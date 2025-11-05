---
name: "Researching Features"
description: "Research external APIs/services/libraries for feature implementation. Interviews user, researches options, confirms choice, gathers implementation notes. Creates .claude/plans file."
version: "1.0.0"
dependencies: []
allowed-tools: ["WebSearch", "WebFetch", "Read", "Write"]
---

# Researching Features Skill

Research external APIs, services, and libraries for implementing features.

## Purpose
When user requests:
- External API integration (payments, auth, messaging, etc.)
- Library/package selection ("what's best for X?")
- Service comparison (cloud providers, databases, analytics)
- Third-party tool evaluation

**NOT for:**
- Internal codebase search (use scout skill)
- Implementation planning of known solutions (use plan skill)

## Workflow

### 1. User Interview (if details unclear)
Ask about:
- **Feature requirements**: What functionality needed?
- **Constraints**: Budget (free/paid), scale, compliance needs
- **Preferences**: Specific tech stack, vendor preferences, setup complexity
- **Timeline**: Proof-of-concept vs production-ready

If user provided details, skip to Step 2.

### 2. Research Options
Use WebSearch to find top 3-5 options:
- Official documentation
- Comparison articles
- Community reviews
- Pricing information
- Integration complexity

Research focus:
- Compatibility with project tech stack
- Ease of integration
- Cost structure
- Reliability and support
- Community adoption

### 3. Present Options & Get Confirmation
Summarize findings:
```
Found 3 options for [feature]:

1. [Option A]
   - Pros: [key advantages]
   - Cons: [limitations]
   - Cost: [pricing]
   - Best for: [use case]

2. [Option B]
   - Pros: [key advantages]
   - Cons: [limitations]
   - Cost: [pricing]
   - Best for: [use case]

3. [Option C]
   - Pros: [key advantages]
   - Cons: [limitations]
   - Cost: [pricing]
   - Best for: [use case]

Recommendation: [Option X] because [reason]

Proceed with [Option X]?
```

Wait for user confirmation.

### 4. Gather Implementation Notes
After confirmation, use WebFetch on official docs to gather:

**Setup & Authentication:**
- Account creation steps
- API key/credential generation
- SDK/library installation
- Environment configuration

**Core Integration:**
- Main endpoints/methods
- Request/response formats
- Authentication patterns
- Example code snippets

**Error Handling:**
- Common error codes
- Retry strategies
- Fallback options

**Production Considerations:**
- Rate limits
- Pricing/quotas
- Webhooks/callbacks
- Testing/sandbox environments

### 5. Create Implementation Plan
Write `.claude/plans/[feature-name].md`:

```markdown
# Implementation Plan: [Feature Name]
*External service: [Service Name]*
*Generated: [timestamp]*

## Service Overview
- Provider: [name]
- Pricing: [free tier / paid]
- Documentation: [link]
- SDK: [language/package]

## Setup Steps
1. Create account at [URL]
2. Generate API key/credentials
3. Install SDK: [package install command]
4. Configure environment variables:
   - [VAR_NAME]: [description]

## Integration Guide

### Authentication
[How to authenticate requests]

### Core Functionality
[Main features to implement]

### Example Implementation
```[language]
[Code example showing typical usage]
```

### Error Handling
[How to handle errors, retries]

## Testing
- Sandbox environment: [URL/config]
- Test credentials: [how to get them]
- Validation steps: [how to verify integration]

## Production Checklist
- [ ] Environment variables set
- [ ] Error handling implemented
- [ ] Rate limiting handled
- [ ] Webhooks configured (if applicable)
- [ ] Billing alerts set up
- [ ] Monitoring/logging added

## Resources
- Docs: [link]
- Examples: [link]
- Community: [forum/discord/slack]
- Support: [contact method]
```

### 6. Return Summary
```
Created implementation plan: .claude/plans/[feature-name].md
Next steps: Review plan, then use scout → plan → build workflow for implementation.
```

## Example Executions

### Example 1: Payment Integration
**User request:** "Add payment processing. Free tier preferred."

**Process:**
1. Ask: "What payment methods? (credit card, digital wallets, crypto?)"
2. Research: Stripe, PayPal, Square
3. Present options with free tier comparison
4. User confirms: Stripe
5. WebFetch Stripe docs for setup, API, webhooks
6. Create `.claude/plans/payment-stripe.md` with integration guide

### Example 2: Email Service
**User request:** "What's the best email service for transactional emails?"

**Process:**
1. Ask: "Email volume? Template needs? SMTP vs API?"
2. Research: SendGrid, Mailgun, AWS SES
3. Present options with pricing/features
4. User confirms: SendGrid
5. Gather SendGrid API docs, templates, deliverability best practices
6. Create `.claude/plans/email-sendgrid.md`

### Example 3: Authentication Provider
**User request:** "Need OAuth login with Google, GitHub."

**Process:**
1. Ask: "Self-hosted or managed? User data requirements?"
2. Research: Auth0, Supabase Auth, NextAuth, Firebase Auth
3. Present options (managed vs open-source)
4. User confirms: Supabase Auth
5. Gather Supabase Auth docs, OAuth setup, user management
6. Create `.claude/plans/auth-supabase.md`

## Plan File Naming Convention

`.claude/plans/[category]-[service].md`

Examples:
- `payment-stripe.md`
- `email-sendgrid.md`
- `auth-supabase.md`
- `analytics-mixpanel.md`
- `storage-s3.md`

## Success Criteria

- ✅ User requirements clarified
- ✅ Researched 3+ options
- ✅ Presented comparison with recommendation
- ✅ User confirmed choice
- ✅ Implementation plan created in `.claude/plans/`
- ✅ Plan includes setup, integration, examples, production checklist

## Error Handling

**If research finds no good options:**
Report to user: "No clear match found. Consider: [alternative approaches]."

**If documentation unavailable:**
Inform user: "Limited docs available for [service]. Recommend choosing [alternative] instead."

**If user requirements conflict:**
Ask for prioritization: "Can't find service that's both free AND supports [feature]. Which is more important?"

## Distinct from Other Skills

| Researching Features | NOT Researching Features |
|---------------------|-------------------------|
| External APIs/services | Internal codebase (scout) |
| Library/tool selection | Implementation planning (plan) |
| Third-party integrations | Building from scratch |
| Service comparison | Code execution |
