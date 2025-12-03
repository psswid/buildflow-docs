# BuildFlow Architecture

## 🎯 Architecture Philosophy

BuildFlow follows **"Depth over Breadth"** principle - one framework (Laravel) implemented with enterprise-grade patterns rather than multiple shallow implementations.

### Core Principles

1. **Domain-Driven Design** - Business logic in pure domain layer
2. **Event-Driven Architecture** - Decoupled contexts via domain events
3. **CQRS** - Separate read/write models for optimization
4. **Contract-First** - OpenAPI spec as single source of truth
5. **Test-Driven Development** - 80%+ coverage with architecture tests

---

## 🏗️ High-Level Architecture

```
┌──────────────────────────────────────────────────────────┐
│               BuildFlow Ecosystem                        │
└──────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                  buildflow-docs                         │
│         (Central Documentation & Contract)              │
│                                                         │
│  • OpenAPI 3.0 Specification (API Contract)             │
│  • Business Requirements (45KB)                         │
│  • Architecture Decision Records (13 ADRs)              │
│  • Domain Analysis (Event Storming)                     │
│  • Implementation Guides                                │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ↓ Defines Contract
                      
┌─────────────────────────────────────────────────────────┐
│              buildflow-laravel-api                      │
│           (Production Backend - Laravel 11)             │
│                                                         │
│  Enterprise Patterns:                                   │
│  • Domain-Driven Design (DDD)                           │
│  • Event-Driven Architecture                            │
│  • CQRS with Read Models                                │
│  • Multi-Tenancy (Row-Level)                            │
│  • JWT Authentication                                   │
│                                                         │
│  Status: 🚧 Phase 1 - Quote Management                 │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ↓ REST API (JSON)
                      
┌─────────────────────────────────────────────────────────┐
│              buildflow-react-web                        │
│              (Frontend - React 18)                      │
│                                                         │
│  • TypeScript                                           │
│  • TanStack Query                                       │
│  • Tailwind CSS + Shadcn/ui                             │
│                                                         │
│  Status: ⏳ Planned (Phase 2)                           │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│         Future Optional Implementations                 │
│         (Learning / Comparison Only)                    │
│                                                         │
│  • buildflow-symfony-api (Symfony experiment)           │
│  • buildflow-nextjs-api (Node.js comparison)            │
│                                                         │
│  Priority: LOW - Only after Laravel is 100% complete   │
└─────────────────────────────────────────────────────────┘
```

---

## 🏛️ Laravel Implementation Architecture

### Modular Monolith Structure

BuildFlow uses **modular monolith** with clear **bounded contexts** (not microservices):

