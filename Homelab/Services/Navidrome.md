---
type: homelab-service
status: running
url: https://navidrome.richarddepierre.com (check Services.md for correct URL)
compose: ~/docker/navidrome/ (no compose file — started directly or via Portainer)
updated: 2026-04-08
---

# Navidrome

Self-hosted music streaming server with Subsonic API compatibility. Used by [[Projects/Uichaa Player]].

## URLs

| Purpose | URL |
|---------|-----|
| Web UI | https://navidrome.richarddepierre.com (confirm in Services.md) |
| LAN direct | http://192.168.1.27:4533 |
| Subsonic API | http://192.168.1.27:4533/rest/ |

## Container

| Container | Image | Status |
|-----------|-------|--------|
| `navidrome-navidrome-1` | `deluan/navidrome:latest` | ✅ Running |

## Setup

- **Data dir**: `~/docker/navidrome/` (navidrome.db, cache)
- **Music**: `/data/music/` on CIFS mount — maps to `/music` inside container
- **Config**: `~/docker/navidrome/navidrome.toml` (via `ND_CONFIGFILE`)
- **Port**: 4533
- **Compose**: No compose file found — container may have been created manually or via Portainer UI

## Library Stats

- ~871 tracks, ~90 albums, ~305 artists (last known)

## Subsonic API

Navidrome exposes the Subsonic API, used by:
- [[Projects/Uichaa Player]] — Flutter Subsonic client

## Dependencies

- `/data/music/` CIFS mount (on tank pool via CT Samba share)

## Restart

```bash
ssh ssh.richarddepierre.com "docker restart navidrome-navidrome-1"
```
