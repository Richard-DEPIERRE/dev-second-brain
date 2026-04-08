---
type: architecture-graph
project: Qhare Mobile
generated: 2026-04-09
tool: graphify + manual Dart analysis
---

# Qhare Mobile — Architecture Graph

> Flutter CRM for energy renovation salespeople. Multi-tenant (per régie), dual B2C/B2B mode, French subsidy logic (MPR, CEE primes).
> Code: `~/freelance/qhare_mobile`

---

## High-Level Architecture

```mermaid
graph TD
    subgraph Entry["Entry & Boot"]
        SPLASH[SplashScreen]
        LOGIN[LoginScreen]
        HOME[HomePage]
    end

    subgraph State["State Layer"]
        BLOC[UserBloc]
        DB[(SQLite\nuserlistqharesplanning.db)]
        SEC[(FlutterSecureStorage\ntokens · btob · language)]
    end

    subgraph API["API Layer (OAuth2 / Dio)"]
        MAIN[MainService\ntoken refresh]
        AUTH[Authentification\nlogin · logout]
        CLIENT[ClientService ★\nleads · filters · docs · comments]
        DEVIS[DevisService\nquotes · invoices · primes · PDF]
        OPS[OperationService\ntravaux · champs · dimensionnement]
        PLAN[PlanningService\ncalendar events]
        MEET[MeetingsService\nposeur RDV]
        NOTIF[NotificationsService]
        SMS[SmsEmailService\ntemplates · send · signature]
        MPR[MprService\nMaPrimeRénov check]
        BTOB[BtoBService\nentité toggle · sites]
        PARC[ParcoursService\npipeline steps]
        FORMS[FormsService\ncustom surveys]
        SEARCH[SearchService]
        MAPS[MapsApi\ndistance matrix]
        VER[VersionService\nforce-update gate]
    end

    subgraph Firebase["Firebase"]
        FCM[Firebase Messaging\npush notifications]
        FCA[Firebase Analytics]
        FCF[Cloud Functions\nversion check endpoint]
    end

    SPLASH --> DB
    SPLASH --> VER
    SPLASH --> FCF
    SPLASH --> LOGIN
    SPLASH --> HOME
    LOGIN --> AUTH
    AUTH --> SEC
    BLOC --> DB
    MAIN --> SEC

    HOME --> NOTIF
    HOME --> PLAN
    HOME --> MEET
    HOME --> FCM

    CLIENT -.-> MAIN
    DEVIS -.-> MAIN
    OPS -.-> MAIN
    PLAN -.-> MAIN
```

---

## Domain Model Map

```mermaid
erDiagram
    USER {
        int id
        string regieId
        bool superadmin
        bool admin
        bool telepro
        bool call
        bool poseur
        bool commercial
        bool devis
        bool mpr
        bool sms
    }

    CLIENT {
        int id
        string numdossier
        string nom
        string etat
        string sousEtat
        float mpr
        float rac
        float cumac
        int regieId
        bool btob
        string siret
    }

    DEVIS_LEAD {
        int id
        string dealName
        float primeCee
        float primeMpr
        float rac
        float prixTtc
        string etatDevis
        int regieId
    }

    OPERATION {
        int id
        string name
        string categoryId
        string nature
        bool btob
    }

    PLANNING {
        int id
        string title
        datetime start
        datetime end
        int userId
        int poseurId
        bool btob
    }

    PARCOURS {
        int id
        string nom
        string couleur
        bool flGagne
        bool flEtapeCourante
    }

    REGIE {
        int id
        string name
        string siret
        string qualibat
        string qualiPv
    }

    USER ||--o{ CLIENT : manages
    USER ||--o{ PLANNING : owns
    REGIE ||--o{ CLIENT : scopes
    REGIE ||--o{ DEVIS_LEAD : scopes
    CLIENT ||--o{ DEVIS_LEAD : has
    CLIENT ||--o{ PLANNING : linked_to
    CLIENT ||--o{ PARCOURS : follows
    DEVIS_LEAD ||--o{ OPERATION : contains
```

---

## Screen Navigation Flow

