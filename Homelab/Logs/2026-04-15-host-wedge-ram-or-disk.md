---
date: 2026-04-15
type: incident-log
status: partially-resolved
severity: high
tags: [proxmox, zfs, ram, hardware, cifs, scrub]
related: [2026-04-08-vm-recovery]
---

# 2026-04-15 — Host Wedge (RAM-or-Disk Silent Corruption)

## Symptoms
- SSH to VM (`ssh.richarddepierre.com` → 192.168.1.27) completed authentication but hung after the MOTD — no shell prompt
- Proxmox web UI reachable, but VM console showed kernel `blocked for more than 120 seconds` hung-task warnings
- CT `/data` CIFS mount unresponsive from VM's side
- Proxmox host SSH slow/partially responsive
- `zpool scrub -p tank` hung indefinitely (commands went into `cv_wait`)
- Later: `zpool status tank` itself **segfaulted**; load average climbed to 33+

## Root Cause Chain
1. **Same scrub from 2026-02-08 had auto-resumed** on a Proxmox reboot between incidents. The paused state from last incident (`zpool scrub -p`) is lost when the pool re-imports — ZFS resumes an in-progress scrub by default.
2. **ZFS scrub was scanning at only 5 MB/s** (normal would be 100+ MB/s on this 4TB HDD) — drive was spending most of its time on retries on problem blocks. This saturated `tank`'s I/O.
3. **Saturated `tank` → CT `/data` via Samba became unresponsive → VM's CIFS mount hung → VM processes touching `/data` went into D state → login shell hung after MOTD.** Same secondary chain as 2026-04-08.
4. **New this time — kernel oopses with corrupted-pointer signature**:
   ```
   Oops: general protection fault, probably for non-canonical address 0x800d91a9a2a03477
   ```
   Same address across 10 oopses (#1 through #10) over a ~30 min window. Non-canonical x86-64 address = a specific memory location keeps being read as a corrupted pointer.
5. **Data errors grew 5 → 8** on `tank` since last incident. ZFS CKSUM errors on drive climbed to 6. **But** SMART on the drive was clean: no reallocated sectors, no pending, no uncorrectable, no CRC errors, overall PASSED. Also zero ATA/SATA-level I/O errors in dmesg.
6. Putting it together: **silent data corruption + same-address kernel oopses + clean SMART + no ATA errors** strongly suggests **memory corruption (non-ECC RAM)** rather than disk failure. Unproven until memtest86+ runs.

## What Was Done

### Fix 1 — Diagnose drive vs RAM ✅
Confirmed drive is healthy per SMART:
```bash
smartctl -H /dev/disk/by-id/ata-ST4000VN006-3CW104_ZW63LFLC   # PASSED
smartctl -A ...   # Reallocated=0, Pending=0, Uncorrectable=0, UDMA CRC=0
```
Confirmed no ATA-level errors in dmesg (only boot-time link-up messages).
Confirmed kernel oopses at identical bogus pointer → classic bit-flip-in-kernel-memory signature.

### Fix 2 — Halt scrub I/O without needing a pool transaction ✅
`zpool scrub -p tank` was hanging in `cv_wait` because the scrub thread was stuck on in-flight I/O (bad sector retries) and wouldn't check the pause flag.
Workaround — kernel module parameter that makes scrub skip I/O without needing to commit a pause transaction:
```bash
echo 1 > /sys/module/zfs/parameters/zfs_vdev_scrub_max_active
echo 1 > /sys/module/zfs/parameters/zfs_no_scrub_io     # the big lever
echo 50 > /sys/module/zfs/parameters/zfs_scan_vdev_limit
```
`zfs_scan_idle` parameter no longer exists in current OpenZFS (harmless permission-denied).
After this, `zpool iostat tank 1` showed reads drop to 0 — drive went idle.

### Fix 3 — Persist the workaround before reboot ✅
```bash
echo 'options zfs zfs_no_scrub_io=1' > /etc/modprobe.d/zfs-no-scrub.conf
update-initramfs -u
```
This ensures that if the scrub auto-resumes on pool import (it does by default), it will do no I/O — preventing re-entry into the same wedged state.

### Fix 4 — Forced reboot ✅
Host was too wedged to recover from userspace (load 33, `zpool status` segfaulting, 30+ D-state `zvol_tq-*` threads, stuck `txg_sync`, `txg_quiesce`). Reboot was the only path.
```bash
systemctl reboot --force
```
Host came back within ~15 minutes. Post-boot: load 0.23, zero kernel oopses, all services running.

### Unintended side effect — scrub "completed" with 0 errors ⚠️
Because `zfs_no_scrub_io=1` was set *before* the scrub resumed on import, the scrub thread traversed the scan state in metadata-only mode and marked itself complete:
```
scan: scrub repaired 0B in 66 days 14:33:52 with 0 errors on 2026-04-15 15:57:53
errors: No known data errors
```
**Consequence**: the list of 8 corrupted files (which we never managed to capture via `zpool status -v tank`) is now lost from the ZFS error log. **The blocks on disk are almost certainly still corrupt** — they just won't be flagged again until something actually reads them or a real scrub runs.

## Final State (post-reboot)

| Component                                | Status                                       |
| ---------------------------------------- | -------------------------------------------- |
| Proxmox host                             | ✅ Stable (load 0.23, no oopses)              |
| CT 100                                   | ✅ Running                                    |
| VM 101                                   | ✅ Running, SSH works                         |
| `tank` pool                              | ✅ ONLINE, 0 reported errors (but see below) |
| `zfs_no_scrub_io=1`                      | ✅ Persisted via `/etc/modprobe.d/zfs-no-scrub.conf` |
| Scrub                                    | Marked complete (artificial, due to workaround) |
| Kernel oopses                            | Stopped completely post-boot                 |
| VM's CIFS `/data` mount                  | Needed `umount -l` + remount to clear stale session |
| Containers                               | Auto-recovered with `restart: unless-stopped` |

## Unresolved / Deferred

### 1. Root cause — RAM or disk? Unknown.
Three hypotheses, can't distinguish yet:
- **(likely)** Bit-flip in non-ECC RAM (transient cosmic ray, or stuck bit in a specific DIMM cell). Same-address oopses strongly suggest a persistent bad bit in kernel memory.
- Corrupt on-disk metadata being fed into kernel structures by the scrub (less likely given clean SMART, but possible).
- Accumulated kernel state corruption over 6-day uptime.

### 2. Hardware context
- 2× 16GB G.Skill Ripjaws V DDR4 (`F4-3200C16-16GVK`), **non-ECC**, running at 2133 MT/s (JEDEC default — XMP/DOCP not enabled, so overclocking is NOT the cause)
- Consumer AMD platform (kvm_amd module loaded)
- EDAC memory controller not registered (can't report errors even if it wanted to — non-ECC)

### 3. The 8 corrupted files
Their identity is lost with the error log. On-disk blocks still likely corrupt. Will resurface if and when something reads them, or when a real scrub runs.

## Next Steps (to do in a maintenance window)

### Phase 1 — Memtest86+
- Boot memtest86+ (Proxmox's GRUB menu usually includes it, or USB)
- **At least one full pass** covering both DIMMs
- Then **pull one stick at a time** and test each in isolation — this identifies which stick is bad if the first pass found errors
- Clean pass → proceed to Phase 2
- Any error → replace the bad stick(s), rerun memtest until clean

### Phase 2 — Drive extended self-test (after RAM verified clean)
```bash
smartctl -t long /dev/disk/by-id/ata-ST4000VN006-3CW104_ZW63LFLC
# ~8 hours, runs on the drive itself, non-disruptive
# Check result later with: smartctl -l selftest /dev/disk/...
```

### Phase 3 — Re-enable and run a real scrub
Only after Phase 1 + 2 are clean:
```bash
echo 0 > /sys/module/zfs/parameters/zfs_no_scrub_io
rm /etc/modprobe.d/zfs-no-scrub.conf
update-initramfs -u
zpool scrub tank
```
Watch `zpool status tank` — if CKSUM errors reappear despite clean RAM and clean SMART, something stranger is going on (SATA cable/controller/PSU ripple). If no errors appear, the incident was likely a transient RAM bit-flip — consider ECC on the next hardware refresh.

### Phase 4 — Prevent auto-resumption of scrubs after reboot (general lesson)
ZFS auto-resumes scrubs on import. When you pause a scrub for I/O reasons, that state only survives while the pool stays imported. Two options:
- Let scrubs complete rather than pausing them long-term
- Keep a persistent `zfs_no_scrub_io=1` only during known-bad-hardware windows, remove otherwise

## Hardware Recommendation (medium-term)

Given this is the second incident in a week driven by storage-layer issues on a single-HDD pool without ECC RAM:
- **ECC RAM** would have caught this failure mode early and *could have self-corrected* transient bit flips. On AMD, consumer Ryzen boards often support UDIMM ECC (AM4/AM5 + certain chipsets) — worth checking your motherboard manual before the next RAM purchase.
- **`tank` is a single-drive vdev with no redundancy** (`ONLINE 0 0 0` on a single `ata-ST4000VN006`). With 8 unrecovered data errors, ZFS can't repair anything because there's no second copy. A mirrored vdev (or at least backup-to-another-pool) would have let ZFS silently heal these corruptions when the scrub found them.

## Commands for next time

If host gets wedged the same way again:
```bash
# From Proxmox node shell (in web UI — doesn't depend on tank):
cat /sys/module/zfs/parameters/zfs_no_scrub_io    # check if safeguard already in place
echo 1 > /sys/module/zfs/parameters/zfs_no_scrub_io
zpool iostat tank 1 5                              # reads should drop to 0 in ~30s
# If zpool scrub -p tank won't pause cleanly, don't wait — just reboot.
# zfs-no-scrub.conf in /etc/modprobe.d already persists the safeguard across reboot.
```

## Storage Summary (post-recovery, 2026-04-15 16:15)

| Pool        | Size  | Used  | Free  | Health                                           |
| ----------- | ----- | ----- | ----- | ------------------------------------------------ |
| tank        | 3.62T | 2.55T | 1.08T | ONLINE, no reported errors (artificial clean — see above) |
| vault_small | 928G  | ~90G  | ~840G | ✅ Clean                                          |
