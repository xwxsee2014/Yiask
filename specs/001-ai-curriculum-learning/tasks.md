# Tasks: Yiask AI智能课程练习平台

**Input**: Design documents from `/specs/001-ai-curriculum-learning/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Feature Name**: Yiask AI智能课程练习平台 (AI-powered English learning platform)
**Tech Stack**: FastAPI backend + React frontend + PostgreSQL + Docker Compose + Alembic
**AI Services**: Dify (text generation), Xinference (TTS/ASR), MinerU (PDF processing)
**Database Versioning**: Alembic (backend/src/alembic.ini)
**Migration Management**: Auto-migration via ENABLE_MIGRATION parameter in .env.backend
**Development Modes**: Local development (recommended) or Docker deployment (quick start)

**Tests**: This implementation requires TDD approach - tests are mandatory per Constitution II

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

Based on plan.md, this is a **web application (Option 2)**: `backend/` and `frontend/` structure

## Development Modes

**⚠️ IMPORTANT**: Choose the right mode for your use case:

### Option 1: Local Development (Required for Backend Development)
- **Use for**: Backend development, database schema changes, migration generation
- Run backend and frontend locally for easier debugging and hot reload
- Use `uvicorn src.main:app --reload` for backend
- Use `npm run dev` for frontend
- Manual migration management: `alembic upgrade head`
- **Why Local**: Only in local mode can you generate and persist migration files

### Option 2: Docker Deployment (For Frontend Debugging or QA Testing Only)
- **Use for**: Frontend local debugging, QA testing, production-like environment testing
- Use Docker Compose for fast deployment with all services containerized
- Migrations are automatically applied when `ENABLE_MIGRATION=true` in .env.backend
- Run `docker-compose up -d` to start all services
- **Why Docker**: Migrations are read-only in Docker mode, cannot generate new migration files
- **Note**: Backend code changes require rebuild (`docker-compose up -d --build backend`)

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per plan.md in backend/ and frontend/ directories
- [ ] T002 Initialize FastAPI backend with Python 3.11 dependencies in backend/requirements.txt
- [ ] T003 Initialize React frontend with TypeScript in frontend/package.json
- [ ] T004 [P] Configure Docker Compose multi-service setup in docker-compose.yml
- [ ] T005 [P] Create environment configuration templates (.env, .env.backend, .env.frontend)
- [ ] T006 [P] Configure PostgreSQL with named volumes in docker-compose.yml
- [ ] T007 Setup linting and formatting tools (ESLint, Prettier, Black, isort)
- [ ] T008 [P] Configure Alembic database versioning in backend/src/alembic.ini

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T009 Setup database schema and SQLAlchemy ORM integration in backend/src/models/
- [ ] T010 [P] Implement JWT authentication framework in backend/src/core/auth.py
- [ ] T011 [P] Setup FastAPI routing structure and middleware in backend/src/api/
- [ ] T012 [P] Create base Pydantic models for API validation in backend/src/schemas/
- [ ] T013 Configure environment configuration management in backend/src/core/config.py
- [ ] T014 Setup error handling and logging infrastructure in backend/src/core/logger.py
- [ ] T015 [P] Create User and UserProfile models in backend/src/models/user.py
- [ ] T016 [P] Implement user registration and login endpoints in backend/src/api/auth.py
- [ ] T017 [P] Setup CORS and security middleware for frontend-backend communication
- [ ] T018 Implement role-based access control (RBAC) in backend/src/core/security.py
- [ ] T019 Setup Alembic migration framework with env.py in backend/alembic/env.py
- [ ] T020 [P] Setup file upload infrastructure for PDF/image/audio files in backend/src/core/storage.py

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - 课程同步内容生成与练习 (Priority: P1) 🎯 MVP

**Goal**: Enable content providers to create course units with materials, trigger AI-generated exercises, review and approve them, and allow learners to practice with generated content

**Independent Test**: Content provider uploads course material → triggers generation → reviews exercises → approves → learner selects course and completes exercises → verify content relevance and quality

### Tests for User Story 1 (MANDATORY - TDD approach) ⚠️

> **NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T021 [P] [US1] Contract test for course management endpoints in backend/tests/contract/test_course_management.py
- [ ] T022 [P] [US1] Contract test for exercise generation workflow in backend/tests/contract/test_exercise_generation.py
- [ ] T023 [P] [US1] Integration test for content provider workflow in backend/tests/integration/test_content_provider_flow.py
- [ ] T024 [P] [US1] Integration test for learner exercise workflow in backend/tests/integration/test_learner_exercise_flow.py

### Models for User Story 1

- [ ] T025 [P] [US1] Create CourseUnit model in backend/src/models/course_unit.py
- [ ] T026 [P] [US1] Create CourseMaterial model in backend/src/models/course_material.py
- [ ] T027 [P] [US1] Create Exercise model in backend/src/models/exercise.py
- [ ] T028 [P] [US1] Create ReviewRecord model in backend/src/models/review.py
- [ ] T029 [P] [US1] Create GenerationTask model in backend/src/models/generation_task.py

### Services for User Story 1

- [ ] T030 [US1] Implement CourseService in backend/src/services/course_service.py (depends on T025, T026)
- [ ] T031 [US1] Implement MinerU integration in backend/src/services/pdf_service.py (depends on T020)
- [ ] T032 [US1] Implement Dify agent integration in backend/src/services/ai_service.py
- [ ] T033 [US1] Implement ExerciseGenerationService in backend/src/services/generation_service.py
- [ ] T034 [US1] Implement ReviewService in backend/src/services/review_service.py (depends on T028)

### API Endpoints for User Story 1

- [ ] T035 [US1] Implement course CRUD endpoints in backend/src/api/courses.py (depends on T030)
- [ ] T036 [US1] Implement material upload endpoints in backend/src/api/courses.py (depends on T031)
- [ ] T037 [US1] Implement exercise generation trigger in backend/src/api/exercises.py (depends on T033)
- [ ] T038 [US1] Implement content review endpoints in backend/src/api/reviews.py (depends on T034)
- [ ] T039 [US1] Implement generation task status tracking in backend/src/api/tasks.py (depends on T029)
- [ ] T040 [US1] Implement exercise retrieval for learners in backend/src/api/exercises.py

### Frontend for User Story 1

- [ ] T041 [P] [US1] Create course management UI in frontend/src/pages/CourseManagement.tsx
- [ ] T042 [P] [US1] Create material upload component in frontend/src/components/CourseUpload.tsx
- [ ] T043 [P] [US1] Create exercise review interface in frontend/src/pages/ReviewExercises.tsx
- [ ] T044 [P] [US1] Create learner exercise interface in frontend/src/pages/Practice.tsx
- [ ] T045 [US1] Create API client services in frontend/src/services/api.ts

### Background Tasks for User Story 1

- [ ] T046 [US1] Implement background exercise generation task (depends on T032, T033)
- [ ] T047 [US1] Implement MinerU PDF processing task (depends on T031)
- [ ] T048 [US1] Setup task progress tracking and status updates

**Checkpoint**: User Story 1 fully functional - content providers can create courses, generate exercises, review them, and learners can practice with approved content

---

## Phase 4: User Story 2 - AI语音朗读功能 (Priority: P2)

**Goal**: Provide AI-powered voice synthesis for exercise questions, recommended answers, and evaluation feedback with clear, natural pronunciation

**Independent Test**: Learner selects exercise → clicks voice play button → verify clear, natural English pronunciation → test on different devices

### Tests for User Story 2 (MANDATORY - TDD approach) ⚠️

- [ ] T049 [P] [US2] Contract test for voice synthesis endpoints in backend/tests/contract/test_voice_synthesis.py
- [ ] T050 [P] [US2] Contract test for audio playback in frontend/tests/contract/test_audio_playback.tsx
- [ ] T051 [P] [US2] Integration test for voice generation workflow in backend/tests/integration/test_voice_workflow.py

### Models for User Story 2

- [ ] T052 [P] [US2] Create VoiceFile model in backend/src/models/voice_file.py

### Services for User Story 2

- [ ] T053 [US2] Implement Xinference TTS integration in backend/src/services/speech_service.py (Xinference CosyVoice2-0.5b)
- [ ] T054 [US2] Implement voice file management service in backend/src/services/voice_service.py (depends on T052)

### API Endpoints for User Story 2

- [ ] T055 [US2] Implement voice synthesis trigger in backend/src/api/exercises.py (depends on T054)
- [ ] T056 [US2] Implement voice file retrieval endpoint in backend/src/api/exercises.py (depends on T054)

### Frontend for User Story 2

- [ ] T057 [P] [US2] Create audio player component in frontend/src/components/AudioPlayer.tsx
- [ ] T058 [P] [US2] Implement voice playback UI in frontend/src/components/VoiceButton.tsx
- [ ] T059 [US2] Integrate voice synthesis with exercise display (depends on T057, T058)

**Checkpoint**: Voice synthesis working - exercises have clear, natural voice朗读 for questions and answers

---

## Phase 5: User Story 2.2 - 语音输入练习功能 (Priority: P2)

**Goal**: Enable learners to answer questions via voice input with real-time evaluation and feedback, including recommended answers

**Independent Test**: Learner sees exercise question → records voice answer → system transcribes → provides quality evaluation → displays recommended answer with voice

### Tests for User Story 2.2 (MANDATORY - TDD approach) ⚠️

- [ ] T060 [P] [US2.2] Contract test for voice input endpoints in backend/tests/contract/test_voice_input.py
- [ ] T061 [P] [US2.2] Contract test for answer evaluation endpoints in backend/tests/contract/test_evaluation.py
- [ ] T062 [P] [US2.2] Integration test for voice-to-evaluation workflow in backend/tests/integration/test_voice_evaluation_flow.py

### Models for User Story 2.2

- [ ] T063 [P] [US2.2] Create VoiceInput model in backend/src/models/voice_input.py
- [ ] T064 [P] [US2.2] Create AnswerEvaluation model in backend/src/models/evaluation.py

### Services for User Story 2.2

- [ ] T065 [US2.2] Implement Xinference ASR integration in backend/src/services/speech_service.py (Whisper model)
- [ ] T066 [US2.2] Implement evaluation service with Dify agent in backend/src/services/evaluation_service.py
- [ ] T067 [US2.2] Implement learning record tracking in backend/src/services/learning_service.py (depends on T063, T064)

### API Endpoints for User Story 2.2

- [ ] T068 [US2.2] Implement voice input submission endpoint in backend/src/api/voice_input.py (depends on T065)
- [ ] T069 [US2.2] Implement answer evaluation endpoint in backend/src/api/evaluation.py (depends on T066)
- [ ] T070 [US2.2] Implement learning record management in backend/src/api/learning_records.py (depends on T067)

### Frontend for User Story 2.2

- [ ] T071 [P] [US2.2] Create voice recording component in frontend/src/components/VoiceRecorder.tsx
- [ ] T072 [P] [US2.2] Create evaluation display component in frontend/src/components/EvaluationResult.tsx
- [ ] T073 [P] [US2.2] Create recommended answer display with voice playback in frontend/src/components/RecommendedAnswer.tsx
- [ ] T074 [US2.2] Integrate voice input with exercise interface (depends on T071, T072, T073)

**Checkpoint**: Voice input functional - learners can speak answers, receive real-time evaluation, and hear recommended responses

---

## Phase 6: User Story 3 - 图文结合场景练习 (Priority: P3)

**Goal**: Support image-based exercises where content providers upload scenario images that learners view while answering questions

**Independent Test**: Content provider creates image-based exercise with scenario image → learner views image → answers question based on visual context

### Tests for User Story 3 (MANDATORY - TDD approach) ⚠️

- [ ] T075 [P] [US3] Contract test for image upload endpoints in backend/tests/contract/test_image_upload.py
- [ ] T076 [P] [US3] Contract test for image retrieval in backend/tests/contract/test_image_retrieval.py
- [ ] T077 [P] [US3] Integration test for image-based exercise workflow in backend/tests/integration/test_image_exercise_flow.py

### Models for User Story 3

- [ ] T078 [P] [US3] Create ExerciseImage model in backend/src/models/exercise_image.py

### Services for User Story 3

- [ ] T079 [US3] Implement image management service in backend/src/services/image_service.py (depends on T078)

### API Endpoints for User Story 3

- [ ] T080 [US3] Implement image upload for exercises in backend/src/api/exercises.py (depends on T079)
- [ ] T081 [US3] Implement image retrieval with exercises in backend/src/api/exercises.py (depends on T079)

### Frontend for User Story 3

- [ ] T082 [P] [US3] Create image upload interface for content providers in frontend/src/components/ImageUpload.tsx
- [ ] T083 [P] [US3] Create image display component for learners in frontend/src/components/ImageDisplay.tsx
- [ ] T084 [US3] Integrate image display with exercise interface (depends on T083)

**Checkpoint**: Image-based exercises working - scenario images display correctly with questions

---

## Phase 7: Learning Progress & Analytics (Supporting Feature)

**Purpose**: Track and display learner progress across all exercises

### Models for Learning Progress

- [ ] T085 [P] Create LearningRecord model in backend/src/models/learning_record.py

### Services for Learning Progress

- [ ] T086 Implement progress tracking and analytics in backend/src/services/progress_service.py

### Frontend for Learning Progress

- [ ] T087 [P] Create progress dashboard in frontend/src/pages/Progress.tsx
- [ ] T088 [P] Create learning history component in frontend/src/components/LearningHistory.tsx

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T089 [P] Add comprehensive API documentation with examples in backend/src/api/
- [ ] T090 [P] Implement performance optimization (database indexes, caching) in backend/src/
- [ ] T091 [P] Add structured JSON logging across all services in backend/src/core/logger.py
- [ ] T092 [P] Implement rate limiting and security hardening in backend/src/core/security.py
- [ ] T093 [P] Create production Docker configuration in docker-compose.prod.yml
- [ ] T094 [P] Add health check endpoints for all services in backend/src/api/health.py
- [ ] T095 [P] Setup database backup and recovery procedures in scripts/backup.sh
- [ ] T096 [P] Run quickstart.md validation and create setup verification script
- [ ] T097 [P] Add frontend error boundaries and loading states in frontend/src/
- [ ] T098 [P] Create API client with retry logic and error handling in frontend/src/services/api.ts
- [ ] T099 [P] Add responsive design for mobile devices in frontend/src/
- [ ] T100 [P] Implement unit tests for all services in backend/tests/unit/
- [ ] T101 [P] Create deployment automation scripts in scripts/deploy.sh
- [ ] T102 [P] Setup monitoring and alerting (if required by deployment)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P2.2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Depends on User Story 1 (needs Exercise model)
- **User Story 2.2 (P2)**: Can start after Foundational (Phase 2) - Depends on User Story 2 (VoiceFile model)
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Depends on User Story 1 (Exercise model)

### Within Each User Story

- Tests (MANDATORY) MUST be written and FAIL before implementation
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all tests for User Story 1 together:
Task: "Contract test for course management endpoints in backend/tests/contract/test_course_management.py"
Task: "Contract test for exercise generation workflow in backend/tests/contract/test_exercise_generation.py"

# Launch all models for User Story 1 together:
Task: "Create CourseUnit model in backend/src/models/course_unit.py"
Task: "Create CourseMaterial model in backend/src/models/course_material.py"
Task: "Create Exercise model in backend/src/models/exercise.py"
Task: "Create ReviewRecord model in backend/src/models/review.py"
Task: "Create GenerationTask model in backend/src/models/generation_task.py"

# After models complete, launch services in parallel where possible:
Task: "Implement MinerU integration in backend/src/services/pdf_service.py"
Task: "Implement Dify agent integration in backend/src/services/ai_service.py"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 2.2 → Test independently → Deploy/Demo
5. Add User Story 3 → Test independently → Deploy/Demo
6. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (P1 - MVP)
   - Developer B: User Story 2 (P2 - voice synthesis)
   - Developer C: User Story 3 (P3 - image exercises)
3. Stories complete and integrate independently

---

## Key Implementation Details

### Database Versioning with Alembic
- **Configuration**: `backend/src/alembic.ini`
- **Auto-migration**: Enabled via `ENABLE_MIGRATION=true` in `.env.backend`
- **Local Development**: Manual migration management (`alembic upgrade head`)
- **Docker Deployment**: Automatic migration on container startup
- **No Init Scripts**: Alembic handles complete database schema creation

### Development Mode Selection
- **Local Development** (Required for Backend): Generate and persist migrations, full debugging capabilities
- **Docker Deployment** (Frontend/QA Only): Read-only migrations, fast testing environment, no migration generation

---

## Task Summary

**Total Tasks**: 102 tasks

**By Phase**:
- Setup: 8 tasks (T001-T008)
- Foundational: 12 tasks (T009-T020)
- User Story 1 (P1): 28 tasks (T021-T048)
- User Story 2 (P2): 11 tasks (T049-T059)
- User Story 2.2 (P5): 15 tasks (T060-T074)
- User Story 3 (P3): 10 tasks (T075-T084)
- Learning Progress: 4 tasks (T085-T088)
- Polish: 14 tasks (T089-T102)

**By Priority**:
- P1 (User Story 1): 36 tasks (MVP)
- P2 (User Story 2): 11 tasks
- P2 (User Story 2.2): 15 tasks
- P3 (User Story 3): 10 tasks

**Parallel Opportunities**: 67 tasks marked with [P]

**Independent Test Criteria**:
- US1: Content provider creates course → generates exercises → reviews/approves → learner practices
- US2: Exercise displays with click-to-play voice朗读, verify audio quality
- US2.2: Learner records voice answer → receives evaluation → sees recommended answer
- US3: Exercise displays with scenario image → learner answers based on visual context

---

## Database Migration & Versioning (Alembic)

**Configuration**:
- **Location**: `backend/src/alembic.ini`
- **Database Connection**: `DATABASE_URL` environment variable
- **Auto-migration**: Controlled by `ENABLE_MIGRATION` parameter in `.env.backend`
- **Initial Setup**: Alembic handles complete database schema creation (no init scripts)

**Migration Workflow**:

### Local Development Mode:
```bash
# Create migration script
alembic revision --autogenerate -m "description"

