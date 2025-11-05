# Initialize Project from PRD

**Usage:** `/init <path-to-prd>`

Initialize project workspace from product requirement document. Sets up workflow structure, analyzes requirements, proposes core features, and prepares orchestration.

## Execution Steps

### Phase 1: Analysis & Setup

1. **Analyze Repository Structure**
   - **Directory scan**: Use `find` or `tree` to get full directory structure
     - Identify src/, lib/, components/, tests/, docs/, config/ directories
     - Note depth and organization patterns (monorepo vs single-package)

   - **Key file detection**: Look for configuration files
     - package.json, package-lock.json (Node.js)
     - requirements.txt, setup.py, pyproject.toml (Python)
     - Cargo.toml (Rust)
     - go.mod (Go)
     - manifest.json (Chrome extension)
     - tsconfig.json, webpack.config.js, vite.config.js (build config)
     - .eslintrc, .prettierrc (code quality)
     - jest.config.js, pytest.ini, vitest.config.js (test config)

   - **Entry points**: Identify main application files
     - index.js, main.js, app.js, server.js (Node.js)
     - __main__.py, app.py, manage.py (Python)
     - main.go (Go)
     - Main.tsx, App.tsx (React)
     - background.js, popup.js, content.js (Chrome extension)

   - **Existing test structure**: Map test organization
     - Test directory location (tests/, __tests__, spec/)
     - Test file naming pattern (*.test.js, *_test.py, *.spec.ts)
     - Test framework indicators (jest, vitest, pytest, unittest, mocha)

   - **Dependencies audit**: Extract from package managers
     - Read package.json dependencies (React, Express, etc.)
     - Read requirements.txt or pyproject.toml
     - Identify external APIs/SDKs in dependencies

   - **Code pattern sampling**: Quick grep for architectural hints
     - Database usage (import db, mongoose, sequelize, prisma, sqlalchemy)
     - Authentication (passport, jwt, auth0, supabase)
     - State management (redux, zustand, pinia, context)
     - API patterns (REST routes, GraphQL schemas, tRPC)

   - **Documentation check**: Look for existing docs
     - README.md, CONTRIBUTING.md, API.md
     - docs/ or documentation/ directory
     - Inline code comments and JSDoc/docstrings

   - **Git analysis** (if available):
     - Recent commits to understand active development areas
     - Branch structure (feature/, develop/, main patterns)
     - .gitignore to understand build artifacts

