# Implementation Guides

This directory contains hands-on guides for implementing BuildFlow using Laravel with DDD, Event-Driven, and CQRS patterns.

## Key Documents

- **[roadmap.md](roadmap.md)** - 10-week implementation plan
  - Phase 0: Foundation (✅ Complete)
  - Phase 1: Quote Management - Showcase (🚧 In Progress)
  - Phase 2: Event-Driven Communication
  - Phase 3: CQRS Implementation
  - Phase 4-5: Complete & Polish

- **[getting-started-laravel.md](getting-started-laravel.md)** - Step-by-step TDD guide
  - Laravel + DDD setup
  - Value Objects implementation
  - Aggregate creation
  - Event handling
  - Testing strategies

- **[testing-strategy.md](testing-strategy.md)** - Testing approach
  - Test pyramid
  - TDD workflow
  - Coverage requirements (80%+ overall, 90%+ domain)
  - Architecture tests

- **[claude-code-guide.md](claude-code-guide.md)** - Using Claude Code for development

## Current Phase: Quote Management (Phase 1)

**Goal:** Build one complete bounded context (QuoteManagement) with full DDD patterns as a showcase.

**What to Build:**
- Quote Aggregate (with full DDD patterns)
- Value Objects (Money, QuoteStatus, etc.)
- Domain Events (QuoteDraftCreated, QuoteAccepted, etc.)
- Application Services
- Tests (Unit, Integration, Feature)

**Follow:** [getting-started-laravel.md](getting-started-laravel.md)

## Development Approach

1. **TDD First** - Write tests before implementation
2. **Domain Pure** - No Laravel in domain layer
3. **Event-Driven** - Use domain events for communication
4. **Architecture Tests** - Enforce layer boundaries

## Related Documentation

- [Domain Model](../domain/) - Business concepts to implement
- [Architecture](../architecture/) - Patterns and decisions
- [API Contract](../api/) - API specification