# Apply migrations manually
alembic upgrade head

# Check current migration status
alembic current

# View migration history
alembic history

# Downgrade to previous version (if needed)
alembic downgrade -1
```

### Docker Deployment Mode:
- **Migrations are automatically applied** when backend container starts
- Triggered when `ENABLE_MIGRATION=true` in `.env.backend`
- **⚠️ IMPORTANT**: Docker mode is read-only - **CANNOT generate new migration files**
- Use only for: Frontend debugging, QA testing, production-like environment testing
- Check migration status (read-only):
  ```bash
  docker-compose exec backend alembic current
  docker-compose exec backend alembic history
  ```
- View backend logs for migration execution:
  ```bash
  docker-compose logs -f backend
  ```
- **For backend development and schema changes**: Switch to Local Development mode

**Best Practices**:
- Always review autogenerated migrations before applying
- Test migrations in development environment first
- Use descriptive migration names
- Keep migrations small and focused
- Document complex schema changes in migration comments
- Migration failures trigger automatic rollback with error logging

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- **TDD is MANDATORY** - tests must be written before implementation per Constitution II
- Verify tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
- External services (Dify, Xinference, MinerU) must be deployed and accessible for full functionality
- Database migrations: Use Alembic (backend/src/alembic.ini) - Local mode for generation, Docker mode for testing only
- Frontend components marked [P] can be developed in parallel with backend endpoints
- Backend development MUST use Local mode to generate migration files
- Docker mode is for frontend debugging and QA testing only (migrations are read-only)
