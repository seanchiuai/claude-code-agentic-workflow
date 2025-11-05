# 🤖 Agent Orchestration Workflow

Multi-agent system for complex coding tasks via Claude Code. Parallel execution. Test-driven. Memory-efficient.

---

## 📑 Quick Navigation

- [Overview](#-what-is-this)
- [Architecture](#️-architecture)
- [Core Workflow](#-core-orchestrated-workflow)
- [When to Use](#-when-to-use-what)
- [File Structure](#-file-structure)
- [Skills Deep Dive](#-skills-deep-dive)
- [Memory Management](#-memory-management)
- [Anti-Patterns](#-anti-patterns-to-avoid)
- [Quick Start](#-quick-start)
- [Validation Strategy](#-validation-strategy)

---

## ⚡ What Is This?

Orchestration system embedded in CLAUDE.md. Auto-triggers on task complexity. Coordinates multiple specialized agents for parallel work.

### 🔧 Skills
Agent-invokable workflows. Codebase-search, execution-planner, test-generator. Return summaries, write files.

### ⚙️ Tasks
Direct sub-agent execution. Builders implement plan phases. Make tests pass. Report completion.

### 🧠 Memory
Hot/warm/cold system. 800 tokens max. Recent tasks + patterns + project context. Auto-decay.

### ✅ Validation
Orchestrator-driven. Git diff + test execution. Per-builder checkpoints. Rollback on failure.

---

## 🏗️ Architecture

**Key Principle:** Orchestration in CLAUDE.md, triggered by complexity. Skills write files + return summaries. Main context stays lean.

### Agent Hierarchy

```
Main Agent (orchestrator)
  ├─> Skill("codebase-search")
  │    └─> Task agents (2-4 parallel) → Scout consolidates
  ├─> Skill("execution-planner")
  │    ├─> Reads scout results
  │    ├─> Skill("test-generator") → Generates tests
  │    └─> Creates phased plan
  └─> Task(builders) → Execute plan, make tests pass
```

### Component Roles

- **Skills** (via Skill tool): codebase-search, execution-planner, test-generator, researching-features
- **Tasks** (via Task tool): Builders executing plan phases
- **Commands** (human-only): Slash commands like /help
- **test-generator**: Mandatory. Tests before implementation.

---

## 🔄 Core Orchestrated Workflow

### Complexity Assessment

- **Simple (< 3 files)** → Direct implementation
- **Complex (3+ files OR unknown areas)** → Full orchestration

### Workflow Phases

1. **Initialize** - Context-pool + Git detection + Checkpoint
2. **Codebase-Search** - 2-4 parallel agents → scout-results.md
3. **Execution-Planner** - Validate tests → Generate tests → Create plan
4. **Execute Builds** - Phased builders → Orchestrator validates

---

### Phase 0: Initialize & Checkpoint

```bash
# Initialize context-pool if missing/stale
IF workflow/tmp/context-pool.json missing OR older than 24h:
  Generate from .claude/memory.md + codebase
  Write to workflow/tmp/context-pool.json

# Detect git and set validation mode
IF git available:
  git rev-parse HEAD > .workflow-checkpoint
  USE_GIT=true
ELSE:
  touch .workflow-start
  USE_GIT=false

# Determine project type
Project: chrome-extension, react, express, python, etc.
```

---

### Phase 1: Codebase-Search (Parallel)

- Launch 2-4 Task agents in parallel
- Each searches different domain (components, services, tests, docs)
- Each returns 200-400 tokens text
- Scout consolidates → `workflow/tmp/scout-results.md`
- Returns <100 token summary to orchestrator

**Output:** Files to modify, files to create, patterns found, dependencies

---

### Phase 2: Execution-Planner (Strategic)

1. Read scout-results.md
2. Generate task name (e.g., "add-autosave")
3. **CRITICAL** Validate test infrastructure EXISTS
   - If missing → Create Phase 0 for setup, skip test-generator
   - If exists → Invoke test-generator skill
4. Invoke test-generator (generates tests BEFORE implementation)
5. Analyze dependencies (what depends on what)
6. Create phased build sequence with test validation
7. Write `workflow/tmp/plan-{task-name}.md`
8. Return <150 token summary

---

### Phase 3: Execute Builds (Phased with Validation)

```bash
For each phase in plan:
  a. Checkpoint (git or timestamp)

  b. Launch builders (parallel if independent)

  c. # Orchestrator validates each builder
     IF USE_GIT:
       git diff $BEFORE_SHA..HEAD --stat
       Orchestrator runs assigned tests
     ELSE:
       find . -newer .build-start
       Orchestrator runs assigned tests

     Tests MUST pass for builder complete

  d. Phase validation: All tests pass

  e. On failure: Rollback phase, retry once
```

⚠️ **Orchestrator Responsibility:** Builders CANNOT self-validate. Orchestrator runs all tests.

---

### Phase 4: Finalize & Update Memory

1. Final validation (git diff + full test suite)
2. Update .claude/memory.md hot section
3. Extract patterns to warm section
4. Test patterns updated in workflow/tests/ (by test-generator)
5. Clean up:
   - workflow/tmp/ (scout-results, plan files)
   - Checkpoints (.workflow-checkpoint, .phase-checkpoint, etc.)
   - context-pool.json only if >24h old

---

### Token Budget

| Operation | Tokens |
|-----------|--------|
| Context-pool init + git detection | 100 |
| Codebase-search invoke + summary | 200 |
| Execution-planner invoke + summary | 250 |
| test-generator (nested) | 150 |
| Launch builders (3 phases) | 250 |
| Orchestrator validation | 150 |
| **Total** | **~1100** |

---

## 🤔 When to Use What?

### Decision Tree

```
User Request arrives
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

### Skill Usage Guidelines

- **codebase-search:** 3+ files OR unknown areas. Find patterns. NOT for external APIs.
- **execution-planner:** After codebase-search. Strategic planning. Phased execution.
- **test-generator:** NEVER invoke directly. execution-planner handles it. Mandatory for features.
- **researching-features:** External API/service research. NOT internal codebase.
- **agent-creating:** User wants new reusable agent (code-reviewer, linter).

---

## 📁 File Structure

```
.claude/
├─ agents/              # User-created persistent agents
├─ skills/              # Core workflow skills
│  ├─ codebase-search/
│  ├─ execution-planner/
│  ├─ test-generator/
│  ├─ researching-features/
│  ├─ agent-creating/
│  └─ skill-creating/
├─ commands/            # Slash commands (human-only)
├─ archive/             # Archived memory (after 5 tasks)
└─ memory.md            # Hot/warm/cold memory (<800 tokens)

workflow/
├─ tests/               # Test pattern library
│  ├─ chrome-extension-patterns.md
│  ├─ react-patterns.md
│  ├─ express-patterns.md
│  └─ python-patterns.md
├─ docs/                # Project-specific documentation
│  ├─ README.md         # Documentation guide
│  ├─ best-practices.md # Coding standards & patterns
│  ├─ api-reference.md  # Internal API documentation
│  └─ {domain}-guide.md # Domain-specific guides
└─ tmp/                 # Ephemeral files (cleaned after task)
   ├─ context-pool.json # 24h TTL, project context
   ├─ scout-results.md  # Codebase-search findings
   └─ plan-{task}.md    # Implementation plan

Checkpoints             # (cleaned after task)
.workflow-checkpoint    # Git: workflow start SHA
.phase-checkpoint       # Git: phase start SHA
.build-checkpoint       # Git: builder SHA
.workflow-start         # Timestamp: workflow start
.phase-start            # Timestamp: phase start
.build-start            # Timestamp: builder start
```

---

## 🔍 Skills Deep Dive

### 🔧 codebase-search Skill

**Purpose:** Parallel codebase search without polluting main context

**Model:** haiku (fast)

**Workflow:**
1. Parse request (target, focus areas)
2. Launch 2-4 Task agents in parallel (components, services, tests, docs)
3. Consolidate findings
4. Write workflow/tmp/scout-results.md (800-2000 tokens)
5. Return <100 token summary

**Output:** Files to modify, files to create, patterns, dependencies

---

### 📋 execution-planner Skill

**Purpose:** Transform scout results into phased execution plan

**Model:** sonnet (strategic)

**Workflow:**
1. Read scout-results.md
2. Generate task name (lowercase-with-hyphens)
3. **CRITICAL** Validate test infrastructure
   - Check package.json test script OR pytest.ini OR test config
   - If missing → Create Phase 0, skip test-generator
4. Invoke test-generator skill (if infra exists)
5. Analyze dependencies
6. Create phased sequence (Phase 1 parallel → Phase 2 sequential → Phase 3 parallel)
7. Write workflow/tmp/plan-{task}.md
8. Return <150 token summary

**Prevents Deadlock:** Checks infra BEFORE invoking test-generator. No fail-fast loop.

---

### 🧪 test-generator Skill

**Purpose:** Generate project-appropriate tests BEFORE implementation

**Model:** sonnet

**Invoked by:** execution-planner skill (NEVER directly)

**Workflow:**
1. Load patterns from workflow/tests/{project-type}-patterns.md
2. Parse requirements (files to test, functionality, mocks needed)
3. Generate test files matching project conventions
4. Update pattern library with new learnings
5. Return test specs (<150 tokens)

**Adapts to:** chrome-extension, react, express, python, react-native, CLI

**Assumes infrastructure exists:** execution-planner validated first. No fail-fast.

---

### 🔬 researching-features Skill

**Purpose:** Research external APIs/services/libraries

**NOT for:** Internal codebase exploration (use codebase-search)

**Use when:** "Integrate Stripe", "What's best solution for X?", external tool research

---

## 🧠 Memory Management

**.claude/memory.md** - Hot/warm/cold system. 800 tokens max. Auto-decay.

### Structure

| Section | Content | Tokens |
|---------|---------|--------|
| **Hot** | Last 2 tasks. Recent work. | 300 max |
| **Warm** | Patterns. Reusable solutions. | 350 max |
| **Cold** | Project structure. Stable info. | 150 max |

### Auto-Decay Rules

- After 5 tasks → Oldest hot task moves to .claude/archive/
- If total >800 tokens → Prune oldest warm patterns
- Cold updates only on major structural changes

### Context-Pool

`workflow/tmp/context-pool.json` - Generated from memory + codebase. 24h TTL. ~400 tokens.

**Contains:** Project type (CRITICAL for test-generator), tech stack, directory structure, test infrastructure, workflow/docs references

---

## ❌ Anti-Patterns to Avoid

### Critical Failures

- ❌ **Skipping context-pool initialization** → Workflow fails
- ❌ **Assuming git exists without fallback** → Breaks non-git projects
- ❌ **Builders self-validating tests** → Orchestrator MUST validate
- ❌ **test-generator failing fast without execution-planner check** → Deadlock
- ❌ **Invoking test-generator directly** → execution-planner handles this

### Performance Issues

- ❌ Orchestrating 1-2 file changes (wasteful overhead)
- ❌ Using codebase-search for external APIs (use researching-features)
- ❌ Reading full scout/plan files in main agent (summaries only)
- ❌ Keeping workflow/tmp/ files after task (clean up)
- ❌ Sequential execution when parallel possible (slower)
- ❌ Parallel builders on dependent files (conflicts)

### Best Practices

- ✅ Complex tasks auto-trigger orchestration
- ✅ Simple tasks execute directly
- ✅ workflow/tmp/ + checkpoints cleaned between tasks
- ✅ Memory captures patterns (<800 tokens)
- ✅ Context-pool initialized before orchestration
- ✅ Git detection automatic with timestamp fallback
- ✅ Orchestrator validates all tests
- ✅ execution-planner checks infra before test-generator
- ✅ Tests written before implementation
- ✅ Phased execution prevents conflicts

---

## 🚀 Quick Start

**1. Clone/Setup Project** → **2. Initialize Project** → **3. Request Feature** → **4. Orchestration Auto-Triggers**

### Step 1: Setup

```bash
# Use this repo as template
git clone [your-repo-url]
cd [project]

# Verify structure
ls .claude/        # memory.md, skills/, agents/
ls workflow/       # tests/, tmp/
```

### Step 2: Initialize Project

**Option A: Automated (Recommended)**

Use `/init` command with your PRD:
```bash
/init path/to/product-requirements.md
```

This will:
- **Analyze entire repository structure** (dirs, files, configs, dependencies, code patterns)
- Analyze PRD and extract tech stack, features, constraints
- **Cross-reference PRD with actual codebase** (flag discrepancies)
- Populate `workflow/docs/` with API docs and best practices (from both PRD and real code)
- Clean `workflow/tests/` (remove irrelevant, add project-specific patterns, seed with existing tests)
- Update `memory.md` with project context (from both repo and PRD)
- Propose core features for your approval (distinguish new vs enhancements)
- Initialize orchestration structure (context-pool.json with actual paths and configs)
- **Generate STATUS-REPORT.html** automatically with full project overview

**Option B: Manual**

Populate `.claude/memory.md` with:
- Project type (chrome-extension, react, express, etc.)
- Tech stack
- Key files
- Directory structure

### Step 3: Request Feature

```bash
# Simple (1-2 files) - direct execution
"Fix typo in README"

# Complex (3+ files) - orchestration triggers
"Add user authentication with JWT"
"Integrate Stripe payments"
"Add autosave with visual feedback"
```

### Step 4: Orchestration Auto-Triggers

CLAUDE.md logic automatically:
1. Assesses complexity
2. Initializes context-pool + git detection
3. Invokes codebase-search skill
4. Invokes execution-planner skill
   - Validates test infrastructure
   - Invokes test-generator (if infra exists)
5. Launches phased builders
6. Orchestrator validates each builder with tests
7. Updates memory
8. Cleans up tmp/ and checkpoints

🎉 **Done!** Tests pass. Memory updated. Pattern library grows. Next task uses learned patterns.

---

## ✅ Validation Strategy

### Orchestrator-Driven Validation

```bash
# Git mode (if git available)
BEFORE_SHA=$(git rev-parse HEAD)
[Builder executes]
git diff $BEFORE_SHA..HEAD --stat         # Verify files
npm test tests/feature.test.js            # Run tests

# Timestamp mode (no git)
touch .build-start
[Builder executes]
find . -newer .build-start                # Verify files
npm test tests/feature.test.js            # Run tests
```

**Critical:** Builders CANNOT self-validate. Orchestrator owns test execution.

### Error Handling

- **Silent failure:** No changes after builder success → Retry with explicit instructions
- **Wrong files:** Compare actual vs expected from plan
- **Test failure:** Rollback phase (git reset or restore), retry once with test output
- **Test infra missing:** execution-planner creates Phase 0, then invokes test-generator
- **Critical failure:** Rollback entire workflow to .workflow-checkpoint

---

## 🎯 Remember

**Quality over quantity.** Concise execution-planners. Test-driven. Memory-efficient. Auto-orchestrated complexity.

📚 Read AGENT-WORKFLOW.md for full details
🔧 Skills in .claude/skills/
🧠 Memory in .claude/memory.md
