# AGENT ORCHESTRATION WORKFLOW

Multi-agent patterns for Claude Code via system prompt (CLAUDE.md).

## Architecture

**Orchestration is embedded in CLAUDE.md, triggered automatically by task complexity.**

```
User Request → CLAUDE.md Logic → Codebase-search → Execution-planner → test-generator → Build → Test Validate
```

### Component Roles
- **Skills** (via Skill tool): codebase-search, execution-planner, test-generator, researching-features, agent-creating, skill-creating
- **Tasks** (via Task tool): Direct sub-agent execution for builds
- **Commands** (human-only): TBD
- **test-generator** (mandatory): Tests generated before implementation, builders make tests pass

## Core Workflow

### 1. Complexity Assessment
- **Simple (< 3 files)**: Direct implementation
- **Complex (3+ files OR unknown areas)**: Codebase-search → Execution-planner → Build

### 2. Orchestrated Execution
```
a. Initialize context-pool if missing/stale (> 24h)
b. Detect git availability, set validation mode (git vs timestamp)
c. Checkpoint: git rev-parse HEAD > .workflow-checkpoint (or touch .workflow-start for non-git)
d. Determine project type from context-pool (chrome-extension, react, express, etc.)
e. Codebase-search: Parallel search (2-4 agents) → workflow/tmp/scout-results.md
f. Execution-planner: Strategic planning
   - Checks test infrastructure (creates Phase 0 if missing)
   - Invokes test-generator skill → generates tests, updates patterns
   - Creates phased execution → workflow/tmp/plan-{task-name}.md
g. Build Phase 1: Parallel independent files
h. Orchestrator validates: git diff (or find -newer) + runs assigned tests
i. Build Phase 2: Sequential dependent files
j. Orchestrator validates: git diff (or find -newer) + runs assigned tests
k. Build Phase 3: Parallel final work
l. Final validation: full test suite + cleanup workflow/tmp/ + cleanup checkpoints
```

### 3. Validation Strategy
**Orchestrator-driven validation (after each builder completes):**
```bash
# Git mode (if git available)
BEFORE_SHA=$(git rev-parse HEAD)
[Builder executes]
git diff $BEFORE_SHA..HEAD --stat       # Orchestrator verifies file changes
npm test tests/feature.test.js          # Orchestrator runs assigned tests
git rev-parse HEAD > .build-checkpoint

# Timestamp mode (if no git)
touch .build-start
[Builder executes]
find . -type f -newer .build-start      # Orchestrator checks changed files
npm test tests/feature.test.js          # Orchestrator runs assigned tests
touch .build-checkpoint
```

**Orchestrator responsibilities:** Verify expected files changed, run tests, enforce pass/fail. Builders cannot self-validate.

Catches: silent failures, wrong files, conflicts, attribution, **broken implementations**.

### 4. Error Handling
- Silent failure → retry once with explicit instructions
- Wrong files → review git/timestamp diff, rollback if needed
- **Test failure → rollback phase (git reset --hard $BEFORE_SHA or restore from timestamp), retry once with test output context**
- **Test infrastructure missing → execution-planner detects early, creates Phase 0 for setup, then invokes test-generator**
- Critical failure → rollback entire workflow (git reset --hard $(cat .workflow-checkpoint) or restore from .workflow-start timestamp)
- Git unavailable → automatic fallback to timestamp-based file tracking via `find -newer`

## Skills

### test-generator Skill
**Purpose:** Generate project-appropriate tests before implementation

**Workflow:**
1. Assumes test infrastructure exists (execution-planner validates first)
2. Load patterns from workflow/tests/{project-type}-patterns.md
3. Generate test files matching project conventions
4. Update pattern library with new learnings
5. Return test specs (< 150 tokens)

**Project Types:** chrome-extension, react, express, python, react-native, CLI, etc.

**Output:** Test files in tests/ + Updated workflow/tests/{type}-patterns.md

**Invoked by:** Execution-planner skill (never directly by orchestrator)

### Codebase-search Skill
**Purpose:** Parallel codebase search without polluting main context

**Workflow:**
1. Launch 2-4 Task agents (haiku) for parallel search domains
2. Each returns text findings (200-400 tokens)
3. Scout consolidates → workflow/tmp/scout-results.md
4. Returns summary (< 100 tokens) to orchestrator

**Output:** workflow/tmp/scout-results.md
```markdown
# Codebase-search Results
## Files to Modify
- path/file.js:120-145 - [change needed]
## Files to Create
- new/path.js - [purpose]
## Patterns Found
- Pattern at file:line - [how to apply]
```

### Execution-planner Skill
**Purpose:** Transform codebase-search results into phased execution plan with test-generator

**Workflow:**
1. Read workflow/tmp/scout-results.md
2. Generate task name from objective (e.g., "add-autosave", "integrate-stripe")
3. **Validate test infrastructure** (check for test framework, config files)
   - If missing → create Phase 0 for test infrastructure setup
   - If exists → proceed to test generation
4. **Invoke test-generator skill** (receives project type from orchestrator, invokes via Skill tool)
   - test-generator loads patterns, generates test files, updates library
   - test-generator returns test specs (< 150 tokens)
5. Analyze dependencies (imports, function calls)
6. Create phased build sequence with test validation per builder
7. Write workflow/tmp/plan-{task-name}.md (ephemeral, cleaned after task)
8. Return summary (< 150 tokens)

