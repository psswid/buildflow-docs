# BuildFlow Multi-Repo Architecture

## Overview

BuildFlow uses a **multi-repository architecture** with a **contract-first approach**. This document explains the architecture, rationale, and how all pieces fit together.

## 🏗️ Repository Structure

```
BuildFlow Ecosystem
│
├── buildflow-docs                     [Central Hub]
│   ├── API Contract (OpenAPI spec)
│   ├── Business Requirements
│   ├── Architecture Docs
│   └── Development Roadmap
│
├── buildflow-laravel-api             [Backend #1]
│   └── Implements API Contract
│
├── buildflow-symfony-api             [Backend #2]
│   └── Implements API Contract
│
├── buildflow-nextjs-api              [Backend #3]
│   └── Implements API Contract
│
├── buildflow-react-web               [Frontend]
│   └── Consumes API Contract
│
├── buildflow-android                 [Mobile - Future]
│   └── Consumes API Contract
│
└── buildflow-ios                     [Mobile - Future]
    └── Consumes API Contract
```

## 🎯 Design Principles

### 1. Contract-First Development

**The API contract is the source of truth.**

```
┌─────────────────────────────────────┐
│     buildflow-docs/api-contract     │
│          (OpenAPI 3.0 Spec)         │
│       ← Single Source of Truth →   │
└─────────────────────────────────────┘
                  ↓
    ┌─────────────┼─────────────┐
    ↓             ↓             ↓
┌────────┐   ┌────────┐   ┌────────┐
│Laravel │   │Symfony │   │Next.js │
│Backend │   │Backend │   │Backend │
└────────┘   └────────┘   └────────┘
    ↓             ↓             ↓
    └─────────────┼─────────────┘
                  ↓
         ┌────────────────┐
         │  React Frontend │
         │  (One Frontend) │
         └────────────────┘
```

**Workflow:**
1. Design API endpoint in OpenAPI spec
2. Implement in backend (Laravel/Symfony/Next.js)
3. Write contract tests to verify compliance
4. Build frontend feature that consumes endpoint
5. Frontend works with ALL backends

### 2. Technology Agnostic Business Logic

**Business requirements are framework-independent.**

The `business/requirements.md` document describes:
- ✅ WHAT the system should do
- ✅ WHY it should do it
- ✅ WHO will use it
- ❌ NOT how to implement it in specific framework

Each backend implements the same business logic in their framework's idioms:

```php
// Laravel
class Quote extends Model {
    public function calculateTotal(): float {
        return $this->lineItems->sum('total') 
             + ($this->subtotal * $this->tax_rate)
             - $this->discount;
    }
}
```

```php
// Symfony
class Quote {
    public function calculateTotal(): float {
        return array_sum(array_column($this->lineItems, 'total'))
             + ($this->subtotal * $this->taxRate)
             - $this->discount;
    }
}
```

```typescript
// Next.js
class Quote {
    calculateTotal(): number {
        return this.lineItems.reduce((sum, item) => sum + item.total, 0)
             + (this.subtotal * this.taxRate)
             - this.discount;
    }
}
```

### 3. Separate Repositories, Shared Standards

**Each repo is independent but follows common standards.**

| Aspect | Strategy |
|--------|----------|
| **API Contract** | Shared OpenAPI spec in `buildflow-docs` |
| **Data Model** | Same entities, relationships in all backends |
| **Business Logic** | Same calculations, validations, workflows |
| **Authentication** | JWT tokens, same structure |
| **Error Handling** | Same error codes and formats |
| **Testing** | Contract tests verify API compliance |
| **Documentation** | Centralized in `buildflow-docs` |

## 📋 Repository Details

### buildflow-docs (Central Hub)

**Purpose:** Single source of truth for API contract, business requirements, and architecture.

**Contents:**
- OpenAPI 3.0 specification
- Business requirements (45KB)
- Development roadmap (70 issues)
- Architecture diagrams
- User documentation

**No Code:** This is documentation only.

**Why separate?**
- Documentation lives longer than any implementation
- Can reference from all repos
- GitHub Pages for public docs
- Issues/discussions in one place

### buildflow-laravel-api

**Purpose:** Laravel implementation of BuildFlow API.

**Tech Stack:**
- Laravel 11+
- Eloquent ORM
- MySQL/PostgreSQL
- PHPUnit for testing
- Laravel Sanctum for auth

**Structure:**
```
buildflow-laravel-api/
├── app/
│   ├── Models/              # Eloquent models
│   ├── Http/
│   │   ├── Controllers/     # API controllers
│   │   ├── Requests/        # Form requests (validation)
│   │   └── Resources/       # JSON resources
│   ├── Services/            # Business logic
│   └── Policies/            # Authorization
├── database/
│   ├── migrations/          # Database migrations
│   └── seeders/             # Test data
├── routes/
│   └── api.php              # API routes
├── tests/
│   ├── Unit/                # Unit tests
│   ├── Feature/             # Feature tests
│   └── Contract/            # API contract tests
└── README.md
```

