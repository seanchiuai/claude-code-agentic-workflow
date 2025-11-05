---
name: "codebase-search"
description: "Parallel internal codebase search. Launches 2-4 search agents, consolidates findings to tmp/scout-results.md, returns summary. Use for 3+ files or unknown areas."
version: "1.0.0"
dependencies: []
allowed-tools: ["Task", "Read", "Write"]
---

# Codebase Search Skill

Coordinate parallel codebase search and consolidate findings.

## Purpose
Launch 2-4 parallel Task agents to search **internal codebase** (not external APIs) for:
- Files to modify
- Files to create
- Patterns to follow
- Dependencies

**NOT for external API research** - use `researching-features` skill.

## Workflow

### 1. Parse Request
Extract:
- **Target**: What to find
- **Focus areas**: Optional directories/patterns
- **Context**: Load `workflow/tmp/context-pool.json` if exists

### 2. Launch Parallel Search Agents (2-4 Task agents)
Launch Task agents in parallel using Task tool, each searching different domain:

**Agent 1: Component/Feature Search**
```
Task(subagent_type="general-purpose",
     prompt="Search for [target] in component/feature files.
     Find: files to modify, patterns used, dependencies.
     Return findings as text (200-400 tokens).")
```

**Agent 2: Logic/API/Service Search**
```
Task(subagent_type="general-purpose",
     prompt="Search for [target] in api/services/handlers.
     Find: files to modify, patterns used, dependencies.
     Return findings as text (200-400 tokens).")
```

**Agent 3: Config/Test/Type Search**
```
Task(subagent_type="general-purpose",
     prompt="Search for [target] in config/test/type files.
     Find: patterns, test structure, type definitions.
     Return findings as text (200-400 tokens).")
```

**Agent 4: Documentation/Examples** (optional, if needed)
```
Task(subagent_type="general-purpose",
     prompt="Search for [target] documentation, examples, comments.
     Find: usage patterns, conventions.
     Return findings as text (200-400 tokens).")
```

### 3. Consolidate Results
Wait for all agents to return. Each agent returns text findings (200-400 tokens).

### 4. Write Consolidated Report
Create `workflow/tmp/scout-results.md`:

```markdown
# Scout Results
*Generated: [timestamp]*

## Files to Modify
- path/file.js:120-145 - [what needs changing]
- path/other.py:45-67 - [what needs changing]

## Files to Create
- new/path.js - [purpose]
- new/util.py - [purpose]

## Patterns Found
- Pattern at file:line - [how to apply]
- Convention in directory/ - [description]

## Dependencies
- Existing: package/module names
- Required: new packages needed

## Implementation Notes
[Key findings from all agents consolidated]
```

### 5. Return Summary
Return brief summary (< 100 tokens) to orchestrator:
```
Found [N] modify, [M] create. Key pattern: [pattern name] at [file:line].
Details in workflow/tmp/scout-results.md.
```

## Output Format

**Single file:** `workflow/tmp/scout-results.md` (800-2000 tokens)
**Return message:** Summary (< 100 tokens) to orchestrator

## Example Execution

**Input:**
```
Find authentication implementation. Context: Express API, MongoDB.
```

**Process:**
1. Launch 4 parallel Task agents (components, services, tests, docs)
2. Agent 1 returns: "Found auth components in api/routes.js:45-89..."
3. Agent 2 returns: "User model at models/user.js:12-34, needs password hash..."
4. Agent 3 returns: "Test pattern at tests/api.test.js uses Jest + supertest..."
5. Agent 4 returns: "No auth docs found, README.md has basic setup..."
6. Consolidate all findings → workflow/tmp/scout-results.md
7. Return: "Found 2 modify (routes, user model), 3 create (middleware, jwt util, tests). Express middleware pattern at middleware/logger.js:15. Details in workflow/tmp/scout-results.md."

## Success Criteria

- ✅ Launched 2-4 Task agents in parallel
- ✅ Each agent returns text (200-400 tokens)
- ✅ Consolidated to workflow/tmp/scout-results.md (800-2000 tokens)
- ✅ Returned summary < 100 tokens to orchestrator
- ✅ File paths verified or marked NEW
- ✅ Patterns include specific file:line references

## Error Handling

**If no files found:**
Return summary: "No existing implementation. Greenfield - recommend standard structure. Details in workflow/tmp/scout-results.md."
Still create scout-results.md with recommendations.

**If ambiguous request:**
Ask for clarification before launching agents.

## Distinct from Other Skills

| Codebase-Search | NOT Codebase-Search |
|-----------------|---------------------|
| Internal codebase search | External API research (researching-features) |
| Find existing patterns | Create implementation plan (execution-planner) |
| Launch search agents | Execute builds (Task tool direct) |
| Output consolidated file | Execute code changes |