```
buildflow-laravel-api/
│
├── app/
│   │
│   ├── SharedKernel/              # Shared across all domains
│   │   ├── Domain/
│   │   │   ├── AggregateRoot.php      # Base for all aggregates
│   │   │   ├── DomainEvent.php        # Event interface
│   │   │   ├── ValueObject.php        # Base for value objects
│   │   │   └── Entity.php             # Base entity
│   │   │
│   │   └── Infrastructure/
│   │       ├── EventBus/              # Event dispatching
│   │       └── Persistence/           # Shared persistence concerns
│   │
│   ├── Domains/                   # Bounded Contexts
│   │   │
│   │   ├── QuoteManagement/      # 🎯 SHOWCASE CONTEXT
│   │   │   │
│   │   │   ├── Domain/           # Pure business logic (NO Laravel)
│   │   │   │   ├── Quote.php             # Aggregate Root
│   │   │   │   ├── LineItem.php          # Entity
│   │   │   │   ├── ValueObjects/
│   │   │   │   │   ├── QuoteId.php
│   │   │   │   │   ├── QuoteNumber.php
│   │   │   │   │   ├── QuoteStatus.php
│   │   │   │   │   ├── Money.php
│   │   │   │   │   └── TaxRate.php
│   │   │   │   ├── Events/
│   │   │   │   │   ├── QuoteDraftCreated.php
│   │   │   │   │   ├── QuoteSent.php
│   │   │   │   │   └── QuoteAccepted.php  # ⭐ Triggers ProjectCreated
│   │   │   │   ├── Exceptions/
│   │   │   │   │   ├── CannotSendEmptyQuote.php
│   │   │   │   │   └── QuoteHasExpired.php
│   │   │   │   └── QuoteRepository.php    # Interface
│   │   │   │
│   │   │   ├── Application/      # Use Cases / Orchestration
│   │   │   │   ├── Commands/
│   │   │   │   │   ├── CreateQuoteDraft.php
│   │   │   │   │   ├── AddLineItem.php
│   │   │   │   │   ├── SendQuote.php
│   │   │   │   │   └── AcceptQuote.php
│   │   │   │   ├── Handlers/
│   │   │   │   │   ├── CreateQuoteDraftHandler.php
│   │   │   │   │   ├── AddLineItemHandler.php
│   │   │   │   │   ├── SendQuoteHandler.php
│   │   │   │   │   └── AcceptQuoteHandler.php
│   │   │   │   └── Queries/
│   │   │   │       ├── QuoteQueryService.php
│   │   │   │       └── QuoteListQuery.php
│   │   │   │
│   │   │   └── Infrastructure/   # Technical Details (Laravel)
│   │   │       ├── Persistence/
│   │   │       │   ├── EloquentQuoteRepository.php
│   │   │       │   ├── QuoteEloquentModel.php
│   │   │       │   └── Projectors/
│   │   │       │       ├── QuoteListProjector.php
│   │   │       │       └── QuoteDashboardStatsProjector.php
│   │   │       ├── Http/
│   │   │       │   ├── QuoteController.php
│   │   │       │   ├── Requests/
│   │   │       │   │   ├── CreateQuoteRequest.php
│   │   │       │   │   └── AcceptQuoteRequest.php
│   │   │       │   └── Resources/
│   │   │       │       └── QuoteResource.php
│   │   │       └── EventListeners/
│   │   │           └── SendQuoteEmail.php
│   │   │
│   │   ├── ProjectManagement/    # Second Context
│   │   │   ├── Domain/
│   │   │   │   ├── Project.php
│   │   │   │   └── Events/
│   │   │   │       └── ProjectCreated.php
│   │   │   ├── Application/
│   │   │   └── Infrastructure/
│   │   │       └── EventListeners/
│   │   │           └── CreateProjectWhenQuoteAccepted.php
│   │   │
│   │   ├── ClientManagement/
│   │   ├── InvoiceManagement/
│   │   ├── DocumentManagement/
│   │   ├── ClientPortal/
│   │   ├── TeamManagement/
│   │   └── Notifications/
│   │
│   └── Http/
│       └── Middleware/
│           └── EnsureOrganizationScope.php
│
├── database/
│   ├── migrations/               # Schema evolution
│   └── seeders/
│
├── tests/
│   ├── Unit/                     # Domain logic (50% of tests)
│   │   └── Domains/
│   │       └── QuoteManagement/
│   │           ├── QuoteTest.php
│   │           ├── ValueObjects/
│   │           └── Events/
│   │
│   ├── Integration/              # Cross-layer (25% of tests)
│   │   └── Domains/
│   │       └── QuoteManagement/
│   │           └── AcceptQuoteFlowTest.php
│   │
│   ├── Feature/                  # HTTP/API (20% of tests)
│   │   └── Api/
│   │       └── QuoteApiTest.php
│   │
│   ├── Architecture/             # Structure enforcement
│   │   ├── DomainLayerTest.php
│   │   └── NamingConventionTest.php
│   │
│   └── Contract/                 # OpenAPI compliance
│       └── QuoteContractTest.php
│
└── config/
    └── buildflow.php             # Feature flags config
```

---

## 🎨 Bounded Contexts (Domains)

### 1. QuoteManagement 💼 **[Phase 1 - SHOWCASE]**

**Responsibility:** Creating and managing quotes

**Aggregate:** Quote (with LineItems)

**Key Events:**
- `QuoteDraftCreated`
- `LineItemAdded`
- `QuoteCalculated`
- `QuoteSent`
- `QuoteAccepted` ⭐ **Triggers project creation**
- `QuoteRejected`

**Business Rules:**
- Cannot send empty quote
- Sent quote cannot be edited
- Only sent quotes can be accepted
- Expired quotes cannot be accepted
- Total = sum(lineItems) + tax - discount

---

### 2. ProjectManagement 🏗️ **[Phase 4]**

**Responsibility:** Tracking project execution

**Aggregate:** Project (with Milestones)

