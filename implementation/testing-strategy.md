# BuildFlow - Testing Strategy

## 🎯 Testing Philosophy

**"If it's not tested, it doesn't work."**

BuildFlow uses a comprehensive testing strategy covering all architectural layers:
- **Unit Tests** - Domain logic (pure business rules)
- **Integration Tests** - Cross-layer interactions
- **Feature Tests** - API endpoints (HTTP)
- **Architecture Tests** - Structural rules enforcement
- **Contract Tests** - API compliance with OpenAPI spec

**Target Coverage:** 80%+ overall, 90%+ for domain layer

---

## 📊 Testing Pyramid

```
         ┌─────────────────┐
         │  E2E Tests      │  < 5%
         │  (Manual/Cypress│
         └─────────────────┘
       ┌───────────────────────┐
       │   Feature Tests       │  ~20%
       │   (API, HTTP)         │
       └───────────────────────┘
     ┌─────────────────────────────┐
     │   Integration Tests         │  ~25%
     │   (Cross-context, Events)   │
     └─────────────────────────────┘
   ┌───────────────────────────────────┐
   │      Unit Tests                   │  ~50%
   │      (Domain, Value Objects)      │
   └───────────────────────────────────┘
```

---

## 🏗️ Testing Layers

### 1. Unit Tests - Domain Layer

**What to test:**
- Value Objects
- Aggregates (business logic)
- Domain Events
- Business rule validation

**Characteristics:**
- ✅ Fast (<1ms per test)
- ✅ No external dependencies
- ✅ Pure PHP logic
- ✅ High coverage (90%+)

