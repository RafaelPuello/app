# CLAUDE.md - DigiDex App Service

Next.js 16 frontend + Django 6.0 backend for main user-facing application (NFC tag management, plant discovery via Pokedex).

**Detailed guides:**
- **Backend**: See `/app/backend/CLAUDE.md` for Django architecture, API contracts, models, database
- **Frontend**: See `/app/frontend/CLAUDE.md` for Next.js pages, React components, sidebar, styling

## Quick Setup

Backend: `cd app/backend && pip install -r requirements.txt && python manage.py migrate && python manage.py runserver 0.0.0.0:8000`

Frontend: `cd app/frontend && npm install && npm run dev`

Docker: `docker compose -f compose.yaml -f compose.override.yaml up`

See README.md for full setup instructions and command reference.

## Architecture

- **Backend**: Django 6.0 + django-ninja REST API at `/app/api/` (NFC tags, plant collections, GBIF integration)
- **Frontend**: Next.js 16 + React 19 at `/app/` (pages: dashboard, plants, home)
- **Auth**: JWT tokens from ID service; validated by backend
- **Data flow**: Frontend → `/app/api/*` (backend) → `/accounts/*`, `/_allauth/*` (ID service)
- **Routing**: Traefik path-based (no stripping); Django handles full `/app/api/` prefix; Next.js `basePath: '/app'`
- **Network**: Both on `digidex-net` external Docker network (Traefik discovery)
- **NFC Features**: Bind/unbind endpoints; PlantLabel model (concrete NFC tag)
- **Pokedex**: Plant search (curated seed + GBIF API); PokedexGrid + PlantForm + NFCScanner components

## Conventions

- API endpoints: RESTful, typed schemas, JWT auth decorators on non-public routes
- Frontend: TypeScript, functional components, React Context for state, `@/` path alias
- Sidebar: Phase 2-4 (responsive mobile hamburger + tablet collapse + glassmorphic desktop)
- Shared styling: `/app/frontend/src/styles` symlinks to `/shared/styles` (design tokens, mixins)

## Gotchas

- **Traefik path handling**: Routes `/app/*` WITHOUT stripping the path prefix. See `.claude/rules/traefik-path-handling.md` for the complete contract and common mistakes.
- **basePath required**: Next.js needs `basePath: '/app'` in config for assets and routing to work behind Traefik (covered in traefik rule).
- **Shared styles symlink**: Frontend expects `src/styles` → `../../shared/styles`. If missing, SCSS imports fail.
- **Port 8000**: Same as CMS backend. Traefik priority rules distinguish them.

## Key Files

- Backend settings: `app/backend/config/settings.py`
- Backend API: `app/backend/config/api.py` (django-ninja router)
- Frontend config: `app/frontend/next.config.ts`
- Sidebar: `app/frontend/src/components/sidebar/` (complex state machine + context)
- Models: `app/backend/domain/models.py`, `app/backend/nfctags/models.py`
- API routes: `app/backend/config/urls.py` (mounts at `'app/api/'`)