**Output:** workflow/tmp/plan-{task-name}.md
```markdown
# Implementation Plan: {task-name}
## Objective: [one line]

## Test Specifications
Generated {N} test files (test-generator skill):
- tests/feature.test.js - Coverage description
Mocking: {APIs, databases, etc.}

## Build Sequence
### Phase 1 (parallel - independent)
- Builder A: src/models/user.js - User model
  - Tests to pass: tests/models/user.test.js
  - Validation: npm test tests/models/user.test.js

- Builder B: src/utils/jwt.js - JWT utilities
  - Tests to pass: tests/utils/jwt.test.js
  - Validation: npm test tests/utils/jwt.test.js

### Phase 2 (sequential - depends Phase 1)
- Builder C: src/api/auth.js (imports user.js, jwt.js)
  - Tests to pass: tests/api/auth.test.js
  - Validation: npm test tests/api/auth.test.js

### Phase 3 (parallel - independent)
- Builder D: docs/auth.md
```

## Memory Management

**`.claude/memory.md`** (< 800 tokens, load at session start):
```markdown
# Project State
## Active Patterns
- Autosave: chrome.storage.local + 30s interval
- Error handling: try-catch-log pattern

## Known Files
- Core: sidepanel.js
- UI: sidepanel.html
```

**Purpose:** Current session context only—what Claude needs to know RIGHT NOW.

**Maintenance:**
- Update after complex tasks to capture new patterns
- When approaching 800 tokens, remove outdated patterns (they're in Git history)
- No archiving—Git commits serve as project history

## File Structure

**Ephemeral (cleaned after task):**
- `workflow/tmp/scout-results.md`
- `workflow/tmp/plan-{task-name}.md`
- `workflow/tmp/context-pool.json` (24h TTL)

**Persistent:**
- `.claude/memory.md` - Current session context only
- `workflow/tests/{project-type}-patterns.md` - Test patterns by project type
- `workflow/docs/` - Project-specific best practices and API documentation

**Checkpoints (during workflow, cleaned after task):**
- `.workflow-checkpoint`
- `.phase-checkpoint`
- `.build-checkpoint`

## Token Budget

| Operation | Tokens |
|-----------|--------|
| Context-pool initialization + git detection | 100 |
| Codebase-search skill invoke + summary | 200 |
| Execution-planner skill invoke + summary | 250 |
| test-generator (nested in execution-planner) | 150 |
| Launch builders (3 phases) | 250 |
| Orchestrator validation (git/timestamp + tests) | 150 |
| **Total** | **~1100** |

**Key optimization:** Codebase-search/execution-planner/test-generator details stay in files, only summaries in main context.

**Note:** execution-planner total is 400 tokens (250 planning + 150 test-generator nested call), but only 250 summary returned to orchestrator.

## Examples

### Complex Task (test-generator Workflow)
```
User: "Add autosave with visual feedback"
→ Initialize context-pool (if missing/stale)
→ Detect git available, use git mode
→ Checkpoint: .workflow-checkpoint created
→ Project type: chrome-extension (from context-pool)
→ 4+ files likely
→ Codebase-search finds: storage API, UI components, state management
→ Execution-planner:
  - Validates test infrastructure exists
  - Invokes test-generator: generates 4 test files (storage, autosave, UI, integration)
  - Creates: workflow/tmp/plan-add-autosave.md with 3 phases
→ Build: 5 parallel/sequential agents, orchestrator validates each with git diff + tests
→ Final validation: full test suite passes
→ Cleanup: workflow/tmp/ files + checkpoints removed
→ Pattern library: workflow/tests/chrome-extension-patterns.md updated
```

### Simple Task
```
User: "Fix typo in README"
→ 1 file
→ Direct edit, no orchestration
```

### External API Task (with test-generator)
```
User: "Integrate Stripe"
→ Initialize context-pool, detect git, checkpoint
→ Project type: express (from context-pool)
→ Skill("researching-features") first
→ Research findings in workflow/tmp/research.md
→ Codebase-search → Execution-planner:
  - Validates test infrastructure exists
  - Invokes test-generator: generates tests mocking Stripe API
  - Creates: workflow/tmp/plan-integrate-stripe.md
→ Build with orchestrator-driven test validation
→ Pattern library: workflow/tests/express-patterns.md updated with Stripe mocking
```

## Anti-Patterns

- ❌ Orchestrating 1-2 file changes (wasteful)
- ❌ Using codebase-search for external APIs (use researching-features)
- ❌ Reading full codebase-search/execution-planner files in main agent (summaries only)
- ❌ Parallel builders on dependent files (conflicts)
- ❌ No error handling for silent failures
- ❌ Keeping workflow/tmp/ files or checkpoints after task
- ❌ **Invoking test-generator skill directly (execution-planner handles this)**
- ❌ **Skipping tests for "simple" features (test-generator mandatory)**
- ❌ **Missing project type in context-pool (test-generator needs this)**
- ❌ **Skipping context-pool initialization (causes workflow failures)**
- ❌ **Assuming git exists without fallback (breaks non-git projects)**
- ❌ **Builders self-validating tests (orchestrator must validate)**
- ❌ **test-generator failing fast without execution-planner checking first (deadlock)**

## Success Indicators

- ✅ Complex tasks auto-trigger orchestration
- ✅ Simple tasks execute directly
- ✅ workflow/tmp/ + checkpoints cleaned between tasks
- ✅ Memory captures patterns (< 800 tokens)
- ✅ **Test patterns accumulate in workflow/tests/**
- ✅ 2-4 parallel agents for complex work
- ✅ **Context-pool initialized before orchestration**
- ✅ **Git detection automatic with fallback to timestamps**
- ✅ **Orchestrator validates all tests (builders don't self-validate)**
- ✅ **execution-planner checks test infrastructure before invoking test-generator**
- ✅ **Tests written before implementation (test-generator)**
- ✅ **Tests pass before phase completion**
- ✅ **Adapts to project type (chrome-extension, react, express, python, etc.)**
- ✅ Phased execution prevents conflicts
- ✅ Rollback available at phase and workflow level