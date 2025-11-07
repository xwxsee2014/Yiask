# Implementation Plan: Yiask AI智能课程练习平台

**Branch**: `001-ai-curriculum-learning` | **Date**: 2025-11-04 | **Spec**: [link to spec.md]
**Input**: 基于AI的课程同步学习平台，集成内容生成、语音朗读和图文结合问题功能

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

**Primary Requirement**: Build an AI-powered English learning platform that synchronizes with formal course materials, generates practice exercises, provides voice synthesis for questions, supports voice input for answers with real-time evaluation, and offers image-based contextual exercises.

**Technical Approach**: Multi-container Docker deployment with separate FastAPI backend, React frontend, and PostgreSQL database. AI services (Dify, Xinference) deployed independently. Uses MinerU for PDF processing. FastAPI background tasks for content generation workflow.

## Technical Context

**Language/Version**: Python 3.11 (FastAPI backend), JavaScript/TypeScript (React frontend)
**Package Management**: uv for virtual environment isolation and package dependency management
**Dependency Format**: pyproject.toml for all Python dependencies (NOT requirements.txt)
**Primary Dependencies**: FastAPI, React, PostgreSQL, Docker Compose, MinerU, Dify (LLMOps), Xinference (TTS/ASR)
**Storage**: PostgreSQL with named volumes for persistence + Alembic for database versioning
**Testing**: pytest (backend), Jest/React Testing Library (frontend)
**Target Platform**: Linux server with Docker Compose deployment
**Project Type**: Web application (separate frontend/backend services)
**Performance Goals**: API response <500ms p95, AI generation first token <1min, support 100 concurrent users
**Constraints**: Multi-container architecture, <100 concurrent users, simple review workflow, no task queue system needed
**Scale/Scope**: Small-scale platform serving <100 concurrent users

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. User Story Driven Development ✓ PASS
- **Status**: COMPLIANT - Feature has 3 P1, 2 P2, and 1 P3 user stories with clear priorities
- **Evidence**:
  - P1: User Story 1 - Course-synchronized content generation (核心价值主张)
  - P2: User Story 2, 2.1 - AI voice features (TTS and speech input)
  - P3: User Story 3 - Image-text combined exercises
- **Justification**: Each story delivers independent value and has explicit acceptance scenarios

### II. Test-First (NON-NEGOTIABLE) ✓ PASS
- **Status**: COMPLIANT - TDD cycle mandated for all stories
- **Evidence**: Implementation order enforced: Models → Services → Endpoints → Integration
- **Action**: Red-Green-Refactor cycle MUST be strictly enforced during implementation
- **Justification**: Test-first ensures requirements clarity and prevents regressions

### III. Independent Testability ✓ PASS
- **Status**: COMPLIANT - Each story includes independent test scenarios
- **Evidence**:
  - Story 1:独立测试 clearly defined in spec
  - Story 2:独立测试 clearly defined in spec
  - Story 2.1:独立测试 clearly defined in spec
  - Story 3:独立测试 clearly defined in spec
- **Justification**: Stories can be demonstrated and tested in isolation

### IV. Phased Implementation ✓ PASS
- **Status**: COMPLIANT - Ordered phases defined
- **Evidence**: Must follow Setup → Foundational → User Stories → Polish
- **Within Stories**: Models → Services → Endpoints → Integration
- **Justification**: Phased approach ensures proper foundation and architectural consistency

### V. Simplicity & Constitution Compliance ✓ PASS
- **Status**: COMPLIANT - Architecture favors simplicity
- **Evidence**:
  - Docker Compose (not Kubernetes) for orchestration
  - FastAPI background tasks (not Celery/RabbitMQ)
  - Simple review workflow
  - <100 user scale (not over-engineered)
- **Justification**: All decisions align with simplicity principle; no complexity violations


## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: Option 2: Web application (backend + frontend services)

Selected structure uses separate backend and frontend directories to support the multi-container Docker deployment with FastAPI backend and React frontend. This enables independent deployment and scaling of services.

### Repository Root Structure

```text
backend/
├── src/
│   ├── models/          # Pydantic/SQLAlchemy models
│   ├── services/        # Business logic services
│   │   ├── ai_service.py      # Dify integration
│   │   ├── speech_service.py  # Xinference integration
│   │   ├── pdf_service.py     # MinerU integration
│   │   └── generation_service.py # Content generation workflow
│   ├── api/             # FastAPI route handlers
│   ├── core/            # Core configurations (auth, db, etc.)
│   └── main.py          # FastAPI application entrypoint
├── alembic/             # Alembic database migration scripts
│   ├── versions/        # Migration files
│   ├── env.py           # Alembic environment configuration
│   └── script.py.mako   # Migration script template
├── alembic.ini          # Alembic configuration file (backend/src/alembic.ini)
├── tests/               # pytest tests
├── Dockerfile           # Backend container definition
├── pyproject.toml       # Python dependencies (uv package management)
└── uv.lock              # Dependency lock file (uv generated)

frontend/
├── src/
│   ├── components/      # React components
│   ├── pages/           # Page components
│   ├── services/        # API client services
│   ├── hooks/           # Custom React hooks
│   ├── types/           # TypeScript type definitions
│   └── App.tsx          # Main application component
├── public/              # Static assets
├── Dockerfile           # Frontend container definition
├── package.json         # Node.js dependencies
└── vite.config.ts       # Vite build configuration

docker-compose.yml       # Multi-container orchestration
.env                     # Root environment variables
.env.backend             # Backend-specific environment variables
│   ├── ENABLE_MIGRATION # Enable database migration trigger (true/false)
│   └── DATABASE_URL     # Database connection URL
.env.frontend            # Frontend-specific environment variables

postgres/
├── data/                # PostgreSQL data persistence (named volume)
└── Dockerfile           # PostgreSQL configuration (custom if needed)
```

