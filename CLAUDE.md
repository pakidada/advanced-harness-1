# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **harness/scaffold monorepo** (codename: yeongyeolsa/영영사) with a FastAPI backend and Next.js 15 frontend. It is a stripped-down template — only `user` and `auth` domains are implemented. Other domains mentioned in README (artist, artwork, curai, etc.) do not exist here.

## Quick Reference

### Backend Commands

```bash
cd backend
uv venv && source .venv/bin/activate
uv pip install -e .                    # Production deps
uv pip install -e .[dev]               # + black, isort, mypy, ruff, pytest, alembic

# Run dev server (module path is backend.main, NOT app.main)
uvicorn backend.main:app --reload --port 28080

# Code quality
black .
isort . --profile black
ruff check --fix .
mypy .
pre-commit run --all-files

# Tests (requires test-db running)
docker-compose up -d                   # Starts pgvector on port 5433
python scripts/init_test_db.py --seed  # Init + seed test data
pytest                                 # Runs with --cov-fail-under=80
pytest tests/path/to/test.py -k "test_name"  # Single test

# Database
alembic upgrade head                   # Run migrations
alembic revision --autogenerate -m "description"  # Create migration
```

### Frontend Commands

```bash
cd frontend
pnpm install
pnpm dev          # Dev server with Turbopack (http://localhost:3000)
pnpm build        # Production build
pnpm lint         # ESLint
```

## Backend Architecture

**Stack:** FastAPI + SQLModel/SQLAlchemy + asyncpg + PostgreSQL (with pgvector)

### Database Layer (`backend/db/orm.py`)

Read/write connection separation with event-loop-scoped engine caching:
- `get_write_session()` / `get_read_session()` — async context managers for service-layer use
- `get_write_session_dependency()` / `get_read_session_dependency()` — FastAPI `Depends()` generators for routers
- Pool: `pool_size=15`, `max_overflow=25`. SSL required in non-dev environments.
- Test DB runs on port **5433** (not 5432) via docker-compose.

### Domain-Driven Design (`backend/domain/`)

Each domain follows: `model.py` → `repository.py` → `service.py`

Currently implemented:
- **`user/`** — User, UserProfile, UserLifestyle, UserPreference, UserDocument, UserPhoto, UserSubscription, UserAccessAudit
- **`user/auth_service.py`** — Email/password auth (bcrypt), JWT access (12h) + refresh (30d) tokens
- **`shared/`** — `BaseRepository[ModelType]` with generic CRUD, `QueryFilterBuilder`, `LocaleFieldSelector`

Key design decisions:
- **No FK constraints at DB level** — referential integrity enforced at application layer only
- **ULID-based string IDs** with type prefixes: `usr_`, `doc_`, `pho_`, `sub_`, `aud_`
- **Soft delete** via `deleted_at` on User, UserDocument, UserPhoto
- **Bcrypt hash stored in `auth_provider_id` field** — this field serves triple duty (OAuth provider ID for social login, bcrypt hash for email login, or None)
- **All enums stored as Text** (not native PG enum), with `from_korean()` / `to_korean()` methods for locale mapping
- **`UserDataLoader`** fetches user relations (profile, lifestyle, preference, subscription) in parallel via `asyncio.gather()`

### API Routes (`backend/api/v1/routers/`)

Only two routers exist:
- **`auth.py`** — `POST /email/sign-up`, `POST /email/login`, `POST /refresh`, `GET /me`
- **`user.py`** — CRUD at `/users` (list, get, create, update, soft-delete)

Prefix: `/api/v1`. Health check: `GET /api/v1/health`.

### Error Handling (`backend/error/`, `backend/middleware/error_handler.py`)

Custom exception hierarchy: `AppException` → `NotFoundError`, `ForbiddenError`, `UnauthorizedError`, `ValidationError`, `ConflictError`, `UserNotFoundSignupRequiredError` (returns HTTP **452**, non-standard).

### Mock Auth (`backend/core/config.py`)

Setting `MOCK_AUTH_ENABLED=true` bypasses all JWT validation — every request is treated as `mock-user-001`. Blocked in production (enforced at startup). Useful for frontend development without real auth.

### Known Issues

- `UserService.list_users` accepts `query`, `status`, `gender` params but silently ignores them — `list_async` only applies `skip`/`limit`
- `refresh_token` table referenced in `scripts/reset_test_db.py` but no SQLModel model exists
- No `alembic/` migrations directory committed yet — needs `alembic init alembic`
- No `tests/` directory exists yet — pytest is configured but tests need to be written

## Frontend Architecture

**Stack:** Next.js 15 (App Router) + React 19 + TypeScript + Tailwind CSS 4

### Route Structure

Minimal scaffold: home page (`/`) + login page (`/login`). The `middleware.ts` has `PROTECTED_PATHS = []` (nothing protected yet).

### Auth Flow (`src/lib/api.ts`, `src/providers/AuthProvider.tsx`)

**Dual token storage:**
- `localStorage` — for client-side auth checks
- HTTP-only cookies — set via internal Next.js route `/api/auth/session` (POST sets, DELETE clears, GET reads)

The `apiRequest<T>()` function auto-retries with token refresh on 401. Cross-tab sync via `storage` event listener.

### Key Files

- `src/lib/api.ts` — API client, all backend calls go through here. Base URL switches on `NEXT_PUBLIC_ENV`
- `src/providers/AuthProvider.tsx` — Auth context: `user`, `isAuthenticated`, `emailLogin`, `emailSignUp`, `logout`, `refreshUser`
- `src/app/api/auth/session/route.ts` — Cookie management (access 12h, refresh 30d, `httpOnly`, `secure` in prod)
- `next.config.ts` — Remote images from `lh3.googleusercontent.com` and `prod-apne2-ygs.s3.amazonaws.com`

### Path Alias

`@/*` maps to `./src/*` (tsconfig paths).

## Infrastructure

### Ports

| Service | Port |
|---------|------|
| Backend dev | 28080 |
| Backend prod (gunicorn) | 8080 |
| Frontend | 3000 |
| Test PostgreSQL | 5433 |

### Docker

- `backend/Dockerfile` — Python 3.12.3-slim + uv, entrypoint at `scripts/entrypoint.sh`
- `backend/docker-compose.yaml` — Only starts test-db (`pgvector/pgvector:pg16` on port 5433), no backend service
- `backend/scripts/entrypoint.sh` — Production: gunicorn (8 workers), Dev: uvicorn with reload

### Pre-commit Hooks

Backend only: black, isort (profile=black), ruff (--fix), trailing-whitespace, end-of-file-fixer, check-yaml, check-merge-conflict, debug-statements.

### Python (`backend/`)

- `requires-python = "==3.12.3"` (exact version)
- mypy: strict mode with `pydantic.mypy` + `sqlalchemy.ext.mypy.plugin`
- pytest: `asyncio_mode = auto`, coverage minimum 80%, excludes routers/auth/curai/admin from coverage