**Example:**
```php
// tests/Unit/Domains/QuoteManagement/QuoteTest.php

use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\ValueObjects\*;
use App\Domains\QuoteManagement\Domain\Events\*;
use App\Domains\QuoteManagement\Domain\Exceptions\*;

describe('Quote Creation', function () {
    test('can create draft quote', function () {
        $quote = Quote::createDraft(
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            number: QuoteNumber::generate(1),
            title: 'Kitchen Renovation',
            currency: Currency::USD(),
            validUntil: now()->addDays(30)
        );
        
        expect($quote)->toBeInstanceOf(Quote::class);
        expect($quote->status()->isDraft())->toBeTrue();
    });
    
    test('creating draft records domain event', function () {
        $quote = Quote::createDraft(/* ... */);
        
        $events = $quote->releaseEvents();
        
        expect($events)->toHaveCount(1);
        expect($events[0])->toBeInstanceOf(QuoteDraftCreated::class);
        expect($events[0]->quoteId)->toBe($quote->id()->toString());
    });
});

describe('Quote Sending', function () {
    test('cannot send empty quote', function () {
        $quote = Quote::createDraft(/* ... */);
        // No line items added
        
        expect(fn() => $quote->send())
            ->toThrow(CannotSendEmptyQuoteException::class);
    });
    
    test('cannot send quote with expired valid until date', function () {
        $quote = Quote::createDraft(
            /* ... */
            validUntil: now()->subDay() // Expired!
        );
        $quote->addLineItem(/* ... */);
        
        expect(fn() => $quote->send())
            ->toThrow(InvalidValidUntilDateException::class);
    });
    
    test('sending quote changes status and records event', function () {
        $quote = $this->createDraftQuoteWithLineItems();
        
        $quote->send();
        
        expect($quote->status()->isSent())->toBeTrue();
        
        $events = $quote->releaseEvents();
        expect($events)->toContain(fn($e) => $e instanceof QuoteSent);
    });
});

describe('Quote Acceptance', function () {
    test('can only accept sent quote', function () {
        $draftQuote = Quote::createDraft(/* ... */);
        
        expect(fn() => $draftQuote->accept())
            ->toThrow(CanOnlyAcceptSentQuoteException::class);
    });
    
    test('cannot accept expired quote', function () {
        $quote = $this->createSentQuote(
            validUntil: now()->subDay()
        );
        
        expect(fn() => $quote->accept())
            ->toThrow(QuoteHasExpiredException::class);
    });
    
    test('accepting quote records QuoteAccepted event', function () {
        $quote = $this->createSentQuote();
        
        $quote->accept();
        
        $events = $quote->releaseEvents();
        $acceptedEvent = collect($events)->first(
            fn($e) => $e instanceof QuoteAccepted
        );
        
        expect($acceptedEvent)->not->toBeNull();
        expect($acceptedEvent->quoteId)->toBe($quote->id()->toString());
        expect($acceptedEvent->totalAmount)->toBe($quote->total()->toFloat());
    });
});

describe('Quote Total Calculation', function () {
    test('total is sum of line items', function () {
        $quote = Quote::createDraft(/* ... */);
        
        $quote->addLineItem(LineItem::create(
            description: 'Item 1',
            quantity: 2,
            unitPrice: Money::fromFloat(100, Currency::USD())
        )); // 200
        
        $quote->addLineItem(LineItem::create(
            description: 'Item 2',
            quantity: 1,
            unitPrice: Money::fromFloat(150, Currency::USD())
        )); // 150
        
        expect($quote->subtotal()->toFloat())->toBe(350.00);
    });
    
    test('total includes tax', function () {
        $quote = Quote::createDraft(/* ... */);
        $quote->setTaxRate(TaxRate::fromPercentage(20)); // 20%
        
        $quote->addLineItem(LineItem::create(
            description: 'Item',
            quantity: 1,
            unitPrice: Money::fromFloat(100, Currency::USD())
        ));
        
        // Subtotal: 100, Tax: 20, Total: 120
        expect($quote->total()->toFloat())->toBe(120.00);
    });
    
    test('total includes discount', function () {
        $quote = Quote::createDraft(/* ... */);
        
        $quote->addLineItem(/* ... total 100 */);
        $quote->applyDiscount(Money::fromFloat(10, Currency::USD()));
        
        // Subtotal: 100, Discount: 10, Total: 90
        expect($quote->total()->toFloat())->toBe(90.00);
    });
});

// Helper methods for tests
function createDraftQuoteWithLineItems(): Quote
{
    $quote = Quote::createDraft(
        clientId: ClientId::generate(),
        organizationId: OrganizationId::generate(),
        number: QuoteNumber::generate(1),
        title: 'Test Quote',
        currency: Currency::USD(),
        validUntil: now()->addDays(30)
    );
    
    $quote->addLineItem(LineItem::create(
        description: 'Test Item',
        quantity: 1,
        unitPrice: Money::fromFloat(100, Currency::USD())
    ));
    
    // Clear events from creation
    $quote->releaseEvents();
    
    return $quote;
}

function createSentQuote(?CarbonImmutable $validUntil = null): Quote
{
    $quote = createDraftQuoteWithLineItems();
    $quote->send();
    $quote->releaseEvents(); // Clear events
    return $quote;
}
```

**Run:**
```bash
./vendor/bin/pest tests/Unit/Domains/QuoteManagement
```

---

### 2. Integration Tests - Application Layer

**What to test:**
- Command handlers
- Query services
- Cross-context communication
- Event listeners
- Sagas/Process Managers

**Characteristics:**
- ⚠️ Medium speed (~10-100ms per test)
- ✅ Database interactions
- ✅ Event dispatching
- ✅ Multiple components

