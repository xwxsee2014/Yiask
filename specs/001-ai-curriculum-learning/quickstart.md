# Quickstart Guide - AI课程练习平台

**Date**: 2025-11-04
**Version**: 1.0.0

This guide will help you get the Yiask AI智能课程练习平台 up and running quickly using Docker Compose.

## Prerequisites

- **Docker** (version 20.10 or later)
- **Docker Compose** (version 2.0 or later)
- **Git** (for cloning the repository)
- **4GB RAM minimum** (8GB recommended)
- **10GB disk space** for images and data

## Architecture Overview

The platform consists of the following services:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │     Backend      │    │   PostgreSQL    │
│   (React)       │◄──►│   (FastAPI)      │◄──►│   Database      │
│   Port: 3000    │    │   Port: 8000     │    │   Port: 5432    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │                       │                       │
         │                       ▼                       │
         │              ┌──────────────────┐              │
         │              │   External AI    │              │
         │              │   Services       │              │
         │              │                  │              │
         │              │  • Dify Agent    │              │
         │              │  • Xinference    │              │
         │              │  • MinerU        │              │
         │              └──────────────────┘              │
         │                                                 │
         └─────────────────────────────────────────────────┘
```

### External Services (Deployed Independently)

- **Dify Platform**: LLMOps platform for AI agent management
  - URL: `http://dify.yourdomain.com` or localhost
  - Used for: Text generation and evaluation agents

- **Xinference**: Model serving platform
  - URL: `http://xinference.yourdomain.com` or localhost
  - Used for: Speech synthesis (CosyVoice2-0.5b) and speech recognition (Whisper)

- **MinerU**: PDF processing service
  - Can be deployed as a service or used as a Python library

## Quick Start (5 minutes)

> **Development Mode**: You have two options for development:
> - **Local Development** (Recommended): Run backend and frontend locally for easier debugging and hot reload
> - **Docker Deployment** (Quick Start): Use Docker Compose for fast deployment with all services containerized

### Step 1: Clone and Setup

```bash
# Clone the repository
git clone <repository-url>
cd yiask

# Check your Docker environment
docker --version
docker-compose --version
```

### Step 2: Configure Environment Variables

Copy the environment template and customize:

```bash
# Create environment files
cp .env.example .env
cp .env.example .env.backend
cp .env.example .env.frontend
```

**Edit `.env` (root configuration):**

```bash
# Database Configuration
POSTGRES_USER=yiask_user
POSTGRES_PASSWORD=secure_password_here
POSTGRES_DB=yiask_db

# Database URL (used by backend)
DATABASE_URL=postgresql://yiask_user:secure_password_here@postgres:5432/yiask_db
```

**Edit `.env.backend` (backend-specific):**

```bash
# Backend Configuration
ENVIRONMENT=development
DEBUG=True
API_V1_STR=/v1
SECRET_KEY=your-secret-key-change-in-production

# Database Configuration
DATABASE_URL=postgresql://user:password@postgres:5432/yiask_db
ENABLE_MIGRATION=true

# CORS Settings
CORS_ORIGINS=["http://localhost:3000", "http://frontend:3000"]

# External Services
DIFY_API_URL=http://dify.yourdomain.com/api/v1
DIFY_API_KEY=your_dify_api_key
DIFY_TEXT_GENERATION_AGENT_ID=your_agent_id
DIFY_EVALUATION_AGENT_ID=your_evaluation_agent_id

XINFERENCE_API_URL=http://xinference.yourdomain.com/api/v1
XINFERENCE_TTS_MODEL_ID=cosyvoice-2-0-5b
XINFERENCE_STT_MODEL_ID=whisper

MINERU_SERVICE_URL=http://mineru.yourdomain.com

# File Storage
UPLOAD_DIR=/app/uploads
VOICE_FILES_DIR=/app/voice_files

# Background Tasks
MAX_GENERATION_TASKS=10
```

**Edit `.env.frontend` (frontend-specific):**

```bash
# Frontend Configuration
REACT_APP_API_URL=http://localhost:8000
REACT_APP_API_V1_STR=/v1
REACT_APP_ENVIRONMENT=development
```

### Step 3: Start the Services

```bash
# Start all services in detached mode
docker-compose up -d

# View logs
docker-compose logs -f

# Or start without detached mode to see output
docker-compose up
```

**Expected Output:**

```
Creating network "yiask_default" with the default driver
Creating volume "yiask_postgres_data" with default driver
Creating yiask_postgres_1 ... done
Creating yiask_backend_1   ... done
Creating yiask_frontend_1  ... done
```

