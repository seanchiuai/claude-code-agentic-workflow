In your responses, sacrifice grammar for concise responses.

## File Structure

All workflow files must use these exact paths:

**Memory & Configuration**
- `.claude/memory.md` - Hot/warm/cold memory (800 tokens max)
- `.claude/skills/codebase-search/SKILL.md` - Codebase-search skill definition
- `.claude/skills/execution-planner/SKILL.md` - Execution-planner skill definition
- `.claude/skills/test-generator/SKILL.md` - Test-Driven Development skill definition
- `.claude/skills/researching-features/SKILL.md` - External API research skill
- `.claude/skills/agent-creating/SKILL.md` - Agent creation skill
- `.claude/skills/skill-creating/SKILL.md` - Skill creation skill

**Test Pattern Library**
- `workflow/tests/{project-type}-patterns.md` - Test patterns by project type (chrome-extension, react, express, python, etc.)

**Ephemeral Files (cleaned after task)**
- `workflow/tmp/context-pool.json` - Shared context cache (400 tokens, 24h TTL)
- `workflow/tmp/scout-results.md` - Consolidated codebase-search findings (800-2000 tokens)
- `workflow/tmp/plan-{task-name}.md` - Implementation plan with phased execution (1500-3000 tokens)

**Validation Checkpoints**
- `.workflow-checkpoint` - Workflow start SHA
- `.phase-checkpoint` - Phase start SHA
- `.build-checkpoint` - Per-builder SHA

## Core Architecture

**Agent hierarchy:**
```
Main Agent (orchestrator following CLAUDE.md)
  ├─> Skill tool → Codebase-search Skill Agent
  │    └─> Task agents (2-4) return text → Scout consolidates to file
  ├─> Skill tool → Execution-planner Skill Agent
  │    ├─> Reads codebase-search file
  │    ├─> Invokes test-generator Skill → generates test files
  │    └─> Creates execution-planner file with test validation
  └─> Task tool → Builder agents (execute from execution-planner, make tests pass)
```

**Key principles:**
- Orchestration logic lives in this file (CLAUDE.md system prompt), not in commands
- Skills are agent-invokable workflows (codebase-search, execution-planner, test-generator, researching-features)
- Tasks are direct sub-agent execution (builders implementing execution-planner phases)
- Commands are human-only shortcuts (/help, /context, etc.)
- Codebase-search/execution-planner/test-generator skills write files, return summaries to keep orchestrator context lean
- **test-generator mandatory**: All feature work requires tests before implementation

## Agent Workflow Patterns

### For Feature Requests and Complex Tasks

When user requests new feature or significant change:

#### 1. Assess Complexity
- **Simple (< 3 files, single domain)**: Direct implementation
- **Medium (3-5 files, known patterns)**: Parallel tasks without full orchestration
- **Complex (3+ files, multiple domains, unknown areas)**: Use orchestrated workflow

#### 2. Check Memory Before Orchestrating
Before invoking codebase-search skill:
- Load `.claude/memory.md` if exists
- Check warm memory for known patterns matching this task
- If pattern exists and recent → skip codebase-search, use pattern directly
- If no pattern or outdated → proceed with codebase-search

#### 3. Orchestrated Workflow (for complex tasks)

**Phase 0: Initialize & Checkpoint**
```
# Initialize context-pool if missing/stale
IF workflow/tmp/context-pool.json missing OR older than 24h:
  Generate from .claude/memory.md + codebase inspection
  Write to workflow/tmp/context-pool.json (~400 tokens)

# Detect git and set validation mode
IF git available:
  git rev-parse HEAD > .workflow-checkpoint
  USE_GIT=true
ELSE:
  touch .workflow-start
  USE_GIT=false

# Determine project type from context-pool
Determine project type from workflow/tmp/context-pool.json
(e.g., "chrome-extension", "react", "express", "python")
```