**Example:**
```php
// tests/Integration/Domains/QuoteManagement/AcceptQuoteFlowTest.php

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Event;
use App\Domains\QuoteManagement\Application\Commands\AcceptQuote;
use App\Domains\QuoteManagement\Application\Handlers\AcceptQuoteHandler;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;

uses(RefreshDatabase::class);

test('accepting quote creates project', function () {
    // Arrange
    Event::fake([QuoteAccepted::class, ProjectCreated::class]);
    
    $organization = Organization::factory()->create();
    $client = Client::factory()->create(['organization_id' => $organization->id]);
    $quote = createQuoteInDatabase([
        'organization_id' => $organization->id,
        'client_id' => $client->id,
        'status' => 'sent',
        'total' => 5000.00,
    ]);
    
    $handler = app(AcceptQuoteHandler::class);
    
    // Act
    $handler->handle(new AcceptQuote($quote->id));
    
    // Assert - Event dispatched
    Event::assertDispatched(QuoteAccepted::class, function ($event) use ($quote) {
        return $event->quoteId === $quote->id
            && $event->totalAmount === 5000.00;
    });
    
    // Assert - Project created (by listener)
    Event::assertDispatched(ProjectCreated::class, function ($event) use ($quote) {
        return $event->clientId === $quote->client_id
            && $event->originQuoteId === $quote->id;
    });
    
    // Assert - Database updated
    $this->assertDatabaseHas('quotes', [
        'id' => $quote->id,
        'status' => 'accepted',
    ]);
    
    $this->assertDatabaseHas('projects', [
        'client_id' => $client->id,
        'origin_quote_id' => $quote->id,
    ]);
});

test('accepting quote sends notification', function () {
    // ... similar test for notification
});

test('rolling back if project creation fails', function () {
    // Arrange
    Event::fake();
    
    // Make project creation fail
    $this->mock(ProjectRepository::class)
        ->shouldReceive('save')
        ->andThrow(new \Exception('Database error'));
    
    $quote = createSentQuote();
    
    // Act & Assert
    expect(fn() => $this->handler->handle(new AcceptQuote($quote->id)))
        ->toThrow(\Exception::class);
    
    // Quote should still be 'sent', not 'accepted'
    $this->assertDatabaseHas('quotes', [
        'id' => $quote->id,
        'status' => 'sent',
    ]);
});
```

**Run:**
```bash
./vendor/bin/pest tests/Integration
```

---

### 3. Feature Tests - HTTP/API Layer

**What to test:**
- API endpoints
- Request validation
- Authentication/Authorization
- Response format
- HTTP status codes

**Characteristics:**
- ⚠️ Slower (~50-200ms per test)
- ✅ Full stack (HTTP → DB)
- ✅ JSON responses
- ✅ Middleware

