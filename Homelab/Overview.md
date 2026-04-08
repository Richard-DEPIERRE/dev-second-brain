---
type: homelab
updated: 2026-04-08
---

# Homelab Overview

## Current Status — All Systems Operational ✅

| Component | Status | Notes |
|-----------|--------|-------|
| Proxmox | ✅ Running | 192.168.1.191 |
| CT (LXC 100) | ✅ Running | 192.168.1.100, Samba/data host |
| servarr-vm (VM 101) | ✅ Running | 192.168.1.27, Docker host |
| tank pool | ✅ Online | Scrub paused — 5 data errors in media files (non-critical) |
| vault_small pool | ✅ Clean | No errors |
| /data CIFS mount | ✅ Healthy | Samba from CT to VM |
| All Docker containers | ✅ Running | See [[Services]] for details |

---

## Hardware

| Host | Role | IP | Access |
|------|------|----|--------|
| Proxmox | Hypervisor | 192.168.1.191 | `ssh proxmox` (LAN only) |
| CT (LXC 100) | Samba/data server | 192.168.1.100 | `ssh servarrct` |
| servarr-vm (VM 101) | Docker host — all containers | 192.168.1.27 | `ssh ssh.richarddepierre.com` |
| Glance host | Dashboard | gssh.richarddepierre.com | `ssh gssh.richarddepierre.com` |

### VM 101 Specs
- 6 vCPUs, 24 GB RAM, 256 GB boot disk (vault_small)
- CPU: x86-64-v2-AES
- PCIe passthrough: `hostpci0: 0000:2d:00` (Intel iGPU — harmless warning on boot)
- `onboot: 1` — auto-starts on Proxmox boot
- OS user: `uichaa`

### CT 100 Specs
- 4 cores, 1 GB RAM, 54 GB root disk (local-lvm)
- Mounts: `/data` (3TB from tank pool), `/docker` (300GB from vault_small)
- Services: Samba, wsdd, Apache, postfix

---

## SSH Access

```bash
ssh proxmox                       # Proxmox — LAN only (192.168.1.191)
ssh servarrct                     # CT (LXC 100) — via Cloudflare Access
ssh ssh.richarddepierre.com       # VM (servarr-vm) — via Cloudflare Access, user: uichaa
ssh gssh.richarddepierre.com      # Glance host — via Cloudflare Access, user: root
```

All remote SSH goes through **Cloudflare Access** (`cloudflared` tunnel).
Key: `~/.ssh/id_rsa_perso`

---

## ZFS Storage Pools (on Proxmox)

| Pool | Disk | Size | Used | Free | Use% | Health |
|------|------|------|------|------|------|--------|
| `tank` | 4TB HDD (ST4000VN006) | 3.62T | 2.55T | 1.08T | 70% | ⚠️ 5 data errors — media files |
| `vault_small` | 1TB SSD (ST1000LM014) | 928G | 89.3G | 839G | 9% | ✅ Clean |

### tank pool — key datasets
- `tank/subvol-100-disk-0` → CT's `/data` (3TB mount, Samba-shared to VM)
  - Contains: `/data/compose/` (legacy Portainer stacks), `/data/media/` (movies, TV, music)

### vault_small pool — key datasets
- `vault_small/vm-101-disk-0` → VM boot disk (256GB zvol → `/dev/zd0`)
- `vault_small/subvol-100-disk-0` → CT's `/docker` (300GB, config files, very lightly used)

### tank pool — known data errors
5 corrupted media files found during scrub (Feb 2026, scrub paused Apr 2026):
```
/data/media/movies/The Running Man (2025)/The.Running.Man.2025.REPACK.1080p.iTunes.WEB-DL.DDP.5.1.Atmos.H.264-CHDWEB.mkv
/data/media/shows/Euphoria (US)/Euphoria.US.S01E05.03.Bonnie.and.Clyde.1080p.AMZN.WEB-DL.DDP5.1.H.264-KiNGS.mkv
/data/media/shows/Euphoria (US)/Euphoria.US.S01E08.REPACK.And.Salt.the.Earth.Behind.You.1080p.AMZN.WEB-DL.DDP5.1.H.264-KiNGS.mkv
/data/media/shows/Euphoria (US)/Euphoria.US.S01E06.The.Next.Episode.1080p.AMZN.WEB-DL.DDP5.1.H.264-KiNGS.mkv
```
To resume the scrub when ready: `ssh proxmox 'zpool scrub tank'`

---

## Architecture — How Everything Connects

