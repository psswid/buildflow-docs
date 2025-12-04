# BuildFlow Phase 5: Production Polish

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Prepare BuildFlow for production deployment with comprehensive documentation, API compliance, performance optimization, security hardening, and deployment configuration.

**Architecture:** Focus on non-functional requirements - observability, security, performance, and developer experience.

**Tech Stack:** OpenAPI/Swagger, PHPStan, Pest Architecture Tests, Laravel Horizon, Docker, GitHub Actions

**Dependencies from Previous Phases:**
- ✅ Phase 1-4: All bounded contexts implemented
- ✅ Event-driven architecture working
- ✅ CQRS read models in place
- ✅ Full test coverage

---

## Week 9: Documentation & Quality

---

### Day 50 (Monday): OpenAPI Documentation

**Goal:** Generate comprehensive API documentation from code

#### Task 1: Install & Configure OpenAPI Generator

**Files:**
- Modify: `composer.json`
- Create: `config/openapi.php`

**Step 1: Install package**

```bash
composer require dedoc/scramble
```

**Step 2: Publish config**

```bash
php artisan vendor:publish --tag=scramble-config
```

**Step 3: Configure Scramble**

```php
<?php
// config/scramble.php
return [
    'info' => [
        'version' => '1.0.0',
        'description' => 'BuildFlow API - Construction Business Management',
    ],
    
    'servers' => [
        ['url' => env('APP_URL') . '/api', 'description' => 'API Server'],
    ],
    
    'middleware' => [
        'web',
        'auth:api', // Require authentication for docs
    ],
    
    'extensions' => [],
];
```

**Step 4: Add OpenAPI attributes to controllers**

```php
<?php
// app/Domains/QuoteManagement/Infrastructure/Http/Controllers/QuoteController.php

use OpenApi\Attributes as OA;

#[OA\Tag(name: 'Quotes', description: 'Quote Management')]
class QuoteController extends Controller
{
    #[OA\Get(
        path: '/api/quotes',
        summary: 'List quotes',
        tags: ['Quotes'],
        security: [['bearerAuth' => []]],
        parameters: [
            new OA\Parameter(name: 'status', in: 'query', required: false, schema: new OA\Schema(type: 'string', enum: ['draft', 'sent', 'accepted', 'rejected'])),
            new OA\Parameter(name: 'client_id', in: 'query', required: false, schema: new OA\Schema(type: 'string', format: 'uuid')),
            new OA\Parameter(name: 'per_page', in: 'query', required: false, schema: new OA\Schema(type: 'integer', default: 20)),
        ],
        responses: [
            new OA\Response(response: 200, description: 'Paginated list of quotes'),
            new OA\Response(response: 401, description: 'Unauthorized'),
        ]
    )]
    public function index(Request $request)
    {
        // ...
    }

    #[OA\Post(
        path: '/api/quotes',
        summary: 'Create a new quote draft',
        tags: ['Quotes'],
        security: [['bearerAuth' => []]],
        requestBody: new OA\RequestBody(
            required: true,
            content: new OA\JsonContent(
                required: ['client_id', 'title', 'valid_until'],
                properties: [
                    new OA\Property(property: 'client_id', type: 'string', format: 'uuid'),
                    new OA\Property(property: 'title', type: 'string', maxLength: 255),
                    new OA\Property(property: 'valid_until', type: 'string', format: 'date'),
                ]
            )
        ),
        responses: [
            new OA\Response(response: 201, description: 'Quote created'),
            new OA\Response(response: 422, description: 'Validation error'),
        ]
    )]
    public function store(CreateQuoteRequest $request)
    {
        // ...
    }
}
```

**Step 5: Commit**

```bash
git add composer.json composer.lock
git add config/scramble.php
git add app/Domains/*/Infrastructure/Http/Controllers/
git commit -m "docs: add OpenAPI documentation with Scramble"
```

---

#### Task 2: OpenAPI Compliance Tests

**Files:**
- Create: `tests/Api/OpenApiComplianceTest.php`

**Step 1: Write compliance tests**

