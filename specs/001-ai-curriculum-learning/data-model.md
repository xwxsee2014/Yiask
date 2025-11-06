# Data Model: AI课程练习平台

**Date**: 2025-11-04
**Feature**: 001-ai-curriculum-learning

## Overview

This data model defines the entities and relationships for the AI-powered English learning platform. The model supports user role separation, course content management, exercise generation, and learning progress tracking.

## Core Entities

### 1. User & Authentication

#### User
```yaml
User:
  id: UUID (PK)
  email: string (unique, indexed)
  password_hash: string (encrypted)
  role: enum[learner, content_provider]
  created_at: timestamp
  updated_at: timestamp
  is_active: boolean

Constraints:
  - Email must be unique across the system
  - Password encryption using bcrypt/argon2
```

#### UserProfile
```yaml
UserProfile:
  id: UUID (PK)
  user_id: UUID (FK → User.id)
  display_name: string
  avatar_url: string (nullable)
  learning_preferences: jsonb (nullable)
  created_at: timestamp
  updated_at: timestamp

Constraints:
  - One-to-one relationship with User
  - Learning preferences stored as JSON for flexibility
```

### 2. Course Content Management

#### CourseUnit
```yaml
CourseUnit:
  id: UUID (PK)
  content_provider_id: UUID (FK → User.id)
  title: string
  description: text
  difficulty_level: enum[beginner, intermediate, advanced]
  subject: string
  vocabulary: jsonb (list of vocabulary items)
  grammar_points: jsonb (list of grammar points)
  status: enum[draft, published, archived]
  created_at: timestamp
  updated_at: timestamp

Constraints:
  - Foreign key to User (content_provider)
  - Status: draft → published → archived lifecycle
  - Vocabulary and grammar stored as JSON arrays
```

#### CourseMaterial
```yaml
CourseMaterial:
  id: UUID (PK)
  course_unit_id: UUID (FK → CourseUnit.id)
  material_type: enum[pdf, text, image]
  order: integer (ordering within course unit)
  file_path: string (for pdf/image)
  text_content: text (for text materials)
  extracted_text: text (for PDF via MinerU)
  extracted_images: jsonb (list of image references)
  upload_timestamp: timestamp
  processed: boolean (MinerU processing status)

Constraints:
  - Foreign key to CourseUnit
  - One course unit can have multiple materials
  - Order field ensures materials are presented in sequence
  - Material types are mutually exclusive
  - MinerU processing tracks extraction status
```

### 3. Exercise System

#### Exercise
```yaml
Exercise:
  id: UUID (PK)
  course_unit_id: UUID (FK → CourseUnit.id)
  exercise_type: enum[q_and_a, image_based_q_and_a]
  question_text: text
  recommended_answer: text (AI-generated model answer)
  explanation: text
  difficulty_score: float (1.0-5.0)
  generation_status: enum[generating, pending_review, approved, rejected]
  reviewed_by: UUID (FK → User.id, nullable)
  reviewed_at: timestamp (nullable)
  created_at: timestamp
  updated_at: timestamp

Constraints:
  - Foreign key to CourseUnit
  - Generation workflow: generating → pending_review → approved/rejected
  - Exercise types: text-based Q&A and image-based Q&A
  - Recommended answer serves as the reference response
```

#### ExerciseImage
```yaml
ExerciseImage:
  id: UUID (PK)
  exercise_id: UUID (FK → Exercise.id)
  image_path: string
  caption: text (nullable)
  alt_text: string (for accessibility)
  uploaded_by: UUID (FK → User.id)
  upload_timestamp: timestamp

Constraints:
  - Foreign key to Exercise
  - Images associated with image_based exercises
  - Accessibility text required
```

### 4. Voice & Audio System

#### VoiceFile
```yaml
VoiceFile:
  id: UUID (PK)
  exercise_id: UUID (FK → Exercise.id)
  file_path: string (audio file location)
  voice_type: enum[question, recommended_answer, evaluation]
  duration_ms: integer
  generated_at: timestamp
  quality_score: float (1.0-5.0)

Constraints:
  - Foreign key to Exercise
  - Voice synthesis via Xinference CosyVoice2-0.5b
  - Three voice types: question, recommended_answer, evaluation
```

#### VoiceInput
```yaml
VoiceInput:
  id: UUID (PK)
  exercise_id: UUID (FK → Exercise.id)
  learner_id: UUID (FK → User.id)
  audio_file_path: string (path to learner's recorded audio)
  transcribed_text: text
  transcription_succeeded: boolean (whether Whisper transcription succeeded)
  confidence_score: float (0.0-1.0, nullable if transcription failed)
  input_timestamp: timestamp

Constraints:
  - Foreign key to Exercise and User
  - Audio file saved for potential re-processing or review
  - Whisper model for speech-to-text
  - Confidence score only available when transcription succeeded
  - Retry mechanism: handled in implementation (max N attempts)
```

