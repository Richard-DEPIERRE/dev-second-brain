---
type: homelab-service
status: running
updated: 2026-04-08
---

# Infrastructure Services

Core services that keep everything running.

---

## Portainer

Docker container management UI.

| | |
|--|--|
| **URL** | https://portainer.richarddepierre.com |
| **Compose** | `~/docker/portainer/compose.yml` |
| **Status** | ✅ Running |

**Containers:**
- `portainer-portainer-1` — `portainer/portainer-ce:latest` (port 9443)
- `portainer-portainer_agent-1` — `portainer/agent:2.33.1` (port 9001)
- `portainer-dozzle-1` — `amir20/dozzle:latest` — log viewer (port 9999 → 8080)

---

## Cloudflared (Cloudflare Tunnel)

Exposes all internal services to the internet via Cloudflare, without opening any firewall ports.

| | |
|--|--|
| **Compose** | `~/docker/cloudflared/docker-compose.yml` |
| **Image** | `cloudflare/cloudflared:latest` |
| **Status** | ✅ Running |
| **Tunnel ID** | `83ddbe44-8711-4b22-b7b7-1e4175806c74` |
| **Manage** | https://dash.cloudflare.com/752d877c7643a692bbaf5872128368e7 |

**How it works:** The container runs `tunnel --no-autoupdate run --token $TUNNEL_TOKEN` and connects to Cloudflare. Each subdomain (`*.richarddepierre.com`) is routed to the appropriate internal port.

---

## Vaultwarden

Self-hosted Bitwarden-compatible password manager.

| | |
|--|--|
| **URL** | https://vault.richarddepierre.com |
| **Compose** | `~/docker/vaultwarden/compose.yml` |
| **Image** | `vaultwarden/server:latest` |
| **Status** | ✅ healthy |
| **Data** | `~/docker/vaultwarden/vw-data/` |

---

## Beszel

Lightweight system stats and monitoring agent.

| | |
|--|--|
| **URL** | https://stats.richarddepierre.com |
| **Compose** | `~/docker/beszel/compose.yaml` |
| **Image** | `henrygd/beszel-agent` |
| **Status** | ✅ Running |
| **Data** | `~/docker/beszel/beszel_agent_data/` |

---

## Immich

Self-hosted photo and video backup (Google Photos alternative).

| | |
|--|--|
| **Compose** | `~/docker/immich/docker-compose.yml` |
| **Status** | ✅ All healthy |

**Containers:**
- `immich_postgres` — `ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0` ✅ healthy
- `immich_redis` — `valkey/valkey:8-bookworm` ✅ healthy
- `immich_machine_learning` — `ghcr.io/immich-app/immich-machine-learning:release` ✅ healthy
- `immich-server` — main app

**Storage:**
- Photos/videos: `/data/immich/library` (on CIFS/tank pool)
- DB: `/home/uichaa/immich/postgres` (local, not on CIFS)

---

## Glance

Homepage dashboard.

| | |
|--|--|
| **SSH** | `ssh gssh.richarddepierre.com` |
| **URL** | https://glance.richarddepierre.com |
| **Config** | `/opt/glance/glance.yml` on glance host |

---

## Kiwix

Offline Wikipedia and ZIM file reader.

| | |
|--|--|
| **URL** | https://wiki.richarddepierre.com |
| **Compose** | `~/docker/kiwix/compose.yml` |
| **Image** | `ghcr.io/kiwix/kiwix-serve:latest` |

---

## MusicBrainz (Picard)

Self-hosted MusicBrainz instance for music tagging.

| | |
|--|--|
| **URL** | https://setmusic.richarddepierre.com |
| **Compose** | `~/docker/musicbrainz/compose.yaml` |
| **Image** | `mikenye/picard` |
| **Config** | `~/docker/musicbrainz/config/` |

---

## Music History

Custom service tracking music listening history.

| | |
|--|--|
| **Compose** | `~/docker/music-history/` |
| **Status** | ✅ Running |

**Containers:**
- `music-history-api` — `node:20-alpine`, Node.js API (port 3000)
- `music-history-db` — PostgreSQL (port 5432, internal only)
  - Data: `~/docker/music-history/database/postgres_data/`
  - ⚠️ **Known issue**: On unclean shutdown, leaves stale `postmaster.pid`. Fix: remove it via alpine container (see [[../Logs/2026-04-08-vm-recovery]])