**Phase 1: Codebase-search (parallel search)**
```
Skill("codebase-search", prompt="Find [target] in codebase. Context: workflow/tmp/context-pool.json")
↓
Codebase-search skill launches 2-4 parallel Task agents
Each returns text findings to codebase-search skill
Scout consolidates → workflow/tmp/scout-results.md
Codebase-search returns brief summary (100 tokens) to orchestrator
```

**Phase 2: Execution-planner (strategic planning with test-generator)**
```
Skill("execution-planner", prompt="Create execution-planner from codebase-search results. Project: {type}. Requirements: [user request]")
↓
Reads: workflow/tmp/scout-results.md
Generates task name from objective (e.g., "add-autosave", "integrate-stripe")
**Validates test infrastructure** (checks for test framework, config, test directory)
  ↓
  IF missing: Create Phase 0 for test setup, skip test-generator invocation
  IF exists: Invoke test-generator skill
Invokes: Skill("test-generator", prompt="Generate tests. Project: {type}. Files: [from codebase-search]")
  ↓
  test-generator assumes infrastructure exists (execution-planner validated first)
  test-generator loads patterns from workflow/tests/{type}-patterns.md
  test-generator generates test files matching project conventions
  test-generator updates pattern library with new learnings
  test-generator returns: Test specs (150 tokens) - files created, mocking, coverage
Creates phased execution execution-planner with test validation per builder
Outputs: workflow/tmp/plan-{task-name}.md
Returns: Execution-planner summary (150 tokens) to orchestrator
```

**Phase 3: Execute Builds (phased with orchestrator-driven validation)**
```
For each phase in execution-planner:
  a. Checkpoint:
     IF USE_GIT: git rev-parse HEAD > .phase-checkpoint
     ELSE: touch .phase-start

  b. Launch builders (parallel if independent, sequential if dependent)

  c. **Orchestrator-driven validation (after each builder completes):**
     IF USE_GIT:
       - BEFORE_SHA=$(git rev-parse HEAD)
       - [Builder executes]
       - git diff $BEFORE_SHA..HEAD --stat (verify expected files changed)
       - Orchestrator runs assigned tests (npm test {test-file}, pytest {test-file}, etc.)
     ELSE:
       - touch .build-start
       - [Builder executes]
       - find . -type f -newer .build-start (verify expected files changed)
       - Orchestrator runs assigned tests
     - Tests must pass for orchestrator to mark builder complete

  d. Phase validation: All tests in phase pass
  e. On test failure:
     IF USE_GIT: git reset --hard .phase-checkpoint
     ELSE: restore from timestamp
     Retry once with test output context
  f. On critical failure: Rollback entire workflow
```

**Phase 4: Finalize & Update Memory**
```
1. Final validation:
   IF USE_GIT:
     - git diff .workflow-checkpoint..HEAD --stat
   ELSE:
     - find . -type f -newer .workflow-start
   - Run full test suite (npm test, pytest, etc.)
2. Update .claude/memory.md hot section with task
3. Extract patterns to warm section
4. Test patterns already updated in workflow/tests/ (by test-generator skill)
5. Clean up:
   - workflow/tmp/ (scout-results.md, plan-{task-name}.md)
   - Checkpoint files (.workflow-checkpoint, .phase-checkpoint, .build-checkpoint, .workflow-start, .phase-start, .build-start)
   - context-pool.json only if > 24h old
```

**Error Handling**
- Silent failures: Detect via git/timestamp diff (no changes after builder reports success)
- Wrong files: Compare actual vs expected from execution-planner
- Test failures: Rollback phase (git reset or restore from timestamp), retry once with test output context
- **Test infrastructure missing: execution-planner detects early, creates Phase 0 for setup, then invokes test-generator**
- Validation failures: Rollback phase, retry once with fixes
- Critical failures: Rollback entire workflow (git reset to .workflow-checkpoint or restore from .workflow-start)
- Git unavailable: Automatic fallback to timestamp-based file tracking

**Total orchestration overhead: ~400-600 tokens** (includes test-generator skill invocation + test generation)