```php
<?php

namespace Tests\Api;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class OpenApiComplianceTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function api_docs_endpoint_returns_valid_openapi(): void
    {
        $response = $this->get('/docs/api.json');

        $response->assertStatus(200);
        $response->assertJsonStructure([
            'openapi',
            'info' => ['title', 'version'],
            'paths',
        ]);
    }

    /** @test */
    public function all_quote_endpoints_are_documented(): void
    {
        $response = $this->get('/docs/api.json');
        $spec = $response->json();

        $this->assertArrayHasKey('/quotes', $spec['paths']);
        $this->assertArrayHasKey('get', $spec['paths']['/quotes']);
        $this->assertArrayHasKey('post', $spec['paths']['/quotes']);
        
        $this->assertArrayHasKey('/quotes/{id}', $spec['paths']);
        $this->assertArrayHasKey('get', $spec['paths']['/quotes/{id}']);
        
        $this->assertArrayHasKey('/quotes/{id}/send', $spec['paths']);
        $this->assertArrayHasKey('post', $spec['paths']['/quotes/{id}/send']);
    }

    /** @test */
    public function all_endpoints_require_authentication(): void
    {
        $response = $this->get('/docs/api.json');
        $spec = $response->json();

        foreach ($spec['paths'] as $path => $methods) {
            if (str_contains($path, '/auth/')) {
                continue; // Skip auth endpoints
            }
            
            foreach ($methods as $method => $details) {
                if (!in_array($method, ['get', 'post', 'put', 'patch', 'delete'])) {
                    continue;
                }
                
                $this->assertArrayHasKey(
                    'security', 
                    $details,
                    "Endpoint {$method} {$path} should require authentication"
                );
            }
        }
    }

    /** @test */
    public function response_matches_documented_schema(): void
    {
        $user = $this->createAuthenticatedUser();
        $quote = $this->createQuote($user->organization_id);

        $response = $this->actingAs($user, 'api')
            ->getJson("/api/quotes/{$quote->id}");

        $response->assertJsonStructure([
            'data' => [
                'id',
                'quote_number',
                'client_id',
                'title',
                'status',
                'total',
                'currency',
                'created_at',
            ]
        ]);
    }
}
```

**Step 2: Commit**

```bash
git add tests/Api/OpenApiComplianceTest.php
git commit -m "test: add OpenAPI compliance tests"
```

---

### Day 51 (Tuesday): Architecture Tests

**Goal:** Enforce architectural boundaries automatically

#### Task 3: Comprehensive Architecture Tests

**Files:**
- Create: `tests/Architecture/DomainBoundariesTest.php`
- Create: `tests/Architecture/LayerDependenciesTest.php`
- Create: `tests/Architecture/NamingConventionsTest.php`

**Step 1: Domain boundaries tests**

```php
<?php

namespace Tests\Architecture;

use PHPUnit\Framework\Attributes\Test;

uses()->group('architecture');

test('domain layer does not depend on infrastructure')
    ->expect('App\Domains\QuoteManagement\Domain')
    ->not->toUse([
        'Illuminate\Database',
        'Illuminate\Http',
        'Illuminate\Support\Facades',
        'Illuminate\Queue',
        'App\Domains\QuoteManagement\Infrastructure',
    ]);

test('domain layer does not depend on application layer')
    ->expect('App\Domains\QuoteManagement\Domain')
    ->not->toUse([
        'App\Domains\QuoteManagement\Application',
    ]);

test('application layer does not depend on infrastructure')
    ->expect('App\Domains\QuoteManagement\Application')
    ->not->toUse([
        'Illuminate\Database',
        'Illuminate\Http',
        'App\Domains\QuoteManagement\Infrastructure',
    ]);

test('bounded contexts do not directly depend on each other domain layers')
    ->expect('App\Domains\QuoteManagement\Domain')
    ->not->toUse([
        'App\Domains\ProjectManagement\Domain',
        'App\Domains\InvoicePayment\Domain',
    ]);

test('project domain does not depend on quote domain directly')
    ->expect('App\Domains\ProjectManagement\Domain')
    ->not->toUse([
        'App\Domains\QuoteManagement\Domain\Quote',
        'App\Domains\QuoteManagement\Domain\QuoteRepository',
    ]);

test('cross-context communication only via events')
    ->expect('App\Domains\ProjectManagement\Infrastructure\EventListeners')
    ->toUse('App\Domains\QuoteManagement\Domain\Events');
```

**Step 2: Layer dependencies tests**

