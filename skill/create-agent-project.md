---
description: Scaffold a new project with Claude Code agent capabilities (workflows, skills, documentation)
allowed-tools: Read, Write, Bash, AskUserQuestion, Glob
---

# Create Agent Project

Scaffolds a new project with Claude Code workflows, skills, and documentation standards.

## What Gets Created

```
<project>/
├── .agent/workflows/development.md    # Unified dev workflow
├── .claude/
│   ├── commands/                      # Planning commands
│   │   ├── feature.md
│   │   ├── bug.md
│   │   └── tech-debt.md
│   └── agents/code-reviewer.md        # Code review agent
├── docs/
│   ├── standards/
│   │   ├── README.md
│   │   └── development-workflow.md
│   ├── architecture.md
│   ├── development-plan.md
│   └── tech-debt.md
└── CLAUDE.md                          # Central quick-reference
```

---

## Phase 1: Gather Project Information

Use `AskUserQuestion` to collect project details. Ask in batches to minimize back-and-forth.

### Batch 1: Basic Info

Ask for:
1. **Project name** - identifier for the project
2. **Project root path** - where to create files (validate exists or will be created)
3. **One-line description** - what is this project?

### Batch 2: Tech Stack

Use multi-select for:

**Frontend** (pick one or none):
- React
- Vue
- Next.js
- Angular
- Svelte
- None

**Backend** (pick one or none):
- .NET
- Node.js/Express
- Python/FastAPI
- Go
- Java/Spring
- None

**Database** (pick one or none):
- MongoDB
- PostgreSQL
- MySQL
- SQLite
- None

**Containerization**:
- Docker
- None

### Batch 3: Git Platform

- GitHub (default)
- GitLab
- Other

### Infer Defaults Based on Selections

After tech stack selection, infer sensible defaults:

| Stack | Port | Install | Run | Lint/Format |
|-------|------|---------|-----|-------------|
| React | 3000 | `npm install` | `npm run dev` | `npm run lint` |
| Vue | 5173 | `npm install` | `npm run dev` | `npm run lint` |
| Next.js | 3000 | `npm install` | `npm run dev` | `npm run lint` |
| .NET | 5001 | `dotnet restore` | `dotnet run` | `dotnet format` |
| Node.js | 3001 | `npm install` | `npm start` | `npm run lint` |
| Python/FastAPI | 8000 | `pip install -r requirements.txt` | `uvicorn main:app --reload` | `ruff check . && ruff format .` |
| Go | 8080 | `go mod download` | `go run .` | `go fmt ./...` |
| MongoDB | 27017 | - | - | - |
| PostgreSQL | 5432 | - | - | - |

### Batch 4: Directory Structure (if needed)

Only ask if multiple services detected:
- Frontend directory name (default: `frontend`)
- Backend directory name (default: `backend`)
- Docker directory name (default: `docker` or `.`)

---

## Phase 2: Clone Templates

```bash
# Clone template repository to temp directory
TEMPLATE_DIR="/tmp/agent-project-templates-$(date +%s)"
git clone --depth 1 https://github.com/Snoww3d/agent-project-templates "$TEMPLATE_DIR"
```

If the clone fails (no network), report the error and suggest the user check their connection.

---

## Phase 3: Create Directory Structure

```bash
PROJECT_ROOT="<collected_path>"

mkdir -p "$PROJECT_ROOT/.agent/workflows"
mkdir -p "$PROJECT_ROOT/.claude/commands"
mkdir -p "$PROJECT_ROOT/.claude/agents"
mkdir -p "$PROJECT_ROOT/docs/standards"
```

---

## Phase 4: Process and Generate Files

For each template file, read it and replace placeholders with collected context.

### Context Object Structure

Build a context object from collected answers:

