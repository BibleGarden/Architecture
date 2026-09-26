# Bible Garden Architecture

## 1. Overview

The system consists of two API services, an admin dashboard, a data pipeline for
data preparation, the Bible Garden app and the Lampada prayer app.

**Principle**: data is prepared in the admin contour, and only verified and finalized data reaches the public contour via export.

The provider and model live in a particular deployment are operational facts,
not architectural ones. The stable selection policy is recorded in
[ADR-0002](decisions/0002-ai-model-provider-policy.md).

```mermaid
graph TB
    subgraph Clients
        DASH[Dashboard-Web<br/>Vue 3 SPA]
        IOS[iOS-App<br/>Bible Garden]
        LAMPADA[Lampada-Mobile<br/>Expo / React Native]
    end

    subgraph AI processing
        AI[Configured AI provider]
    end

    subgraph Admin Contour
        ADM_API[Dashboard-API<br/>FastAPI]
        ADM_DB[(cep_admin<br/>MySQL 8.4)]
        PARSER[bible-parser<br/>scripts + MFA]
    end

    subgraph Public Contour
        PUB_API[Bible-API<br/>FastAPI]
        PUB_DB[(cep_public<br/>MySQL 8.4)]
    end

    PARSER --> ADM_DB

    DASH -->|JWT| ADM_API
    ADM_API --> ADM_DB
    ADM_API -.->|read stats| PUB_DB

    IOS -->|API Key| PUB_API
    LAMPADA -->|API Key| PUB_API
    PUB_API --> PUB_DB
    PUB_API -->|consent-gated prayer content| AI

    PUB_API -->|import| ADM_API
```

## 2. GitHub Repositories

