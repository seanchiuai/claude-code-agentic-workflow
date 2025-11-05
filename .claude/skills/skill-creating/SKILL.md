---
name: "Skill Creating"
description: "Create new skill for repeatable workflow. Use when user wants reusable skill pattern. Creates .claude/skills/[name]/SKILL.md."
version: "1.0.0"
dependencies: []
allowed-tools: ["Read", "Write", "Bash"]
---

# Skill Creating Skill

Create new skills for repeatable workflows that multiple agents can use.

## Purpose
When user requests new skill for:
- Repeatable workflow pattern (multi-step process)
- Cross-agent functionality (multiple agents invoke same pattern)
- Standardized procedures (code formatting, deployment, migration)

**Difference from agents:**
- **Skills**: Reusable workflows that ANY agent can invoke via Skill tool
- **Agents**: Specialized personas for specific repetitive tasks

## Workflow

### 1. Parse Request
Extract:
- **Skill name**: Gerund form, kebab-case (generating-commits, validating-migrations)
- **Purpose**: What workflow it automates
- **Invocation trigger**: When agents should use it

### 2. Design Skill Structure
Create `.claude/skills/[name]/SKILL.md` with frontmatter + instructions:

```markdown
---
name: "[Skill Name]"
description: "[2-4 sentences: what it does, when to use, what it returns]"
version: "1.0.0"
dependencies: []
allowed-tools: ["Tool1", "Tool2"]
model: "haiku" or "sonnet"
---

# [Skill Name]

[Brief purpose statement]

## Workflow

### 1. [First Step]
[Instructions]

### 2. [Second Step]
[Instructions]

### 3. [Third Step]
[Instructions]

## Output Format
[What the skill returns to invoking agent]

## Success Criteria
- ✅ [Check 1]
- ✅ [Check 2]
```

### 3. Create Skill Directory
```bash
mkdir -p .claude/skills/[name]
# Create SKILL.md
# Create resources/ subdirectory if needed for templates/examples
```

### 4. Write Skill File
Include:
- **Clear invocation trigger**: When should agents use this?
- **Numbered workflow**: Step-by-step instructions
- **Output format**: What does skill return?
- **Success criteria**: How to verify completion?
- **Error handling**: What to do if steps fail?

### 5. Return Summary
```
Created skill: .claude/skills/[name]/SKILL.md
Purpose: [one line]
Invoke: Skill("[name]", prompt="[context]")
```

## Skill Design Principles

**1. Atomic Workflow**
Each skill performs one clear workflow, not multiple unrelated tasks.

**2. Agent-Invokable**
Skills are called by agents via Skill tool, not by users directly.

**3. Reusable**
Multiple agents in different contexts should be able to use the skill.

**4. Output-Focused**
Skills return structured output (files, summaries) to invoking agent.

**5. Stateless**
Each invocation is independent, no persistent state between calls.

## Example Skills

### Example 1: Generating Commit Messages
```markdown
---
name: "Generating Commit Messages"
description: "Generates conventional commit messages from git diffs. Use when creating commits. Returns formatted message."
version: "1.0.0"
dependencies: []
allowed-tools: ["Bash", "Read"]
model: "haiku"
---

# Generating Commit Messages

Generate conventional commit messages following project conventions.

## Workflow

### 1. Get Staged Changes
```bash
git diff --staged
```

### 2. Analyze Changes
Identify:
- Files modified
- Nature of changes (feat, fix, refactor, docs, etc.)
- Scope (component/module affected)
- Breaking changes

### 3. Generate Message
Format:
```
<type>(<scope>): <summary>

<body>

<footer>
```

Types: feat, fix, docs, style, refactor, test, chore

### 4. Return Message
Return formatted message to invoking agent.

## Output Format
```
feat(auth): add JWT token validation

Implement JWT validation middleware with expiry checks.
Includes refresh token handling and error responses.

Breaking change: Auth header now required for protected routes
```

## Success Criteria
- ✅ Summary under 50 characters
- ✅ Type matches conventional commit types
- ✅ Body explains what and why
- ✅ Breaking changes noted in footer if applicable
```