### 5. Evaluation & Learning Progress

#### AnswerEvaluation
```yaml
AnswerEvaluation:
  id: UUID (PK)
  voice_input_id: UUID (FK → VoiceInput.id)
  accuracy_score: float (1.0-5.0)
  grammar_score: float (1.0-5.0)
  feedback_text: text
  overall_score: float (1.0-5.0)
  evaluated_at: timestamp

Constraints:
  - Foreign key to VoiceInput
  - Only created for VoiceInput with transcription_succeeded=true
  - Multi-dimensional evaluation via Dify agent (accuracy + grammar)
  - Generated via real-time text analysis
```

#### LearningRecord
```yaml
LearningRecord:
  id: UUID (PK)
  learner_id: UUID (FK → User.id)
  exercise_id: UUID (FK → Exercise.id)
  voice_input_id: UUID (FK → VoiceInput.id, nullable)
  evaluation_id: UUID (FK → AnswerEvaluation.id, nullable)
  completion_time_seconds: integer
  attempts_count: integer
  completion_status: enum[in_progress, completed]
  started_at: timestamp
  completed_at: timestamp (nullable)

Constraints:
  - Foreign keys to User, Exercise, VoiceInput, AnswerEvaluation
  - Tracks learning session progress
  - Multiple attempts per exercise allowed
  - Record created when learner STARTS an exercise
  - No record = learner hasn't started yet

States:
  - in_progress: Started but not finished (includes abandoned/quit)
  - completed: Successfully finished (submitted answer and received evaluation)
```

### 6. Content Review & Approval

#### ReviewRecord
```yaml
ReviewRecord:
  id: UUID (PK)
  exercise_id: UUID (FK → Exercise.id)
  reviewer_id: UUID (FK → User.id)
  status: enum[approved, rejected, modified]
  review_comments: text (nullable)
  original_content: jsonb (exercise snapshot before modification)
  reviewed_at: timestamp

Constraints:
  - Foreign key to Exercise and User
  - Snapshots original content for audit trail
  - Supports approval, rejection, and modification workflows
```

### 7. Generation Workflow

#### GenerationTask
```yaml
GenerationTask:
  id: UUID (PK)
  course_unit_id: UUID (FK → CourseUnit.id)
  status: enum[pending, in_progress, completed, failed]
  task_type: enum[exercise_generation, voice_synthesis]
  progress_percentage: integer (0-100)
  initiated_by: UUID (FK → User.id)
  initiated_at: timestamp
  completed_at: timestamp (nullable)
  error_message: text (nullable)

Constraints:
  - Foreign key to CourseUnit and User
  - Tracks background task progress
  - Supports retry logic on failure (handled in application layer with exponential backoff)
```

## Entity Relationships

```
User (1) ←→ (1) UserProfile
User (1) ←→ (N) CourseUnit (as content_provider)
CourseUnit (1) ←→ (N) CourseMaterial
CourseUnit (1) ←→ (N) Exercise
Exercise (1) ←→ (N) ExerciseImage
Exercise (1) ←→ (N) VoiceFile
Exercise (1) ←→ (N) VoiceInput
VoiceInput (1) ←→ (1) AnswerEvaluation
Exercise (1) ←→ (1) ReviewRecord (content provider's review)
CourseUnit (1) ←→ (N) GenerationTask
```

## State Transitions

### Exercise Generation Workflow
```
Draft CourseMaterial → Trigger Generation → GenerationTask (pending)
    ↓
GenerationTask (pending) → GenerationTask (in_progress) → Exercise (generating)
    ↓
Exercise (generating) → Exercise (pending_review) → ReviewRecord (created)
    ↓
ReviewRecord (status) → Exercise (approved/rejected)
    ↓
Exercise (approved) → VoiceFile (generated) → Published for learners
```

### Voice Generation Failure Workflow
```
Exercise (approved) → VoiceFile Generation Task
    ↓
GenerationTask (voice_synthesis, pending) → GenerationTask (in_progress)
    ↓
VoiceFile (generated successfully) → Exercise (ready for learners with voice)
    ↓ OR
GenerationTask (voice_synthesis, failed)
    ↓
Exercise (approved) → Exercise (ready with degraded experience - text only)
    ↓
GenerationTask (voice_synthesis, failed) → Retry Counter (max 3 attempts)
    ↓
On retry 1, 2 → Auto-retry with exponential backoff
    ↓
On retry 3 (final attempt) → GenerationTask (failed) → Manual intervention
    ↓
API endpoint: POST /exercises/{exercise_id}/retry-voice-generation
    ↓
Content provider can manually trigger retry → New GenerationTask (voice_synthesis)
```

### Learning Session Workflow
```
Learner selects Exercise → LearningRecord (created, in_progress)
    ↓
VoiceInput recorded → Transcribe with Whisper (with retry mechanism)
    ↓ (two outcomes)
    ├── transcription_succeeded=true → AnswerEvaluation → LearningRecord (completed)
    └── transcription_succeeded=false → Fail (after max N retries)

No LearningRecord exists → Learner hasn't started exercise yet
```

