---
type: personal-project
status: in-progress
stack: [Go, Flutter, WebSocket, LiveKit, PostgreSQL, Docker]
code_path: "~/fun-projects/paracord-backend | ~/fun-projects/paracord-frontend"
---

# Paracord

Discord-inspired real-time chat app — personal learning project.

## Components

### Backend (`~/fun-projects/paracord-backend`)
- **Language**: Go
- **Framework**: Gin
- **WebSocket**: Gorilla WebSocket
- **Database**: PostgreSQL
- **Run**: `docker compose up`
- Requires Docker & Docker Compose

### Frontend (`~/fun-projects/paracord-frontend`)
- **Framework**: Flutter v3.24.5 (managed via FVM)
- **State**: BLoC
- **Real-time**: WebSockets + LiveKit (voice/video)
- Requires running backend instance

## Local Dev
```bash
# Start backend
cd ~/fun-projects/paracord-backend
docker compose up

# Start frontend
cd ~/fun-projects/paracord-frontend
fvm flutter run
```

## Notes
- Workspace file: `~/fun-projects/paracord.code-workspace`
- LiveKit for voice/video channels
- This is a learning project for Go + real-time systems
