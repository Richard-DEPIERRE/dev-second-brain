---
type: homelab
---

# Homelab Overview

## Hardware

| Host | Role | IP | Access |
|------|------|----|--------|
| Proxmox | Hypervisor | 192.168.1.191 | `ssh proxmox` (LAN only) |
| CT (LXC 100) | Samba/data server, services host | 192.168.1.100 | `ssh servarrct` |
| servarr-vm (VM 101) | Docker host — all media/app containers | 192.168.1.27 | `ssh ssh.richarddepierre.com` (via Cloudflare) |
| Glance host | Dashboard | gssh.richarddepierre.com | `ssh gssh.richarddepierre.com` |

## SSH Access

```bash
ssh proxmox               # Proxmox — LAN only (192.168.1.191)
ssh servarrct             # CT (LXC 100) — via Cloudflare Access
ssh ssh.richarddepierre.com   # VM (servarr-vm) — via Cloudflare Access, user: uichaa
ssh gssh.richarddepierre.com  # Glance host — via Cloudflare Access, user: root
```

All remote SSH goes through **Cloudflare Access** (`cloudflared` in `~/.ssh/config`).  
Key: `~/.ssh/id_rsa_perso`

## ZFS Storage Pools (on Proxmox)

| Pool | Disk | Size | Use | Health |
|------|------|------|-----|--------|
| `tank` | 4TB HDD (ST4000VN006) | 3.62T | Media, Portainer stacks, CT data | ⚠️ 5 data errors (scrub 2026-02-08) |
| `vault_small` | 1TB SSD (ST1000LM014) | 928G | VM boot disks, CT docker config | ✅ Clean |

### tank pool — key datasets
- `tank/subvol-100-disk-0` → CT's `/data` — Samba-shared to VM as `/data`
  - Contains: `/data/compose/` (Portainer stacks), `/data/media/` (movies, TV, music)
  - 2.6T used / 3.0T (87% full)

### vault_small pool — key datasets
- `vault_small/vm-101-disk-0` → VM boot disk (256GB, zvol)
- `vault_small/subvol-100-disk-0` → CT's `/docker` (300GB, 1% used)

## Proxmox VMs

| VMID | Name | Status | Specs | Disk |
|------|------|--------|-------|------|
| 101 | servarr-vm | ✅ Running (192.168.1.27) | 6 cores, 24 GB RAM | 256 GB (vault_small) |

VM notes:
- CPU: x86-64-v2-AES
- PCIe passthrough: `hostpci0: 0000:2d:00` (Intel iGPU — logs a warning but VM boots fine)
- `onboot: 1` — auto-starts on Proxmox boot

## Known Issue — zvol udev symlink
On Proxmox reboot, `/dev/zvol/` symlinks may not be created automatically.  
**Fix is permanent**: `zfs-zvol-udev.service` installed at `/etc/systemd/system/`.  
If VM won't start with "no zvol device link" error:
```bash
ssh proxmox "udevadm trigger --action=add /dev/zd0 && udevadm settle"
```

## Architecture — How Everything Connects

```
Proxmox (192.168.1.191)
├── tank pool (4TB HDD)
│   └── LXC 100 /data ──[Samba]──► VM /data (CIFS mount: //192.168.1.100/data)
│       ├── /data/compose/16  → servarr stack (sonarr/radarr/lidarr/bazarr/qbit)
│       ├── /data/compose/17  → jellyfin
│       ├── /data/compose/29  → navidrome
│       ├── /data/compose/31  → music-history
│       └── /data/media/      → movies, TV, music files
└── vault_small pool (1TB SSD)
    ├── VM 101 boot disk (256G zvol → /dev/zd0)
    └── LXC 100 /docker (300G, config files)

VM (192.168.1.27) — Docker containers
├── Portainer-managed (from /data/compose/): jellyfin, navidrome, servarr stack, music-history
├── Local stacks (~/docker/): paracord, paradou-bot, portainer, vaultwarden, immich, cloudflared
└── /data CIFS mount (soft, vers=2.0) — CRITICAL: if tank I/O is saturated, this fails
```

**⚠️ Important**: If `tank` pool is running a scrub, the Samba share becomes unresponsive, which causes all Portainer-managed containers to fail to start. Stop the scrub first:
```bash
ssh proxmox 'zpool scrub -p tank'   # pause
ssh proxmox 'zpool scrub -s tank'   # stop completely
```

## Networking
- External: Cloudflare tunnels (ID: `83ddbe44-8711-4b22-b7b7-1e4175806c74`)
- Domain: `*.richarddepierre.com`
- Manage at: https://dash.cloudflare.com/752d877c7643a692bbaf5872128368e7

## Dashboard
- Glance: config at `/opt/glance/glance.yml` on glance host
- Users: `uichaa` (personal), `mattis` (shared)

## Incident Logs
- [[Logs/2026-04-08-vm-recovery]] — VM not starting, zvol udev fix, tank scrub I/O saturation