### Example 2: Validating Migrations
```markdown
---
name: "Validating Migrations"
description: "Validates database migration files for safety and conventions. Use before committing migrations. Returns validation report."
version: "1.0.0"
dependencies: []
allowed-tools: ["Read", "Grep", "Bash"]
model: "sonnet"
---

# Validating Migrations

Validate database migration files for safety, reversibility, conventions.

## Workflow

### 1. Find Migration Files
Locate new/modified migration files:
```bash
git diff --name-only | grep migrations/
```

### 2. Check Migration Safety
For each file, verify:
- **Naming convention**: Timestamp prefix (YYYYMMDDHHMMSS_description)
- **Up migration exists**: Forward migration defined
- **Down migration exists**: Rollback migration defined
- **No data loss**: DROP/DELETE operations commented or justified
- **Indexes defined**: Performance considerations
- **Foreign keys**: Referential integrity preserved

### 3. Test Migrations (if possible)
```bash
# Dry run if test DB available
npm run migrate:dry-run
```

### 4. Return Validation Report
Report format:
```markdown
## Migration Validation Report

### migration_20250104_143022_add_auth.sql
- ✅ Naming convention
- ✅ Up migration
- ✅ Down migration
- ⚠️  DROP COLUMN without backup - RISKY
- ✅ Indexes defined
- ✅ Foreign keys preserved

Recommendation: Add backup step before DROP COLUMN
```

## Output Format
Return validation report with:
- ✅ Pass
- ⚠️  Warning
- ❌ Fail

## Success Criteria
- ✅ All migrations checked
- ✅ Naming conventions validated
- ✅ Reversibility verified
- ✅ Risky operations flagged
```

### Example 3: Formatting Code
```markdown
---
name: "Formatting Code"
description: "Auto-formats code using project formatters. Use before commits. Returns formatting status."
version: "1.0.0"
dependencies: []
allowed-tools: ["Bash", "Read"]
model: "haiku"
---

# Formatting Code

Auto-format code using project's configured formatters.

## Workflow

### 1. Detect Formatters
Check for:
- prettier (JS/TS)
- black (Python)
- gofmt (Go)
- rustfmt (Rust)
- Other language-specific formatters

### 2. Run Formatters
```bash
# JS/TS
npx prettier --write .

# Python
black .

# Go
gofmt -w .

# Rust
cargo fmt
```

### 3. Check Results
```bash
git diff --stat
```

### 4. Return Status
```
Formatted 12 files:
- src/api/auth.js
- src/models/user.py
- tests/integration.go

Run git diff to review changes.
```

## Output Format
Return:
- Number of files formatted
- List of changed files
- Command to review changes

## Success Criteria
- ✅ Formatters detected and run
- ✅ All files formatted
- ✅ No formatter errors
```

## File Structure

```
.claude/skills/
├── scout/
│   └── SKILL.md
├── plan/
│   └── SKILL.md
├── generating-commits/
│   └── SKILL.md
├── validating-migrations/
│   ├── SKILL.md
│   └── resources/
│       └── safe-migration-checklist.md
└── formatting-code/
    └── SKILL.md
```

## Success Criteria

- ✅ Created `.claude/skills/[name]/SKILL.md`
- ✅ Frontmatter includes name, description, tools, model
- ✅ Workflow has numbered steps
- ✅ Output format defined
- ✅ Success criteria listed
- ✅ Error handling included

## Error Handling

**If skill purpose unclear:**
Ask user to clarify workflow before creating skill.

**If similar skill exists:**
Check `.claude/skills/` directory, suggest using/extending existing skill.

## Distinct from Other Skills

| Skill Creating | NOT Skill Creating |
|---------------|-------------------|
| Create reusable workflows | Create specialized agents (agent-creating) |
| Multi-agent patterns | One-time task execution |
| Standardized procedures | Project-specific implementations |
| Agent-invokable via Skill tool | User-invokable commands |
