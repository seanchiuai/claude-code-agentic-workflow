# Generate Project Status Report

Generate a comprehensive HTML status report saved to `/STATUS-REPORT.html`.

**Gather this information:**

1. **Project Overview** - Name, type, tech stack from context or inspection
2. **Memory Status** - Parse `.claude/memory.md` (hot/warm/cold sections with task counts)
3. **Active Plans** - List files in `specs/` with timestamps and summaries
4. **Recent Activity** - Git log last 10 commits (if git available) with dates and messages
5. **File Structure** - Key directories and file counts
6. **Temporary State** - Check `tmp/` for active context-pool, scout results
7. **Skills & Commands** - List available in `.claude/skills/` and `.claude/commands/`
8. **Validation Status** - Check for checkpoint files (`.workflow-checkpoint`, `.phase-checkpoint`)

**HTML Requirements:**

- Basic CSS styling (dark mode friendly)
- Color-coded status indicators
- Summary cards at top (4 key metrics)
- Footer with generation timestamp
- No external dependencies (inline all CSS/JS)

**Output:**

List what you have included in the report. In your resposne and report, sacrifise grammar for conciseness

Keep data gathering efficient - don't read large files completely, use sampling.