```php
<?php

namespace Tests\Architecture;

test('controllers are in infrastructure layer')
    ->expect('App\Domains\*\Infrastructure\Http\Controllers')
    ->toExtend('App\Http\Controllers\Controller');

test('handlers are in application layer')
    ->expect('App\Domains\*\Application\Handlers')
    ->toHaveSuffix('Handler');

test('commands are in application layer')
    ->expect('App\Domains\*\Application\Commands')
    ->toBeFinal()
    ->toBeReadonly();

test('value objects are final and immutable')
    ->expect('App\Domains\*\Domain\ValueObjects')
    ->toBeFinal();

test('aggregates extend AggregateRoot')
    ->expect('App\Domains\QuoteManagement\Domain\Quote')
    ->toExtend('App\SharedKernel\Domain\AggregateRoot');

test('domain events implement DomainEvent interface')
    ->expect('App\Domains\*\Domain\Events')
    ->toImplement('App\SharedKernel\Domain\DomainEvent');

test('repositories have interface in domain, implementation in infrastructure')
    ->expect('App\Domains\*\Domain\*Repository')
    ->toBeInterface();
```

**Step 3: Naming conventions tests**

```php
<?php

namespace Tests\Architecture;

test('controllers have Controller suffix')
    ->expect('App\Domains\*\Infrastructure\Http\Controllers')
    ->toHaveSuffix('Controller');

test('requests have Request suffix')
    ->expect('App\Domains\*\Infrastructure\Http\Requests')
    ->toHaveSuffix('Request');

test('resources have Resource suffix')
    ->expect('App\Domains\*\Infrastructure\Http\Resources')
    ->toHaveSuffix('Resource');

test('projectors have Projector suffix')
    ->expect('App\Domains\*\Infrastructure\Projectors')
    ->toHaveSuffix('Projector');

test('event listeners describe what they do')
    ->expect('App\Domains\*\Infrastructure\EventListeners')
    ->toMatch('/^(Create|Send|Update|Log|Notify|Generate).*When.*$/');

test('exceptions end with Exception or domain-specific suffix')
    ->expect('App\Domains\*\Domain\Exceptions')
    ->toHaveSuffix('Exception')
    ->or->toMatch('/(Cannot|Invalid|NotFound)$/');
```

**Step 4: Commit**

```bash
git add tests/Architecture/
git commit -m "test: add comprehensive architecture tests"
```

---

### Day 52 (Wednesday): Static Analysis

**Goal:** Achieve PHPStan level 8 compliance

#### Task 4: PHPStan Configuration & Fixes

**Files:**
- Create: `phpstan.neon`
- Modify: Various domain files for type safety

**Step 1: Configure PHPStan**

```neon
# phpstan.neon
includes:
    - ./vendor/larastan/larastan/extension.neon

parameters:
    level: 8
    
    paths:
        - app/Domains
        - app/SharedKernel
        
    excludePaths:
        - app/Domains/*/Infrastructure/Persistence/*EloquentModel.php
        
    ignoreErrors:
        # Allow dynamic property access on Eloquent models
        - '#Access to an undefined property#'
        
    reportUnmatchedIgnoredErrors: false
    
    # Strict rules
    checkMissingIterableValueType: true
    checkGenericClassInNonGenericObjectType: true
    
    # Custom rules
    treatPhpDocTypesAsCertain: false
```

**Step 2: Fix type issues in domain layer**

Add strict types and return types:

```php
<?php

declare(strict_types=1);

namespace App\Domains\QuoteManagement\Domain;

// Add typed properties
class Quote extends AggregateRoot
{
    /** @var array<int, LineItem> */
    private array $lineItems = [];
    
    // Add return types to all methods
    public function addLineItem(LineItem $item): void
    {
        // ...
    }
    
    /** @return array<int, LineItem> */
    public function lineItems(): array
    {
        return $this->lineItems;
    }
}
```

**Step 3: Run PHPStan**

```bash
./vendor/bin/phpstan analyse --level=8
```

**Step 4: Commit**

```bash
git add phpstan.neon
git add app/Domains/
git add app/SharedKernel/
git commit -m "chore: achieve PHPStan level 8 compliance"
```

---

### Day 53 (Thursday): Performance Optimization

**Goal:** Optimize query performance and add caching

#### Task 5: Database Query Optimization