## Complexity Tracking

**No violations** - All architecture decisions align with simplicity principle.

---

## Phase 0: Research ✓ COMPLETED

**Output**: `research.md` (generated)

All technical decisions clarified through specification and clarification sessions:
- ✓ Python package management: uv for virtual environment isolation and dependency management
- ✓ Dependency specification: pyproject.toml for all Python dependencies (NOT requirements.txt)
- ✓ Multi-container orchestration: Docker Compose
- ✓ AI services: Dify as LLMOps platform layer
- ✓ Task management: FastAPI background tasks
- ✓ PDF processing: MinerU component
- ✓ Configuration: .env files
- ✓ Database: PostgreSQL with named volumes
- ✓ Database versioning: Alembic (backend/src/alembic.ini)
- ✓ Migration trigger: ENABLE_MIGRATION parameter in .env.backend
- ✓ Initial database: Full Alembic control (no init scripts)
- ✓ Connection config: DATABASE_URL environment variable
- ✓ Failure handling: Auto-rollback + error logging

All decisions comply with Constitution principles.

---

## Phase 1: Design & Contracts ✓ COMPLETED

**Outputs Generated**:
- ✓ `data-model.md` - Complete entity model with relationships, validation rules, and indexes
- ✓ `contracts/api-spec.yaml` - OpenAPI 3.0 specification with all endpoints
- ✓ `quickstart.md` - Comprehensive setup and usage guide
- ✓ Agent context updated via `update-agent-context.sh claude`

**Design Highlights**:
- 11 core entities with clear relationships
- RESTful API with 40+ endpoints (complete REST implementation)
- Complete quickstart guide for 5-minute setup
- Docker Compose multi-service architecture
- Simplicity-focused: REST-only approach (no GraphQL complexity)

---

## Post-Phase 1 Constitution Re-Check

**Re-evaluation after design completion:**

### I. User Story Driven Development ✓ PASS
- **Status**: COMPLIANT - Design fully supports independent story delivery
- **Evidence**: Data model supports all P1/P2/P3 stories with clear entity boundaries
- **Updated**: Entity relationships enable parallel story implementation

### II. Test-First (NON-NEGOTIABLE) ✓ PASS
- **Status**: COMPLIANT - Architecture enables contract testing
- **Evidence**: API contracts (OpenAPI/GraphQL) provide executable specifications
- **Updated**: Contract tests can be generated from API specifications

### III. Independent Testability ✓ PASS
- **Status**: COMPLIANT - Service boundaries clearly defined
- **Evidence**: Microservice architecture with clear interfaces
- **Updated**: Each service can be tested independently via API contracts

### IV. Phased Implementation ✓ PASS
- **Status**: COMPLIANT - Phases clearly defined in quickstart
- **Evidence**: Setup → Foundational → User Stories → Polish sequence documented
- **Updated**: Implementation order: Models → Services → Endpoints → Integration

### V. Simplicity & Constitution Compliance ✓ PASS
- **Status**: COMPLIANT - No complexity violations
- **Evidence**:
  - Docker Compose (not Kubernetes) ✓
  - FastAPI background tasks (not Celery/RabbitMQ) ✓
  - Simple review workflow ✓
  - REST-only API design (removed GraphQL to avoid unnecessary complexity) ✓
- **Updated**: All Phase 1 deliverables maintain simplicity principle

**Overall Gate Status**: ✅ **ALL GATES PASS** - Proceed to `/speckit.tasks`

---

## API Design Decision: REST-Only ✓

**Decision Made**: Removed GraphQL schema in favor of REST-only approach

**Rationale**:
- **Simplicity Principle**: Following Constitution V - avoids unnecessary complexity
- **Project Scale**: <100 users doesn't require GraphQL's advanced features
- **Tooling**: FastAPI auto-generates excellent REST API documentation
- **File Uploads**: REST multipart/form-data better suited for PDF/image uploads
- **Implementation**: Faster to develop and test with single API pattern

**Current API Design**:
- OpenAPI 3.0 specification with 40+ endpoints
- RESTful design following standard conventions
- Auto-generated documentation at `/docs`
- All endpoints defined in `contracts/api-spec.yaml`

**Package Management Approach**:
- Use `uv` for creating and managing Python virtual environments
- All Python dependencies specified in `pyproject.toml` (NO requirements.txt)
- Dependency lock file `uv.lock` ensures reproducible builds
- uv provides faster dependency resolution compared to pip

Future enhancement: GraphQL can be added later if complex dashboard queries become a bottleneck.

---

## Deliverables Summary

All planning artifacts have been successfully generated and are ready for implementation:

### Documentation Files
1. ✅ **spec.md** - Feature specification with 37 clarified requirements
2. ✅ **plan.md** - This implementation plan (current file)
3. ✅ **research.md** - Technical research and decision consolidation
4. ✅ **data-model.md** - Complete data model with 11 entities
5. ✅ **quickstart.md** - Setup and usage guide

### API Contracts
6. ✅ **contracts/api-spec.yaml** - OpenAPI 3.0 specification (40+ endpoints, REST-only)

### Agent Context
8. ✅ **CLAUDE.md** - Updated with Python/FastAPI, React, PostgreSQL, Docker, Dify, Xinference, MinerU

### Ready for Next Phase
The `/speckit.tasks` command can now be executed to generate the task breakdown and begin implementation following the planned architecture.

---

**Command**: `/speckit.plan` completed successfully
**Status**: ✅ All phases complete, ready for `/speckit.tasks`
**Next Action**: Run `/speckit.tasks` to generate implementation tasks

