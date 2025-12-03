# Architecture Documentation

This directory contains technical architecture decisions and design patterns for BuildFlow.

## Key Documents

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Complete architecture overview
  - DDD structure (Domain, Application, Infrastructure)
  - Event-Driven communication patterns
  - CQRS implementation
  - Multi-tenancy design

- **[decisions/](decisions/)** - Architecture Decision Records (ADRs)
  - 14 ADRs documenting key technical decisions
  - See [decisions/SUMMARY.md](decisions/SUMMARY.md) for index

## Core Architectural Patterns

### Domain-Driven Design (DDD)
- Pure domain layer (no framework dependencies)
- Aggregates as consistency boundaries
- Value Objects for immutability
- Repository interfaces for persistence abstraction

**See:** [ADR-011: Domain-Driven Design](decisions/011-domain-driven-design.md)

### Event-Driven Architecture
- Domain events for cross-context communication
- Event bus for decoupling
- Event sourcing for audit trail
- Sagas for complex workflows

**See:** [ADR-012: Event-Driven Architecture](decisions/012-event-driven-architecture.md)

### CQRS (Basic)
- Separate read and write models
- Projectors for read model updates
- Denormalized data for queries
- Optimized for different access patterns

**See:** [ADR-013: CQRS Basic](decisions/013-cqrs-basic.md)

## Key Decisions

### Strategic
- **Laravel-First Strategy** - [ADR-014](decisions/014-laravel-first-strategy.md)
- **API-First Approach** - [ADR-002](decisions/002-api-first-approach.md)
- **Multi-Repository Strategy** - [ADR-001](decisions/001-multi-repository-strategy.md)

### Infrastructure
- **PostgreSQL Primary Database** - [ADR-007](decisions/007-postgresql-primary-database.md)
- **Cloud File Storage** - [ADR-005](decisions/005-cloud-file-storage.md)
- **JWT Authentication** - [ADR-003](decisions/003-jwt-authentication.md)

### Quality
- **Contract Testing Strategy** - [ADR-008](decisions/008-contract-testing-strategy.md)
- **Feature Flags for Tiers** - [ADR-009](decisions/009-feature-flags-for-tiers.md)

## Related Documentation

- [Domain Model](../domain/) - Business concepts and rules
- [Implementation Guide](../implementation/) - How to build
- [API Contract](../api/) - API specification
