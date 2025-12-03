# Domain Documentation

This directory contains domain modeling documentation using Domain-Driven Design (DDD) principles.

## Key Documents

- **[event-storming.md](event-storming.md)** - Event storming workshop results
  - 8 Bounded Contexts identified
  - Domain events catalog
  - Aggregate boundaries
  - Cross-context communication

## Bounded Contexts

1. **QuoteManagement** - Quote creation, versioning, acceptance
2. **ProjectManagement** - Active project tracking
3. **InvoiceManagement** - Invoicing and payments
4. **ClientManagement** - Client information
5. **DocumentManagement** - File storage
6. **UserManagement** - Authentication & teams
7. **ClientPortal** - Client self-service
8. **NotificationManagement** - Alerts & reminders

## Domain-Driven Design

BuildFlow uses DDD to model complex business logic. See:
- [ADR-011: Domain-Driven Design](../architecture/decisions/011-domain-driven-design.md)
- [ADR-012: Event-Driven Architecture](../architecture/decisions/012-event-driven-architecture.md)
- [Getting Started Guide](../implementation/getting-started-laravel.md)

## Related Documentation

- [Business Requirements](../business/requirements.md) - What we're building
- [Architecture](../architecture/ARCHITECTURE.md) - How it's structured
- [Implementation Guide](../implementation/getting-started-laravel.md) - How to build