### Step 4: Verify Installation

Open your browser and navigate to:

- **Frontend**: http://localhost:3000
- **Backend API Docs**: http://localhost:8000/docs
- **Backend Health Check**: http://localhost:8000/health

### Step 5: Initial Setup

```bash
# Check database connection
curl http://localhost:8000/health

# Create initial admin user (via API or directly in database)
# The first registered user will be able to create courses
```

## Usage Workflow

### For Content Providers:

1. **Register an account** at http://localhost:3000/register
2. **Create a course unit** (title, difficulty, vocabulary, grammar points)
3. **Upload course materials** (PDF files, text, or images)
4. **Trigger exercise generation** - backend will:
   - Process PDFs with MinerU
   - Generate exercises via Dify agent
   - Create voice files via Xinference
5. **Review generated content** - approve or reject exercises
6. **Publish exercises** - learners can now access them

### For Learners:

1. **Register an account** at http://localhost:3000/register
2. **Browse available courses** (published status only)
3. **Select a course unit** to practice
4. **Complete exercises**:
   - Listen to AI-generated voice朗读
   - Answer via voice input
   - Receive real-time evaluation
   - View recommended answers
5. **Track progress** - see your learning history

## Development Workflow

### Backend Development

#### Option 1: Local Development (Recommended for Development)

```bash
# Prerequisites:
# - Python 3.11+
# - PostgreSQL (local installation)
# - Node.js 18+ (for frontend)
# - Git

# 1. Setup Python virtual environment
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Setup database
# Create PostgreSQL database:
createdb yiask_db

# Update .env.backend with local database URL:
# DATABASE_URL=postgresql://username:password@localhost:5432/yiask_db

# 4. Setup environment
# Copy .env.backend.example to .env.backend and update:
cp .env.backend.example .env.backend
# Edit .env.backend with your local settings

# 5. Run database migrations
# Ensure ENABLE_MIGRATION=true in .env.backend
alembic upgrade head

# 6. Start backend server (with hot reload)
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# 7. In another terminal, run tests
pytest

# 8. Run specific test file
pytest tests/test_exercises.py -v

# 9. Access API documentation
# Open browser to: http://localhost:8000/docs

# Database Management (Local):
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

#### Option 2: Docker Deployment (Quick Start)

```bash
# Prerequisites:
# - Docker and Docker Compose installed

# 1. Start all services with Docker Compose
# Migrations are automatically applied on backend container startup
# This happens when ENABLE_MIGRATION=true in .env.backend
docker-compose up -d

# 2. Monitor backend startup and migration status
# Watch the backend logs to see migration progress
docker-compose logs -f backend

# Expected log output:
# INFO  - Database migrations applied successfully
# INFO  - Started server process
# INFO  - Uvicorn running on http://0.0.0.0:8000

# 3. Access backend container (if needed for debugging)
docker-compose exec backend bash

# 4. Run tests inside container
docker-compose exec backend pytest

# 5. Run specific test file inside container
docker-compose exec backend pytest tests/test_exercises.py -v

# 6. Access API documentation
# Open browser to: http://localhost:8000/docs

# Database Management (Docker):
# NOTE: Migrations are handled automatically by the backend container
# Do NOT manually run 'alembic upgrade head' in Docker mode

# Check migration status (read-only - for debugging)
docker-compose exec backend alembic current

# View migration history (read-only - for debugging)
docker-compose exec backend alembic history

# Create new migration (for development):
# Switch to local development mode and use the commands from "Option 1: Local Development"
# Then rebuild the Docker image to include the migration

# View backend logs (includes automatic migration execution logs)
docker-compose logs -f backend

# Migration failure handling:
# - Automatic rollback on error
# - Error details logged to backend container logs
# - Check docker-compose logs if container fails to start
# - If migration fails, check DATABASE_URL and ENABLE_MIGRATION in .env.backend
```

### Frontend Development

#### Option 1: Local Development (Recommended for Development)

```bash
# Prerequisites:
# - Node.js 18+
# - npm or yarn

# 1. Install dependencies
cd frontend
npm install

# 2. Setup environment
# Copy .env.frontend.example to .env.frontend and update:
cp .env.frontend.example .env.frontend
# Edit .env.frontend with your settings (e.g., API URL)

# 3. Start development server (hot reload)
npm run dev

# 4. In another terminal, run tests
npm test

# 5. Build for production
npm run build

# 6. Preview production build locally
npm run preview