#### 4. Skill Usage Guidelines

**Use `codebase-search` skill when:**
- Task involves 3+ files OR unknown code areas
- Need to find existing implementations/patterns
- Exploring unfamiliar parts of codebase
- NOT for external API research (use researching-features)

**Use `execution-planner` skill when:**
- Codebase-search results need strategic planning
- Multiple builders will work in parallel
- Complex dependencies between steps
- Always use AFTER codebase-search completes

Execution-planner skill automatically:
- Invokes test-generator skill to generate tests before implementation
- Receives project type from orchestrator
- Specifies test validation per builder

Execution-planner must specify:
- Phased execution (Phase 1 parallel, Phase 2 sequential, Phase 3 parallel)
- File ownership per builder (prevents conflicts)
- Dependencies between files (determines sequencing)
- Test validation per builder (which tests must pass)
- Success criteria and validation steps

**Use `test-generator` skill when:**
- NEVER invoke directly - Execution-planner skill handles this
- Execution-planner skill invokes automatically for all feature work (after validating infrastructure exists)
- Generates project-appropriate unit tests
- Adapts to project type (chrome-extension, react, express, python, etc.)
- Assumes test infrastructure exists (execution-planner validates first)

**Use `researching-features` skill when:**
- Integrating external API/service/library
- Need to research which external tool to use
- User asks "what's the best solution for X?"
- NOT for internal codebase exploration

**Use `agent-creating` skill when:**
- User explicitly wants new specialized agent
- Repetitive pattern that needs reusable agent
- Example: code-reviewer, doc-generator, linter

**Use `skill-creating` skill when:**
- User explicitly wants new skill
- Repeatable workflow that multiple agents could use

#### 5. Direct Task Usage (skip orchestration)

When execution-planner is already clear:
```
Task(subagent_type="general-purpose", model="sonnet",
     prompt="Implement X. Files: [specific paths]. Pattern: [specific approach]")
```

Use direct tasks when:
- Simple 1-2 file changes
- Clear implementation path (no exploration needed)
- Pattern already known from memory

#### 6. Model Selection Strategy

- **haiku**: Codebase-search skill (fast search), simple edits, documentation
- **sonnet**: Execution-planner skill (strategic thinking), core implementation, complex logic
- **opus**: Very complex architectural decisions (rare)

#### 7. Builder Coordination & Validation

**Orchestrator-driven validation:**
```bash
# Git mode (if git available)
BEFORE_SHA=$(git rev-parse HEAD)
[Builder executes]
git diff $BEFORE_SHA..HEAD --stat         # Orchestrator verifies file changes
git diff $BEFORE_SHA..HEAD --name-only    # List files
npm test tests/feature.test.js            # Orchestrator runs assigned tests
git rev-parse HEAD > .build-checkpoint

# Timestamp mode (if no git)
touch .build-start
[Builder executes]
find . -type f -newer .build-start        # Orchestrator checks changed files
npm test tests/feature.test.js            # Orchestrator runs assigned tests
touch .build-checkpoint

# Validation checks
- No changes after success = silent failure → retry with explicit instructions
- Unexpected files changed = scope creep → review and adjust
- Expected files missing = incomplete → retry or report failure
- Tests fail = builder incomplete → rollback and retry
```

**Orchestrator responsibilities:** Verify expected files changed, run tests, enforce pass/fail. Builders cannot self-validate.

**Builder completion format:**
```
✅ [Component name] complete
- [Key implementation detail]
- [Integration point or validation result]
```

No build-status-*.md files needed - git diff provides ground truth.

### Memory Management

**Load memory at session start:**
```
IF .claude/memory.md exists:
  Load into context (~800 tokens)
  Check for relevant patterns
```

**Update memory after orchestrated task:**
```
Hot memory: Add task entry (150 tokens)
Warm memory: Extract new patterns or update existing
If > 5 tasks in hot: Archive oldest to .claude/archive/YYYY-MM-DD.md
If total > 1000 tokens: Prune oldest warm patterns
```

