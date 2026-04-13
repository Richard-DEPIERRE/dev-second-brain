---
date: 2026-04-08
type: incident-log
status: resolved
severity: high
tags: [proxmox, zfs, docker, cifs]
---

# 2026-04-08 — VM Recovery & ZFS Scrub Incident

## Symptoms
- servarr-vm (VMID 101) would not start from Proxmox GUI
- Error: `TASK ERROR: timeout: no zvol device link for 'vm-101-disk-0' found after 300 sec`

## Root Cause Chain
1. **Primary**: `/dev/zvol/` directory did not exist on Proxmox — udev had not created the symlink for `vault_small/vm-101-disk-0` → `/dev/zd0`
2. **Secondary**: `tank` ZFS pool had an active scrub running (started 2026-02-08), consuming all disk I/O at ~34 MB/s, saturating the 4TB HDD
3. **Tertiary**: The scrub I/O saturation caused the `tank` pool (which backs the CT's `/data` Samba share) to become unresponsive, killing Docker containers that bind-mount `/data/compose/*`

## What Was Done

### Fix 1 — Restore zvol device link ✅
```bash
ssh proxmox
udevadm trigger --action=add /dev/zd0
udevadm settle
# Result: /dev/zvol/vault_small/vm-101-disk-0 -> ../../zd0 restored
```

### Fix 2 — Make zvol udev fix permanent ✅
Created `/etc/systemd/system/zfs-zvol-udev.service` on Proxmox:
```ini
[Unit]
Description=Retrigger udev for ZFS zvol symlinks after pool import
After=zfs-import.target
Requires=zfs-import.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'sleep 5 && udevadm trigger --action=add --subsystem-match=block && udevadm settle'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
`systemctl enable zfs-zvol-udev.service`

### Fix 3 — VM started ✅
```bash
qm start 101
# VM is now running at 192.168.1.27
```

Note: VM logged a warning about IGD passthrough (`hostpci0: 0000:2d:00`):
`IGD device is unsupported in legacy mode` — this is a warning, not fatal. VM boots fine.

### Fix 4 — Reduce scrub I/O impact ⚠️ Partial
```bash
echo 1 > /sys/module/zfs/parameters/zfs_scrub_min_time_ms
# This makes the scrub yield more often to other I/O
```
The `zpool scrub -p tank` (proper pause) could not complete — Proxmox SSH too slow due to I/O load.

## Final State (fully resolved)

| Component                                           | Status                                         |
| --------------------------------------------------- | ---------------------------------------------- |
| servarr-vm (101)                                    | ✅ Running (192.168.1.27)                       |
| /dev/zvol/ symlink                                  | ✅ Restored (permanent fix via systemd service) |
| zfs-zvol-udev.service                               | ✅ Installed, enabled                           |
| tank pool scrub                                     | ✅ Paused (`zpool scrub -p tank`)               |
| /data CIFS mount (VM→CT)                            | ✅ Healthy                                      |
| Portainer-managed containers                        | ✅ All running                                  |
| Jellyfin, Navidrome, Sonarr, Radarr, Lidarr, Bazarr | ✅ Up                                           |
| music-history-db                                    | ✅ Healthy (stale postmaster.pid removed)       |

## Resolution (end of session)

### Scrub paused ✅
After force reboot of the Proxmox host, SSH became responsive again and:
```bash
ssh proxmox 'zpool scrub -p tank'  # → "scrub_paused" ✅
```

### All containers auto-recovered ✅
With scrub paused, CIFS mount became healthy and all `restart: unless-stopped` containers came back automatically within ~3 minutes of VM boot.

### music-history-db fixed ✅
Was crash-looping with `FATAL: bogus data in lock file "postmaster.pid"` — stale PID file from unclean shutdown. Fixed by:
```bash
docker stop music-history-db
docker run --rm -v /home/uichaa/docker/music-history/database/postgres_data:/data alpine rm /data/postmaster.pid
docker start music-history-db
# → now healthy
```

### Pending — tank pool data errors
The scrub found **5 data errors** on `tank` (media pool). To investigate after things stabilize:
```bash
ssh proxmox 'zpool status -v tank'  # see which files are affected
ssh proxmox 'zpool scrub tank'      # resume scrub when ready
```
These errors are on media files, not OS/VM disks (`vault_small` is clean).

## Storage Summary (at time of incident)

| Pool | Size | Used | Free | Health |
|------|------|------|------|--------|
| tank | 3.62T | 2.55T | 1.08T | ⚠️ 5 data errors |
| vault_small | 928G | 89.2G | 839G | ✅ Clean |

CT `/data` (tank): 2.6T used / 3.0T (87%)  
VM `/docker` (vault_small): 899M used / 300G (1%)  
VM boot disk (vault_small): 88.2G used / 256G

## Architecture Discovered

```
Proxmox (192.168.1.191)
├── tank pool (4TB HDD) → CT /data (Samba share)
│   └── contains: /data/compose/* (Portainer stacks)
│                 /data/media/* (movies, TV, music)
└── vault_small pool (1TB SSD)
    ├── CT /docker (LXC config)
    └── VM boot disk (vm-101-disk-0, 256GB)

CT (LXC 100, 192.168.1.100)
├── Samba: shares /data → VM mounts as //192.168.1.100/data
└── Services: Apache, Samba, wsdd, postfix

VM (VMID 101, 192.168.1.27)
├── Docker (Portainer-managed)
├── /data → CIFS mount from CT (soft mount, vers=2.0)
│   └── /data/compose/16  → servarr (sonarr/radarr/lidarr/bazarr/qbit)
│       /data/compose/17  → jellyfin
│       /data/compose/29  → navidrome
│       /data/compose/31  → music-history
└── ~/docker/ → local docker stacks (paracord, paradou-bot, portainer, etc.)
```

## Containers Status (updated post-recovery)

| Container | Status | Notes |
|-----------|--------|-------|
| gluetun | ✅ healthy | VPN gateway |
| vaultwarden | ✅ healthy | Password manager |
| immich (redis/postgres/ml) | ✅ healthy | Photo backup |
| portainer + agent + dozzle | ✅ running | Stack manager |
| cloudflared | ✅ running | Tunnel |
| paradou-bot | ✅ running | Discord bot |
| paracord-frontend + adminer | ✅ running | |
| beszel-agent | ✅ running | Stats |
| music-history-api | ✅ running | |
| music-history-db | ✅ healthy | Fixed: removed stale postmaster.pid |
| jellyfin + jellyseerr | ✅ running | |
| navidrome | ✅ running | |
| sonarr/radarr/lidarr/bazarr | ✅ running | |
| qbittorrent/prowlarr/nzbget/slskd | ✅ running | Via gluetun VPN |
| paracord-backend/postgres/livekit | ❌ exited 255 | Down 4+ weeks, unrelated to this incident |
