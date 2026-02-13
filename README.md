# Advanced Harness

Claude Code를 위한 고급 스킬/커맨드 모음 모노레포. FastAPI 백엔드 + Next.js 15 프론트엔드 프로젝트에 Claude Code 자동화 스킬, MCP 서버 연동, 프레임워크 가이드라인을 통합하여 개발 생산성을 극대화합니다.

## 프로젝트 구조

```
advanced-harness/
├── backend/          # Python FastAPI 백엔드 (PostgreSQL, SQLModel)
├── frontend/         # Next.js 15 프론트엔드 (React 19, TypeScript, Tailwind CSS 4)
├── ebook/            # 전자책 리소스
├── .claude/          # Claude Code 설정, 스킬, 커맨드, 훅
│   ├── skills/       # 프로젝트 스킬 (17개)
│   └── commands/     # 슬래시 커맨드 (3개)
├── .agents/skills/   # 에이전트 스킬 (4개)
├── CLAUDE.md         # Claude Code 프로젝트 가이드
└── .mcp.json         # MCP 서버 설정
```

## 기술 스택

| 영역 | 기술 |
|------|------|
| **Backend** | Python 3.12, FastAPI, SQLModel, PostgreSQL, asyncpg |
| **Frontend** | Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS 4, MUI |
| **AI Agent** | Pydantic AI, SSE streaming |
| **Infra** | AWS ECS, ECR, S3, Docker |
| **CI/CD** | GitHub Actions |

## Skills 목록

### Agent Skills (`.agents/skills/`)

| 스킬 | 설명 |
|------|------|
| `programmatic-seo` | 템플릿+데이터로 SEO 최적화 페이지를 대량 생성하는 12가지 플레이북 가이드 |
| `remotion-best-practices` | React 기반 영상 제작 프레임워크 Remotion의 애니메이션/오디오/자막 등 27개 도메인 룰 |
| `vercel-react-best-practices` | Vercel 엔지니어링 기준 React/Next.js 성능 최적화 57개 룰 |
| `web-design-guidelines` | Vercel Web Interface Guidelines 기반 UI 코드 접근성/UX 리뷰 |

### Project Skills (`.claude/skills/`)

| 스킬 | 설명 |
|------|------|
| `brand-guidelines` | Anthropic 공식 브랜드 컬러와 타이포그래피(Poppins/Lora) 적용 |
| `copyright-safe-style-transfer` | Replicate FLUX Pro 모델로 이미지 스타일을 저작권 안전하게 변환 |
| `docx` | Word 문서(.docx) 생성/편집/분석 - docx-js 생성, XML 직접 편집 |
| `error-tracking` | Sentry v8 에러 트래킹 및 성능 모니터링 통합 패턴 가이드 |
| `evan-insight-blog-writer` | evan-insight 블로그용 투자 분석 글 작성 - 어그로 두괄식, 쉬운 용어 |
| `frontend-design` | 독창적이고 프로덕션급 프론트엔드 UI 생성 디자인 가이드 |
| `internal-comms` | 사내 커뮤니케이션(3P 업데이트, 뉴스레터, FAQ 등) 작성 가이드 |
| `mcp-builder` | MCP(Model Context Protocol) 서버 Python/TypeScript 구축 가이드 |
| `mermaid` | Mermaid 다이어그램 생성 - 플로우차트, ER, 간트 등 23종 지원 |
| `pdf` | PDF 읽기/병합/분할/회전/워터마크/생성/OCR 등 전방위 처리 |
| `ppt-brand-guidelines` | VRL 프레젠테이션 브랜드 가이드라인 (라임그린 + 다크네이비) |
| `pptx` | PowerPoint 생성/편집/분석 - HTML→PPTX 변환, OOXML 편집 |
| `pytest-backend-testing` | FastAPI 백엔드 pytest 테스팅 가이드 - 유닛/통합/비동기/목킹 |
| `skill-developer` | Claude Code 스킬 생성/관리 가이드 - 트리거, 훅, 500줄 룰 |
| `threads-writer` | Threads SNS 글쓰기 - 11가지 템플릿으로 자동 작성 |
| `xlsx` | 엑셀 스프레드시트 생성/편집/분석 - openpyxl/pandas, 수식 검증 |
| `yebi-startup-writer` | 예비창업패키지 사업계획서 PSST 프레임워크 기반 docx 직접 작성 |

### Commands (`.claude/commands/`)

| 커맨드 | 설명 |
|--------|------|
| `dev-docs-update` | 컨텍스트 컴팩션 전 개발 문서(active task, 세션 상태) 업데이트 |
| `dev-docs` | 전략적 개발 계획 생성 - `dev/active/`에 plan/context/tasks 구조화 |
| `route-research-for-testing` | 편집된 라우트 자동 감지 후 auth-route-tester로 스모크 테스트 |

## 빠른 시작

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

## MCP 서버 연동

`.mcp.json`에 설정된 MCP 서버:

- **Context7** - 라이브러리 공식 문서 조회
- **Magic (21st.dev)** - 모던 UI 컴포넌트 생성
- **Sequential Thinking** - 구조화된 다단계 추론
- **Playwright** - 브라우저 자동화/E2E 테스트
- **Replicate** - AI 모델 실행 (이미지 생성/변환)
- **Pencil** - .pen 파일 디자인 편집

## License

Private repository.
