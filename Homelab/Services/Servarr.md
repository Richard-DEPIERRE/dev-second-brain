---
type: homelab-service
status: running
compose: ~/docker/servarr/compose.yaml
updated: 2026-04-08
---

# Servarr Stack

The *arr stack — automated media collection management. All download traffic routes through the Gluetun VPN container.

## URLs

| Service | URL | Port |
|---------|-----|------|
| Sonarr (TV) | https://sonarr.richarddepierre.com | 8989 |
| Radarr (Movies) | https://radarr.richarddepierre.com | 7878 |
| Lidarr (Music) | https://lidarr.richarddepierre.com | 8686 |
| Bazarr (Subtitles) | https://bazarr.richarddepierre.com | 6767 |
| Prowlarr (Indexers) | https://prowlarr.richarddepierre.com | 9696 |
| qBittorrent | https://qbit.richarddepierre.com | 8181 |
| Slskd (Soulseek) | https://slskd.richarddepierre.com | — |
| NZBGet (Usenet) | https://nzbget.richarddepierre.com | 6789 |
| Gluetun VPN | https://gluetun.richarddepierre.com | 8000 |

## Containers

| Container | Image | Status | Network |
|-----------|-------|--------|---------|
| `gluetun` | `qmcgaw/gluetun` | ✅ healthy | servarrnetwork (172.39.0.2) |
| `qbittorrent` | `lscr.io/linuxserver/qbittorrent` | ✅ Running | via gluetun |
| `prowlarr` | `lscr.io/linuxserver/prowlarr` | ✅ Running | via gluetun |
| `nzbget` | `lscr.io/linuxserver/nzbget` | ✅ Running | via gluetun |
| `slskd` | `slskd/slskd` | ✅ healthy | via gluetun |
| `sonarr` | `lscr.io/linuxserver/sonarr` | ✅ Running | 172.39.0.3 |
| `radarr` | `lscr.io/linuxserver/radarr` | ✅ Running | 172.39.0.4 |
| `lidarr` | `blampe/lidarr` | ✅ Running | 172.39.0.5 |
| `bazarr` | `lscr.io/linuxserver/bazarr` | ✅ Running | 172.39.0.6 |
| `deunhealth` | `qmcgaw/deunhealth` | ✅ Running | none |

## Setup

- **Compose**: `~/docker/servarr/compose.yaml` on the VM
- **Config dirs**: `~/docker/servarr/{sonarr,radarr,lidarr,bazarr,prowlarr,qbittorrent,slskd,gluetun,nzbget}/`
- **Media**: `/data/media/` — all *arr apps write here, Jellyfin reads from here
- **VPN**: AirVPN via Gluetun — qbittorrent, prowlarr, nzbget, slskd all use `network_mode: service:gluetun`
- **Docker network**: `servarrnetwork` (172.39.0.0/24)
- **qBittorrent UI**: VueTorrent mod installed (`DOCKER_MODS=ghcr.io/vuetorrent/vuetorrent-lsio-mod:latest`)

## Data Flow

```
Prowlarr (indexer hub)
    ├──► Sonarr (TV shows) ──► qBittorrent ──► /data/media/shows/
    ├──► Radarr (movies)   ──► qBittorrent ──► /data/media/movies/
    └──► Lidarr (music)    ──► qBittorrent ──► /data/media/music/
                           └──► Slskd (Soulseek) ──► /data/media/music/

Bazarr ──► Sonarr + Radarr (pulls subtitle info, downloads .srt files)

NZBGet (Usenet) ──► used by *arr apps as alternative downloader
```

## Gluetun VPN Gateway

All download clients (qbittorrent, prowlarr, nzbget, slskd) share Gluetun's network stack.
- Provider: AirVPN
- Healthcheck: pings google.com every 20s
- If Gluetun is unhealthy, `deunhealth` automatically restarts qbittorrent
- Forwarded VPN port configured in `.env` as `FIREWALL_VPN_INPUT_PORTS`

## Dependencies

- `/data/media/` CIFS mount must be healthy for all *arr apps
- Gluetun must be healthy before qbittorrent/prowlarr/nzbget/slskd start

## Restart

```bash
ssh ssh.richarddepierre.com "cd ~/docker/servarr && docker compose up -d"
```
