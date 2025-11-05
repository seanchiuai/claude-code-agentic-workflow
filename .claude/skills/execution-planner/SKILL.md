---
name: "execution-planner"
description: "Transform codebase-search findings into phased execution plan. Creates workflow/tmp/plan-{task-name}.md, returns summary. Use after codebase-search completes."
version: "1.0.0"
dependencies: []
allowed-tools: ["Read", "Write", "Skill"]
---

# Execution Planner Skill

Transform codebase-search results into actionable phased execution strategy.

## Purpose
Create implementation plan from codebase-search findings:
- **Input**: workflow/tmp/scout-results.md + user requirements + project type
- **Output**: workflow/tmp/plan-{task-name}.md (phased execution plan)
- **Focus**: Strategic planning, dependencies, phased execution, test-driven development

**NOT for:**
- Searching codebase (codebase-search)
- External API research (researching-features)
- Executing builds (Task tool)

## Workflow

### 1. Load Inputs
Read:
- `workflow/tmp/scout-results.md` - Codebase-search findings
- User requirements from prompt (includes project type)
- `workflow/tmp/context-pool.json` (optional) - Project conventions
- `.claude/memory.md` (optional) - Known patterns

### 2. Generate Task Name
Create descriptive task name from objective:
- Format: lowercase-with-hyphens
- Examples: "add-autosave", "integrate-stripe", "fix-auth-bug"
- Keep concise (2-4 words)

### 3. Validate Test Infrastructure
**CRITICAL:** Check test infrastructure BEFORE invoking test-generator skill.

Check project has test setup:
```
- package.json has test script OR pytest.ini exists OR test config present
- Test framework installed (Jest, Vitest, pytest, etc.)
- Test directory exists (tests/, __tests__/, etc.)
```

**If missing:**
- Create Phase 0 in plan: "Set up test infrastructure"
- Include setup instructions in plan
- Skip test-generator invocation
- Note that tests cannot run until infrastructure ready
- Builders proceed without test validation initially

**If exists:** Proceed to invoke test-generator skill

### 4. Invoke Test Generator Skill (if infrastructure exists)
Generate tests before implementation:
```
Skill("test-generator", prompt="Generate tests for {task-name}.
Project type: {project-type}
Requirements: {user requirements}
Files to implement: {from codebase-search results}")
```

Test-generator skill returns test specifications (< 150 tokens):
- Test files created
- Mocking strategy
- Coverage areas
- Builders must make tests pass

**Note:** test-generator assumes infrastructure exists (we validated in step 3)

### 5. Analyze Dependencies
Identify:
- **Independent files**: Can be built in parallel
- **Dependent files**: Must be built sequentially
- **Critical path**: What must come first

Example:
```
utils/jwt.js → models/user.js → middleware/auth.js → api/routes.js
(foundation)   (uses jwt)       (uses user model)    (uses middleware)
```

### 6. Create Phased Build Sequence
Organize into phases:

**Phase 1 (parallel):** Independent files, no dependencies on each other
**Phase 2 (sequential):** Dependent files, imports from Phase 1
**Phase 3 (parallel):** Independent validation/docs (tests already created by test-generator)

Each builder phase includes:
- **Test validation**: Run tests after implementation
- **Expected tests to pass**: Which test files builder must satisfy

### 7. Write Plan File
Create `workflow/tmp/plan-{task-name}.md` (ephemeral, cleaned after task):

```markdown
# Implementation Plan: {Task Name}
## Objective: [one line description]

## Test Specifications
{From test-generator skill - which tests created, what they cover}

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
- Builder D: docs/auth.md - Documentation
- Builder E: Update integration tests (if needed)

## File Ownership
- Phase 1: No conflicts (independent modules)
- Phase 2: Builder C doesn't overlap with others
- Phase 3: No conflicts (separate domains)

## Validation Strategy
- Per-builder: Git diff + run assigned tests
- Phase complete: All tests in phase pass
- Final: Full test suite (npm test)

## Success Criteria
- [ ] All phases complete without conflicts
- [ ] All tests pass (test-generator + existing)
- [ ] No regressions in existing features
```

Task name format: `lowercase-with-hyphens` (e.g., `add-autosave`)

### 8. Return Summary
Return brief summary (< 150 tokens) to orchestrator:
```
Plan created: {task-name}, 3 phases, 5 builders (2 parallel → 1 sequential → 2 parallel).
Tests: {N} test files created by test-generator skill.
Phase 1: Foundation (jwt, models). Phase 2: Integration (auth, routes). Phase 3: Docs.
Details in workflow/tmp/plan-{task-name}.md.
```

## Output Format

**Single file:** `workflow/tmp/plan-{task-name}.md` (1500-3000 tokens, ephemeral)
**Return message:** Summary (< 150 tokens) to orchestrator

## Plan Structure

### Must Include

**Build Sequence:**
- Phased execution (which phase, parallel or sequential)
- Builder assignments (who builds what)
- File dependencies (what imports what)

**File Ownership:**
- Which builder owns which files
- Conflict prevention (no two builders edit same file)

**Success Criteria:**
- Measurable validation steps
- How to verify completion

