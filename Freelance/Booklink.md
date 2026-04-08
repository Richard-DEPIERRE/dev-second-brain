---
type: freelance-project
status: active
client: Horyond Agency
stack: [Flutter, Firebase, FlutterFlow]
code_path: "/Volumes/SSD Richard/freelance/booklink/booklink"
backlog_path: "/Volumes/SSD Richard/freelance/booklink/backlog"
---

# Booklink

Flutter mobile app — managed via [[Horyond Agency]].

## Status

🟢 **Active** — TestFlight builds being actively pushed (v2.6.0, v3.0.0 seen in emails).

## Architecture

Originally built with FlutterFlow, then modified by a second development team:
- Screens in `lib/screens/` — each has `{page}_model.dart` + `{page}_widget.dart`
- FlutterFlow routing via `lib/flutter_flow/nav/nav.dart`
- Firebase backend: Auth (Google, Email, Apple), Firestore
- FVM (Flutter Version Manager) for version pinning

## Key Folders
```
booklink/
├── lib/
│   ├── screens/       ← all views
│   ├── flutter_flow/  ← FlutterFlow utilities & router
│   ├── backend/schema ← Firebase data models
│   └── auth/          ← login methods
├── backlog/           ← separate Firebase backlog project (web)
```

## Deployment
- iOS via TestFlight (App Store Connect)
- Apple auth keys in project root (`.p8` files)

## Contracts & Invoices
Stored at `/Volumes/SSD Richard/freelance/booklink/`:
- `CONTRAT BOOKLINK.pdf`
- `devis-booklink.pdf`
- `contrat cession droit 2.pdf`

## Notes
- Code is on the SSD, not in `~/freelance/` — make sure SSD is mounted before working
- Backlog is a separate Firebase web project (`backlog/`)
