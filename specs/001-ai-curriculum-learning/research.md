# Research Report: AI课程练习平台

**Date**: 2025-11-04
**Feature**: 001-ai-curriculum-learning

## Executive Summary

All technical decisions have been clarified through the specification and clarification sessions. This research consolidates the architectural decisions and integration patterns for the AI-powered English learning platform.

## Technical Decisions

### 1. Multi-Container Orchestration

**Decision**: Docker Compose (docker-compose.yaml)

**Rationale**: For a platform serving <100 concurrent users with FastAPI + React + PostgreSQL architecture, Docker Compose provides the optimal balance of simplicity and functionality. YAML-based service definitions with built-in .env file support make it easy to manage and maintain compared to Kubernetes. Local Docker builds with health checks ensure proper service startup order.

**Alternatives Considered**: Docker Swarm (too complex for scale), Kubernetes (over-engineered), Podman Compose (similar but less mature ecosystem)

### 2. AI Service Integration Pattern

**Decision**: Dify as LLMOps Platform Layer

**Rationale**: Dify acts as the abstraction layer between the application and commercial model APIs (DeepSeek/Qwen3). Rather than direct API calls, the system uses Dify-created agents for both exercise content generation and learner response evaluation. This provides better workflow management, prompt orchestration, and agent-based architecture benefits.

**Implementation**:
- Dify agent for AI text generation (exercise content creation)
- Dify agent for real-time text analysis (learner answer evaluation)
- Xinference for speech services (CosyVoice2-0.5b TTS, Whisper ASR)

### 3. Task Management for Offline Content Generation

**Decision**: FastAPI Background Tasks + Database Status Tracking

**Rationale**: Given the simple review workflow (direct confirmation, no complex multi-step process), using FastAPI's built-in background tasks plus database status polling is sufficient. This avoids the complexity and overhead of Celery/RabbitMQ while meeting the requirements for offline content generation with simple status tracking.

**Architecture**:
- Background task initiated when content provider triggers generation
- Task updates database status: pending → generating → pending_review → approved/published
- Frontend polls for progress updates
- No separate task queue system needed

### 4. PDF Processing Pipeline

**Decision**: MinerU Component

**Rationale**: MinerU provides comprehensive PDF parsing capabilities including OCR text recognition and image extraction. This is essential for processing course materials uploaded by content providers, enabling automated content extraction for exercise generation.

**Integration**: MinerU processes uploaded PDFs, extracts both text content and images, makes content available to the AI generation pipeline.

### 5. Configuration Management

**Decision**: Root .env + Service-Specific Environment Files

**Rationale**: Provides centralized environment management through root .env while allowing service-specific overrides through .env.backend and .env.frontend. This pattern enables easy local development and production deployments while maintaining service isolation.

**Structure**:
- `.env` - Shared configuration
- `.env.backend` - Backend-specific variables
- `.env.frontend` - Frontend-specific variables

### 6. Database Persistence

**Decision**: PostgreSQL with Named Volumes + Init Scripts

**Rationale**: Named volumes provide reliable data persistence across container restarts with better data safety than bind mounts. Init scripts enable automatic database schema creation on first run, ensuring consistent deployment.

**Implementation**:
- Named volume for data persistence
- SQL initialization scripts for schema setup
- Custom PostgreSQL configuration if needed

## Dependencies Analysis

| Service | Purpose | Integration Pattern |
|---------|---------|-------------------|
| MinerU | PDF content parsing & OCR | Python library/API within backend |
| Dify | AI text generation & evaluation | REST API calls to Dify agents |
| Xinference | Speech synthesis & recognition | REST API calls to deployed models |
| PostgreSQL | Data persistence | SQLAlchemy ORM connection |
| FastAPI | Backend API framework | Application core |
| React | Frontend UI | Vite build system |

## Architecture Benefits

1. **Simplicity**: All components align with simplicity principle - no over-engineering
2. **Scalability**: Separate containers enable independent scaling as needed
3. **Maintainability**: Clear service boundaries and well-defined interfaces
4. **Flexibility**: Environment-based configuration for different deployment scenarios
5. **Testability**: Independent services enable isolated testing

## Risk Mitigation

- **AI Service Reliability**: Dify provides abstraction layer, can swap underlying models
- **PDF Processing**: MinerU handles various PDF formats, fallback to manual input available
- **Speech Services**: Local Xinference deployment ensures availability
- **Data Persistence**: Named volumes with regular backups recommended

## Compliance with Constitution

All research decisions comply with the Specify Constitution principles:
- ✓ User Story Driven: Architecture supports independent story delivery
- ✓ Test-First: Structure enables contract tests before integration
- ✓ Independent Testability: Separated services with clear interfaces
- ✓ Phased Implementation: Setup → Foundational → Stories → Polish sequence
- ✓ Simplicity: No complexity violations, all decisions favor simplicity

## Conclusion

The research confirms that all technical decisions are appropriate for the <100 concurrent user scale and aligned with the Constitution principles. The architecture provides a solid foundation for implementing the AI curriculum learning platform with proper separation of concerns and maintainability.
