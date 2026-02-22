# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

This is a **monorepo** containing both backend (FastAPI) and frontend (Next.js) applications:

- `backend/` - Python FastAPI backend with PostgreSQL
- `frontend/` - Next.js 15 frontend with TypeScript and Tailwind CSS

## Backend Development

### Prerequisites

- Python 3.12.3 (exact version, see `backend/pyproject.toml`)
- Docker & Docker Compose

### Setup

```bash
cd backend
uv venv
source .venv/bin/activate
uv sync
uv sync --group dev  # Install dev dependencies (black, isort, mypy, ruff, pytest)
```

### Running the Backend

```bash
# Development server (note: module is backend.main not app.main)
cd backend
uvicorn backend.main:app --reload --port 28080

# Docker Compose (test DB only)
cd backend
docker-compose up
```

### Code Quality

```bash
cd backend
black .                           # Format code
isort . --profile black          # Sort imports
ruff check --fix .               # Lint with auto-fix
mypy .                           # Type checking
pre-commit run --all-files       # Run all pre-commit hooks
```

### Testing

```bash
cd backend
pytest                                      # Run all tests (requires test DB)
pytest --cov=backend --cov-report=html      # With coverage report
pytest tests/unit/domain/user/ -v          # Specific domain tests
```

Test DB is a local PostgreSQL container (port 5433) defined in `docker-compose.yaml`. Start it with `docker-compose up test-db` before running tests. Init scripts: `scripts/init_test_db.py`, `scripts/reset_test_db.py`.

### Backend Architecture

**Framework:** FastAPI with async/await pattern using SQLModel + SQLAlchemy

**Database Layer:**

- **Read/Write Separation**: Separate database connections for read and write operations
- Event loop-based caching of engines and sessionmakers in `backend/db/orm.py`
- Session factories:
  - `get_write_session()` / `get_write_session_dependency()` for write operations
  - `get_read_session()` / `get_read_session_dependency()` for read operations
- PostgreSQL with asyncpg driver; SSL required in staging/production, optional in development

**Domain-Driven Design:**

- Business logic organized in `backend/domain/{entity}/` directories
- Current domains:
  - `user` - User model, service, repository, auth_service, enums
  - `shared` - Base repository, query helpers, raw query repository

**API Structure:**

- Versioned endpoints under `/api/v1/` prefix
- Routers in `backend/api/v1/routers/`: `auth.py`, `user.py`
- DTOs in `backend/dtos/`: `auth.py`, `user.py`
- Main app creation in `backend/main.py` via `create_application()`
- Health check: `GET /api/v1/health`

**Utilities:**

- `backend/utils/logger.py` - Logging utilities
- `backend/utils/password.py` - Password hashing (bcrypt)
- `backend/error/` - Custom error types
- `backend/middleware/error_handler.py` - Global exception handler
- `backend/scripts/verify_trigram_support.py` - DB trigram extension check

**Configuration:**

- Settings in `backend/core/config.py` using Pydantic BaseSettings
- Environment variables required: database credentials (read/write), JWT config
- `MOCK_AUTH_ENABLED=true` enables mock auth for development/testing (blocked in production)
- CORS: localhost origins in development; configure production domains in `config.py`

**Docker:**

- `Dockerfile` - Production image using `python:3.12.3-slim` + uv, entrypoint: `scripts/entrypoint.sh`
- `docker-compose.yaml` - Local test DB only (`pgvector/pgvector:pg16` on port 5433)

## Frontend Development

### Prerequisites

- Node.js 20+
- pnpm package manager

### Setup

```bash
cd frontend
pnpm install
```

### Running the Frontend

```bash
cd frontend
pnpm dev          # Development server with Turbopack (http://localhost:3000)
pnpm build        # Production build
pnpm start        # Start production server
pnpm lint         # Run ESLint
```

### E2E Testing (Playwright)

