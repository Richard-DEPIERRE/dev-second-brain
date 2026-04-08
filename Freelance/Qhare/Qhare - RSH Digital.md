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
- **Framework**: Flutter (Dart)
- **State**: flutter_bloc + freezed
- **Navigation**: GetX (Get.to / Get.offAllNamed)
- **HTTP**: Dio + OAuth2 bearer tokens
- **Local DB**: sqflite
- **Auth storage**: flutter_secure_storage
- **Push notifications**: Firebase Messaging + firebase_analytics
- **Maps**: google_maps_flutter + flutter_polyline_points
- **PDF**: pdf package (generation) + flutter_pdfview (viewing)
- **i18n**: fr + es (flutter_localizations + custom GoogleTranslatorInit)
- Apple auth keys in `~/freelance/qhare_mobile/` (multiple `.p8` files)

## Notes
- Code at `~/freelance/qhare_mobile/`
- Mockups at `/Volumes/SSD Richard/freelance/Qhare/*.mockup`
- Architecture graph: [[Architecture]]