**Files:**
- Create: `app/Domains/QuoteManagement/Infrastructure/Persistence/QuoteQueryOptimizer.php`
- Add indexes via migrations

**Step 1: Create composite indexes migration**

```php
<?php
// database/migrations/xxxx_add_performance_indexes.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        // Quote indexes
        Schema::table('quotes', function (Blueprint $table) {
            $table->index(['organization_id', 'status', 'created_at']);
            $table->index(['organization_id', 'client_id', 'status']);
            $table->index(['valid_until', 'status']); // For expiry checks
        });

        // Read model indexes
        Schema::table('quote_list_view', function (Blueprint $table) {
            $table->index(['organization_id', 'status', 'created_at']);
            $table->index(['organization_id', 'valid_until']); // Expiring soon
        });

        // Event store indexes for replay
        Schema::table('event_store', function (Blueprint $table) {
            $table->index(['aggregate_type', 'occurred_at']);
        });
    }

    public function down(): void
    {
        // Drop indexes
    }
};
```

**Step 2: Add query caching**

```php
<?php

namespace App\Domains\QuoteManagement\Application\Queries;

use Illuminate\Support\Facades\Cache;

class QuoteQueryService
{
    private const CACHE_TTL = 60; // 1 minute

    public function getDashboardStats(string $organizationId): array
    {
        return Cache::remember(
            "quote_stats:{$organizationId}",
            self::CACHE_TTL,
            fn() => $this->fetchDashboardStats($organizationId)
        );
    }

    public function invalidateStatsCache(string $organizationId): void
    {
        Cache::forget("quote_stats:{$organizationId}");
    }

    private function fetchDashboardStats(string $organizationId): array
    {
        // Actual query logic
    }
}
```

**Step 3: Add cache invalidation to projectors**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\Projectors;

class QuoteDashboardProjector
{
    public function __construct(
        private QuoteQueryService $queryService
    ) {}

    public function onQuoteAccepted(QuoteAccepted $event): void
    {
        // Update stats...
        
        // Invalidate cache
        $this->queryService->invalidateStatsCache($event->organizationId);
    }
}
```

**Step 4: Commit**

```bash
git add database/migrations/*_add_performance_indexes.php
git add app/Domains/QuoteManagement/Application/Queries/
git add app/Domains/QuoteManagement/Infrastructure/Projectors/
git commit -m "perf: add database indexes and query caching"
```

---

#### Task 6: Performance Tests

**Files:**
- Create: `tests/Performance/QueryPerformanceTest.php`

**Step 1: Write performance tests**

```php
<?php

namespace Tests\Performance;

use App\Domains\QuoteManagement\Application\Queries\QuoteQueryService;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class QueryPerformanceTest extends TestCase
{
    use RefreshDatabase;

    private QuoteQueryService $queryService;

    protected function setUp(): void
    {
        parent::setUp();
        $this->queryService = app(QuoteQueryService::class);
        
        // Seed test data
        $this->seedQuotes(1000);
    }

    /** @test */
    public function list_query_completes_under_50ms(): void
    {
        $start = microtime(true);
        
        $this->queryService->list('org-1', perPage: 20);
        
        $duration = (microtime(true) - $start) * 1000;
        
        $this->assertLessThan(50, $duration, "Query took {$duration}ms");
    }

    /** @test */
    public function dashboard_stats_completes_under_20ms(): void
    {
        $start = microtime(true);
        
        $this->queryService->getDashboardStats('org-1');
        
        $duration = (microtime(true) - $start) * 1000;
        
        $this->assertLessThan(20, $duration, "Query took {$duration}ms");
    }

    /** @test */
    public function search_completes_under_100ms(): void
    {
        $start = microtime(true);
        
        $this->queryService->search('org-1', 'kitchen');
        
        $duration = (microtime(true) - $start) * 1000;
        
        $this->assertLessThan(100, $duration, "Search took {$duration}ms");
    }

    private function seedQuotes(int $count): void
    {
        // Bulk insert for speed
        $quotes = [];
        for ($i = 0; $i < $count; $i++) {
            $quotes[] = [
                'id' => \Str::uuid()->toString(),
                'organization_id' => 'org-1',
                'quote_number' => 'Q-' . str_pad((string)$i, 6, '0', STR_PAD_LEFT),
                'client_id' => 'client-1',
                'client_name' => 'Test Client ' . $i,
                'title' => 'Quote ' . $i . ($i % 10 === 0 ? ' kitchen' : ''),
                'status' => ['draft', 'sent', 'accepted'][$i % 3],
                'total' => rand(100000, 1000000),
                'currency' => 'USD',
                'valid_until' => now()->addDays(30),
                'line_items_count' => rand(1, 10),
                'created_at' => now()->subDays(rand(0, 365)),
                'updated_at' => now(),
            ];
        }
        
        \DB::table('quote_list_view')->insert($quotes);
    }
}
```

**Step 2: Commit**

```bash
git add tests/Performance/
git commit -m "test: add performance benchmarks"
```

---

### Day 54 (Friday): Security Hardening

**Goal:** Implement security best practices

#### Task 7: Security Middleware & Policies

**Files:**
- Create: `app/Http/Middleware/ValidateOrganizationAccess.php`
- Create: `app/Policies/QuotePolicy.php`
- Update: `app/Providers/AuthServiceProvider.php`

**Step 1: Organization access middleware**

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ValidateOrganizationAccess
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();
        
        if (!$user) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        // Inject organization scope for all queries
        app()->instance('current_organization_id', $user->organization_id);

        return $next($request);
    }
}
```

