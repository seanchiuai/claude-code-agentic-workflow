# Project Memory
*Last updated: [timestamp]*
*Token budget: < 800 tokens (current: ~450)*

## Hot Memory (Last 2 Tasks - 300 tokens max)
*Auto-decay: Moves to archive after 5 new tasks*

### Task 1: [Most Recent]
- **Date**: [YYYY-MM-DD HH:MM]
- **Type**: [Feature/Bug/Refactor/Research]
- **Scope**: [Brief description - 1 line]
- **Files touched**: [3-5 key files]
- **Pattern used**: [Key pattern applied]
- **Builders**: [Number of parallel agents]
- **Outcome**: [✅ Success / ⚠️ Partial / ❌ Failed]

### Task 2: [Second Most Recent]
- **Date**: [YYYY-MM-DD HH:MM]
- **Type**: [Feature/Bug/Refactor/Research]
- **Scope**: [Brief description - 1 line]
- **Files touched**: [3-5 key files]
- **Pattern used**: [Key pattern applied]
- **Builders**: [Number of parallel agents]
- **Outcome**: [✅ Success / ⚠️ Partial / ❌ Failed]

---

## Warm Memory (Project Patterns - 400 tokens max)
*Reusable patterns discovered during tasks*

### Architecture Patterns
- **[Pattern name]**: [Where it lives - file:line]
  - *Usage*: [When to apply this pattern]
  - *Example*: [Concrete code location]

### Code Conventions
- **Import style**: [ES6 modules / CommonJS / etc]
- **Error handling**: [try-catch pattern / error boundaries / etc]
- **Testing framework**: [Jest / Mocha / Pytest / etc]
- **File organization**: [Feature-based / Layer-based / etc]

### Common Solutions
- **[Problem]**: [Solution pattern at file:line]
- **[Problem]**: [Solution pattern at file:line]

---

## Cold Memory (Project Context - 150 tokens max)
*Stable information about project structure*

### Project Type
- **Stack**: [Language/framework - e.g., "React + Express + MongoDB"]
- **Architecture**: [Monolith / Microservices / etc]
- **Deployment**: [Where it runs - AWS / Vercel / etc]

### Key Files (Top 10)
- `[file path]` - [Role - 3-5 words]
- `[file path]` - [Role - 3-5 words]
- `[file path]` - [Role - 3-5 words]

### Directory Structure
```
project/
├── [dir]/ - [Purpose]
├── [dir]/ - [Purpose]
└── [dir]/ - [Purpose]
```

### Dependencies (Critical only)
- [package]: [version] - [Why critical]
- [package]: [version] - [Why critical]

---

## Memory Management Rules

### Auto-Decay (Main Agent Responsibility)
1. After 5 completed tasks → Oldest hot task moves to `.claude/archive/[date].md`
2. If total tokens > 800 → Prune oldest warm patterns
3. Cold memory updates only on major structural changes

### Update Triggers
- **Hot**: After every orchestrated task completion
- **Warm**: When new pattern discovered or existing pattern used
- **Cold**: Project setup, major refactors, dependency changes

### Token Budget Enforcement
```
Hot:  2 tasks × 150 tokens = 300 tokens max
Warm: Patterns + conventions = 350 tokens max
Cold: Project context       = 150 tokens max
--------------------------------
Total:                        800 tokens max
```

---

## Usage Guidelines

### For Main Agent
**When to load:**
- Start of new session (if file exists)
- Before complexity assessment (check if pattern exists in warm memory)

**When to update:**
- After orchestrated task completes (add to hot memory)
- New pattern discovered (add to warm memory)
- Project structure changes (update cold memory)

**What to check:**
- Before invoking scout → Check warm memory for known patterns
- Before invoking plan → Check hot memory for similar recent tasks
- Before launching builders → Use cold memory for context-pool generation

### For Sub-Agents
**Do NOT read or modify this file.** Sub-agents use:
- `workflow/tmp/context-pool.json` (generated from cold memory)
- `workflow/tmp/scout-results.md` (from codebase-search skill)
- `workflow/tmp/plan-{task-name}.md` (from execution-planner skill)