## Validation Rules

### CourseUnit
- Title: required, max 200 characters
- Difficulty level: must match content complexity
- Status transitions: draft → published → archived (no backward transitions)

### CourseMaterial
- Order: required, sequential integer starting from 1
- Order must be unique within a course unit
- Material type determines file_path/text_content usage

### Exercise
- Question text: required, max 2000 characters
- Recommended answer: required (AI-generated model answer)
- Generation status: workflow-based transitions only

### VoiceInput
- Audio file path: required (learner's recorded voice file)
- transcription_succeeded: required boolean field
- confidence_score: 0.0-1.0 range (nullable if transcription failed)
- transcribed_text: required when transcription succeeded
- Audio file: required format validation (mp3, wav, m4a)
- Evaluation only created if transcription_succeeded=true
- Retry mechanism: implementation detail (max N attempts)

### AnswerEvaluation
- Accuracy and grammar scores: 1.0-5.0 range
- Feedback text: optional but recommended
- Overall score: calculated from sub-scores or provided independently
- Fluency score:暂未实现 (待AI模型选择)

### LearningRecord
- Record created when learner starts exercise (select + begin)
- completion_status: in_progress → completed
- in_progress: includes both active attempts and abandoned/quit sessions
- No record exists = learner hasn't started exercise yet
- Multiple attempts allowed per exercise

## Indexes & Performance

### Recommended Indexes
```sql
-- User lookup
CREATE INDEX idx_user_email ON "User"(email);

-- Course content queries
CREATE INDEX idx_course_unit_provider ON "CourseUnit"(content_provider_id);
CREATE INDEX idx_course_unit_status ON "CourseUnit"(status);
CREATE INDEX idx_course_unit_difficulty ON "CourseUnit"(difficulty_level);
CREATE INDEX idx_course_material_unit_order ON "CourseMaterial"(course_unit_id, "order");

-- Exercise queries
CREATE INDEX idx_exercise_course_unit ON "Exercise"(course_unit_id);
CREATE INDEX idx_exercise_status ON "Exercise"(generation_status);

-- Learning progress
CREATE INDEX idx_learning_record_learner ON "LearningRecord"(learner_id);
CREATE INDEX idx_learning_record_completion ON "LearningRecord"(completion_status);

-- Voice files
CREATE INDEX idx_voice_file_exercise ON "VoiceFile"(exercise_id);
```

## Voice Generation Failure Handling

### Overview
Voice synthesis failures are handled through automatic retry mechanisms and degraded mode operation. Exercises remain usable even when voice generation fails.

### Failure Handling Strategy
1. **Automatic Retry**: Service layer implements exponential backoff retry (max 3 attempts)
2. **Degraded Mode**: Failed exercises are usable without voice (text-only mode)
3. **Manual Retry**: Content providers can manually trigger retry via API endpoint
4. **Error Tracking**: All failures logged in GenerationTask.error_message

### Constraints
- Voice generation failure does not block exercise usage
- Learners can still practice with text-only exercises
- Retry mechanism handled in application layer (no database persistence)
- Max retry attempts: 3 (configurable per environment)
- Manual retry available after automatic retries exhausted

## Data Retention

- **User Data**: Permanent until account deletion
- **Voice Files**: 1 year retention (configurable)
- **Learning Records**: 3 years retention (for progress tracking)
- **Generation Tasks**: 30 days (cleanup old task logs)
- **Exercise Content**: Permanent (educational content value)

## Security Considerations

- **PII Protection**: User emails encrypted at rest
- **Voice Data**: Audio files stored securely, access-controlled
- **Password Security**: bcrypt/argon2 hashing with salt
- **Audit Trail**: ReviewRecord maintains content change history
- **Access Control**: Role-based access to sensitive fields

## Database Migration & Versioning

**Migration Tool**: Alembic

**Configuration**:
- **Location**: `backend/src/alembic.ini`
- **Database Connection**: `DATABASE_URL` environment variable
- **Auto-migration**: Controlled by `ENABLE_MIGRATION` parameter in `.env.backend`
- **Initial Setup**: Alembic handles complete database schema creation (no init scripts)

**Migration Workflow**:
1. Database migrations are automatically applied when backend container starts
2. Triggered when `ENABLE_MIGRATION=true` in `.env.backend`
3. Migration failures automatically trigger rollback
4. Error details logged to backend container logs

**Migration Management**:
- Create new migrations: `alembic revision --autogenerate -m "description"`
- View current version: `alembic current`
- View migration history: `alembic history`
- Manual upgrade: `alembic upgrade head`
- Manual downgrade: `alembic downgrade -1`

**Best Practices**:
- Always review autogenerated migrations before applying
- Test migrations in development environment first
- Use descriptive migration names
- Keep migrations small and focused
- Document complex schema changes in migration comments