```mermaid
flowchart TD
    SPLASH([SplashScreen /]) -->|token valid| HOME([HomePage /home])
    SPLASH -->|no token| LOGIN([LoginScreen /login])
    LOGIN -->|success| HOME

    HOME --> CLIENTS([ClientsLeads])
    HOME --> PLANNING([Planning /])
    HOME --> MEETINGS([Meetings])
    HOME --> SETTINGS([Settings /settings])
    HOME --> NOTIFS([Notifications /notifications])
    HOME --> INFO([Information /information])

    CLIENTS --> SEARCH([Search])
    CLIENTS --> FILTER([ClientFilter])
    CLIENTS --> CLIENT_INFO([ClientInfo])

    CLIENT_INFO --> DOCS([ClientDocuments])
    CLIENT_INFO --> COMMENTS([Comments])
    CLIENT_INFO --> DEVIS_FAC([DevisFacture])
    CLIENT_INFO --> SEND_MAIL([SendMailSms])
    CLIENT_INFO --> SITES([Sites — B2B])
    CLIENT_INFO --> MODIFY_CLIENT([ClientModify])
    CLIENT_INFO --> LEADS_MOD([LeadsModify])

    DEVIS_FAC --> MOD_DEVIS([ModifyDevis])
    DEVIS_FAC --> MOD_OPS([ModifyOperations])
    DEVIS_FAC --> MOD_PROD([ModifyProduits])
    DEVIS_FAC --> MOD_PREST([ModifyPrestations])
    DEVIS_FAC --> MOD_PRIMES([ModifyPrimes])
    DEVIS_FAC --> MOD_FACTURE([ModifyFacture])
    DEVIS_FAC --> REGLEMENTS([AjouterReglements])
    DEVIS_FAC --> DP_MAIRIE([ModifyDpMairie])
    DEVIS_FAC --> RENOVATION([Renovation])

    PLANNING --> ADD_PLAN([AddPlanning])
```

---

## Service → Model Dependency Map

```mermaid
graph LR
    subgraph Services
        CS[ClientService ★]
        DS[DevisService]
        OS[OperationService]
        PS[PlanningService]
        MS[MeetingsService]
        SS[SmsEmailService]
        BS[BtoBService]
        PAS[ParcoursService]
        FS[FormsService]
        NS[NotificationsService]
        RS[RdvService]
        MPRS[MprService]
        MAPS[MapsApi]
        SEAS[SearchService]
    end

    subgraph Models
        UM[UserModel]
        CM[ClientModel]
        DM[DevisLeadModel]
        OM[OperationInfoModel]
        PM[PlanningModel]
        RDV[RdvModel]
        NM[NotificationsModel]
        SIM[SiteModel]
        PAM[ParcoursModel]
        FM[FormsModel]
        EM[EmailModel]
        SM[SmsModel]
        DOC[DocumentModel]
        COM[CommentModel]
        PROD[ProduitsInfoModel]
        PRES[PrestationInfoModel]
    end

    CS --> CM
    CS --> DOC
    CS --> COM
    DS --> DM
    DS --> PROD
    DS --> PRES
    OS --> OM
    PS --> PM
    MS --> RDV
    SS --> EM
    SS --> SM
    BS --> SIM
    PAS --> PAM
    FS --> FM
    NS --> NM
    RS --> RDV
    SEAS --> CM
```

---

## Key Architectural Insights

### God Nodes (highest centrality)

| Node | Why it matters |
|------|---------------|
| `ClientService` | Used by virtually every screen — the gravitational center of the graph |
| `DevisService` | Most complex service (~1800 lines, 30+ endpoints) — entire quote-to-invoice lifecycle |
| `ClientModel` | The core entity; carries 50+ fields covering property, lead source, financials, roles |
| `DevisLeadModel` | Aggregates quote, invoice, operations, products, primes, financing, DP mairie |

### Invisible Root: Régie

Every entity (user, client, devis, operation, product, site, RDV) carries `regieId`. The régie (installation company) is the invisible multi-tenant root scoping all data. There is no superadmin cross-régie view in the app.

### B2B / B2C Bifurcation

```mermaid
graph TD
    APP[App boot] --> SEC{BtobUtils\n_isBtob?}
    SEC -->|false| B2C[B2C mode\nnormal header color\nstandard forms]
    SEC -->|true| B2B[B2B mode\npurple header tint\nSIRET fields\nSite multi-address\ndifferent endpoints]
```