**Example:**
```php
// tests/Feature/Api/QuoteApiTest.php

use Illuminate\Foundation\Testing\RefreshDatabase;
use App\Models\User;
use App\Models\Client;

uses(RefreshDatabase::class);

describe('POST /api/quotes', function () {
    test('authenticated user can create quote', function () {
        $user = User::factory()->create();
        $client = Client::factory()->create([
            'organization_id' => $user->organization_id
        ]);
        
        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                'client_id' => $client->id,
                'title' => 'Kitchen Renovation',
                'valid_until' => now()->addDays(30)->toDateString(),
            ]);
        
        $response->assertStatus(201)
                 ->assertJsonStructure(['id', 'message']);
        
        $this->assertDatabaseHas('quotes', [
            'client_id' => $client->id,
            'title' => 'Kitchen Renovation',
            'status' => 'draft',
        ]);
    });
    
    test('unauthenticated request returns 401', function () {
        $response = $this->postJson('/api/quotes', [
            'client_id' => 'some-id',
            'title' => 'Test',
            'valid_until' => now()->addDays(30)->toDateString(),
        ]);
        
        $response->assertStatus(401);
    });
    
    test('validation fails for missing required fields', function () {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                // Missing client_id, title, valid_until
            ]);
        
        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['client_id', 'title', 'valid_until']);
    });
    
    test('validation fails for invalid client_id', function () {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                'client_id' => 'non-existent-id',
                'title' => 'Test',
                'valid_until' => now()->addDays(30)->toDateString(),
            ]);
        
        $response->assertStatus(422)
                 ->assertJsonValidationErrors(['client_id']);
    });
    
    test('cannot create quote for client in different organization', function () {
        $user = User::factory()->create();
        $otherOrgClient = Client::factory()->create(); // Different org
        
        $response = $this->actingAs($user, 'api')
            ->postJson('/api/quotes', [
                'client_id' => $otherOrgClient->id,
                'title' => 'Test',
                'valid_until' => now()->addDays(30)->toDateString(),
            ]);
        
        $response->assertStatus(403); // Forbidden
    });
});

describe('POST /api/quotes/{id}/accept', function () {
    test('can accept sent quote', function () {
        $user = User::factory()->create();
        $quote = createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'status' => 'sent',
        ]);
        
        $response = $this->actingAs($user, 'api')
            ->postJson("/api/quotes/{$quote->id}/accept");
        
        $response->assertStatus(200)
                 ->assertJson(['message' => 'Quote accepted successfully']);
        
        $this->assertDatabaseHas('quotes', [
            'id' => $quote->id,
            'status' => 'accepted',
        ]);
    });
    
    test('cannot accept draft quote', function () {
        $user = User::factory()->create();
        $quote = createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'status' => 'draft',
        ]);
        
        $response = $this->actingAs($user, 'api')
            ->postJson("/api/quotes/{$quote->id}/accept");
        
        $response->assertStatus(422)
                 ->assertJson([
                     'message' => 'Can only accept sent quotes'
                 ]);
    });
});

describe('GET /api/quotes', function () {
    test('lists quotes for authenticated user organization', function () {
        $user = User::factory()->create();
        
        createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'title' => 'Quote 1',
        ]);
        createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'title' => 'Quote 2',
        ]);
        
        // Different org - should not appear
        createQuoteInDatabase([
            'organization_id' => 'other-org-id',
            'title' => 'Quote 3',
        ]);
        
        $response = $this->actingAs($user, 'api')
            ->getJson('/api/quotes');
        
        $response->assertStatus(200)
                 ->assertJsonCount(2, 'data')
                 ->assertJsonPath('data.0.title', 'Quote 1')
                 ->assertJsonPath('data.1.title', 'Quote 2');
    });
    
    test('can filter by status', function () {
        $user = User::factory()->create();
        
        createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'status' => 'draft',
        ]);
        createQuoteInDatabase([
            'organization_id' => $user->organization_id,
            'status' => 'sent',
        ]);
        
        $response = $this->actingAs($user, 'api')
            ->getJson('/api/quotes?status=sent');
        
        $response->assertStatus(200)
                 ->assertJsonCount(1, 'data');
    });
});
```

**Run:**
```bash
./vendor/bin/pest tests/Feature/Api
```

---

### 4. Architecture Tests

**What to test:**
- Layer dependencies
- Naming conventions
- Code structure rules
- Domain purity

**Characteristics:**
- ✅ Fast
- ✅ Prevent architectural violations
- ✅ Enforce consistency

