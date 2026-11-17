# Crypto Yield Maxi Docs

This folder keeps the durable project documentation for the current app in this repository.

## Core Docs

- [architecture.md](architecture.md): backend/frontend structure, data flow, active cron jobs, tables
- [api.md](api.md): current public API surface and response shapes
- [deployment.md](deployment.md): production deployment notes, env vars, migrations
- [frontend-pages.md](frontend-pages.md): current routes and important UX behavior
- [exchanges.md](exchanges.md): active exchange coverage and curated guide lists
- [exchange-symbol-mapping.md](exchange-symbol-mapping.md): symbol canonicalization and cross-venue matching
- [filters.md](filters.md): centralized and environment-driven filtering thresholds
- [state-machines.md](state-machines.md): sync, cache, retention and recovery flows
- [STRING_UTILS.md](STRING_UTILS.md): string normalization helper behavior

## Current Product Shape

The app currently ships these major surfaces:

- `/`: landing page with product overview and a static `380+` exchange stat
- `/delta-neutral`: funding extraction opportunities, same-exchange and cross-exchange modes
- `/arbitrage`: two-leg scanner (Price / Funding modes) with server-side filter/sort/pagination
- `/lending`: lending pools grouped by asset
- `/vaults`: vault opportunities with notice modal and APR history chart
- `/airdrop`: curated airdrop opportunities from the database
- `/guide`: educational guide + curated exchange directory
- `/about`: project/about/support page
- `/trending`: placeholder screen
- `/health`: frontend route handler returning the deployed component and release SHA (not a UI page)

## Active Sync Jobs

The cron worker (`backend/src/cron/index.ts`) runs:

- Arbitrage scanner every 5 minutes
- Delta-neutral funding extraction every hour at `:03`
- Lending pools every hour at `:07`
- Vaults every hour at `:09`
- Hourly history snapshot guards at `:12` and `:42`
- Telegram reports at `:10` (top opportunities) and `:15` (hourly summary)
- Daily backup at `02:30 UTC`, retention cleanup at `03:30 UTC`, history self-heal at `04:45 UTC`

There is no exchange-directory cron. `/api/exchanges` is built from the live client registry, while the guide page uses curated frontend data plus `ref_links` overrides.

## Notes

- Temporary audit reports and one-off checklists are intentionally not kept here once the resulting fixes are merged.
- Backend-only operational notes may also live under `backend/docs/`; frontend deployment/CI notes live under `frontend/docs/`.
