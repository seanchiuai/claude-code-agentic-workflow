---
name: "test-generator"
description: "Test-Driven Development skill. Generates unit tests before implementation. Adapts to project type (Chrome extension, React, Express, Python, etc.). Reads/updates pattern library."
version: "1.0.0"
dependencies: []
allowed-tools: ["Read", "Write", "Glob", "Grep"]
---

# Test Generator Skill

Generate project-appropriate unit tests before implementation.

## Purpose
Create test files based on:
- **Project type** (chrome-extension, react, express, python, etc.)
- **Requirements** (what needs to be implemented)
- **File specifications** (from codebase-search results)
- **Existing patterns** (from workflow/tests/{type}-patterns.md)

**Mandatory for all feature work** - invoked by execution-planner skill during Phase 2.

## Workflow

### 1. Load Test Patterns

**IMPORTANT:** Assumes test infrastructure exists (execution-planner validates before invoking this skill).

If infrastructure was missing, execution-planner would have created Phase 0 and NOT invoked this skill.

Read `workflow/tests/{project-type}-patterns.md`:
- Infrastructure details (framework, config, commands)
- Setup patterns (mocking, fixtures)
- Code patterns (common test structures)
- Learned patterns (from previous runs)

**If pattern file missing:**
Create basic pattern file from LLM knowledge for this project type.

### 2. Parse Requirements

From input, extract:
- **Files to test**: List of implementation files
- **Functionality**: What each file should do
- **Dependencies**: External APIs, services, Chrome APIs to mock
- **Test location**: Where tests should live (tests/, __tests__/, etc.)

### 3. Generate Test Files

For each file to implement, create corresponding test file:

**Test file naming conventions:**
- React/Node: `{filename}.test.js` or `__tests__/{filename}.test.js`
- Python: `test_{filename}.py` in `tests/` directory
- Match project's existing test structure if present

**Test structure:**
```
- Setup/teardown (beforeEach, afterEach, fixtures)
- Mock external dependencies (APIs, Chrome APIs, database)
- Test cases covering main functionality
- Edge cases and error handling
- Use patterns from pattern library
```

**Example for Chrome extension storage function:**
```js
// tests/storage.test.js
import { saveUserData, getUserData } from '../src/storage';

describe('Storage operations', () => {
  beforeEach(() => {
    jest.clearAllMocks();
    global.chrome.storage.local.get.mockClear();
    global.chrome.storage.local.set.mockClear();
  });

  it('should save user data to chrome.storage.local', async () => {
    const userData = { name: 'John', preferences: { theme: 'dark' } };

    await saveUserData(userData);

    expect(chrome.storage.local.set).toHaveBeenCalledWith(
      { userData },
      expect.any(Function)
    );
  });

  it('should retrieve user data from storage', async () => {
    const mockData = { userData: { name: 'John' } };
    chrome.storage.local.get.mockImplementation((keys, callback) => {
      callback(mockData);
    });

    const result = await getUserData();

    expect(result).toEqual({ name: 'John' });
    expect(chrome.storage.local.get).toHaveBeenCalledWith('userData', expect.any(Function));
  });

  it('should handle storage errors gracefully', async () => {
    chrome.storage.local.get.mockImplementation((keys, callback) => {
      callback(null);
    });

    const result = await getUserData();

    expect(result).toBeNull();
  });
});
```

### 4. Identify New Patterns

While generating tests, note any new patterns discovered:
- New mocking approaches
- Edge cases specific to this project
- Useful test utilities created

### 5. Update Pattern Library

Append new patterns to `workflow/tests/{project-type}-patterns.md` under "Learned Patterns" section:

```markdown
## Learned Patterns

### Pattern Name (Added YYYY-MM-DD)
Description of pattern and when to use it.

```code example```
```

### 6. Create Test Specification Summary

Return to execution-planner skill (< 150 tokens):
```
Generated {N} test files for {project-type}:
- tests/feature-a.test.js: {brief description of coverage}
- tests/feature-b.test.js: {brief description of coverage}

Mocking strategy: {chrome APIs, HTTP requests, database, etc.}
Coverage: {main cases + edge cases + errors}

Builders must make these tests pass.
```

## Project Type Adaptations

### chrome-extension
- Mock chrome.* APIs (storage, runtime, tabs, etc.)
- Test message passing between components
- DOM manipulation for popups/sidepanels
- Framework: Jest + jsdom

### react
- Component rendering tests
- User interaction tests (clicks, inputs)
- Hook testing
- API mocking with MSW
- Framework: Vitest + Testing Library

### express
- Endpoint testing with supertest
- Middleware testing
- Database mocking
- Auth flow testing
- Framework: Jest + supertest

### python
- Function and class testing
- File operation mocking
- API mocking with unittest.mock
- Parametrized tests
- Framework: pytest

### react-native
- Component tests with Testing Library Native
- Platform-specific mocking
- Navigation testing
- Framework: Jest + Testing Library Native

### CLI tools (Node/Python)
- stdio mocking
- Command argument testing
- Exit code validation
- Framework: Jest or pytest

## Output Format

**Test files:** Write to appropriate test directory (tests/, __tests__/, etc.)
**Pattern updates:** Append to workflow/tests/{project-type}-patterns.md
**Return message:** Test specification summary (< 150 tokens)

## Example Execution

**Input:**
```
Project: chrome-extension
Requirements: Add autosave functionality
Files: src/storage.js (save/load functions), src/autosave.js (interval timer)
```

**Process:**
1. Load: Read workflow/tests/chrome-extension-patterns.md
2. Parse: 2 files need tests, Chrome storage API needs mocking
3. Generate:
   - tests/storage.test.js (save/load with chrome.storage.local mocks)
   - tests/autosave.test.js (setInterval mocking, save trigger tests)
4. Pattern: Note new pattern for timer mocking
5. Update: Append timer pattern to chrome-extension-patterns.md
6. Return: "Generated 2 test files: storage.test.js (save/load/errors), autosave.test.js (interval/trigger/cleanup). Mocking chrome.storage.local + timers. Builders must make tests pass."

**Note:** Infrastructure already validated by execution-planner before this skill was invoked.

## Success Criteria

- ✅ **Assumes test infrastructure exists (execution-planner validated first)**
- ✅ Pattern library loaded for project type
- ✅ Test files created matching project conventions
- ✅ All mocks identified and implemented
- ✅ Coverage includes happy path + edge cases + errors
- ✅ New patterns appended to pattern library
- ✅ Summary returned < 150 tokens

## Error Handling

**No test infrastructure:**
Should never happen - execution-planner validates before invoking this skill.
If it does happen, report error and ask orchestrator to check execution-planner logic.

**Unknown project type:**
Fail and ask for explicit project type or create generic pattern.

**No pattern file:**
Create basic pattern file from LLM knowledge for this project type.

**Can't determine test location:**
Ask or use standard convention (tests/ for most projects).

## Distinct from Other Skills

| Test-Generator | NOT Test-Generator |
|-----------------|---------------------|
| Generate test files | Implement features (builders) |
| Mock external dependencies | Search for patterns (codebase-search) |
| Project-type specific tests | Create implementation plan (execution-planner) |
| Write tests before code | Research external APIs (researching-features) |