**Example:**
```php
// tests/Architecture/DomainLayerTest.php

use PHPUnit\Framework\TestCase;

test('domain layer does not depend on infrastructure', function () {
    expect('App\Domains\QuoteManagement\Domain')
        ->not->toUse([
            'Illuminate\Database',
            'Illuminate\Http',
            'Illuminate\Support\Facades',
        ]);
});

test('domain layer does not depend on application layer', function () {
    expect('App\Domains\QuoteManagement\Domain')
        ->not->toUse('App\Domains\QuoteManagement\Application');
});

test('value objects are final', function () {
    expect('App\Domains\QuoteManagement\Domain\ValueObjects')
        ->classes()
        ->toBeFinal();
});

test('value objects extend ValueObject base class', function () {
    expect('App\Domains\QuoteManagement\Domain\ValueObjects')
        ->classes()
        ->toExtend('App\SharedKernel\Domain\ValueObject');
});

test('domain events implement DomainEvent interface', function () {
    expect('App\Domains\QuoteManagement\Domain\Events')
        ->classes()
        ->toImplement('App\SharedKernel\Domain\DomainEvent');
});

test('domain events are immutable (readonly properties)', function () {
    expect('App\Domains\QuoteManagement\Domain\Events')
        ->classes()
        ->toHaveConstructorsWithReadOnlyParameters();
});

test('aggregates extend AggregateRoot', function () {
    expect('App\Domains\QuoteManagement\Domain\Quote')
        ->toExtend('App\SharedKernel\Domain\AggregateRoot');
});

test('repository interfaces are in domain', function () {
    expect('App\Domains\QuoteManagement\Domain\QuoteRepository')
        ->toBeInterface();
});

test('controllers are thin (max 20 lines per method)', function () {
    expect('App\Http\Controllers')
        ->classes()
        ->toHaveMethodLengthLessThan(20);
});

test('handlers use constructor injection', function () {
    expect('App\Domains\QuoteManagement\Application\Handlers')
        ->classes()
        ->toHaveConstructor();
});
```

**Run:**
```bash
./vendor/bin/pest tests/Architecture
```

---

### 5. Contract Tests - API Compliance

**What to test:**
- API responses match OpenAPI spec
- Request/response schemas
- Status codes
- Headers

**Characteristics:**
- ⚠️ Medium speed
- ✅ Validates OpenAPI compliance
- ✅ Prevents API drift

**Example:**
```php
// tests/Contract/QuoteApiContractTest.php

use Spectator\Spectator;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

beforeEach(function () {
    Spectator::using('openapi.yaml');
});

test('POST /quotes matches OpenAPI spec', function () {
    $user = User::factory()->create();
    $client = Client::factory()->create([
        'organization_id' => $user->organization_id
    ]);
    
    $this->actingAs($user, 'api')
        ->postJson('/api/quotes', [
            'client_id' => $client->id,
            'title' => 'Kitchen Renovation',
            'valid_until' => now()->addDays(30)->toDateString(),
        ])
        ->assertValidRequest()
        ->assertValidResponse(201);
});

test('GET /quotes response matches schema', function () {
    $user = User::factory()->create();
    
    createQuoteInDatabase([
        'organization_id' => $user->organization_id,
    ]);
    
    $this->actingAs($user, 'api')
        ->getJson('/api/quotes')
        ->assertValidRequest()
        ->assertValidResponse(200);
});

test('validation errors match OpenAPI spec', function () {
    $user = User::factory()->create();
    
    $this->actingAs($user, 'api')
        ->postJson('/api/quotes', [
            // Missing required fields
        ])
        ->assertValidRequest()
        ->assertValidResponse(422);
});
```

**Run:**
```bash
./vendor/bin/pest tests/Contract
```

---

## 🎭 Test Doubles

### When to use what:

**Fake:**
```php
// In-memory implementation
class FakeQuoteRepository implements QuoteRepository
{
    private array $quotes = [];
    
    public function save(Quote $quote): void
    {
        $this->quotes[$quote->id()->toString()] = $quote;
    }
    
    public function findById(QuoteId $id): ?Quote
    {
        return $this->quotes[$id->toString()] ?? null;
    }
}

// Usage in tests
test('handler saves quote', function () {
    $repo = new FakeQuoteRepository();
    $handler = new CreateQuoteDraftHandler($repo);
    
    $quoteId = $handler->handle(new CreateQuoteDraft(/* ... */));
    
    expect($repo->findById($quoteId))->not->toBeNull();
});
```

**Mock:**
```php
// When you need to verify interactions
test('handler dispatches events', function () {
    $eventBus = Mockery::mock(EventBus::class);
    $eventBus->shouldReceive('dispatch')
             ->once()
             ->with(Mockery::type(QuoteAccepted::class));
    
    $handler = new AcceptQuoteHandler($repo, $eventBus);
    $handler->handle(new AcceptQuote($quoteId));
});
```

