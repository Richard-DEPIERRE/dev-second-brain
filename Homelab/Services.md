---
type: homelab
---

# Homelab Services

All services run via the Cloudflare tunnel on `*.richarddepierre.com`. The media stack runs inside **servarr-vm** (currently stopped — see [[Overview]]).

## Infrastructure

| Service | URL | What it does |
|---------|-----|--------------|
| Proxmox | https://proxmox.richarddepierre.com | Hypervisor management |
| Portainer | https://portainer.richarddepierre.com | Docker container management |
| Beszel | https://stats.richarddepierre.com | System stats & monitoring |
| Vaultwarden | https://vault.richarddepierre.com | Self-hosted Bitwarden (passwords) |
| Glance | `gssh.richarddepierre.com` (SSH) | Homepage dashboard |
| Gluetun | https://gluetun.richarddepierre.com | VPN gateway for downloads |

## Media

| Service | URL | What it does |
|---------|-----|--------------|
| Jellyfin | https://jelly.richarddepierre.com | Media server (movies, TV, music) |
| JellySearch | https://search.richarddepierre.com | Jellyfin search UI |
| Bazarr | https://bazarr.richarddepierre.com | Subtitle management |
| Radarr | https://radarr.richarddepierre.com | Movie collection manager |
| Sonarr | https://sonarr.richarddepierre.com | TV show collection manager |
| Lidarr | https://lidarr.richarddepierre.com | Music collection manager |
| Prowlarr | https://prowlarr.richarddepierre.com | Indexer manager (connects to *arr) |
| MusicBrainz (SetMusic) | https://setmusic.richarddepierre.com | Self-hosted MusicBrainz |

## Downloads

| Service | URL | What it does |
|---------|-----|--------------|
| qBittorrent | https://qbit.richarddepierre.com | Torrent client (routes via Gluetun) |
| qBittorrent Web | https://qbw.richarddepierre.com/qb/torrents | Alternative qBit UI |
| Slskd | https://slskd.richarddepierre.com | Soulseek client (music) |
| NZBGet | https://nzbget.richarddepierre.com | Usenet downloader |

## Knowledge

| Service | URL | What it does |
|---------|-----|--------------|
| Kiwix | https://wiki.richarddepierre.com | Offline Wikipedia/ZIM files |

## External Services (linked in dashboard)

| Service | URL | What it does |
|---------|-----|--------------|
| Eweka | https://www.eweka.nl | Usenet provider |
| Geekhub | https://geek-hub.com.au | NZB indexer |
| NZBGeek | https://nzbgeek.info | NZB indexer |

## Service Connections

```
Prowlarr ──→ Radarr
         ──→ Sonarr
         ──→ Lidarr

Radarr ──→ qBittorrent (via Gluetun VPN)
Sonarr ──→ qBittorrent (via Gluetun VPN)
Lidarr ──→ qBittorrent (via Gluetun VPN)
       ──→ Slskd

Bazarr ──→ Radarr + Sonarr (subtitle sync)

Jellyfin ──→ media files downloaded by *arr stack
JellySearch ──→ Jellyfin API
```

## Compose File Locations (on VM at /data/compose/)

All Portainer-managed stacks live on the CIFS mount. SSH to VM then:

| Stack | Path | Services |
|-------|------|---------|
| servarr | `/data/compose/16` | sonarr, radarr, lidarr, bazarr, qbittorrent, prowlarr, gluetun |
| jellyfin | `/data/compose/17` | jellyfin |
| navidrome | `/data/compose/29` | navidrome |
| music-history | `/data/compose/31` | music-history-api, music-history-db |

Local stacks (in `~/docker/` on VM, not affected by CIFS):
- portainer, vaultwarden, immich, cloudflared, paradou-bot, paracord, beszel, revolt, wger, kiwix, musicbrainz

## Restart Commands (when /data is accessible)
```bash
ssh ssh.richarddepierre.com
cd /data/compose/17 && docker compose up -d   # jellyfin
cd /data/compose/29 && docker compose up -d   # navidrome
cd /data/compose/16 && docker compose up -d   # servarr stack
cd /data/compose/31 && docker compose up -d   # music-history
```

## Notes
- [[Projects/Uichaa Player]] connects to Navidrome (`https://setmusic.richarddepierre.com` or similar) — 871 tracks, 90 albums, 305 artists
- Navidrome music files are on the CIFS mount (`/data/media/music/` or similar, type: smb2 per Navidrome logs)
- All Portainer-managed containers depend on `/data` CIFS mount being healthy
- ⚠️ If tank pool scrub is running, CIFS mount becomes unresponsive — stop scrub first (see [[Overview]])
- Downloads route through Gluetun VPN to avoid ISP exposure