# 7. Access frontend
# Open browser to: http://localhost:3000
```

#### Option 2: Docker Deployment (Quick Start)

```bash
# Prerequisites:
# - Docker and Docker Compose installed

# Frontend is automatically started with Docker Compose
# See "Option 2: Docker Deployment" in Backend Development section

# Access frontend container (if needed for debugging)
docker-compose exec frontend sh

# Install dependencies inside container
docker-compose exec frontend npm install

# Run development server inside container
docker-compose exec frontend npm run dev

# Build for production inside container
docker-compose exec frontend npm run build

# View logs
docker-compose logs -f frontend
```

### Database Management

```bash
# Access PostgreSQL
docker-compose exec postgres psql -U yiask_user -d yiask_db

# View database tables
\dt

# View table structure
\d+ users

# Backup database
docker-compose exec postgres pg_dump -U yiask_user yiask_db > backup.sql

# Restore database
docker-compose exec -T postgres psql -U yiask_user yiask_db < backup.sql
```

## Troubleshooting

### Service Won't Start

```bash
# Check logs
docker-compose logs [service_name]

# Common issues:
# - Port already in use: Change ports in docker-compose.yml
# - Permission denied: Check file permissions
# - Missing environment variables: Verify .env files
```

### Database Connection Errors

```bash
# Verify PostgreSQL is running
docker-compose exec postgres pg_isready -U yiask_user

# Check database URL format
# Should be: postgresql://user:password@host:port/database
```

### External Service Connectivity

```bash
# Test Dify connection from backend
curl -H "Authorization: Bearer YOUR_API_KEY" \
     http://dify.yourdomain.com/api/v1/agent YOUR_AGENT_ID

# Test Xinference connection
curl http://xinference.yourdomain.com/api/v1/models

# Test MinerU (if deployed as service)
curl -X POST http://mineru.yourdomain.com/extract \
     -F "file=@test.pdf"
```

### Permission Issues

```bash
# Fix file permissions
sudo chown -R $USER:$USER ./uploads ./voice_files
chmod -R 755 ./uploads ./voice_files
```

### Reset Everything

```bash
# Stop and remove all containers
docker-compose down

# Remove volumes (WARNING: deletes all data)
docker-compose down -v

# Remove images (optional)
docker-compose down --rmi all

# Restart from scratch
docker-compose up -d
```

## Configuration Reference

### Docker Compose Services

| Service | Port | Health Check | Description |
|---------|------|--------------|-------------|
| postgres | 5432 | `/` | PostgreSQL database |
| backend | 8000 | `/health` | FastAPI application |
| frontend | 3000 | N/A | React application |

### Environment Variables

**Backend (`.env.backend`):**

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `DATABASE_URL` | Yes | PostgreSQL connection | `postgresql://...` |
| `DIFY_API_URL` | Yes | Dify platform URL | `http://dify:5001` |
| `DIFY_API_KEY` | Yes | Dify API key | `app-xxx` |
| `XINFERENCE_API_URL` | Yes | Xinference service URL | `http://xinference:9997` |
| `SECRET_KEY` | Yes | JWT secret | `change-me` |

**Frontend (`.env.frontend`):**

| Variable | Required | Description | Example |
|----------|----------|-------------|---------|
| `REACT_APP_API_URL` | Yes | Backend API URL | `http://localhost:8000` |

## Performance Tuning

### For Development

```yaml
# docker-compose.yml
services:
  backend:
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
```

### For Production

```yaml
# docker-compose.prod.yml
services:
  backend:
    replicas: 2
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

## API Documentation

After starting the backend, visit:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **OpenAPI JSON**: http://localhost:8000/openapi.json

## Security Notes

⚠️ **Important**: This is a development setup. For production:

1. Change all default passwords
2. Use strong SECRET_KEY
3. Enable HTTPS
4. Set up proper CORS origins
5. Implement rate limiting
6. Use environment-specific .env files
7. Enable database SSL
8. Set up proper backup strategy

## Next Steps

1. **External Services Setup**: Deploy Dify, Xinference, and MinerU
2. **User Management**: Implement proper user onboarding flow
3. **Content Moderation**: Add automated content checks
4. **Analytics**: Track learning progress and engagement
5. **Mobile Support**: Consider React Native for mobile apps
6. **Caching**: Implement Redis for performance
7. **Monitoring**: Add Prometheus/Grafana for metrics

## Support

- **Issues**: Create an issue in the repository
- **Documentation**: See `docs/` directory
- **API Reference**: http://localhost:8000/docs
- **Community**: Join our Discord/Slack channel

## License

This project is licensed under the MIT License - see the LICENSE file for details.