**Step 2: Quote policy**

```php
<?php

namespace App\Policies;

use App\Models\User;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;

class QuotePolicy
{
    public function viewAny(User $user): bool
    {
        return true; // All authenticated users can list
    }

    public function view(User $user, QuoteListView $quote): bool
    {
        return $user->organization_id === $quote->organization_id;
    }

    public function create(User $user): bool
    {
        return in_array($user->role, ['owner', 'manager']);
    }

    public function update(User $user, QuoteListView $quote): bool
    {
        return $user->organization_id === $quote->organization_id
            && in_array($user->role, ['owner', 'manager']);
    }

    public function send(User $user, QuoteListView $quote): bool
    {
        return $this->update($user, $quote);
    }

    public function accept(User $user, QuoteListView $quote): bool
    {
        return $this->update($user, $quote);
    }

    public function delete(User $user, QuoteListView $quote): bool
    {
        return $user->organization_id === $quote->organization_id
            && $user->role === 'owner';
    }
}
```

**Step 3: Register policies**

```php
<?php
// app/Providers/AuthServiceProvider.php

use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;
use App\Policies\QuotePolicy;

class AuthServiceProvider extends ServiceProvider
{
    protected $policies = [
        QuoteListView::class => QuotePolicy::class,
    ];
}
```

**Step 4: Apply in controller**

```php
<?php

class QuoteController extends Controller
{
    public function show(string $id)
    {
        $quote = $this->queries->findById($id);
        
        $this->authorize('view', $quote);
        
        return new QuoteResource($quote);
    }

    public function send(string $id)
    {
        $quote = $this->queries->findById($id);
        
        $this->authorize('send', $quote);
        
        // ... send logic
    }
}
```

**Step 5: Commit**

```bash
git add app/Http/Middleware/
git add app/Policies/
git add app/Providers/AuthServiceProvider.php
git commit -m "security: add authorization policies and organization scoping"
```

---

#### Task 8: Security Tests

**Files:**
- Create: `tests/Security/AuthorizationTest.php`
- Create: `tests/Security/TenantIsolationTest.php`

**Step 1: Authorization tests**

```php
<?php

namespace Tests\Security;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class AuthorizationTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function user_cannot_access_other_organization_quotes(): void
    {
        $user1 = $this->createUser('org-1');
        $user2 = $this->createUser('org-2');
        $quote = $this->createQuote('org-1');

        $response = $this->actingAs($user2, 'api')
            ->getJson("/api/quotes/{$quote->id}");

        $response->assertStatus(403);
    }

    /** @test */
    public function field_worker_cannot_create_quotes(): void
    {
        $user = $this->createUser('org-1', role: 'field_worker');

        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                'client_id' => 'client-123',
                'title' => 'Test',
                'valid_until' => now()->addDays(30)->toDateString(),
            ]);

        $response->assertStatus(403);
    }

    /** @test */
    public function manager_can_create_quotes(): void
    {
        $user = $this->createUser('org-1', role: 'manager');
        $client = $this->createClient('org-1');

        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                'client_id' => $client->id,
                'title' => 'Test Quote',
                'valid_until' => now()->addDays(30)->toDateString(),
            ]);

        $response->assertStatus(201);
    }
}
```

