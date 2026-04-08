---
type: homelab
updated: 2026-04-08
---

# Homelab Services

All services run via the Cloudflare tunnel on `*.richarddepierre.com`.

## Quick Status

| Service | URL | Status |
|---------|-----|--------|
| Proxmox | https://proxmox.richarddepierre.com | ✅ |
| Portainer | https://portainer.richarddepierre.com | ✅ |
| Beszel | https://stats.richarddepierre.com | ✅ |
| Vaultwarden | https://vault.richarddepierre.com | ✅ |
| Glance | `ssh gssh.richarddepierre.com` | ✅ |
| Jellyfin | https://jelly.richarddepierre.com | ✅ |
| Jellyseerr | https://search.richarddepierre.com | ✅ |
| Bazarr | https://bazarr.richarddepierre.com | ✅ |
| Radarr | https://radarr.richarddepierre.com | ✅ |
| Sonarr | https://sonarr.richarddepierre.com | ✅ |
| Lidarr | https://lidarr.richarddepierre.com | ✅ |
| Prowlarr | https://prowlarr.richarddepierre.com | ✅ |
| qBittorrent | https://qbit.richarddepierre.com | ✅ |
| Slskd | https://slskd.richarddepierre.com | ✅ |
| NZBGet | https://nzbget.richarddepierre.com | ✅ |
| Navidrome | https://navidrome.richarddepierre.com | ✅ |
| MusicBrainz | https://setmusic.richarddepierre.com | ✅ |
| Kiwix | https://wiki.richarddepierre.com | ✅ |
| Gluetun | https://gluetun.richarddepierre.com | ✅ |
| Paracord | — | ⚠️ Frontend only, backend down |

## Detailed Notes

- [[Services/Jellyfin]] — Media server + Jellyseerr request manager
- [[Services/Servarr]] — *arr stack: Sonarr, Radarr, Lidarr, Bazarr, Prowlarr, qBittorrent, Slskd, NZBGet, Gluetun VPN
- [[Services/Navidrome]] — Music streaming, Subsonic API
- [[Services/Infrastructure]] — Portainer, Cloudflared, Vaultwarden, Beszel, Immich, Glance, Kiwix, MusicBrainz, Music History
- [[Services/Paracord]] — Paracord homelab deployment (partially down)

## External Services

| Service | URL | Purpose |
|---------|-----|---------|
| Eweka | https://www.eweka.nl | Usenet provider |
| Geekhub | https://geek-hub.com.au | NZB indexer |
| NZBGeek | https://nzbgeek.info | NZB indexer |

## Compose File Locations (on VM)

All compose stacks live in `~/docker/` on the VM:

| Stack | Path | Services |
|-------|------|---------|
| servarr | `~/docker/servarr/compose.yaml` | gluetun, qbittorrent, prowlarr, sonarr, radarr, lidarr, bazarr, slskd, nzbget |
| jellyfin | `~/docker/jellyfin/compose.yaml` | jellyfin, jellyseerr |
| navidrome | `~/docker/navidrome/` | navidrome (no compose file, started manually) |
| music-history | `~/docker/music-history/` | music-history-api, music-history-db |
| portainer | `~/docker/portainer/compose.yml` | portainer-ce, portainer-agent, dozzle |
| vaultwarden | `~/docker/vaultwarden/compose.yml` | vaultwarden |
| immich | `~/docker/immich/docker-compose.yml` | immich-server, immich-ml, postgres, redis |
| cloudflared | `~/docker/cloudflared/docker-compose.yml` | cloudflared |
| paracord | `~/docker/paracord/` | paracord-frontend, backend (stopped) |
| paradou-bot | `~/docker/paradou-bot/docker-compose.yml` | paradou-bot |
| beszel | `~/docker/beszel/compose.yaml` | beszel-agent |
| kiwix | `~/docker/kiwix/compose.yml` | kiwix-serve |
| musicbrainz | `~/docker/musicbrainz/compose.yaml` | musicbrainz (picard) |
