---
type: freelance-project
status: active
client: AquaGymFit
stack: [Flutter, Firebase, BLoC]
code_path: "~/freelance/aquagymfit (old) | ~/freelance/new_aquagymfit (refactor)"
archive_path: "/Volumes/SSD Richard/freelance/aquagymfit"
---

# AquaGymFit

Mobile app for aquatic fitness classes — booking, subscriptions, class management.

## Status

🟡 **Refactor in progress** — the original `aquagymfit/` project (FlutterFlow-based) is being fully rewritten in `new_aquagymfit/` with clean architecture.

## Projects

### `~/freelance/aquagymfit` — Original (FlutterFlow)
- Built with FlutterFlow, then partially modified by a second team
- Mixed state management: Provider + GetX + BLoC
- 140+ packages, FlutterFlow routing (`FFRoute`/`FFParameters`)
- Firebase auth: Google, Apple, Email/Password, Anonymous
- Still the version in production

### `~/freelance/new_aquagymfit` — Refactor
- Complete rewrite from scratch, same features and design
- Feature-based clean architecture
- BLoC only for state management
- ~60 packages
- Standard Flutter `ThemeData` + `GoRouter` with `StatefulShellRoute`
- Domain-specific repositories (replaced 1226-line `StoreFirebase` god-class)
- Unit tests per feature

## Key Features
- Class booking with real-time scheduling
- Calendar integration
- Waitlist management
- Subscription management
- Multi-provider auth (Google, Apple, Email, Anonymous)

## Stack
- **Framework**: Flutter
- **Backend**: Firebase (Firestore, Auth, Cloud Functions)
- **State**: BLoC (new) / Provider+GetX+BLoC mix (old)
- **Routing**: GoRouter (new) / FFRoute (old)

## Files on Archive (SSD)
- `Facture/` — invoices sent to client (devis, factures from ~2022)
- Various Apple auth keys (`.p8` files)