```json
{
  "PROJECT_NAME": "my-project",
  "PROJECT_ROOT": "/path/to/project",
  "PROJECT_DESCRIPTION": "A cool project",
  "TECH_STACK": "React, .NET, MongoDB",
  "GENERATED_DATE": "2024-01-15",

  "frontend": true,
  "FRONTEND_DIR": "frontend",
  "FRONTEND_TECH": "React",
  "FRONTEND_INSTALL_CMD": "npm install",
  "FRONTEND_RUN_CMD": "npm run dev",
  "FRONTEND_LINT_CMD": "npm run lint",

  "backend": true,
  "BACKEND_DIR": "backend",
  "BACKEND_TECH": ".NET",
  "BACKEND_INSTALL_CMD": "dotnet restore",
  "BACKEND_RUN_CMD": "dotnet run",
  "BACKEND_FORMAT_CMD": "dotnet format",

  "python": false,

  "database": true,
  "DATABASE_TECH": "MongoDB",

  "docker": true,
  "DOCKER_DIR": "docker",
  "services": [
    { "name": "frontend", "port": 3000 },
    { "name": "backend", "port": 5001 },
    { "name": "mongodb", "port": 27017 }
  ],

  "github": true,
  "gitlab": false,

  "skills": [
    { "name": "/start-app", "description": "Start the application" }
  ],

  "KEY_FILES": "- `backend/Program.cs` - Entry point\n- `frontend/src/App.tsx` - Root component"
}
```

### Template Processing

For each template:
1. Read the `.hbs` file
2. Replace `{{VARIABLE}}` with context values
3. Process `{{#if condition}}...{{/if}}` blocks
4. Process `{{#each array}}...{{/each}}` loops
5. Write the processed content to the target file

### Files to Generate

| Template | Target |
|----------|--------|
| `CLAUDE.md.hbs` | `CLAUDE.md` |
| `workflows/development.md.hbs` | `.agent/workflows/development.md` |
| `commands/feature.md.hbs` | `.claude/commands/feature.md` |
| `commands/bug.md.hbs` | `.claude/commands/bug.md` |
| `commands/tech-debt.md.hbs` | `.claude/commands/tech-debt.md` |
| `agents/code-reviewer.md.hbs` | `.claude/agents/code-reviewer.md` |
| `docs/standards/README.md.hbs` | `docs/standards/README.md` |
| `docs/standards/development-workflow.md.hbs` | `docs/standards/development-workflow.md` |
| `docs/architecture.md.hbs` | `docs/architecture.md` |
| `docs/development-plan.md.hbs` | `docs/development-plan.md` |
| `docs/tech-debt.md.hbs` | `docs/tech-debt.md` |

---

## Phase 5: Create Project-Specific Skills

Based on tech stack, create helpful utility skills:

### If Docker is enabled

Create `.claude/commands/start-app.md`:
```markdown
---
description: Start the application using Docker
allowed-tools: Bash
---

# Start Application

```bash
cd {{DOCKER_DIR}}
docker compose up -d
```

**Services**:
{{#each services}}
- {{this.name}}: http://localhost:{{this.port}}
{{/each}}
```

### If E2E tests detected

Create `.claude/commands/run-e2e.md`:
```markdown
---
description: Run E2E tests
allowed-tools: Bash
---

# Run E2E Tests

```bash
cd {{FRONTEND_DIR}}
{{E2E_CMD}}
```
```

---

## Phase 6: Cleanup and Next Steps

1. Remove temp clone directory (if used)
2. List all created files
3. Provide next steps:

```
## Scaffolding Complete!

Created files:
- CLAUDE.md
- .agent/workflows/development.md
- .claude/commands/feature.md
- .claude/commands/bug.md
- .claude/commands/tech-debt.md
- .claude/agents/code-reviewer.md
- docs/standards/README.md
- docs/standards/development-workflow.md
- docs/architecture.md
- docs/development-plan.md
- docs/tech-debt.md

## Next Steps

1. Review and customize `CLAUDE.md` for your project specifics
2. Fill in `docs/architecture.md` with your system diagrams
3. Update `docs/development-plan.md` with your roadmap phases
4. Initialize git if not already: `git init`
5. Test the workflows:
   - `/feature` - Plan a new feature
   - `/create-feature` - Implement with full workflow
```

---

## Error Handling

- If project root doesn't exist, ask if it should be created
- If files already exist, ask before overwriting
- If git clone fails, use local templates or inline fallbacks
- Validate all required information is collected before proceeding