```
Internet
    │
    ▼
Cloudflare Tunnel (tunnel ID: 83ddbe44-8711-4b22-b7b7-1e4175806c74)
    │    *.richarddepierre.com
    ▼
cloudflared container (VM) ──► routes to internal services
    │
    ▼
Proxmox (192.168.1.191)
├── tank pool (4TB HDD)
│   └── CT LXC 100 → /data (3TB Samba share)
│       ├── /data/compose/  → legacy Portainer stack files (not actively used)
│       └── /data/media/    → movies, TV, music files
└── vault_small pool (1TB SSD)
    ├── VM 101 boot disk (256GB zvol → /dev/zd0)
    └── CT /docker (300GB, config storage)

CT (LXC 100, 192.168.1.100)
└── Samba: shares /data → VM mounts as //192.168.1.100/data (CIFS, soft, vers=2.0)

VM (VMID 101, 192.168.1.27) — Main Docker host
├── /data → CIFS mount from CT ⚠️ Critical: fails if tank I/O is saturated
│   └── /data/media/ → media files served to Jellyfin, Navidrome, *arr
└── ~/docker/ → all compose stacks (local, not on CIFS)
    ├── servarr/   → gluetun, qbittorrent, prowlarr, sonarr, radarr, lidarr, bazarr, slskd, nzbget
    ├── jellyfin/  → jellyfin, jellyseerr
    ├── navidrome/ → navidrome (music at /data/music)
    ├── music-history/ → music-history-api, music-history-db (postgres)
    ├── portainer/ → portainer-ce, portainer-agent, dozzle
    ├── vaultwarden/ → vaultwarden
    ├── immich/    → immich-server, immich-ml, postgres, valkey/redis
    ├── cloudflared/ → cloudflared (Cloudflare tunnel)
    ├── paracord/  → paracord-frontend (running), backend/postgres/livekit (stopped 4+ weeks)
    ├── paradou-bot/ → discord bot (Node.js)
    ├── beszel/    → beszel-agent (stats)
    ├── kiwix/     → kiwix-serve (offline wiki)
    └── musicbrainz/ → MusicBrainz Picard
```

### Service dependency chain
```
gluetun (VPN) ──► qbittorrent, prowlarr, nzbget, slskd (all traffic via VPN)
prowlarr ──► sonarr, radarr, lidarr (indexer sync)
sonarr/radarr/lidarr ──► qbittorrent (download requests)
lidarr ──► slskd (music via Soulseek)
bazarr ──► sonarr + radarr (subtitle sync)
jellyfin ──► /data/media/ (reads downloaded files)
navidrome ──► /data/music/ (reads music files)
```

---

## Networking

- External: Cloudflare tunnels, domain `*.richarddepierre.com`
- Tunnel ID: `83ddbe44-8711-4b22-b7b7-1e4175806c74`
- Manage: https://dash.cloudflare.com/752d877c7643a692bbaf5872128368e7
- All services exposed via Cloudflare — no ports open directly to internet

### Internal Docker Networks
- `servarrnetwork` (172.39.0.0/24) — used by servarr stack
  - gluetun: 172.39.0.2, sonarr: 172.39.0.3, radarr: 172.39.0.4, lidarr: 172.39.0.5, bazarr: 172.39.0.6, slskd: 172.39.0.7

---

## Known Issues & Quirks

### zvol udev symlink (FIXED — permanent)
On Proxmox reboot, `/dev/zvol/` symlinks may not be created automatically.
**Permanent fix installed**: `/etc/systemd/system/zfs-zvol-udev.service`
If VM won't start with "no zvol device link" error (shouldn't happen now):
```bash
ssh proxmox "udevadm trigger --action=add /dev/zd0 && udevadm settle"
```

### tank scrub I/O saturation (CRITICAL)
If `tank` pool scrub is running, it saturates the 4TB HDD (~34 MB/s), making Samba unresponsive.
This cascades: CIFS mount times out → Docker containers that bind-mount `/data/` fail to start.
```bash
ssh proxmox 'zpool scrub -p tank'   # pause scrub
ssh proxmox 'zpool scrub -s tank'   # stop scrub completely
```

### Portainer-managed stacks (legacy)
Portainer stacks in `/data/compose/` are mostly old configs — actual compose files now live in `~/docker/` on the VM.
Active CIFS-dependent path: `/data/media/` (media files).

---

## Useful Commands

```bash
# Check all container status
ssh ssh.richarddepierre.com "docker ps -a --format '{{.Names}} | {{.Status}}'"

# ZFS pool health
ssh proxmox 'zpool status tank && zpool list'

# Check /data is accessible from VM
ssh ssh.richarddepierre.com 'ls /data/media/ | head -5'

# Restart all servarr services
ssh ssh.richarddepierre.com "cd ~/docker/servarr && docker compose up -d"

# Restart specific stacks
ssh ssh.richarddepierre.com "cd ~/docker/jellyfin && docker compose up -d"
ssh ssh.richarddepierre.com "cd ~/docker/navidrome && docker compose up -d"
```

---

## Dashboard

- Glance: config at `/opt/glance/glance.yml` on glance host (`ssh gssh.richarddepierre.com`)
- Portainer: https://portainer.richarddepierre.com
- Beszel (stats): https://stats.richarddepierre.com

---

## Incident Logs

- [[Logs/2026-04-08-vm-recovery]] — VM not starting (zvol udev), tank scrub I/O saturation, full recovery