**README.md includes:**
- Setup instructions
- Link to API contract
- Testing guide
- Deployment guide

### buildflow-symfony-api

**Purpose:** Symfony implementation of BuildFlow API.

**Tech Stack:**
- Symfony 7+
- Doctrine ORM
- PostgreSQL
- PHPUnit for testing
- Symfony Security

**Structure:**
```
buildflow-symfony-api/
├── src/
│   ├── Entity/              # Doctrine entities
│   ├── Controller/          # API controllers
│   ├── Repository/          # Data repositories
│   ├── Service/             # Business logic
│   └── Security/            # Authentication
├── config/
│   ├── packages/            # Bundle config
│   └── routes/              # API routes
├── migrations/              # Database migrations
├── tests/
│   ├── Unit/
│   ├── Functional/
│   └── Contract/            # API contract tests
└── README.md
```

### buildflow-nextjs-api

**Purpose:** Next.js API routes implementation.

**Tech Stack:**
- Next.js 14+
- Prisma ORM
- PostgreSQL
- Jest for testing
- NextAuth.js

**Structure:**
```
buildflow-nextjs-api/
├── src/
│   ├── app/
│   │   └── api/             # API routes
│   ├── lib/
│   │   ├── db.ts            # Database client
│   │   ├── auth.ts          # Authentication
│   │   └── services/        # Business logic
│   └── types/               # TypeScript types
├── prisma/
│   ├── schema.prisma        # Database schema
│   └── migrations/          # Migrations
├── tests/
│   ├── unit/
│   ├── integration/
│   └── contract/            # API contract tests
└── README.md
```

### buildflow-react-web

**Purpose:** React SPA frontend that works with ANY backend.

**Tech Stack:**
- React 18+
- TypeScript
- Vite
- React Router
- TanStack Query (React Query)
- Tailwind CSS + Shadcn/ui

**Structure:**
```
buildflow-react-web/
├── src/
│   ├── api/                 # API client (generated from OpenAPI)
│   ├── components/          # Reusable components
│   ├── features/            # Feature modules
│   │   ├── clients/
│   │   ├── quotes/
│   │   ├── projects/
│   │   └── invoices/
│   ├── hooks/               # Custom hooks
│   ├── routes/              # Route definitions
│   ├── types/               # TypeScript types (from OpenAPI)
│   └── utils/
├── public/
├── tests/
└── README.md
```

**Key Feature:** Backend URL is configurable:
```typescript
// .env
VITE_API_BASE_URL=http://localhost:8000/api/v1  // Laravel
// or
VITE_API_BASE_URL=http://localhost:8001/api/v1  // Symfony
// or
VITE_API_BASE_URL=http://localhost:3000/api/v1  // Next.js
```

## 🔄 Development Workflow

### Phase 1: Design API Endpoint

**In `buildflow-docs/api-contract/openapi.yaml`:**

```yaml
paths:
  /clients:
    get:
      summary: List all clients
      responses:
        '200':
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Client'
```

### Phase 2: Implement in Backend

**In `buildflow-laravel-api`:**

```php
// routes/api.php
Route::get('/clients', [ClientController::class, 'index']);

// app/Http/Controllers/ClientController.php
public function index()
{
    $clients = Client::where('organization_id', auth()->user()->organization_id)
        ->paginate(20);
    
    return ClientResource::collection($clients);
}
```

### Phase 3: Write Contract Tests

**In `buildflow-laravel-api/tests/Contract/ClientTest.php`:**

```php
public function test_list_clients_matches_contract()
{
    $response = $this->getJson('/api/v1/clients');
    
    // Validate against OpenAPI schema
    $response->assertJsonStructure([
        'data' => [
            '*' => ['id', 'name', 'email', 'phone', 'created_at']
        ],
        'meta' => ['current_page', 'total']
    ]);
}
```

### Phase 4: Build Frontend Feature

**In `buildflow-react-web/src/features/clients/ClientList.tsx`:**

```typescript
import { useQuery } from '@tanstack/react-query';
import { clientsApi } from '@/api/clients'; // Auto-generated from OpenAPI

export function ClientList() {
  const { data, isLoading } = useQuery({
    queryKey: ['clients'],
    queryFn: () => clientsApi.list()
  });
  
  // Render UI...
}
```

### Phase 5: Test with Multiple Backends

```bash
# Test with Laravel
VITE_API_BASE_URL=http://localhost:8000/api/v1 npm run dev

# Test with Symfony
VITE_API_BASE_URL=http://localhost:8001/api/v1 npm run dev

# Test with Next.js
VITE_API_BASE_URL=http://localhost:3000/api/v1 npm run dev
```