```bash
cd frontend
pnpm test:e2e          # Run E2E tests
pnpm test:e2e:ui       # Playwright UI mode
pnpm test:e2e:debug    # Debug mode
pnpm test:e2e:report   # Show last test report
```

### Frontend Architecture

**Framework:** Next.js 15 (App Router) with React 19, TypeScript, Tailwind CSS 4

**Project Structure:**

- `src/app/` - Next.js App Router pages and API routes
  - `/` - Home page
  - `/login` - Login page
  - `/api/auth/session` - Session management API route
  - `error.tsx`, `loading.tsx` - Global error/loading states
- `src/components/` - Reusable React components
  - `layout/` - `Navbar.tsx`, `Footer.tsx`
  - `ui/` - shadcn/ui components (alert, badge, button, card, checkbox, dialog, input, label, select, skeleton, textarea)
- `src/lib/` - Core utilities
  - `api.ts` - API client with JWT token management (localStorage)
  - `utils.ts` - General utilities (cn helper, etc.)
- `src/providers/` - React context providers
  - `AuthProvider.tsx` - Authentication context (email login/signup, Firebase)
- `src/types/index.ts` - TypeScript type definitions
- `src/constants/enums.ts` - Application enums
- `src/fonts/` - Local font files
- `src/middleware.ts` - Route protection middleware

**Key Technologies:**

- **Styling:** Tailwind CSS 4, shadcn/ui (Radix UI primitives)
- **State Management:** React hooks and context providers
- **Authentication:** JWT tokens stored in localStorage + Firebase (Google auth)
- **UI Components:** shadcn/ui, Radix UI, Lucide React icons, Recharts
- **E2E Testing:** Playwright

**Configuration:**

- `next.config.ts` - Next.js config, Turbopack enabled
- `postcss.config.mjs` - PostCSS with Tailwind CSS 4
- `playwright.config.ts` - Playwright E2E test configuration
- `components.json` - shadcn/ui configuration
- Environment variables required: `NEXT_PUBLIC_API_URL_DEV`, `NEXT_PUBLIC_API_URL_PROD`, `NEXT_PUBLIC_ENV`

## Development Workflow

### Environment Files

- `backend/.env` - Database credentials (read/write), JWT config, `MOCK_AUTH_ENABLED`, `ENVIRONMENT`
- `frontend/.env` - `NEXT_PUBLIC_API_URL_DEV`, `NEXT_PUBLIC_API_URL_PROD`, `NEXT_PUBLIC_ENV`

### Git Workflow

- Main branch: `main`
- Create feature branches for development

### Key Design Patterns

**Backend:**

- Domain-Driven Design with clear separation of concerns
- Repository pattern for data access
- DTO pattern for API contracts
- Dependency injection via FastAPI's `Depends()`
- Async/await throughout with proper session management

**Frontend:**

- App Router with Client/Server component separation
- Context-based auth state via `AuthProvider`
- API calls centralized in `src/lib/api.ts` with automatic token management

### Important Notes

- **Backend module path:** Use `backend.main:app` not `app.main:app` when running uvicorn
- **Database sessions:** Always use appropriate read/write session factory from `backend/db/orm.py`
  - Use `get_write_session_dependency()` for endpoints that modify data
  - Use `get_read_session_dependency()` for read-only endpoints
- **Frontend API calls:** Centralized in `src/lib/api.ts`; tokens stored in localStorage with keys `app_access_token` / `app_refresh_token`
- **Authentication flow:** JWT-based (email/password) + Firebase for social login; `AuthProvider` manages session state
- **Mock auth:** Set `MOCK_AUTH_ENABLED=true` in `backend/.env` for development without real credentials (disabled in production)
- **Pre-commit hooks:** Backend uses black, isort, ruff, and other checks via `.pre-commit-config.yaml`
- **Adding new domains:** Create `backend/domain/{name}/` with `model.py`, `service.py`, `repository.py`; register router in `backend/main.py`
- **Adding new UI components:** Use `pnpm dlx shadcn@latest add <component>` to add shadcn/ui components
