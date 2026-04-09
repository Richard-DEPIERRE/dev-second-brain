---
type: freelance-project
status: active
client: RSH Digital / Qhare
stack: [Flutter]
code_path: "~/freelance/qhare_mobile"
archive_path: "/Volumes/SSD Richard/freelance/Qhare"
---

# Qhare — RSH Digital

Flutter mobile CRM for energy renovation salespeople (solar panels, heat pumps, insulation, etc.), contracted through RSH Digital. Longest-running active contract — monthly invoicing since October 2023.

## Status

🟢 **Active** — ongoing monthly contract with RSH Digital.

## Contract
- Service contract signed: `CONTRAT DE PRESTATION DE SERVICE DE DÉVELOPPEMENT MOBILE FLUTTER`
- Stored at `/Volumes/SSD Richard/freelance/Qhare/`

## Invoicing History
Monthly factures stored at `/Volumes/SSD Richard/freelance/Qhare/facture/`:

| Period | File |
|--------|------|
| 2023-10 to 2023-12 | Facture 2023:10–12 RSH DIGITAL |
| 2024-01 to 2024-12 | Facture 2024:01–12 RSH DIGITAL |
| 2025-01 to 2025-11 | Facture 2025:01–11 RSH DIGITAL |
| 2025-12 | FAC 2025-12 RSH DIGITAL |
| 2026-01 | FAC-2026-000001 RSH DIGITAL |
| 2026-02 | FAC-2026-000002 RSH DIGITAL |
| 2026-03 | FAC-2026-000003 RSH DIGITAL |

Also: `Courrier du 2025-11-26.pdf` and `STATUTS Signé-Horyond Agency.pdf` in same folder.

## Stack
- **Framework**: Flutter (Dart) — version managed via FVM (`.fvmrc`)
- **State**: flutter_bloc + freezed (migrating to 100% BLoC — see Refactor)
- **Navigation**: GetX (Get.to / Get.offAllNamed), routes in `lib/routes.dart`
- **HTTP**: Dio + OAuth2 bearer tokens (refresh handled by `MainService`)
- **Local DB**: sqflite (`lib/database/`)
- **Auth storage**: flutter_secure_storage (tokens, B2B toggle, language)
- **Push notifications**: Firebase Messaging + firebase_analytics
- **Maps**: google_maps_flutter + flutter_polyline_points
- **PDF**: pdf package (generation) + flutter_pdfview (viewing)
- **i18n**: fr + es via ARB files (`lib/l10n/`) + GoogleTranslatorInit fallback
- Apple auth keys in `~/freelance/qhare_mobile/` (multiple `.p8` files)

## Apps & Environments

| Flavor | App Name | Bundle ID | Distribution |
|--------|----------|-----------|--------------|
| `dev` | Qhare Dev | `com.crm-qhare.qhare.dev` | TestFlight (iOS QA) |
| `prod` | Qhare | `com.crm-qhare.qhare` | App Store + Play Store |

Environment currently switched manually in `lib/utils/env_variable.dart`. Flutter build flavors are a planned refactor item.

## Branching Strategy — GitFlow

```
master               → production releases (tagged)
develop              → integration branch
feature/<name>       → single task (PR → develop)
bigfeature/<name>    → large initiative, acts as sub-develop (PR → develop when complete)
hotfix/<name>        → urgent fix (master → master + develop)
release/<version>    → release prep (develop → master + develop)
```

## Active Branches

| Branch | Purpose | Status |
|--------|---------|--------|
| `master` | Production | stable |
| `develop` | Integration | active |
| `bigfeature/refactor` | Full app refactor (BLoC, flavors, English) | 🟡 in progress |

## Active Refactor (bigfeature/refactor)

The full app refactor is tracked on `bigfeature/refactor`. Individual tasks branch off it as `feature/<task>` and PR back into it. Final merge → `develop` when all tasks complete.

Refactor goals:
1. **BLoC migration** — 100% BLoC for all non-trivial state (from scattered StatefulWidget)
2. **Build flavors** — proper `dev`/`prod` Flutter flavor setup (replace manual env switch)
3. **English codebase** — all variable/method/class names in English (UI stays French/Spanish via l10n)
4. **Service cleanup** — remove BuildContext from services; error handling in BLoC/widgets

## Development Conventions

- All code in **English** — UI strings only via `lib/l10n/` ARB files (never hardcoded)
- Tests for BLoC, services, model serialization — skip pure UI rendering
- One service per domain, no BuildContext in services
- `dart run build_runner build --delete-conflicting-outputs` after @freezed changes
- `flutter gen-l10n` after ARB file changes

## Notes
- Code at `~/freelance/qhare_mobile/`
- CLAUDE.md in repo root — full project reference for Claude Code
- Mockups at `/Volumes/SSD Richard/freelance/Qhare/*.mockup`
- Architecture graph: [[Architecture]]
