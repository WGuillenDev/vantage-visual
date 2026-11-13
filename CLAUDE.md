# CLAUDE.md

Guidance for AI coding assistants working in this repository.

## Project

VANTAGE is a web application built for the 2026 NASA Space Apps Challenge. It turns open Earth-observation hazard data into map-based information about landslide hazard and road accessibility for communities in Costa Rica.

The system is split into two repositories:

- `vantage-visual` (this repository): the frontend, an installable PWA.
- `vantage-engine`: the backend (REST API, data pipeline and modeling). It is owned by another team. Never modify it from here.

**Status: in preparation.** No application code exists yet. Do not assume files, scripts or dependencies that are not present in the tree; check first.

## Frontend scope

In scope:

- Rendering maps, statistics and location-based views from the backend REST API.
- Loading, error, empty and offline states for every screen.
- PWA installation, offline cache and local notifications.
- Sending user actions (citizen reports, scenario requests) to the API.

Out of scope:

- Any hazard, probability or routing computation. The client displays results; it never computes or alters them.
- Direct access to databases or third-party data providers.
- Storing API keys or any other secret.

## Simplified architecture

```
Browser (PWA)
├── Screens (React)
│     └── API client (typed from the OpenAPI contract)
│           └── HTTPS · REST/JSON · /api/v1 ──► vantage-engine
├── Map layer (CesiumJS)
└── Service worker (offline cache, local notifications)
```

- The frontend depends only on the versioned REST contract (`/api/v1`) published by the backend.
- Every API response carries metadata (source, data date, computation date) that the UI must display.
- Until the real API is available, screens are built against mock data that follows the same contract. Switching between mock data and the real API is a configuration change, not a code change.
- Endpoint paths and JSON field names come from the contract and are written in Spanish. Keep them exactly as the contract defines them.

## Confirmed technologies

| Concern | Technology |
| --- | --- |
| Framework and build | React 18, TypeScript, Vite |
| Styling | Tailwind CSS |
| Routing | React Router |
| Server state | TanStack Query |
| UI state | Zustand |
| Globe and maps | CesiumJS with Resium |
| Charts | Recharts |
| PWA and offline | vite-plugin-pwa (Workbox), Notification API |
| API client | openapi-fetch with types generated from the OpenAPI contract |
| Tests | Vitest (unit and component), Playwright (end to end) |

Do not add a dependency without a clear need; prefer what is already listed.

## Planned components

Planned screens: Map, Statistics, My zone, Institutional view and Report.

| Component | Responsibility |
| --- | --- |
| `GlobeView` | 3D globe with the global hazard layer |
| `CountryMap` | Canton map by hazard level, with a detail panel |
| `RegionMap` | Road segments and communities in the pilot region |
| `StatsPage` | Canton ranking and exposure statistics |
| `ValidationPanel` | Model validation results, shown as they are |
| `MiZonaPage` | Location-based hazard level and guidance |
| `WhereToGo` | Suggested destinations with a link to external navigation |
| `InstitutionalPage` | Ranking of critical road segments; mark a segment closed or open |
| `ReportPage` | Citizen report form and nearby reports |
| `SourceBadge` | Source and timestamp for every figure |

## Product rules that affect code

- No figure is shown without its source and timestamp.
- Hazard levels are conveyed by text plus color, never by color alone. Minimum contrast is 4.5:1 and every screen is keyboard navigable.
- User location is used in memory only. Never persist it and never log it.
- No secrets in the client. Only public configuration, through `VITE_` variables.
- Model outputs are estimates under stated assumptions. Never label them as predictions or as calibrated probabilities, and show validation results unmodified.
- The UI must keep working offline with the last cached data and show how old that data is.
- UI text is in Spanish (the audience is in Costa Rica). Code, identifiers, comments, commits and documentation in this repository are in English.

## Conventions

- **Language:** English for code, identifiers, technical comments, README, CLAUDE.md, branch names, commits, GitHub issues, pull request titles and descriptions, and API documentation. Spanish only for user-facing UI text and for the Spanish names defined by the API contract.
- **Branches:** `main` is stable and protected. Work happens in the personal branches `dev/wilber` and `dev/ivan` and reaches `main` only through pull requests. Merges are done manually by a team member on GitHub.
- **Task IDs:** frontend tasks are numbered per sprint (`F-S0-01`, `F-S1-03`, ...). Issue and pull request titles start with the task ID, for example `F-S1-03 Add loading, error, empty and offline states`.
- **Commits:** Conventional Commits are mandatory, in English, imperative mood (`feat:`, `fix:`, `docs:`, `chore:`, `test:`, `refactor:`).
- **Authorship:** every commit keeps the developer's real Git identity. Do not add AI co-author trailers or "Generated with" signatures to commits or pull requests.
- **Pull requests:** title and description in English, small and focused on a single task, with `Closes #N` for the related issue.
- **TypeScript:** strict mode; avoid `any`; function components and hooks.
- **API access:** always through the typed client. Never hand-write response types and never call `fetch` directly from a component.
- **Tests:** Vitest files next to the code they cover; Playwright for the end-to-end demo path.
- **Never commit:** `.env` files, build output or large data files.