**Step 2: Tenant isolation tests**

```php
<?php

namespace Tests\Security;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class TenantIsolationTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function list_only_returns_own_organization_quotes(): void
    {
        $user1 = $this->createUser('org-1');
        $this->createQuote('org-1', 'Quote A');
        $this->createQuote('org-1', 'Quote B');
        $this->createQuote('org-2', 'Quote C'); // Different org

        $response = $this->actingAs($user1, 'api')
            ->getJson('/api/quotes');

        $response->assertStatus(200);
        $response->assertJsonCount(2, 'data');
        $response->assertJsonMissing(['title' => 'Quote C']);
    }

    /** @test */
    public function dashboard_stats_only_include_own_organization(): void
    {
        $user1 = $this->createUser('org-1');
        $this->seedQuotesForOrg('org-1', 10);
        $this->seedQuotesForOrg('org-2', 5);

        $response = $this->actingAs($user1, 'api')
            ->getJson('/api/quotes/dashboard');

        $response->assertStatus(200);
        $response->assertJson(['total_quotes' => 10]);
    }

    /** @test */
    public function cannot_manipulate_other_org_resources_via_id_guessing(): void
    {
        $user = $this->createUser('org-1');
        $otherQuote = $this->createQuote('org-2');

        // Try to send someone else's quote
        $response = $this->actingAs($user, 'api')
            ->postJson("/api/quotes/{$otherQuote->id}/send");

        $response->assertStatus(403);
    }
}
```

**Step 3: Commit**

```bash
git add tests/Security/
git commit -m "test: add security and tenant isolation tests"
```

---

## Week 10: Deployment & Monitoring

---

### Day 55-56: CI/CD Pipeline

#### Task 9: GitHub Actions Workflow

**Files:**
- Create: `.github/workflows/ci.yml`
- Create: `.github/workflows/deploy.yml`

**Step 1: CI workflow**

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: buildflow
          POSTGRES_PASSWORD: secret
          POSTGRES_DB: buildflow_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          extensions: mbstring, pdo, pdo_pgsql
          coverage: xdebug

      - name: Install Dependencies
        run: composer install --prefer-dist --no-progress

      - name: Copy .env
        run: cp .env.testing .env

      - name: Generate Key
        run: php artisan key:generate

      - name: Run Migrations
        run: php artisan migrate --force

      - name: Run PHPStan
        run: ./vendor/bin/phpstan analyse --level=8

      - name: Run Tests
        run: ./vendor/bin/pest --coverage --min=80

      - name: Run Architecture Tests
        run: ./vendor/bin/pest --group=architecture

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Security Audit
        run: composer audit
```

**Step 2: Commit**

```bash
git add .github/workflows/
git commit -m "ci: add GitHub Actions CI pipeline"
```

---

### Day 57-58: Docker & Deployment

#### Task 10: Docker Configuration

**Files:**
- Create: `Dockerfile`
- Create: `docker-compose.yml`
- Create: `docker-compose.prod.yml`

**Step 1: Dockerfile**

```dockerfile
# Dockerfile
FROM php:8.3-fpm-alpine

# Install dependencies
RUN apk add --no-cache \
    postgresql-dev \
    libzip-dev \
    && docker-php-ext-install pdo pdo_pgsql zip opcache

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Copy application
COPY . .

# Install dependencies
RUN composer install --no-dev --optimize-autoloader

# Set permissions
RUN chown -R www-data:www-data storage bootstrap/cache

# Expose port
EXPOSE 9000

