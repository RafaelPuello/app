# CLAUDE.md - DigiDex App Service

**Service-specific guides:**
- **Backend**: See `/app/backend/CLAUDE.md` for detailed Django backend architecture, API endpoints, testing, and database setup
- **Frontend**: See `/app/frontend/CLAUDE.md` for Next.js frontend pages, components, styling, and development

This file provides overall service context.

## Project Overview

The App service is the main user-facing application for DigiDex. It combines a Django backend (minimal, mainly for serving the frontend) with a Next.js 16 frontend that provides the primary user interface for managing collections and interacting with NFC tags.

## Service Composition

- **Backend**: Django 6.0 (lightweight, serves Next.js and API proxy)
- **Frontend**: Next.js 16 + React 19 + Tailwind CSS (full application UI)

## Development Setup

### Backend

```bash
cd app/backend

# Install dependencies
pip install -r requirements.txt

# Run development server
python manage.py runserver 0.0.0.0:8000
```

**Settings**: `app/backend/config/settings.py` (single settings file for dev/prod)

### Frontend

```bash
cd app/frontend

# Install dependencies
npm install

# Development server (port 3000)
npm run dev

# Build for production
npm run build

# Start production build locally
npm start

# Linting
npm run lint
```

**Ports**:
- Frontend: 3000 (development)
- Backend: 8000

### Docker Compose (Development)

```bash
# From app/ directory
docker compose -f compose.yaml -f compose.override.yaml up
```

## Project Structure

```
app/
├── backend/                  # Django application
│   ├── config/              # Django settings and URL config
│   │   ├── settings.py      # Main Django settings
│   │   ├── asgi.py
│   │   ├── wsgi.py
│   │   └── urls.py
│   ├── manage.py
│   └── requirements.txt
│
├── frontend/                # Next.js application
│   ├── src/
│   │   ├── app/            # Next.js App Router pages
│   │   │   ├── dashboard/  # Dashboard pages
│   │   │   ├── plants/     # Plant listing/management
│   │   │   └── page.tsx    # Home page
│   │   ├── components/     # Reusable React components
│   │   ├── lib/            # Utilities and types
│   │   │   └── interfaces/ # TypeScript interfaces
│   │   └── styles/         # Global CSS
│   ├── public/             # Static assets
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.ts
│   └── README.md
│
├── .github/                # GitHub Actions workflows
├── compose.yaml
├── compose.override.yaml    # Development overrides
└── .gitmodules

```

## Frontend Architecture

Next.js 16 + React 19 + TypeScript application with:
- App Router based pages (home, dashboard, plants)
- TypeScript for type safety across components
- Tailwind CSS v4 for utility-first styling
- API clients for NFC tags and GBIF integration
- Path alias `@/` for clean imports pointing to `src/`

Component organization: Reusable React components in `src/components/`, pages in `src/app/`, utilities and API clients in `src/lib/`

See `/app/frontend/CLAUDE.md` for page structure, component organization, styling configuration, and development workflows.

## Backend Architecture

Django 6.0 backend with REST API for NFC tags and plant collections via django-ninja. Lightweight service focused on API endpoints rather than serving templates. Features:
- Domain-based app structure (domain, nfctags, botany)
- PlantLabel model for plant collections
- REST API at `/api/` with Swagger documentation
- Pytest-based testing with API parity checks
- Session and JWT authentication support

See `/app/backend/CLAUDE.md` for detailed configuration, API endpoints, testing patterns, and database setup.

## Linting & Code Quality

### Frontend (Next.js/ESLint)

```bash
cd frontend
npm run lint          # Check for linting issues
npm run lint -- --fix # Auto-fix issues (if supported)
```

**ESLint Config**: `eslint.config.mjs`

### Backend (Django)

```bash
cd backend
# (Minimal - main app is frontend-focused)
```

## Testing

### Frontend

```bash
cd frontend
# Testing setup can be added with jest/vitest
# Currently: manual testing recommended during development
```

### Backend

```bash
cd backend
# Tests can be added as the backend grows
```

## API Communication

The frontend communicates with the CMS and other services through:

1. **Direct API calls** - To CMS backend at `/api/` (via Traefik proxy)
2. **Authentication** - Via the ID service
3. **Data fetching** - REST endpoints from CMS

The Traefik reverse proxy (in development and production) handles routing:
- `/api/*` → CMS backend:8000
- `/accounts/*` → ID service backend:8000
- All other paths → App frontend:3000

## Environment Variables

### Frontend

Create `.env.local` for local overrides:

```bash
# API configuration
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

### Backend

Set via environment or `.env` file:

```bash
DEBUG=True
SECRET_KEY=your-secret-key
DATABASE_URL=sqlite:///db.sqlite3
ALLOWED_HOSTS=localhost,127.0.0.1
```

## Docker & Deployment

### Local Development

```bash
docker compose -f compose.yaml -f compose.override.yaml up
```

The `compose.override.yaml` enables hot reloading for development.

### Production Build

1. **Frontend**: Next.js builds to `.next` directory
2. **Backend**: Serves frontend and static assets
3. **Docker**: Multi-stage build optimizes image size

Build:
```bash
docker build -t digidex-app:latest .
```

## Common Tasks

### Add a New Page

1. Create file in `src/app/[feature]/page.tsx`
2. Export default React component
3. Add navigation link in layout/header

Example:
```typescript
// src/app/mypage/page.tsx
export default function MyPage() {
  return (
    <main>
      <h1>My Page</h1>
    </main>
  );
}
```

### Add a New Component

1. Create file in `src/components/MyComponent.tsx`
2. Export component
3. Import and use in pages or other components

### Update Tailwind Config

Edit `tailwind.config.ts` to customize theme:
- Colors, fonts, spacing
- Breakpoints, animations
- Plugin configuration

### Install Dependencies

```bash
# Frontend
cd frontend
npm install <package-name>

# Backend
cd backend
pip install <package-name>
pip freeze > requirements.txt
```

## Troubleshooting

### Port Already in Use

```bash
# Find and kill process on port 3000
lsof -ti:3000 | xargs kill -9

# Or specify different port
npm run dev -- -p 3001
```

### Next.js Build Errors

```bash
# Clear build cache
rm -rf .next

# Rebuild
npm run build
```

### Django Issues

```bash
# Check migrations
python manage.py showmigrations

# Check settings
python manage.py shell
>>> from django.conf import settings
>>> print(settings.DEBUG)
```

## Documentation Links

- [Next.js Documentation](https://nextjs.org/docs)
- [React Documentation](https://react.dev)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Django Documentation](https://docs.djangoproject.com/)
