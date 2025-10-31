# Spec-Driven Development (SDD) Framework

This project implements a **Spec-Driven Development** framework that enables AI-guided feature development through structured workflows.

## Architecture Overview

### Core Workflow (Sequential Phases)
1. **Specify** (`/speckit.specify`) → Create feature specification from natural language
2. **Clarify** (`/speckit.clarify`) → Resolve ambiguities with targeted questions  
3. **Plan** (`/speckit.plan`) → Generate technical implementation plan
4. **Tasks** (`/speckit.tasks`) → Break down into actionable, dependency-ordered tasks
5. **Implement** (`/speckit.implement`) → Execute task list with validation checkpoints

### Key Components

- **`.specify/`** - Framework infrastructure (templates, scripts, memory)
- **`.github/prompts/`** - AI agent prompt definitions for each workflow command
- **`specs/[###-feature-name]/`** - Per-feature documentation (auto-generated)
- **`.specify/memory/constitution.md`** - Project governance principles

## Essential Patterns

### Feature Branch Workflow
```bash
# Features follow naming: ###-descriptive-name (e.g., 001-user-auth)
git checkout -b "001-user-authentication"  
# OR for non-git: export SPECIFY_FEATURE="001-user-authentication"
```

### Document Structure (Per Feature)
```
specs/001-feature-name/
├── spec.md          # Business requirements & user stories
├── plan.md          # Technical architecture & decisions  
├── tasks.md         # Implementation task breakdown
├── research.md      # Technical research & decisions
├── data-model.md    # Entity relationships & validation
├── quickstart.md    # Integration scenarios
├── contracts/       # API specifications (OpenAPI/GraphQL)
└── checklists/      # Quality validation checklists
```

### Script Integration Points
All bash scripts in `.specify/scripts/bash/` use:
- **Absolute paths** - Always use full paths in script calls
- **JSON output** - Scripts support `--json` flag for programmatic use
- **Error handling** - Scripts exit with clear error codes and messages

Key scripts:
- `check-prerequisites.sh` - Validates workflow state
- `create-new-feature.sh` - Initializes feature branch/directory  
- `update-agent-context.sh` - Syncs AI agent context files

## Development Guidelines

### Working with Specifications
- **User stories must have priorities** (P1, P2, P3) for independent delivery
- **Requirements must be testable** - avoid vague adjectives without metrics
- **Success criteria must be measurable** - quantify performance/UX targets
- **Each story should be independently implementable**

### Task Organization
- **Phase-based execution**: Setup → Foundation → User Stories → Polish
- **Parallel markers `[P]`** indicate tasks that can run simultaneously
- **Story mapping `[US1]`, `[US2]`** enables independent story delivery
- **Dependencies must be explicit** in task descriptions

### Quality Gates
- **Constitution compliance** - All plans checked against `.specify/memory/constitution.md`
- **Coverage validation** - Every requirement must map to tasks
- **Checklist validation** - Quality checklists must pass before implementation
- **Traceability** - Requirements → Tasks → Implementation linkage

## Critical Commands & Workflows

### Essential Scripts (Call with absolute paths)
```bash
# Initialize new feature
./.specify/scripts/bash/create-new-feature.sh --json "feature description"

# Check workflow prerequisites  
./.specify/scripts/bash/check-prerequisites.sh --json --require-tasks

# Update AI agent context
./.specify/scripts/bash/update-agent-context.sh [copilot|claude|cursor-agent]
```

### Prompt File Integration
When implementing new commands, follow the pattern in `.github/prompts/`:
- Load context via `check-prerequisites.sh`
- Use progressive disclosure for large documents
- Apply validation checkpoints between phases
- Generate structured outputs (markdown tables, numbered lists)

### Agent Context Management
The framework auto-maintains agent-specific instruction files:
- **GitHub Copilot**: `.github/copilot-instructions.md` (this file)
- **Claude**: `CLAUDE.md` 
- **Cursor**: `.cursor/rules/specify-rules.mdc`
- **Others**: See `update-agent-context.sh` for full list

## Error Handling & Recovery

### Common Issues
- **Branch not on feature format** - Feature branches must be `###-name` pattern
- **Missing prerequisites** - Run `check-prerequisites.sh` to validate workflow state  
- **Constitution violations** - Address principle conflicts before proceeding
- **Incomplete checklists** - Quality gates must pass before implementation

### Recovery Patterns
- **Restart from spec** - Re-run `/speckit.specify` to rebuild from requirements
- **Incremental fixes** - Use `/speckit.clarify` to resolve specification ambiguities
- **Task regeneration** - Re-run `/speckit.tasks` after plan changes

## Project-Specific Conventions

### File Naming & Structure
- **Feature IDs**: Zero-padded 3-digit prefixes (001, 002, 003...)  
- **Templates**: All in `.specify/templates/` with `-template.md` suffix
- **Scripts**: Bash scripts in `.specify/scripts/bash/` with `.sh` extension
- **Prompts**: AI commands in `.github/prompts/` with `.prompt.md` suffix

### Multi-Agent Support  
Framework supports multiple AI agents simultaneously by maintaining separate context files. Each agent gets project-specific instructions while sharing the same underlying specification workflow.

### Constitution-Driven Development
The `.specify/memory/constitution.md` file defines non-negotiable project principles. All planning phases validate against these principles with explicit compliance gates.

---

*Auto-updated by SDD framework. Manual additions preserve between `<!-- MANUAL ADDITIONS START -->` and `<!-- MANUAL ADDITIONS END -->` markers.*