**Memory structure:**
- Hot (last 2 tasks): Recent work, 300 tokens max
- Warm (patterns): Reusable solutions, 400 tokens max
- Cold (context): Project structure, 150 tokens max

### Context Pool Management

**Generate when:**
- First orchestrated task in session
- Major project changes (config files modified, restructure)
- Existing pool older than 24 hours

**Contents:**
- **Project type** (chrome-extension, react, express, python, etc.) - CRITICAL for test-generator
- Tech stack, architecture
- Directory structure and conventions
- Known patterns and entry points
- Dependencies and execution environment
- Test infrastructure (framework, runner, config location)

**Usage:**
- Sub-agents (codebase-search, execution-planner, test-generator, builders) load for shared context
- test-generator skill reads project type to adapt test generation
- Prevents redundant discovery (~800 token savings per task)

### Execution Decision Tree

```
User Request
    ↓
Is it conversational/informational? → Answer directly
    ↓
Is it simple single file change? → Edit directly
    ↓
Is it 2-3 similar files, known pattern? → Parallel tasks, no orchestration
    ↓
Is it 3+ files OR unknown area OR multiple domains?
    ↓
Check memory: Known pattern exists?
    ├─> YES: Use pattern, maybe skip codebase-search
    └─> NO:  Full orchestration (codebase-search → execution-planner → build)
```

### Anti-Patterns to Avoid

- ❌ Orchestrating simple 1-2 file changes (wasteful overhead)
- ❌ Using codebase-search for external API research (use researching-features)
- ❌ Reading full codebase-search-results.md as main agent (read summary from skill only)
- ❌ Reading full execution-planner-{task-name}.md as main agent (read summary from skill only)
- ❌ Keeping workflow/tmp/ files or checkpoints after task completion (clean up)
- ❌ Not updating memory after complex tasks (lost learning)
- ❌ Sequential execution when parallel possible (slower)
- ❌ Creating build-status-*.md files (use git/timestamp diff for validation)
- ❌ Running parallel builders on dependent files (causes conflicts)
- ❌ Using aggregate git diff for parallel builders (can't attribute changes)
- ❌ Execution-planner without file ownership and dependencies (enables conflicts)
- ❌ No error handling for silent failures (builder says success, no changes)
- ❌ Assuming git available (need timestamp-based fallback for non-git projects)
- ❌ Invoking test-generator skill directly from orchestrator (Execution-planner skill handles this)
- ❌ Skipping tests for "simple" features (test-generator mandatory for all features)
- ❌ Not specifying project type in context-pool (test-generator needs this)
- ❌ **Skipping context-pool initialization (causes workflow failures)**
- ❌ **Builders self-validating tests (orchestrator must validate)**
- ❌ **test-generator failing fast without execution-planner checking first (deadlock)**

### Success Indicators

- ✅ Complex tasks trigger orchestration automatically
- ✅ Simple tasks execute directly without overhead
- ✅ workflow/tmp/ + checkpoints cleaned between tasks
- ✅ Memory captures reusable patterns (< 800 tokens)
- ✅ Test patterns accumulate in workflow/tests/ over time
- ✅ 2-4 parallel agents for complex work
- ✅ Main agent context stays lean (< 50% capacity after task)
- ✅ **Context-pool initialized before orchestration**
- ✅ **Git detection automatic with fallback to timestamps**
- ✅ **Orchestrator validates all tests (builders don't self-validate)**
- ✅ **execution-planner checks test infrastructure before invoking test-generator**
- ✅ Phased execution prevents file conflicts
- ✅ Silent failures detected and handled
- ✅ **Tests written before implementation (test-generator)**
- ✅ **Tests pass before phase completion**
- ✅ **Builders adapt to project type (chrome-extension, react, express, etc.)**
- ✅ Rollback available at phase and workflow level

When designing implementation execution-planners, agents, or any markdown files, be concise. Do not repeat yourself. Quality over Quantity.