**Spy:**
```php
// Laravel's Event::fake() is a spy
Event::fake([QuoteAccepted::class]);

// Act
$handler->handle(new AcceptQuote($quoteId));

// Assert
Event::assertDispatched(QuoteAccepted::class);
```

---

## 📈 Coverage Goals

| Layer | Target Coverage | Priority |
|-------|----------------|----------|
| Domain (Aggregates, VOs) | 90%+ | Critical |
| Application (Handlers) | 80%+ | High |
| Infrastructure | 60%+ | Medium |
| HTTP Controllers | 70%+ | Medium |
| **Overall** | **80%+** | High |

**Check coverage:**
```bash
./vendor/bin/pest --coverage --min=80
```

---

## 🚀 CI/CD Integration

**GitHub Actions:**
```yaml
# .github/workflows/tests.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_DB: buildflow_test
          POSTGRES_PASSWORD: secret
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 8.3
          extensions: mbstring, pdo, pdo_pgsql
      
      - name: Install Dependencies
        run: composer install
      
      - name: Run Tests
        run: ./vendor/bin/pest --coverage --min=80
        env:
          DB_CONNECTION: pgsql
          DB_HOST: localhost
          DB_PORT: 5432
          DB_DATABASE: buildflow_test
          DB_USERNAME: postgres
          DB_PASSWORD: secret
      
      - name: Architecture Tests
        run: ./vendor/bin/pest tests/Architecture
      
      - name: Contract Tests
        run: ./vendor/bin/pest tests/Contract
```

---

## 💡 Best Practices

### 1. Test Naming

✅ **Good:**
```php
test('cannot send empty quote')
test('accepting quote records domain event')
test('user can create quote via API')
```

❌ **Bad:**
```php
test('test1')
test('it works')
test('quote')
```

### 2. Arrange-Act-Assert Pattern

```php
test('accepting quote creates project', function () {
    // Arrange - Setup
    $quote = createSentQuote();
    Event::fake();
    
    // Act - Execute
    $this->handler->handle(new AcceptQuote($quote->id()));
    
    // Assert - Verify
    Event::assertDispatched(ProjectCreated::class);
});
```

### 3. One Assertion Focus Per Test

✅ **Good:**
```php
test('accepting quote changes status to accepted')
test('accepting quote records QuoteAccepted event')
test('accepting quote creates project')
```

❌ **Bad:**
```php
test('accepting quote does everything', function () {
    // Tests status, events, project creation, notifications...
    // Too much in one test!
});
```

### 4. Test Data Builders

```php
// tests/Helpers/QuoteBuilder.php
class QuoteBuilder
{
    private ClientId $clientId;
    private string $title = 'Default Quote';
    private Money $total;
    // ...
    
    public static function aQuote(): self
    {
        return new self();
    }
    
    public function withTitle(string $title): self
    {
        $this->title = $title;
        return $this;
    }
    
    public function withTotal(float $amount): self
    {
        $this->total = Money::fromFloat($amount, Currency::USD());
        return $this;
    }
    
    public function sent(): self
    {
        $this->status = QuoteStatus::sent();
        return $this;
    }
    
    public function build(): Quote
    {
        $quote = Quote::createDraft(/* ... */);
        // Apply configured properties
        return $quote;
    }
}

// Usage in tests
test('example', function () {
    $quote = QuoteBuilder::aQuote()
        ->withTitle('Kitchen Renovation')
        ->withTotal(5000.00)
        ->sent()
        ->build();
    
    // Use $quote in test
});
```

---

## 🎓 Learning Resources

- **PHPUnit Documentation** - https://phpunit.de/
- **Pest PHP** - https://pestphp.com/
- **Test-Driven Development by Kent Beck**
- **Growing Object-Oriented Software, Guided by Tests**

---

**Next:** Start with unit tests for value objects, then aggregates, then move up the pyramid!

---

**Last Updated:** 2024-11-12
