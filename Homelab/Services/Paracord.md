---
type: homelab-service
status: partial
compose: ~/docker/paracord/
updated: 2026-04-08
---

# Paracord (Self-hosted)

The homelab deployment of [[../../Projects/Paracord]] — Discord clone (Go backend + Flutter frontend).

## Containers

| Container | Image | Status |
|-----------|-------|--------|
| `paracord-frontend-paracord-frontend-1` | `paracord-frontend-paracord-frontend` | ✅ Running (port 5000→80) |
| `paracord-backend-adminer-1` | `adminer` | ✅ Running (port 8081→8080) |
| `paracord-backend-backend-1` | `paracord-backend-backend` | ❌ Exited (255) — down 4+ weeks |
| `paracord-backend-postgres-1` | `postgres:16-alpine` | ❌ Exited (255) — down 4+ weeks |
| `paracord-backend-livekit-1` | `livekit/livekit-server:latest` | ❌ Exited (255) — down 4+ weeks |

## Compose

- Frontend: `~/docker/paracord/paracord-frontend/`
- Backend: `~/docker/paracord/paracord-backend/docker-compose.yml`
  - Go backend (port 8080), PostgreSQL 16 (port 5432), LiveKit (ports 7880, 20000)
  - Backend source code lives here too: `cmd/`, `internal/`, `go.mod`, etc.

## Notes

The backend stack has been down for 4+ weeks (pre-dates the 2026-04-08 incident — unrelated).
Frontend and adminer are still running but the app is non-functional without the backend.

To restart the full stack:
```bash
ssh ssh.richarddepierre.com "cd ~/docker/paracord/paracord-backend && docker compose up -d"
```
