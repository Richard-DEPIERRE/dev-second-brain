---
type: personal-project
status: in-progress
stack: [Flutter, Subsonic API, Navidrome]
code_path: "~/fun-projects/uichaa_player"
---

# Uichaa Player

Production-ready Navidrome/Subsonic music streaming client for Flutter. Aims for Spotify-level features.

## Features
- Gapless playback
- Background audio + lockscreen controls
- Offline downloads
- Global search (tracks, albums, artists, playlists)
- Queue management
- Artwork caching
- Synced lyrics (LRC format with real-time highlighting)
- Listening analytics + Spotify Wrapped-style year summaries
- Playlist management

## Stack
- **Framework**: Flutter
- **Protocol**: Subsonic API (compatible with Navidrome, Airsonic, etc.)
- Connects to a self-hosted Navidrome instance (homelab)

## Connection to Homelab
- Streams from the Navidrome/music server in the homelab
- See [[Homelab/Services]] for the Navidrome instance URL

## Notes
- This is a personal app — not on the App Store
- The name "Uichaa" appears in the homelab glance dashboard users (`uichaa`)