### Optional Sections

- Implementation notes (patterns to follow)
- Risk mitigation (potential issues)
- Dependencies (packages to install)
- Test infrastructure setup (if test-generator skill failed)

### Required for Test-Driven Development

- **Test Specifications**: Summary from test-generator skill
- **Per-builder test validation**: Which tests each builder must pass
- **Validation commands**: Test runner commands per builder

## Example Execution

**Input:**
```
Create plan from codebase-search results for authentication feature.
Project type: express
Requirements: JWT auth for API.
Codebase-search results: workflow/tmp/scout-results.md
```

**Process:**
1. Read workflow/tmp/scout-results.md
2. Generate task name: "add-jwt-auth"
3. Validate test infrastructure: jest.config.js exists, test script present ✓
4. Invoke test-generator skill: Skill("test-generator", project=express, files=[jwt.js, user.js, auth.js, routes.js])
5. Receive test specs: "Generated 4 test files (jwt, user, auth, routes). Mocking database + HTTP."
6. Analyze dependencies: jwt utils → user model → middleware → routes
7. Create 3 phases with test validation per builder
8. Write workflow/tmp/plan-add-jwt-auth.md
9. Return summary

**Output (plan file):**
```markdown
# Implementation Plan: add-jwt-auth

## Objective
Add JWT-based authentication to API with protected routes

## Test Specifications
Generated 4 test files (test-generator skill):
- tests/utils/jwt.test.js - Token generation/verification
- tests/models/user.test.js - Password hashing, auth methods
- tests/middleware/auth.test.js - JWT middleware validation
- tests/api/routes.test.js - Auth endpoints, protected routes
Mocking: Database (Mongoose) + HTTP (supertest)

## Build Sequence

### Phase 1 (parallel - independent)
- Builder A: utils/jwt.js - JWT utilities (generate/verify tokens)
  - Tests to pass: tests/utils/jwt.test.js
  - Validation: npm test tests/utils/jwt.test.js

- Builder B: models/user.js - Add password hashing, auth methods
  - Tests to pass: tests/models/user.test.js
  - Validation: npm test tests/models/user.test.js

### Phase 2 (sequential - depends Phase 1)
- Builder C: middleware/auth.js - JWT verification middleware (imports jwt.js, user.js)
  - Tests to pass: tests/middleware/auth.test.js
  - Validation: npm test tests/middleware/auth.test.js

- Builder D: api/routes.js - Add auth routes, protect endpoints (imports middleware/auth.js)
  - Tests to pass: tests/api/routes.test.js
  - Validation: npm test tests/api/routes.test.js

### Phase 3 (parallel - independent)
- Builder E: docs/api.md - Update API docs

## File Ownership
- Phase 1: A owns jwt.js, B owns user.js (no conflicts)
- Phase 2: C owns middleware, D owns routes (no overlap)
- Phase 3: E owns docs (separate file)

## Validation Strategy
- Per-builder: Git diff + run assigned test file
- Phase complete: All tests in phase pass
- Final: Full test suite (npm test)

## Success Criteria
- [ ] All builders complete without file conflicts
- [ ] All tests pass (4 test-generator + existing suite)
- [ ] No breaking changes to existing API
- [ ] JWT tokens work for protected routes

## Implementation Notes
- Follow Express middleware pattern at middleware/logger.js:15
- Use bcrypt for password hashing (see models/session.js:45)
- Error format: { error: 'message', code: 'CODE' }
```

**Return summary:**
```
Plan created: add-jwt-auth, 3 phases, 5 builders (2 parallel → 2 sequential → 1).
Tests: 4 test files created (jwt, user, middleware, routes).
Foundation (jwt, user) → Integration (middleware, routes) → Docs.
Details in workflow/tmp/plan-add-jwt-auth.md.
```

## Success Criteria

- ✅ Generated task name from objective
- ✅ **Validated test infrastructure BEFORE invoking test-generator**
- ✅ Invoked test-generator skill (or created Phase 0 for test setup)
- ✅ Created workflow/tmp/plan-{task-name}.md
- ✅ Phased execution defined (parallel/sequential)
- ✅ File ownership prevents conflicts
- ✅ Dependencies explicit
- ✅ Test validation per builder specified
- ✅ Success criteria measurable
- ✅ Returned summary < 150 tokens

## Error Handling

**If codebase-search results missing:**
```
ERROR: Codebase-search results not found at workflow/tmp/scout-results.md
Run codebase-search skill first.
```

**If project type missing:**
Fail and ask orchestrator for explicit project type.

**If test infrastructure missing:**
Create Phase 0 in plan with test setup instructions.
Skip test-generator invocation (can't generate tests without infrastructure).
Note in plan that tests cannot run until infrastructure ready.
Builders proceed without test validation initially.

**If requirements unclear:**
Ask for clarification before proceeding.

## Distinct from Other Skills

| Execution-Planner | NOT Execution-Planner |
|-------------------|----------------------|
| Strategic planning | Codebase search (codebase-search) |
| Phased execution | External API research (researching-features) |
| Dependency analysis | Executing builds (Task tool) |
| File ownership | Creating agents (agent-creating) |
