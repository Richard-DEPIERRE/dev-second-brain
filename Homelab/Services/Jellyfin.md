---
type: homelab-service
status: running
url: https://jelly.richarddepierre.com
compose: ~/docker/jellyfin/compose.yaml
updated: 2026-04-08
---

# Jellyfin

Media server for movies, TV shows, and music. The main frontend for all downloaded media.

## URLs

| Purpose | URL |
|---------|-----|
| Web UI | https://jelly.richarddepierre.com |
| LAN direct | http://192.168.1.27:8096 |
| Request movies/shows | https://search.richarddepierre.com (Jellyseerr) |

## Containers

| Container | Image | Status |
|-----------|-------|--------|
| `jellyfin` | `lscr.io/linuxserver/jellyfin:latest` | ✅ Running |
| `jellyseerr` | `fallenbagel/jellyseerr:latest` | ✅ Running |

## Setup

- **Compose**: `~/docker/jellyfin/compose.yaml` on the VM
- **Config**: `~/docker/jellyfin/config/`
- **Media**: `/data/media/` (CIFS mount from CT → tank pool)
  - Movies: `/data/media/movies/`
  - TV shows: `/data/media/shows/`
  - Music: `/data/media/music/`
- **Hardware transcoding**: Intel iGPU passthrough (`/dev/dri` device mapped)
- **Ports**: 8096 (HTTP), 7359/udp (service discovery), 1900/udp (client discovery)

## Jellyseerr

Request management UI — users can request movies and TV shows which get sent to Radarr/Sonarr.
- Port: 5055
- Config: `~/docker/jellyfin/jellyseerr/`

## Dependencies

- `/data/media/` CIFS mount must be healthy (fails if tank scrub is running)
- Radarr/Sonarr/Lidarr populate the media files

## Restart

```bash
ssh ssh.richarddepierre.com "cd ~/docker/jellyfin && docker compose up -d"
```