---

## Example Populated Memory

```markdown
# Project Memory
*Last updated: 2025-01-04 14:30:22*
*Token budget: < 800 tokens (current: 720)*

## Hot Memory (Last 2 Tasks)

### Task 1: User Authentication System
- **Date**: 2025-01-04 14:15
- **Type**: Feature
- **Scope**: Added JWT auth with protected routes
- **Files touched**: api/routes.js, models/user.js, middleware/auth.js, utils/jwt.js, tests/auth.test.js
- **Pattern used**: Express middleware chain + Passport.js pattern
- **Builders**: 2 parallel (auth logic + tests)
- **Outcome**: ✅ Success - All tests passing

### Task 2: Payment Integration
- **Date**: 2025-01-03 09:45
- **Type**: Feature
- **Scope**: Integrated Stripe payment processing
- **Files touched**: api/payments.js, services/stripe.js, models/transaction.js, tests/payments.test.js
- **Pattern used**: Service layer pattern for external APIs
- **Builders**: 2 parallel (stripe service + payment routes)
- **Outcome**: ✅ Success - Webhook handling working

---

## Warm Memory (Project Patterns)

### Architecture Patterns
- **Express middleware chain**: middleware/logger.js:15-30
  - *Usage*: All middleware follows this pattern (req, res, next)
  - *Example*: middleware/auth.js uses same pattern for JWT verification

- **Service layer for external APIs**: services/stripe.js:1-120
  - *Usage*: Wrap all third-party API calls in service layer
  - *Example*: services/email.js, services/storage.js follow same pattern

- **Error handling middleware**: api/routes.js:120-135
  - *Usage*: try-catch with next(err) pattern, centralized error handler
  - *Example*: All route handlers use this pattern

### Code Conventions
- **Import style**: ES6 modules (import/export)
- **Error handling**: try-catch with next(err), central error middleware at app.js:89
- **Testing framework**: Jest + supertest for API, @testing-library/react for frontend
- **File organization**: Feature-based (auth/, payments/, users/)

### Common Solutions
- **Password hashing**: models/user.js:45-67 uses bcrypt with pre-save hook
- **Token validation**: utils/jwt.js:12-23 handles JWT verify with error handling
- **Database connection**: config/db.js:8-25 uses Mongoose with connection pooling
- **Environment config**: config/env.js:1-30 validates required env vars on startup

---

## Cold Memory (Project Context)

### Project Type
- **Stack**: Express.js (Node 18) + MongoDB (Mongoose) + React (Vite)
- **Architecture**: Monolithic backend API + SPA frontend
- **Deployment**: Backend on Railway, frontend on Vercel

### Key Files
- `src/app.js` - Express app setup, middleware chain
- `src/api/routes.js` - Route definitions and handlers
- `config/db.js` - Database connection
- `models/user.js` - User model with auth methods
- `middleware/auth.js` - JWT authentication middleware
- `services/stripe.js` - Stripe integration
- `tests/api.test.js` - API integration tests
- `client/src/App.tsx` - Frontend entry point
- `client/src/api/client.ts` - API client with auth headers
- `package.json` - Dependencies and scripts

### Directory Structure
```
project/
├── src/
│   ├── api/ - Route handlers
│   ├── models/ - Database models
│   ├── middleware/ - Express middleware
│   ├── services/ - External API wrappers
│   └── utils/ - Helper functions
├── config/ - Configuration files
├── tests/ - Backend tests
├── client/ - React frontend
└── scripts/ - Build and deployment scripts
```

### Dependencies (Critical)
- express: ^4.18.0 - Core web framework
- mongoose: ^8.0.0 - MongoDB ODM
- jsonwebtoken: ^9.0.0 - JWT auth
- stripe: ^14.0.0 - Payment processing
- bcrypt: ^5.1.0 - Password hashing
```

---

## Archive Trigger

When hot memory reaches 5 tasks, oldest task moves to:
`.claude/archive/2025-01-03-payment-integration.md`

Archive files are NOT loaded by main agent (keep context clean).