**Key Events:**
- `ProjectCreated` (from QuoteAccepted)
- `ProjectStarted`
- `MilestoneCompleted`
- `ProjectCompleted`

**Listens To:**
- `QuoteAccepted` → Creates new Project

---

### 3. ClientManagement 👥 **[Phase 4]**

**Responsibility:** Managing client relationships

**Aggregate:** Client (simple)

**Key Events:**
- `ClientRegistered`
- `ClientTagged`

---

### 4. InvoiceManagement 💰 **[Phase 4]**

**Responsibility:** Billing and payments

**Aggregate:** Invoice (with Payments)

**Key Events:**
- `InvoiceGenerated` (from Quote)
- `InvoiceSent`
- `PaymentReceived`
- `InvoiceBecameOverdue`

---

### 5. DocumentManagement 📄 **[Phase 4]**

**Responsibility:** File storage and organization

**Aggregate:** Document

**Key Events:**
- `DocumentUploaded`
- `DocumentShared`

---

### 6. ClientPortal 🌐 **[Phase 4 - Pro Tier]**

**Responsibility:** Client self-service

**Listens To:** All relevant events to sync portal view

---

### 7. TeamManagement 👨‍👩‍👧‍👦 **[Phase 4 - Business Tier]**

**Responsibility:** Multi-user access

---

### 8. Notifications 📧 **[Cross-Cutting]**

**Responsibility:** Email/SMS communication

**Listens To:** All significant events

---

## 🔄 Event-Driven Communication

### How Contexts Communicate

```
┌──────────────────────────────────────────┐
│      Quote Management Context            │
│                                          │
│  User accepts quote                      │
│    ↓                                     │
│  Quote.accept() called                   │
│    ↓                                     │
│  QuoteAccepted event recorded            │
│    ↓                                     │
│  Repository saves Quote                  │
│    ↓                                     │
│  Events dispatched to Event Bus          │
└────────────────┬─────────────────────────┘
                 │
                 ↓ QuoteAccepted Event
                 
        ┌────────────────┐
        │   Event Bus    │ (Laravel Event Dispatcher)
        └────┬──────┬────┘
             │      │
     ┌───────┘      └────────┐
     ↓                       ↓
┌─────────────────┐   ┌─────────────────┐
│ Project Context │   │  Notifications  │
│                 │   │    Context      │
│ Listener:       │   │                 │
│ CreateProject   │   │ Listener:       │
│ WhenQuoteAccepted│   │ SendQuoteEmail │
└─────────────────┘   └─────────────────┘
```

**Benefits:**
- ✅ Zero coupling between contexts
- ✅ Easy to add new reactions
- ✅ Testable in isolation
- ✅ Audit trail of all events

---

## 📊 CQRS - Read/Write Separation

### Write Side (Commands)

**Optimized for:** Business logic, consistency, validation

```php
// Write through Domain Model
$quote = Quote::createDraft(...);
$quote->addLineItem($item);
$quote->send();

$repository->save($quote);  // Dispatches events
```

**Characteristics:**
- Uses Aggregates (normalized)
- Enforces business rules
- Records domain events
- Transactional consistency

---

### Read Side (Queries)

**Optimized for:** Fast queries, complex joins, dashboards

```php
// Read from denormalized view
$quotes = QuoteListView::forOrganization($orgId)
    ->where('status', 'sent')
    ->paginate(20);

$stats = QuoteDashboardStats::where('organization_id', $orgId)->first();
```

**Characteristics:**
- Denormalized tables
- Pre-calculated aggregates
- Optimized indexes
- Eventually consistent

---

### Synchronization (Projectors)

```php
// QuoteListProjector listens to events
class QuoteListProjector
{
    public function onQuoteAccepted(QuoteAccepted $event): void
    {
        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'status' => 'accepted',
                'accepted_at' => $event->acceptedAt,
            ]);
        
        // Update dashboard stats
        $this->updateStats($event->organizationId);
    }
}
```

---

## 🔐 Multi-Tenancy

### Row-Level Isolation

Every tenant-scoped table has `organization_id`:

```php
Schema::create('quotes', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->uuid('organization_id');  // Tenant isolation
    $table->uuid('client_id');
    // ... other fields
    
    $table->foreign('organization_id')
          ->references('id')->on('organizations');
    
    $table->index(['organization_id', 'status']);  // Compound index
});
```

