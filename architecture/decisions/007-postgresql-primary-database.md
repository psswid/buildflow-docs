# ADR 007: PostgreSQL as Primary Database

**Status:** Accepted

**Date:** 2024-11-12

---

## Context

BuildFlow needs a relational database that:
- Supports complex queries and relationships
- Works across all backend implementations
- Handles JSON data (for settings, metadata)
- Provides good performance at scale
- Has good Laravel/Symfony support

---

## Decision

**Use PostgreSQL as the primary database for all implementations.**

### Key Features Used
- JSON/JSONB columns for flexible data (tags, settings)
- Full-text search for clients/projects
- Foreign key constraints for data integrity
- Transactions for quote/invoice workflows
- Indexes for multi-tenant performance

### Version
- PostgreSQL 14+ (for better JSON performance)

---

## Consequences

### Positive
- ✅ Feature-rich and mature
- ✅ Excellent JSON support
- ✅ Advanced indexing (GIN, GIST)
- ✅ Full-text search built-in
- ✅ Open source and free
- ✅ Great performance at scale
- ✅ Strong data integrity
- ✅ Good ORM support in all frameworks

### Negative
- ⚠️ Slightly more complex than MySQL
- ⚠️ Larger resource footprint
- ⚠️ Need to learn PostgreSQL-specific features

---

## Alternatives Considered

**MySQL**: Rejected - JSON support less mature, fewer advanced features  
**SQLite**: Rejected - not suitable for multi-user SaaS  
**MongoDB**: Rejected - need relational structure and ACID guarantees  
**MariaDB**: Rejected - similar to MySQL, no advantages

---

## Implementation Notes

### Laravel
```php
// config/database.php
'pgsql' => [
    'driver' => 'pgsql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'database' => env('DB_DATABASE', 'buildflow'),
    // ...
]
```

### Symfony
```yaml
# config/packages/doctrine.yaml
doctrine:
    dbal:
        driver: 'pdo_pgsql'
        server_version: '14'
```

---

**Related:** ADR-004 (multi-tenancy uses PostgreSQL features)
