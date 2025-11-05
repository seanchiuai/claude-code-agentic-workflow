# Project Documentation

This directory contains project-specific documentation referenced by agents during development.

## Purpose

Agents consult these docs during orchestrated workflows to:
- Understand project-specific coding standards
- Reference internal API contracts
- Apply domain-specific patterns
- Maintain consistency across implementations

## Structure

**`best-practices.md`** - Coding standards, architectural patterns, conventions
**`api-reference.md`** - Internal API documentation, endpoints, data contracts
**`{domain}-guide.md`** - Domain-specific implementation guides (e.g., auth-guide.md, data-guide.md)

## Usage

Agents automatically reference these docs when:
- Context-pool initialization (includes doc references)
- Codebase-search (finds relevant patterns)
- Execution-planner (applies project standards)
- Builders (implement according to documented patterns)

## Maintenance

Update docs when:
- New architectural patterns emerge
- API contracts change
- Domain-specific best practices solidify
- Common pitfalls discovered

Keep docs concise - agents have token limits. Focus on what's unique to this project, not general programming knowledge.
