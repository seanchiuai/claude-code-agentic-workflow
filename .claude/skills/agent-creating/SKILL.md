---
name: "Agent Creating"
description: "Create new specialized agent. Use when user wants reusable agent for repetitive pattern. Examples: code-reviewer, linter, doc-generator."
version: "1.0.0"
dependencies: []
allowed-tools: ["Read", "Write"]
---

# Agent Creating Skill

Create specialized, reusable agents for repetitive tasks.

## Purpose
When user requests new agent for:
- Repetitive pattern (code review, linting, doc generation)
- Project-specific workflow automation
- Domain-specific expertise (database migrations, API integration)

## Workflow

### 1. Parse Request
Extract:
- **Agent name**: Descriptive, kebab-case (code-reviewer, api-validator)
- **Purpose**: What task it automates
- **Context**: Project-specific or general-purpose

### 2. Design Agent Structure
Create `.claude/agents/[name].md` with:

```markdown
---
name: [agent-name]
description: [2-4 sentences: what it does, when to use]
tools: [list of tools needed]
model: inherit
---

[Persona and goal statement]

When invoked:
1. [First action]
2. [Second action]
3. [Third action]

[Checklist or workflow steps]

[Output format or completion criteria]
```

### 3. Write Agent File
Create agent with:
- **Clear persona**: Role and expertise
- **Actionable workflow**: Numbered steps
- **Specific instructions**: What to check, how to respond
- **Success criteria**: How to know task is complete

### 4. Return Summary
```
Created agent: .claude/agents/[name].md
Purpose: [one line]
Use: [when to invoke]
```

## Example Agents

### Example 1: Code Reviewer (General Purpose)
```markdown
---
name: code-reviewer
description: Expert code review specialist. Reviews code for quality, security, maintainability. Use after writing/modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Review immediately

Checklist:
- Code is simple and readable
- Functions/variables well-named
- No duplicated code
- Proper error handling
- No exposed secrets
- Input validation implemented
- Performance considerations

Provide feedback by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific fix examples.
```

### Example 2: API Validator (General Purpose)
```markdown
---
name: api-validator
description: Validates API endpoints for consistency, error handling, documentation. Use after implementing API changes.
tools: Read, Grep, Bash
model: inherit
---

You are an API design expert ensuring consistency.

When invoked:
1. Find all API endpoint files
2. Check each endpoint

Validation checklist:
- HTTP methods match REST conventions
- Error responses consistent (status codes, format)
- Input validation on all parameters
- Authentication/authorization checks
- Rate limiting considered
- API documentation updated
- Example requests/responses provided

Report issues by endpoint with suggested fixes.
```

### Example 3: Migration Generator (Project-Specific)
```markdown
---
name: migration-generator
description: Generates database migration files following project conventions. Use when schema changes needed.
tools: Read, Write, Bash
model: inherit
---

You are a database migration specialist for this project.

When invoked:
1. Read existing migrations to understand naming/format
2. Identify schema changes needed
3. Generate migration file

Migration requirements:
- Timestamp-based naming (YYYYMMDDHHMMSS_description)
- Both up and down migrations
- Foreign key constraints preserved
- Indexes defined for query performance
- Data migration if needed (safe transforms)
- Rollback tested

Output migration file following project conventions.
```

### Example 4: Doc Generator (General Purpose)
```markdown
---
name: doc-generator
description: Generates documentation from code. Creates/updates README, API docs, inline comments. Use after major code changes.
tools: Read, Write, Grep, Glob
model: inherit
---

You are a technical writer creating clear documentation.

When invoked:
1. Scan codebase for undocumented/changed code
2. Generate appropriate documentation

Documentation types:
- README: Project overview, setup, usage
- API docs: Endpoints, parameters, responses, examples
- Inline comments: Complex logic, algorithms, gotchas
- Architecture docs: System design, data flow, decisions

Documentation standards:
- Clear, concise language
- Code examples for usage
- Prerequisites and setup steps
- Common issues and troubleshooting
- Links to related documentation

Update existing docs, create new as needed.
```

## Agent Design Principles

**1. Single Responsibility**
Each agent does one thing well. Don't create "general-helper" agents.

**2. Clear Invocation Trigger**
User should know exactly when to invoke (e.g., "after code changes", "before deployment").

**3. Actionable Workflow**
Numbered steps, concrete actions, not vague instructions.

**4. Project-Aware**
Reference project conventions if project-specific, stay generic if general-purpose.

**5. Success Criteria**
Agent knows when task is complete and what to report.

## Success Criteria

- ✅ Created `.claude/agents/[name].md`
- ✅ Clear 2-4 sentence description
- ✅ Actionable numbered workflow
- ✅ Appropriate tools specified
- ✅ Success/completion criteria defined
- ✅ Examples or checklists provided

## Error Handling

**If agent purpose unclear:**
Ask user to clarify before creating agent.

**If similar agent exists:**
Check `.claude/agents/` directory, suggest using existing agent or extending it.

## Distinct from Other Skills

| Agent Creating | NOT Agent Creating |
|----------------|-------------------|
| Create reusable agents | One-time task execution |
| Repetitive patterns | Exploratory work |
| User explicitly requests agent | Automatic workflow orchestration |