## 🧪 Contract Testing Strategy

### What is Contract Testing?

Contract tests verify that backend implementations match the OpenAPI specification.

### Contract Test Suite

Each backend repo has a `tests/Contract/` directory:

```php
// tests/Contract/ContractTestCase.php
abstract class ContractTestCase extends TestCase
{
    protected function assertMatchesOpenApiSchema(
        TestResponse $response, 
        string $schemaName
    ) {
        // Load OpenAPI spec from buildflow-docs
        $spec = yaml_parse_file(__DIR__ . '/../../openapi.yaml');
        
        // Validate response against schema
        $schema = $spec['components']['schemas'][$schemaName];
        $this->assertTrue(
            $this->validateAgainstSchema($response->json(), $schema)
        );
    }
}
```

```php
// tests/Contract/ClientContractTest.php
class ClientContractTest extends ContractTestCase
{
    public function test_list_clients_matches_contract()
    {
        $response = $this->getJson('/api/v1/clients');
        
        $response->assertStatus(200);
        $this->assertMatchesOpenApiSchema($response, 'ClientList');
    }
    
    public function test_create_client_matches_contract()
    {
        $response = $this->postJson('/api/v1/clients', [
            'name' => 'Test Client',
            'email' => 'test@example.com'
        ]);
        
        $response->assertStatus(201);
        $this->assertMatchesOpenApiSchema($response, 'Client');
    }
}
```

### CI/CD Pipeline

```yaml
# .github/workflows/test.yml
name: Contract Tests

on: [push, pull_request]

jobs:
  contract-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      # Checkout buildflow-docs to get OpenAPI spec
      - name: Checkout API Contract
        uses: actions/checkout@v3
        with:
          repository: yourusername/buildflow-docs
          path: api-contract
      
      - name: Run Contract Tests
        run: php artisan test --testsuite=Contract
```

## 📊 Pros and Cons

### Multi-Repo Advantages ✅

1. **Independence** - Each implementation can evolve separately
2. **Portfolio** - Each repo is standalone portfolio piece
3. **Learning** - Compare different approaches
4. **CI/CD** - Separate pipelines, faster builds
5. **Teams** - Different teams can work independently
6. **Deployment** - Deploy backends independently
7. **Clarity** - Clear separation of concerns

### Multi-Repo Challenges ⚠️

1. **Coordination** - Changes to API contract affect all repos
2. **Duplication** - Some code patterns repeated
3. **Complexity** - More repos to manage
4. **Synchronization** - Keeping implementations in sync

### Mitigation Strategies

1. **API Contract First** - Design contract before implementation
2. **Contract Tests** - Automated verification of compliance
3. **Shared Docs** - Centralized in buildflow-docs
4. **Versioning** - Semantic versioning for API
5. **Communication** - Clear process for API changes

## 🔄 API Versioning Strategy

### Version in URL
```
/api/v1/clients
/api/v2/clients (future)
```

### Breaking Changes
- Increment major version (v1 → v2)
- Maintain v1 for 6 months minimum
- Document migration path

### Non-Breaking Changes
- Update minor version in OpenAPI spec
- Add new endpoints or optional fields
- All backends must implement within 1 sprint

## 🚀 Getting Started

### For Backend Developers

1. **Read the contract**
   ```bash
   git clone https://github.com/yourusername/buildflow-docs.git
   cat buildflow-docs/api-contract/openapi.yaml
   ```

2. **Choose your framework**
   - Laravel → clone buildflow-laravel-api
   - Symfony → clone buildflow-symfony-api
   - Next.js → clone buildflow-nextjs-api

3. **Follow the setup guide** in that repo's README

4. **Implement features** following the roadmap

5. **Write contract tests** to verify compliance

### For Frontend Developers

1. **Read the contract**
   ```bash
   git clone https://github.com/yourusername/buildflow-docs.git
   ```

2. **Clone frontend**
   ```bash
   git clone https://github.com/yourusername/buildflow-react-web.git
   ```

3. **Generate TypeScript types** from OpenAPI spec
   ```bash
   npm run generate-types
   ```

4. **Point to any backend** in `.env`

5. **Build features** that consume the API

## 📖 Additional Resources

- [OpenAPI Specification](https://swagger.io/specification/)
- [Contract Testing](https://martinfowler.com/bliki/ContractTest.html)
- [Microservices Patterns](https://microservices.io/patterns/index.html)
- [API Design Best Practices](https://restfulapi.net/)

## 🤝 Contributing

See [buildflow-docs/CONTRIBUTING.md](https://github.com/yourusername/buildflow-docs/blob/main/CONTRIBUTING.md)

---

**Questions?** Open an issue in [buildflow-docs](https://github.com/yourusername/buildflow-docs/issues)

**Last Updated:** November 12, 2025