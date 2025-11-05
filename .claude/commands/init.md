# Initialize Project from PRD

**Usage:** `/init <path-to-prd>`

Initialize project workspace from product requirement document. Sets up workflow structure, analyzes requirements, proposes core features, prepares orchestration.

**CRITICAL: MUST COMPLETE ALL 5 PHASES BEFORE REPORTING "DONE"**

---

## Phase 1: Analysis & Setup

**Deliverables:**
1. Repository analysis complete (project type, structure, dependencies, tests, docs)
2. Scattered markdown files organized into workflow/docs/, .claude/archive/, workflow/tmp/
3. PRD read, cross-referenced with codebase, discrepancies flagged
4. CLAUDE.md updated (only if PRD requires workflow customization)
5. workflow/docs/ populated: api-reference.md, best-practices.md, {domain}-guide.md
6. workflow/tests/ cleaned (only relevant patterns kept, seeded with existing examples)
7. .claude/memory.md updated (cold: project context, hot: initialization task)

**Analysis scope:**
- Scan repo: directory structure, config files, entry points, test setup, dependencies, code patterns, existing docs, git history
- Categorize markdown: specs, documentation, notes, history, tests, scratch
- Organize markdown: consolidate duplicates, archive history, move temp files, flag deletions
- Extract from PRD: project type, tech stack, features, constraints, architecture, APIs
- Synthesize: actual codebase + PRD requirements + discrepancies

**Output files:**
- CLAUDE.md (if edited)
- workflow/docs/api-reference.md
- workflow/docs/best-practices.md
- workflow/docs/{domain}-guide.md (as needed)
- workflow/tests/{project-type}-patterns.md (cleaned/enhanced)
- .claude/memory.md

---

## Phase 2: Core Feature Extraction

**Deliverables:**
1. Core features extracted from PRD (5-12 features, core only)
2. Features marked [NEW] or [ENHANCE existing]
3. Features saved to workflow/tmp/features.md
4. Changes reported (files created/edited, markdown reorganization, repo insights)
5. Core features proposed to user with options

**Extraction criteria:**
- Core = MVP essential, foundational, or blocking
- Compare codebase vs PRD (what exists vs what's needed)
- Format: numbered list, 1-line descriptions

**Report to user:**
- Files created/edited (CLAUDE.md, workflow/docs/, workflow/tests/, memory.md)
- Markdown organization (moved/archived/merged/deleted)
- Repo insights (size, existing features, test coverage)
- Core features (numbered, marked [NEW]/[ENHANCE]) - also saved to workflow/tmp/features.md

**Ask:** "Do you agree with this core feature list?"
**Options:** Yes (proceed to Phase 3) / No, modify / Let me revise

---

## Phase 3: Iteration (if needed)

**Deliverables:**
1. User feedback incorporated
2. Updated feature list presented (update workflow/tmp/features.md)
3. Agreement obtained (or max 5 iterations reached)

**Flow:**
- If "No" → update list → update workflow/tmp/features.md → re-present → ask again
- Iterate until "Yes"

---

## Phase 4: Store & Initialize Orchestration

**Deliverables:**
1. Agreed features stored in .claude/memory.md warm section
2. workflow/tmp/context-pool.json created (< 400 tokens)
3. Placeholder plans created: workflow/tmp/plan-{feature-name}.md (stubs only)
4. Skill preparation notes documented (which features need which skills)

**DO NOT:**
- Invoke skills or tasks
- Start implementation
- Run orchestration

**context-pool.json structure:**
```json
{
  "project_type": "...",
  "tech_stack": [...],
  "architecture": "...",
  "test_framework": "...",
  "test_config": "...",
  "test_directory": "...",
  "entry_points": [...],
  "key_directories": {...},
  "core_features": [...],
  "existing_features": [...],
  "documentation": {...},
  "dependencies": {...}
}
```

---

## Phase 5: Reporting

**Deliverables:**
1. STATUS-REPORT.html generated (dark mode, inline CSS, no external deps)
2. Final summary presented to user

**HTML sections:**
1. **Feature Status Dashboard** - all features from memory.md (status, stage, tests, blockers, agents)
2. **Project Size & Scope** - size estimate, feature counts
3. **Deep Progress Detail** - expected phases, skills needed, complexity, dependencies
4. **Repository Overview** - file count, type, stack, test/doc status
5. **Initialization Summary** - repo discoveries, PRD extraction, discrepancies, next actions

**HTML requirements:**
- Dark mode friendly CSS
- Color-coded status indicators
- Summary cards (4 key metrics)
- Footer with timestamp
- All CSS/JS inline

**Final summary format:**
```
✅ Repository: [count] files analyzed
✅ Project type: [type]
✅ Documentation: [count] files created
✅ Test patterns: [status]
✅ Core features: [count] identified ([NEW]/[ENHANCE])
✅ Markdown organization: [count] files processed
✅ Orchestration: Prepared (not executed)

Next: "Ready to implement. View STATUS-REPORT.html. Request feature or say 'implement all'"
```

---

## Error Handling

- PRD path invalid → report error, ask for path
- PRD unclear → extract best-effort, note ambiguities
- PRD contradicts codebase → report discrepancies in Phase 2
- Project type ambiguous → ask user (show evidence)
- User never agrees (5 iterations) → ask manual edit
- No test infrastructure → note in context-pool
- Empty codebase → rely on PRD, note in report
- Large codebase (10k+ files) → sample, note limitations

---

## Validation Checklist (before reporting "done")

**Output this after message:**

- [ ] Phase 1: All 7 deliverables created
- [ ] Phase 2: Features extracted, workflow/tmp/features.md created, changes reported, user asked
- [ ] Phase 3: User agreement obtained (or max iterations), workflow/tmp/features.md updated if needed
- [ ] Phase 4: Features stored, context-pool created, plans stubbed
- [ ] Phase 5: STATUS-REPORT.html generated, final summary presented
- [ ] All files use exact paths (workflow/docs/, workflow/tmp/, .claude/archive/)
- [ ] Markdown organization complete (scattered files processed)
- [ ] Dual source of truth (codebase + PRD synthesized)