2. **Organize Scattered Development Markdown Files**
   - **Find all markdown files**: Search entire repository for *.md files
     - Exclude: node_modules/, venv/, dist/, build/, .git/
     - Identify scattered: notes, specs, plans, logs, TODOs, dev journals

   - **Categorize by content type**: Sample each file to determine purpose
     - **Development plans/specs**: Feature specs, architecture docs, implementation plans
     - **Project documentation**: API docs, guides, how-tos, references
     - **Meeting notes/decisions**: ADRs, decision logs, meeting notes
     - **Development history**: Changelogs, progress logs, dev journals
     - **Test documentation**: Test plans, coverage reports, QA notes
     - **Temporary/scratch**: WIP notes, debugging logs, temp files

   - **Organize into workflow structure**:
     - **workflow/docs/**: Move project documentation
       - API documentation → `workflow/docs/api-reference.md` (consolidate if multiple)
       - Implementation guides → `workflow/docs/{domain}-guide.md`
       - Keep existing README.md, CONTRIBUTING.md in root

     - **.claude/archive/**: Archive development history
       - Create `.claude/archive/YYYY-MM-DD/` dated folders
       - Move: meeting notes, decision logs, old plans, dev journals
       - Preserve chronological order

     - **workflow/tmp/**: Move active/temporary files
       - Move: WIP notes, scratch files, debug logs
       - Mark for cleanup after initialization

     - **Consolidate duplicates**: Merge similar content
       - Multiple API docs → single `workflow/docs/api-reference.md`
       - Multiple guides on same topic → single coherent guide
       - Note merged sources in file header

     - **Delete obsolete**: Remove clearly outdated/irrelevant files
       - Ask user for confirmation before deleting (list files)
       - Examples: old TODOs for shipped features, outdated specs

   - **Create organization report**: Document what was moved/merged/deleted
     - List: original location → new location
     - Note: merged files (sources)
     - Note: files marked for deletion (pending user approval)

3. **Read PRD**
   - Load PRD from provided path
   - Extract: project type, tech stack, features, constraints, architecture
   - Identify: core domain, external APIs/services, testing requirements
   - **Cross-reference with repo structure**: Validate PRD against actual codebase
     - Flag discrepancies (PRD says React, repo has Vue)
     - Note missing features mentioned in PRD
     - Identify existing features not in PRD

4. **Determine Project Type**
   - Synthesize from both repo analysis AND PRD
   - Project type: chrome-extension, react, express, python, next.js, etc.
   - Framework/library versions (critical for test-generator)
   - Validate test infrastructure exists or plan to add
   - Document actual tech stack (may differ from PRD)

5. **Update CLAUDE.md (if needed)**
   - Add project-specific context to CLAUDE.md if PRD contains:
     - Unique architectural patterns
     - Special orchestration rules
     - Project-specific validation requirements
   - Keep additions minimal (< 200 tokens)
   - Only edit if PRD requires workflow customization

6. **Populate workflow/docs/**
   - Create `api-reference.md`: Internal API documentation
     - Extract API contracts, endpoints, interfaces from PRD AND existing code
     - Sample actual API code to document real patterns
     - Document data models, schemas, types found in codebase
     - Include authentication/authorization patterns (both PRD and actual implementation)
     - **Incorporate organized markdown from step 2** (previously scattered API docs)

   - Create `best-practices.md`: Project-specific coding standards
     - Extract coding conventions from PRD
     - **Analyze existing code samples** for actual patterns used
     - Error handling patterns (from existing code)
     - Document naming conventions (from repo analysis)
     - File organization rules (from directory structure)

   - Create domain-specific guides as needed:
     - `{domain}-guide.md` for each major domain (e.g., auth-guide.md, payments-guide.md)
     - **Reference actual implementation files** discovered in repo analysis
     - **Incorporate organized markdown from step 2** (previously scattered guides)
     - Implementation patterns for complex features
     - Integration guidelines for external services

7. **Clean workflow/tests/**
   - Keep only test patterns matching project type
   - Remove irrelevant patterns (e.g., if react project, remove python-patterns.md)
   - Add missing test pattern file for project type if needed
   - **Seed with existing test examples** from repo analysis
   - Enhance relevant pattern file with PRD-specific test requirements

8. **Update memory.md**
   - **Cold section**: Add project context from BOTH repo analysis AND PRD
     - Project type, tech stack, architecture overview
     - Directory structure (from repo scan), entry points (discovered)
     - Testing infrastructure location (actual paths found)
     - Key dependencies and external integrations
   - **Warm section**: Leave empty (patterns added during work)
   - **Hot section**: Add initialization task
   - Keep total < 800 tokens

### Phase 2: Core Feature Extraction

9. **Extract Core Features**
   - Analyze PRD for CORE features (not nice-to-have)
   - **Compare with existing codebase**: Identify what's built vs what's needed
   - Core = essential for MVP, blocking other features, or foundational
   - Distinguish: new features to build vs existing features to enhance
   - Format as numbered list with brief description (1 line each)
   - Aim for 5-12 core features

10. **Report Changes**
    - List all files edited/created:
      - CLAUDE.md (if edited)
      - workflow/docs/ files (list each)
      - workflow/tests/ files (removed/kept/added)
      - memory.md
    - Report markdown organization from step 2:
      - Files moved to workflow/docs/
      - Files archived to .claude/archive/
      - Files moved to workflow/tmp/
      - Files merged (note sources)
      - Files proposed for deletion (pending approval)
    - Report repository insights discovered:
      - Project size (file count, LOC estimate)
      - Existing feature completeness
      - Test coverage status
    - Keep format concise, use bullets

11. **Propose Core Features**
    - Present numbered list of core features
    - Mark each: [NEW] or [ENHANCE existing]
    - Ask: "Do you agree with this core feature list?"
    - Provide options:
      - Yes (proceed to Phase 3)
      - No, modify (user provides changes)
      - Let me revise (re-analyze PRD with different criteria)

### Phase 3: Iteration (if user disagrees)

12. **Handle Disagreement**
    - If user says "No" or provides modifications:
      - Update feature list based on feedback
      - Re-present list
      - Ask again for agreement
    - Iterate until user agrees
    - Maximum 5 iterations (then ask user to manually edit)

### Phase 4: Store & Initialize Orchestration

13. **Store Agreed Features in memory.md**
    - Update warm section with core features as patterns to implement
    - Format: Feature name → expected files/domains → existing or new
    - Keep < 400 tokens

14. **Initialize Orchestration Structure**
    - Create `workflow/tmp/context-pool.json`:
      ```json
      {
        "project_type": "...",
        "tech_stack": [...],
        "architecture": "...",
        "test_framework": "...",
        "test_config": "...",
        "test_directory": "...",
        "entry_points": [...],
        "key_directories": {
          "src": "...",
          "tests": "...",
          "docs": "...",
          "config": "..."
        },
        "core_features": [...],
        "existing_features": [...],
        "documentation": {
          "api_reference": "workflow/docs/api-reference.md",
          "best_practices": "workflow/docs/best-practices.md",
          "domain_guides": [...]
        },
        "dependencies": {
          "main": [...],
          "dev": [...],
          "external_apis": [...]
        }
      }
      ```
    - Populate from BOTH repo analysis AND PRD
    - Keep < 400 tokens

15. **Prepare (but don't run) Orchestration**
    - Create placeholder execution plans for each core feature:
      - `workflow/tmp/plan-{feature-name}.md` (stub, not full plan)
      - Contains: Feature name, estimated files, dependencies, existing code to reference
      - Mark as "PENDING - run codebase-search + execution-planner when ready"

    - Create skill preparation notes:
      - Which features need codebase-search (unknown areas or enhancements)
      - Which features need test-generator (all of them)
      - Which features can use direct tasks (known patterns from repo analysis)

    - DO NOT invoke skills or tasks yet
    - DO NOT start implementation

### Phase 5: Reporting

16. **Generate HTML Status Report**
    - Save to `/STATUS-REPORT.html`

    **Gather this information:**

    1. **Feature Status Dashboard**
       - Parse `.claude/memory.md` (hot, warm, cold) for all features/tasks
       - For each feature:
         - Name & short description
         - Current status: Not started / In progress / Blocked / Shipped
         - Stage details:
           - If not shipped, which phase? (planning/building/testing/doc)
           - List tests/checks run (failures/errors or success)
           - Any blockers, issues, or TODOs
           - Links to related API docs/spec/plans if available
         - Show which agent(s) or workflows will work on it
       - Visual indicators: color/emoji for each status

    2. **Project Size & Scope**
       - Estimate project size (small/medium/big/very big) based on memory.md + repo analysis
       - Count features/tasks currently tracked & shipped

    3. **Deep Progress Detail**
       - For each feature (not started during init), show:
         - Expected build/test phases
         - Which skills will be needed (codebase-search, execution-planner, test-generator)
         - Estimated complexity
         - Dependencies between features

    4. **Repository Overview**
       - File count, project type, tech stack
       - Test infrastructure status
       - Documentation status

    5. **Initialization Summary**
       - What was discovered in repo analysis
       - What was extracted from PRD
       - Any discrepancies flagged
       - Next recommended actions

    **HTML Requirements:**

    - Basic CSS styling (dark mode friendly)
    - Color-coded status indicators
    - Summary cards at top (4 key metrics)
    - Footer with generation timestamp
    - No external dependencies (inline all CSS/JS)

    **Output:**

    List what you included in report. Sacrifice grammar for conciseness.
    Keep data gathering efficient - use sampling for large files.

## Important Notes

- **Repository analysis first**: ALWAYS scan entire repo before reading PRD to ground understanding in reality
- **Dual source of truth**: Synthesize from BOTH codebase (what exists) AND PRD (what's desired)
- **Minimal CLAUDE.md edits**: Only if PRD requires workflow customization
- **No implementation**: This command only sets up structure
- **User must agree**: Cannot proceed to Phase 4 without user agreement on core features
- **Status report auto-generated**: Phase 5 creates HTML report automatically after initialization
- **Concise output**: Sacrifice grammar for brevity in all reports
- **Context pool is critical**: Must be created for orchestration to work
- **Workflow/docs mandatory**: All projects need api-reference.md and best-practices.md
- **Workflow/docs ground in reality**: Use actual code samples from repo analysis, not just PRD theory
- **Test patterns pruned**: Only keep relevant patterns for project type
- **Flag discrepancies**: If PRD contradicts codebase, report to user in Phase 2

## Error Handling

- PRD path invalid → Report error, ask for correct path
- PRD unclear → Extract best-effort, note ambiguities in report
- PRD contradicts codebase → Report discrepancies in Phase 2, ask user for guidance
- Project type ambiguous → Ask user to clarify (show evidence from both repo and PRD)
- User never agrees (5+ iterations) → Ask user to manually edit memory.md
- No test infrastructure detected → Note in context-pool, plan Phase 0 for setup
- Empty or minimal codebase → Rely more on PRD, note in final report
- Large codebase (10k+ files) → Sample key areas instead of full scan, note limitations

17. **Final Report**
    - Summarize initialization:
      - ✅ Repository: [file count] files analyzed
      - ✅ Project type: [type]
      - ✅ Documentation: [count] files created (list them like a root tree)
      - ✅ Test patterns: [count] files (removed/kept/added)
      - ✅ Core features: [count] identified (list them - [new] new, [enhance] enhancements)
      - ✅ Markdown organization: [count] files moved/archived/merged
      - ✅ Orchestration: Prepared (not executed)
    - Next steps:
      - "Ready to implement core features. View STATUS-REPORT.html for details."
      - "To start: request a specific feature or say 'implement all core features'"