### Global Scopes

```php
// Automatically filters by organization
class Quote extends Model
{
    protected static function booted()
    {
        static::addGlobalScope('organization', function (Builder $builder) {
            if (auth()->check()) {
                $builder->where('organization_id', auth()->user()->organization_id);
            }
        });
    }
}
```

---

## 🧪 Testing Strategy

### Test Pyramid

```
         ┌─────────────────┐
         │  E2E Tests      │  5%
         └─────────────────┘
       ┌───────────────────────┐
       │   Feature Tests       │  20%
       │   (API, HTTP)         │
       └───────────────────────┘
     ┌─────────────────────────────┐
     │   Integration Tests         │  25%
     │   (Cross-context, Events)   │
     └─────────────────────────────┘
   ┌───────────────────────────────────┐
   │      Unit Tests                   │  50%
   │      (Domain, Value Objects)      │
   └───────────────────────────────────┘
```

### Coverage Goals

| Layer | Target | Priority |
|-------|--------|----------|
| Domain | 90%+ | Critical |
| Application | 80%+ | High |
| Infrastructure | 60%+ | Medium |
| **Overall** | **80%+** | High |

---

## 🚀 Deployment Architecture

### Development
```
Docker Compose:
  - Laravel (PHP-FPM)
  - PostgreSQL
  - Redis
  - Mailhog (email testing)
```

### Production (Planned)
```
AWS / DigitalOcean:
  - Laravel (behind Nginx)
  - Managed PostgreSQL
  - Redis Cluster
  - S3 / Spaces (files)
  - CloudFlare CDN
```

---

## 📋 Technology Decisions

All major architectural decisions are documented as ADRs:

| Decision | ADR | Impact |
|----------|-----|--------|
| Multi-repo structure | [ADR-001](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/001-multi-repository-strategy.md) | 🔴 Critical |
| API-First approach | [ADR-002](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/002-api-first-approach.md) | 🔴 Critical |
| Domain-Driven Design | [ADR-011](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/011-domain-driven-design.md) | 🔴 Critical |
| Event-Driven Architecture | [ADR-012](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/012-event-driven-architecture.md) | 🔴 Critical |
| CQRS Basic | [ADR-013](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/013-cqrs-basic.md) | 🟡 Important |

[See full ADR index](https://github.com/psswid/buildflow-docs/tree/main/docs/architecture/decisions)

---

## 🎯 Why This Architecture?

### For Portfolio

**Traditional Portfolio Projects:**
- ❌ CRUD with Eloquent models
- ❌ Business logic in controllers
- ❌ No domain modeling
- ❌ Shallow implementation

**BuildFlow:**
- ✅ **DDD** - Shows understanding of complex domain modeling
- ✅ **Event-Driven** - Shows understanding of decoupling
- ✅ **CQRS** - Shows understanding of optimization
- ✅ **Architecture Tests** - Shows care for maintainability

### For Job Interviews

**When they ask about DDD:**
> "I used DDD in BuildFlow. The Quote aggregate has business rules like 'cannot send empty quote' enforced in the domain, not controllers. I can show you the code."

**When they ask about events:**
> "My Quote and Project contexts are decoupled via events. When Quote is accepted, it dispatches QuoteAccepted event, and ProjectManagement context listens and creates a Project. Zero coupling."

**When they ask about CQRS:**
> "I have separate read models. QuoteListView is denormalized with client names pre-joined. Projectors update it on events. Write model stays normalized."

---

## 📚 Further Reading

- [Domain Analysis](https://github.com/psswid/buildflow-docs/blob/main/DOMAIN_ANALYSIS_EVENT_STORMING.md)
- [Implementation Roadmap](https://github.com/psswid/buildflow-docs/blob/main/IMPLEMENTATION_ROADMAP.md)
- [Laravel DDD Starter Guide](https://github.com/psswid/buildflow-docs/blob/main/LARAVEL_DDD_STARTER_GUIDE.md)
- [Testing Strategy](https://github.com/psswid/buildflow-docs/blob/main/TESTING_STRATEGY.md)

---

**Last Updated:** 2024-11-12  
**Architecture Version:** 2.0 (Laravel-First with Enterprise Patterns)  
**Status:** Phase 1 Implementation