B2B toggle persists in `FlutterSecureStorage`. All service calls append `btob`/`BtoB_mobile` params. `BtoBService.choixEntite()` syncs the toggle server-side.

### Authentication & Session Lifecycle

```mermaid
sequenceDiagram
    participant App
    participant Storage as FlutterSecureStorage
    participant API as Backend OAuth2

    App->>Storage: read access_token
    alt token valid
        App->>API: API call with Bearer token
    else 401 or expired
        App->>Storage: read refresh_token
        App->>API: POST /oauth/token (refresh_token grant)
        API-->>App: new access_token + refresh_token
        App->>Storage: write new tokens
    else refresh fails
        App->>App: DisconnectUser().disconnect()
        App->>App: Get.offAllNamed('/login')
    end
```

### Financial Subsidy Logic (MPR + CEE)

The app models the full French energy renovation subsidy ecosystem:

| Subsidy | Model field | Service |
|---------|------------|---------|
| **MaPrimeRénov (MPR)** | `ClientModel.mpr`, `DevisLeadModel.primeMpr` | `MprService` + `ClientService.check_avis` |
| **CEE primes** | `DevisLeadModel.primeCee`, `DevisLeadModel.cumac` | `OperationService.get_primes` |
| **RAC** (reste à charge) | `ClientModel.rac`, `DevisLeadModel.rac` | Computed in `DevisService.recalculer` |
| **Avis Fiscal** (DGFiP) | `AvisResponseModel` | `ClientService.check_avis` → DGFiP API |
| **Financement** (loans) | `FinancementInfo`, `FinancementModel` | `DevisService` |

### Push Notification Deep Links

Firebase push notifications can navigate directly to:
- `lead/{id}` → `ClientInfo`
- `information_qhare` → `InformationPage`

Handled in both foreground (`_firebaseMessagingBackgroundHandlerInApp`) and background tap (`_handleMessage`). Badge count managed via `flutter_new_badger`.

### Force-Update Gate

`SplashScreen` calls `VersionService` → Firebase Cloud Function `getMinimumVersion`. If the installed version is below minimum, the app shows a mandatory update dialog before routing anywhere.

---

## Communities (graphify — native code layer)

> Note: graphify only analyzed non-Dart files (iOS/Android boilerplate, Firebase functions).
> The Dart architecture above was analyzed separately.
> Source: `~/freelance/qhare_mobile/graphify-out/GRAPH_REPORT.md`

| Community | Nodes | Purpose |
|-----------|-------|---------|
| iOS App Entry Point | AppDelegate → FlutterAppDelegate | iOS native bootstrap |
| Flutter Plugin Registration | GeneratedPluginRegistrant (iOS + Android) | Auto-generated plugin wiring |
| Android App Entry Point | MainActivity | Android native bootstrap |
| Qhare Flutter App | qhare ↔ Flutter | Core framework relationship |
| iOS Bridging Header | Runner-Bridging-Header.h | Obj-C/Swift bridge |
| Firebase Cloud Functions | index.js | Version check cloud function |

---

## Files of Interest

| Path | Purpose |
|------|---------|
| `lib/main.dart` | App boot, Firebase init, UserBloc provision, GoogleTranslatorInit |
| `lib/routes.dart` | Named route map (6 top-level routes) |
| `lib/bloc/bloc/user_bloc.dart` | Single BLoC — loads user from SQLite |
| `lib/service/main_service.dart` | BaseApi + token refresh |
| `lib/service/client_service.dart` | ★ Most-used service |
| `lib/service/devis_service.dart` | Most complex service (~1800 lines) |
| `lib/utils/btob_utils.dart` | Global B2B mode toggle |
| `lib/utils/disconnect.dart` | Clears tokens + SQLite on logout |
| `lib/translator/google_translator.dart` | Wraps app in fr→es translation layer |
| `lib/database/user_database.dart` | SQLite user persistence |
| `functions/index.js` | Firebase Cloud Function — version gate |
| `.env` / `.env.production` | API base URL, OAuth2 client credentials |
| `graphify-out/graph.html` | Interactive graph (open in browser) |
| `graphify-out/GRAPH_REPORT.md` | graphify audit report |