CMD ["php-fpm"]
```

**Step 2: docker-compose.yml**

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/var/www/html
    depends_on:
      - db
      - redis
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./docker/nginx.conf:/etc/nginx/conf.d/default.conf
      - .:/var/www/html
    depends_on:
      - app

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: buildflow
      POSTGRES_USER: buildflow
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

**Step 3: Commit**

```bash
git add Dockerfile docker-compose.yml docker-compose.prod.yml docker/
git commit -m "infra: add Docker configuration"
```

---

### Day 59: Monitoring & Logging

#### Task 11: Structured Logging & Health Checks

**Files:**
- Create: `app/Http/Controllers/HealthController.php`
- Modify: `config/logging.php`

**Step 1: Health check endpoint**

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Redis;

class HealthController extends Controller
{
    public function check()
    {
        $checks = [
            'database' => $this->checkDatabase(),
            'redis' => $this->checkRedis(),
            'storage' => $this->checkStorage(),
        ];

        $healthy = !in_array(false, $checks, true);

        return response()->json([
            'status' => $healthy ? 'healthy' : 'unhealthy',
            'checks' => $checks,
            'timestamp' => now()->toIso8601String(),
        ], $healthy ? 200 : 503);
    }

    private function checkDatabase(): bool
    {
        try {
            DB::select('SELECT 1');
            return true;
        } catch (\Exception $e) {
            return false;
        }
    }

    private function checkRedis(): bool
    {
        try {
            Redis::ping();
            return true;
        } catch (\Exception $e) {
            return false;
        }
    }

    private function checkStorage(): bool
    {
        return is_writable(storage_path());
    }
}
```

**Step 2: Add route**

```php
// routes/api.php
Route::get('/health', [HealthController::class, 'check']);
```

**Step 3: Commit**

```bash
git add app/Http/Controllers/HealthController.php
git add routes/api.php
git commit -m "ops: add health check endpoint"
```

---

### Day 60: Final Documentation & Review

#### Task 12: README & Documentation

**Files:**
- Update: `README.md`
- Create: `docs/ARCHITECTURE.md`
- Create: `docs/API.md`
- Create: `docs/DEPLOYMENT.md`

**Step 1: Update README**

```markdown
# BuildFlow Laravel API

Enterprise-grade construction business management API built with DDD, Event-Driven Architecture, and CQRS.

## Quick Start

```bash
# Clone and install
git clone https://github.com/yourorg/buildflow-laravel-api
cd buildflow-laravel-api
composer install
cp .env.example .env
php artisan key:generate

# Database
php artisan migrate
php artisan db:seed

# Start
php artisan serve
```

## Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for detailed architecture documentation.

### Key Patterns
- **DDD (Domain-Driven Design)** - Rich domain models with business logic
- **Event-Driven** - Cross-context communication via domain events
- **CQRS** - Separate read/write models for optimal performance

### Bounded Contexts
- Quote Management
- Project Management
- Invoice & Payment
- Client Management

## API Documentation

Interactive API docs available at `/docs/api` when running.

## Testing

```bash
# All tests
./vendor/bin/pest

# With coverage
./vendor/bin/pest --coverage

# Architecture tests only
./vendor/bin/pest --group=architecture

# Performance tests
./vendor/bin/pest --group=performance
```

## Deployment

See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for deployment instructions.

## License

MIT
```

**Step 2: Commit**

```bash
git add README.md docs/
git commit -m "docs: comprehensive documentation"
```

---

## Phase 5 Success Criteria

```
[ ] OpenAPI documentation complete and accessible
[ ] All endpoints documented with examples
[ ] PHPStan level 8 passing
[ ] Architecture tests enforcing boundaries
[ ] Performance benchmarks passing (<50ms queries)
[ ] Security policies implemented
[ ] Tenant isolation verified
[ ] CI/CD pipeline working
[ ] Docker configuration ready
[ ] Health checks implemented
[ ] Documentation complete
```

## Phase 5 Completion Checklist

```
[ ] 100% API documentation coverage
[ ] PHPStan clean at level 8
[ ] 15+ architecture tests passing
[ ] 5+ security tests passing
[ ] 3+ performance benchmarks passing
[ ] CI pipeline runs in <5 minutes
[ ] Docker builds successfully
[ ] Health endpoint returns 200
[ ] README updated with full instructions
[ ] All ADRs referenced in docs
```

---

## Final Project Statistics

After completing all 5 phases:

| Metric | Target | Actual |
|--------|--------|--------|
| Total Tests | 150+ | |
| Code Coverage | 80%+ | |
| Architecture Tests | 20+ | |
| PHPStan Level | 8 | |
| API Endpoints | 25+ | |
| Domain Events | 15+ | |
| Read Models | 6+ | |
| Bounded Contexts | 4 | |
```

---