All repositories are under the [Bible-Garden](https://github.com/Bible-Garden) organization.

| Repository | Local directory | Description |
|------------|----------------|-------------|
| [Bible-API](https://github.com/Bible-Garden/Bible-API) | `Bible-API/` | Public read-only API for the iOS app |
| [Dashboard-API](https://github.com/Bible-Garden/Dashboard-API) | `Dashboard-API/` | Admin API — data management, quality control, export |
| [Dashboard-Web](https://github.com/Bible-Garden/Dashboard-Web) | `Dashboard-Web/` | Vue 3 admin dashboard — alignment review and QC |
| [iOS-App](https://github.com/Bible-Garden/iOS-App) | — | Bible Garden iOS app |
| [Lampada-Mobile](https://github.com/BibleGarden/Lampada-Mobile) | `Lampada-Mobile/` | Prayer app with local journal and optional AI features |
| [Architecture](https://github.com/Bible-Garden/Architecture) | `Architecture/` | This documentation |

## 3. Data Flow

```mermaid
flowchart LR
    A[Download texts<br/>bible-parser] --> B[Forced Alignment<br/>MFA]
    B --> C[Save to cep_admin<br/>bible-parser]
    C --> D[Quality checks<br/>quality scripts]
    D --> E[Manual fixes<br/>Dashboard-Web]
    E --> F[Import to cep_public<br/>Bible-API]
    F --> G[iOS-App]
```

### What happens at each step

1. **bible-parser** downloads Bible texts from open sources
2. **MFA** aligns text with audio (word-level timestamps)
3. Results are saved to `cep_admin` database (voice_alignments, voice_chapters)
4. **Python scripts** analyze quality and write anomalies to voice_anomalies
5. Operator uses **Dashboard-Web** to review anomalies and apply manual corrections (voice_manual_fixes)
6. **Bible-API** fetches data from Dashboard-API and loads it into `cep_public` (manual_fixes applied)
7. **iOS-App** receives clean data via Bible-API

## 4. Services

### Bible-API

Public read-only API for the iOS app. Minimal, fast, no DB writes (except import and request logging).

- **Repo**: [Bible-Garden/Bible-API](https://github.com/Bible-Garden/Bible-API)
- **Port**: 8084
- **Auth**: API Key (`X-API-Key`)
- **DB**: `cep_public` (SELECT only, INSERT during import and request stats logging)
- **Production**: `https://api.bible.garden/api`
- **Request stats**: middleware logs authenticated API requests with resolved application identity to `api_requests` (background thread)

### Dashboard-API

API for the dashboard and data management. Read + write.

- **Repo**: [Bible-Garden/Dashboard-API](https://github.com/Bible-Garden/Dashboard-API)
- **Port**: 8085
- **Auth**: JWT (admin endpoints), API Key (read endpoints)
- **DB**: `cep_admin` (full access), `cep_public` (read-only, for API stats)

### Dashboard-Web

Vue 3 SPA for reviewing and correcting forced alignment.

- **Repo**: [Bible-Garden/Dashboard-Web](https://github.com/Bible-Garden/Dashboard-Web)
- **Port**: 5174
- **Stack**: Vue 3, TypeScript, PrimeVue, TailwindCSS, Chart.js
- **Connects to**: Dashboard-API (`/bible-api` proxy), alignment-api (`/alignment-api` proxy)
- **Pages**: Voices, Anomalies, Inspect, Alignment Tasks, API Stats

### bible-parser

Data pipeline: text downloading, forced alignment, DB loading. Not a running service — utility scripts.

- **Stack**: PHP 8.3, Python 3, MFA (Docker), ffmpeg
- **DB**: `cep_admin` (direct write)

### iOS-App

Bible Garden iOS app — listen to the Bible with pauses and multilingual verse-by-verse playback.

- **Repo**: [Bible-Garden/iOS-App](https://github.com/Bible-Garden/iOS-App)
- **Connects to**: Bible-API (`https://api.bible.garden/api`)

### Lampada-Mobile

Lampada is a separate Expo application for guided prayer. Prayer sessions,
answers, recordings, transcripts, favourites and settings are stored locally on
the device. It uses Bible API for generated questions, optional transcription,
contextual scripture selection, scripture text and narration.

Prayer content is sensitive personal data. Its transfer is governed by the
[Lampada data processing rules](privacy/lampada-data-processing.md) and
[ADR-0001](decisions/0001-lampada-ai-data-processing.md). Production AI
processing requires purpose-specific explicit consent and a paid provider
contract with optional provider logging and data sharing disabled.

## 5. Endpoints

### Bible-API

| Method | Path | Tag |
|--------|------|-----|
| GET | `/api/languages` | Languages |
| GET | `/api/translations` | Translations |
| GET | `/api/translations/{code}/books` | Translations |
| GET | `/api/excerpt_with_alignment` | Excerpts |
| GET/HEAD | `/api/audio/{translation}/{voice}/{book}/{chapter}.mp3` | Audio |
| GET | `/api/about` | About |
| GET | `/api/version-check` | Version |
| POST | `/api/ai/question` | AI |
| POST | `/api/ai/transcribe` | AI |
| POST | `/api/ai/scripture` | AI |
| POST | `/api/cache/clear` | Cache |
| GET | `/api/import` | Import |

Bible-API also runs `RequestStatsMiddleware` that logs authenticated API requests with application identity to `api_requests`. It excludes docs, `/api/health`, audio OPTIONS, authentication failures and unattributed 4xx responses. Dynamic paths are normalized (e.g. `/api/audio/*`, `/api/translations/*/books`).

### Dashboard-API

| Method | Path | Tag | Auth |
|--------|------|-----|------|
| POST | `/api/auth/login` | Auth | — |
| GET | `/api/languages` | Languages | API Key |
| GET | `/api/translations` | Translations | API Key |
| GET | `/api/translation_info` | Translations | API Key |
| GET | `/api/translations/{code}/books` | Translations | API Key |
| GET | `/api/excerpt_with_alignment` | Excerpts | API Key |
| GET | `/api/chapter_with_alignment` | Excerpts | API Key |
| GET/HEAD | `/api/audio/{translation}/{voice}/{book}/{chapter}.mp3` | Audio | API Key |
| PUT | `/api/translations/{code}` | Admin | JWT |
| PUT | `/api/voices/{code}` | Admin | JWT |
| GET | `/api/voices/{code}/anomalies` | Admin | JWT |
| POST | `/api/voices/anomalies` | Admin | JWT |
| PATCH | `/api/voices/anomalies/{code}/status` | Admin | JWT |
| POST | `/api/voices/manual-fixes` | Admin | JWT |
| GET | `/api/check_translation` | Admin | JWT |
| GET | `/api/check_voice` | Admin | JWT |
| POST | `/api/cache/clear` | Admin | JWT |
| GET | `/api/data` | Export | API Key |
| GET | `/api/stats/summary?days=30` | Statistics | JWT |
| GET | `/api/stats/recent?limit=50` | Statistics | JWT |

## 6. Databases

### cep_public (public, read-only)

Contains only finalized data for the iOS app.

| Table | Purpose |
|-------|---------|
| `languages` | Languages (en, ru, uk) |
| `bible_books` | Bible book reference (66 books) |
| `translations` | Translations (active=1 only) |
| `translation_books` | Books per translation |
| `translation_verses` | Verse texts |
| `translation_titles` | Section headings |
| `translation_notes` | Verse footnotes |
| `voices` | Voices (active=1 only) |
| `voice_alignments` | Verse timings (with manual_fixes applied) |
| `api_requests` | Raw API request log (14-day retention) |
| `api_request_daily_stats` | Aggregated daily API stats (permanent) |

Adding a **new language** is not just a `languages` row — `bible_books` carries per-language columns (`short_name_*`, `full_name_*`) and code in Bible-API, bible-parser and the evaluation set enumerates `ru`/`uk`/`en` in ~100 enumerated places across six repositories, including the despair rule of `app/safety.py`. The complete checklist is `Bible-API/architect/adding-a-language.md`.

### cep_admin (admin, full access)

Contains all data, including working and technical tables.

| Table | Purpose |
|-------|---------|
| `languages` | Languages |
| `bible_books` | Bible book reference |
| `bible_stat` | Expected verse counts (for integrity checks) |
| `translations` | All translations (including inactive) |
| `translation_books` | Books per translation |
| `translation_verses` | Verse texts |
| `translation_titles` | Section headings |
| `translation_notes` | Verse footnotes |
| `voices` | All voices (including inactive) |
| `voice_alignments` | Verse timings (original, from MFA) |
| `voice_manual_fixes` | Manual timing corrections |
| `voice_anomalies` | Detected alignment anomalies |
| `voice_chapters` | MFA technical data (input/output/timecodes) |
| `phinxlog` | Migration history |

### Database relationship

```mermaid
flowchart RL
    subgraph Dashboard-API
        CEP[(cep_admin)]
    end

    subgraph Bible-API
        PUB[(cep_public)]
    end

    PUB -->|GET /api/import| CEP
```

During import, Bible-API calls `GET /api/data` on Dashboard-API. On the Dashboard-API side, `COALESCE(vmf.begin, va.begin)` is applied — `cep_public.voice_alignments` receives the final values. Bible-API works without COALESCE.

## 7. Directory Structure

```
cep/
├── Bible-API/           # Bible-API (FastAPI) — github.com/Bible-Garden/Bible-API
│   ├── app/
│   │   ├── main.py            # Entry point, routers
│   │   ├── excerpt.py         # Content endpoint (simplified, no COALESCE)
│   │   ├── audio.py           # MP3 streaming with Range support
│   │   ├── about.py           # About page
│   │   ├── version_check.py   # Version check
│   │   ├── import_data.py     # Data import from Dashboard-API
│   │   ├── middleware.py       # RequestStatsMiddleware (logs requests to DB)
│   │   ├── aggregate_stats.py # Daily stats aggregation (cron)
│   │   ├── auth.py            # API Key only
│   │   ├── models.py          # Pydantic models
│   │   ├── database.py        # Connection to cep_public
│   │   └── config.py          # Env variables
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── Dashboard-API/       # Dashboard-API (FastAPI) — github.com/Bible-Garden/Dashboard-API
│   ├── app/
│   │   ├── main.py            # Entry point, all admin endpoints
│   │   ├── excerpt.py         # Content endpoints (with COALESCE)
│   │   ├── audio.py           # MP3 streaming with Range support
│   │   ├── checks.py          # Integrity checks
│   │   ├── auth.py            # API Key + JWT
│   │   ├── data.py            # Data export for Bible-API
│   │   ├── stats.py           # API usage statistics (reads cep_public)
│   │   ├── models.py          # Pydantic models
│   │   ├── database.py        # Connection to cep_admin
│   │   └── config.py          # Env variables
│   ├── migrations/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── Dashboard-Web/       # Dashboard-Web (Vue 3) — github.com/Bible-Garden/Dashboard-Web
│   ├── src/
│   │   ├── Components/        # Vue components
│   │   ├── composables/       # useApi, useAuth, useAlignmentTasks, useAudioPlayback
│   │   ├── services/          # api.ts (axios), auth.ts (JWT)
│   │   ├── config/            # api.ts (endpoints config)
│   │   ├── types/             # TypeScript types
│   │   ├── utils/             # audio.ts
│   │   └── router/            # Vue Router
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── bible-parser/        # Data pipeline — github.com/MariaPaypoint/bible-parser
│   ├── alignment/             # Forced alignment (MFA)
│   ├── parsing/               # Text parsing
│   ├── quality/               # Anomaly analysis scripts
│   ├── audio/                 # Audio processing
│   └── api/                   # Minimal FastAPI (health check only)
│
├── db/                  # Database setup
│   ├── docker-compose.yml     # MySQL 8.4 (port 3308)
│   └── setup_cep_public.sql   # DDL for cep_public
│
└── Architecture/        # Architecture — github.com/Bible-Garden/Architecture
    └── architecture.md
```

## 8. Docker Compose

Each service has its own `docker-compose.yml`. MySQL is shared via the `mysql_default` network.

```yaml
# db/docker-compose.yml
services:
  mysql:
    image: mysql:8.4
    container_name: cep-mysql
    ports: ["3308:3306"]

# Bible-API/docker-compose.yml
services:
  bible-api:
    container_name: bible-api
    build: .
    ports: ["9084:8000"]
    env_file: .env
    volumes:
      - ${AUDIO_DIR}:/audio
    networks: [mysql_default]

# Dashboard-API/docker-compose.yml
services:
  dashboard-api:
    container_name: dashboard-api
    build: .
    ports: ["8085:8000"]
    env_file: .env
    volumes:
      - ${AUDIO_DIR}:/audio
    networks: [mysql_default]

# Dashboard-Web/docker-compose.yml
services:
  dashboard-web:
    build: .
    container_name: dashboard-web
    ports: ["9086:5173"]
```

## 9. Production

- `bible.garden` — static site only (no API)
- `api.bible.garden` — Bible-API (public read-only)
- SSL via Let's Encrypt, nginx as reverse proxy
- Import is done per-translation to keep memory usage low

## 12. Data Import

Each service owns its own database. Data is transferred via API, not by direct access to another service's DB. Bible-API fetches data from Dashboard-API.

```mermaid
sequenceDiagram
    participant Operator
    participant Public as Bible-API
    participant Admin as Dashboard-API
    participant CepAdmin as cep_admin
    participant CepPublic as cep_public

    Operator->>Public: GET /api/import?translation=syn
    Public->>Admin: GET /api/data?translation=syn
    Admin->>CepAdmin: SELECT (active data + COALESCE manual_fixes)
    CepAdmin-->>Admin: data
    Admin-->>Public: JSON
    Public->>CepPublic: apply one translation transaction
    CepPublic-->>Public: OK
    Public-->>Operator: report (record counts)
```

## 11. API Request Statistics

Tracks Bible-API usage by Bible Garden, Lampada and operations.

### Architecture

```mermaid
flowchart LR
    IOS[Bible Garden] -->|request| PUB[Bible-API]
    LAMP[Lampada] -->|request| PUB
    OPS[Operations] -->|request| PUB
    PUB -->|middleware logs| RAW[(api_requests<br/>cep_public)]
    CRON[Cron 2:00 AM] -->|aggregate_stats.py| AGG[(api_request_daily_stats<br/>cep_public)]
    RAW -.->|yesterday's data| AGG
    CRON -->|purge > 14 days| RAW
    ADM[Dashboard-API] -->|cross-DB read| RAW
    ADM -->|cross-DB read| AGG
    DASH[Dashboard-Web<br/>/stats page] -->|JWT| ADM
```

### Components

- **Bible-API `middleware.py`**: `RequestStatsMiddleware` — logs authenticated requests with `bible-garden`, `lampada` or `ops` identity in a background thread. Normalizes dynamic paths (`/api/audio/*`, `/api/translations/*/books`). Excludes docs, `/api/health`, audio OPTIONS, 403/404/405 and 4xx validation responses without an authenticated application.
- **Bible-API `aggregate_stats.py`**: Cron script — aggregates past raw data by day and endpoint for each application and for `all`, plus per-application and overall daily totals; purges raw rows older than 14 days.
- **Dashboard-API `stats.py`**: Two JWT-protected endpoints that read
  `cep_public` through cross-database queries:
  - `GET /api/stats/summary?days=30` returns totals, the immediately preceding
    period, application breakdown, traffic groups, daily totals and group series, filtered top
    endpoints, slow endpoints from retained raw rows, and today's live data.
    `days` is an exact calendar window including today. Top endpoints can be
    narrowed with `top_group=scripture|ai|other` and `top_endpoint=<substring>`;
    filtering happens before the top-20 limit.
  - `GET /api/stats/recent?limit=50` returns retained raw requests and accepts
    `endpoint`, `status`, `method`, `client_pseudonym` and `application` filters.
- **Traffic grouping**: `/api/ai/*` belongs to `ai`; the remaining `/api/*`
  routes belong to `scripture`; all other paths belong to `other`.
- **Trend availability**: request, error, and response-time comparisons use
  equal adjacent calendar windows. The current unique-IP total covers the
  available raw portion of the selected period (at most 14 calendar dates).
  Its comparison is returned only when retained raw rows cover both complete
  windows; otherwise the previous value is `null` rather than a partial count.
- **Dashboard-Web `ApiStats.vue`**: Summary, application and traffic-group cards, period
  deltas, switchable daily metrics and Scripture/AI series, server-filtered top
  endpoints, slow endpoints, and server-filtered recent requests.

Contract updated on 2026-09-26 against `Dashboard-API/app/stats.py`,
`Dashboard-Web/src/Components/ApiStats.vue`, and Bible-API's
`app/aggregate_stats.py` retention job.

### Tables (in cep_public)

| Table | Retention | Purpose |
|-------|-----------|---------|
| `api_requests` | 14 days | Raw request log (application, endpoint, method, status, response time, keyed client pseudonym, user agent); historical and pre-switch old-writer rows use `unknown` |
| `api_request_daily_stats` | Permanent | Aggregates per day, endpoint and application, including overall `all` endpoint and `_total_` rows (historical and pre-switch rows use `unknown`) |

### Dashboard-API: `GET /api/data[?translation=alias]`

Returns finalized data as JSON. The `translation` parameter (translation alias) is optional.

**Without parameter** — all active data:
1. Reference tables: `languages`, `bible_books`
2. All active translations: `translations` (active=1) + `translation_books`, `translation_verses`, `translation_titles`, `translation_notes`
3. All active voices: `voices` (active=1)
4. `voice_alignments` with COALESCE(vmf.begin, va.begin) applied

**With parameter** `?translation=syn` — single translation data:
1. Reference tables: `languages`, `bible_books`
2. Specified translation + its `translation_books`, `translation_verses`, `translation_titles`, `translation_notes`
3. Voices for this translation: `voices` (active=1)
4. `voice_alignments` for this translation only

### Bible-API: `GET /api/import[?translation=alias]`

Calls Dashboard-API, fetches data, loads into `cep_public`:

**Without parameter** — full resync:
1. Requests `GET /api/data/manifest` and writes the reference tables
2. Fetches each active translation separately and applies it in its own
   transaction, without a global `TRUNCATE`
3. Verifies the imported counts against the manifest and returns the report

**With parameter** `?translation=syn` — single translation update:
1. Requests `GET /api/data?translation=syn` from Dashboard-API
2. Deletes this translation's data from `cep_public`
3. Inserts received data
4. Reference tables (`languages`, `bible_books`) are always updated
5. Returns report
