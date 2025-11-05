# Generate Project Status Report

Generate a comprehensive HTML status report saved to `/STATUS-REPORT.html`.

**Gather this information:**

1. **Feature Status Dashboard**
   - Parse `.claude/memory.md` (hot, warm, cold) for all features/tasks, listing each feature being worked toward shipping.
   - For each feature:
     - Name & short description
     - Current status: Not started / In progress (building/testing/doc) / Blocked / Shipped
     - Stage details:
       - If not shipped, which phase? (e.g. API dev, UI building, documentation, test runs)
       - List tests/checks run (point to failures/errors if any, or test success)
       - Any blockers, issues, or TODOs
       - Links to related API docs/spec/plans if available
     - Show which agent(s) or workflows touched/are working on it
  - Visual indicators: color/emoji for each status

2. **Project Size & Scope**
   - Estimate project size (small/medium/big/very big) based on memory.md context (e.g., number of shipped features, complexity, file structure)
   - Count features/tasks currently tracked & shipped

3. **Deep Progress Detail**
   - For each in-progress feature, show:
     - All build/test phases so far
     - Problems encountered or open issues
     - Check if API docs/spec implemented/used (scan for /api/, /specs/, or mentioned in memory)
     - Whether it has been user-tested, unit-tested, or integrated

4. **Archive/History**
   - List previously shipped features (from memory.md/history/archive if present) with brief outcome and date

5. **Other signals (optional, if present)**
   - Which step in overall workflow the project is at (from cold/warm memory)
   - Any evidence of external review, validation, or external API integration in recorded memory

**Report purpose**:  
Show *feature-by-feature* progress and problems to help project owner understand, at a glance, what's shipped, stuck, still building, or under test—based only on `.claude/memory.md` and minimal file inspection.


**HTML Requirements:**

- Basic CSS styling (dark mode friendly, theme matching the project is applicable)
- Color-coded status indicators
- Summary cards at top (4 key metrics)
- Footer with generation timestamp
- No external dependencies (inline all CSS/JS)

**Output:**

List what you have included in the report. In your resposne and report, sacrifise grammar for conciseness

Keep data gathering efficient - don't read large files completely, use sampling.
