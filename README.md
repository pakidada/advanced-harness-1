# Advanced Harness

A monorepo of advanced skills/commands for Claude Code. Integrates Claude Code automation skills and framework guidelines into a FastAPI backend + Next.js 15 frontend project to maximize development productivity.

## Tech Stack

| Area | Technology |
|------|-----------|
| **Backend** | Python 3.12, FastAPI, SQLModel, PostgreSQL, asyncpg |
| **Frontend** | Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS 4, MUI |
| **AI Agent** | Pydantic AI, SSE streaming |

## Skills (15)

| # | Skill | Description |
|---|-------|-------------|
| 1 | `brand-guidelines` | Apply Anthropic official brand colors and typography (Poppins/Lora) |
| 2 | `docx` | Word document (.docx) creation/editing/analysis - docx-js generation, direct XML editing |
| 3 | `error-tracking` | Sentry v8 error tracking and performance monitoring integration pattern guide |
| 4 | `fastapi-backend-guidelines` | FastAPI backend DDD development guide - SQLModel ORM, repository pattern, async/await |
| 5 | `frontend-design` | Design guide for creating distinctive, production-grade frontend UIs |
| 6 | `mermaid` | Mermaid diagram generation - supports 23 types including flowcharts, ER diagrams, Gantt charts |
| 7 | `nextjs-frontend-guidelines` | Next.js 15 frontend guide - App Router, shadcn/ui, Tailwind CSS 4, Korean localization |
| 8 | `pdf` | Comprehensive PDF processing: read/merge/split/rotate/watermark/create/OCR |
| 9 | `ppt-brand-guidelines` | VRL presentation brand guidelines (lime green + dark navy) |
| 10 | `pptx` | PowerPoint creation/editing/analysis - HTML-to-PPTX conversion, OOXML editing |
| 11 | `pytest-backend-testing` | FastAPI backend pytest testing guide - unit/integration/async/mocking |
| 12 | `skill-developer` | Claude Code skill creation/management guide - triggers, hooks, 500-line rule |
| 13 | `vercel-react-best-practices` | React/Next.js performance optimization guide based on Vercel Engineering standards |
| 14 | `web-design-guidelines` | UI code accessibility/UX review based on Vercel Web Interface Guidelines |
| 15 | `yebi-startup-writer` | Pre-startup package business plan writing based on PSST framework, direct docx output |

## Commands (3)

| Command | Description |
|---------|-------------|
| `/dev-docs-update` | Update dev docs (active tasks, session state) before context compaction |
| `/dev-docs` | Generate strategic development plans - structured plan/context/tasks in `dev/active/` |
| `/route-research-for-testing` | Auto-detect edited routes and run smoke tests via auth-route-tester |

## Agents (12)

Specialized agents that autonomously handle complex subtasks.

| # | Agent | Description |
|---|-------|-------------|
| 1 | `auth-route-debugger` | Authentication issue debugging specialist (401/403 errors, cookies, JWT, Keycloak) |
| 2 | `auth-route-tester` | Authenticated API route testing - DB change verification and code review |
| 3 | `auto-error-resolver` | Automatic TypeScript compilation error detection and resolution |
| 4 | `code-architecture-reviewer` | Code quality, architectural consistency, and system integration review |
| 5 | `code-refactor-master` | Comprehensive refactoring: file restructuring, dependency tracking, component extraction |
| 6 | `documentation-architect` | Developer docs, API docs, and data flow diagram generation |
| 7 | `frontend-error-fixer` | Build-time/runtime frontend error diagnosis and resolution (using browser-tools MCP) |
| 8 | `plan-reviewer` | Pre-implementation development plan review - risk assessment, gap analysis |
| 9 | `planner` | Generate structured development plans (plan/context/tasks) in `dev/active/` |
| 10 | `refactor-planner` | Code structure analysis and step-by-step refactoring strategy planning |
| 11 | `threads-aggro-writer` | Viral Threads posts on developer/startup topics (Korean) |
| 12 | `web-research-specialist` | Technical issue research across GitHub Issues, Reddit, Stack Overflow, etc. |

## Quick Start

### Backend

```bash
cd backend
uv venv && source .venv/bin/activate
uv pip install -e .[dev]
uvicorn backend.main:app --reload --port 28080
```

### Frontend

```bash
cd frontend
pnpm install
pnpm dev  # http://localhost:3000
```
