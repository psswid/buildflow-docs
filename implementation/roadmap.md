# BuildFlow Laravel - Implementation Roadmap

## 🎯 Goal

Build a production-ready, enterprise-grade Laravel application using DDD, Event-Driven Architecture, and CQRS patterns. This is a **depth-first, learning-focused** implementation.

**Priority:** One framework (Laravel), full depth, enterprise patterns.

---

## 📊 Project Phases

| Phase | Focus | Duration | Outcome |
|-------|-------|----------|---------|
| 0 | Foundation | 1 week | Project structure, base infrastructure |
| 1 | Core Aggregate (Quote) | 2 weeks | Full DDD implementation showcase |
| 2 | Event-Driven | 1 week | Cross-context communication |
| 3 | CQRS & Reads | 1 week | Optimized read models |
| 4 | Remaining Contexts | 3 weeks | Complete domain implementation |
| 5 | Production Polish | 2 weeks | Monitoring, testing, deployment |

**Total:** ~10 weeks (2.5 months) for enterprise-grade MVP

---

## 🏗️ Phase 0: Foundation (Week 1)

### Day 1-2: Project Setup

**Tasks:**
1. Create Laravel project
2. Setup folder structure
3. Configure database (PostgreSQL)
4. Setup testing framework
5. Configure CI/CD basics

**Commands:**
```bash
# Create project
composer create-project laravel/laravel buildflow-laravel-api
cd buildflow-laravel-api

# Install dependencies
composer require tymon/jwt-auth
composer require ramsey/uuid
composer require --dev pestphp/pest
composer require --dev pestphp/pest-plugin-laravel
composer require --dev phpstan/phpstan
composer require --dev larastan/larastan

# Setup Pest
./vendor/bin/pest --init

# Create folder structure
mkdir -p app/SharedKernel/{Domain,Infrastructure}
mkdir -p app/Domains/QuoteManagement/{Domain,Application,Infrastructure}
mkdir -p tests/{Unit,Feature,Integration,Architecture}
```

**Folder Structure:**
```
app/
├── SharedKernel/
│   ├── Domain/
│   │   ├── AggregateRoot.php
│   │   ├── DomainEvent.php
│   │   ├── ValueObject.php
│   │   └── Entity.php
│   └── Infrastructure/
│       ├── EventBus/
│       └── Persistence/
│
├── Domains/
│   └── QuoteManagement/
│       ├── Domain/
│       ├── Application/
│       └── Infrastructure/
│
├── Http/
│   └── Controllers/
│       └── Api/
│
└── Providers/
```

**Deliverable:** ✅ Clean project structure with documentation

**Commit:** `feat: setup Laravel project with DDD structure`

---

### Day 3-4: Shared Kernel

**Tasks:**
1. Create base abstractions
2. Setup UUID generator
3. Create event bus interface
4. Basic value objects

**Files to Create:**

```php
// app/SharedKernel/Domain/AggregateRoot.php
abstract class AggregateRoot
{
    private array $domainEvents = [];
    
    protected function record(DomainEvent $event): void
    {
        $this->domainEvents[] = $event;
    }
    
    public function releaseEvents(): array
    {
        $events = $this->domainEvents;
        $this->domainEvents = [];
        return $events;
    }
}

// app/SharedKernel/Domain/DomainEvent.php
interface DomainEvent
{
    public function eventName(): string;
    public function occurredOn(): DateTimeImmutable;
    public function aggregateId(): string;
}

// app/SharedKernel/Domain/ValueObject.php
abstract class ValueObject
{
    abstract public function equals(self $other): bool;
}

// app/SharedKernel/Domain/Identifier.php
abstract class Identifier extends ValueObject
{
    protected function __construct(protected string $value) {}
    
    public function toString(): string
    {
        return $this->value;
    }
    
    public function equals(ValueObject $other): bool
    {
        return $other instanceof static 
            && $this->value === $other->value;
    }
    
    public function __toString(): string
    {
        return $this->value;
    }
}
```

**Tests:**
```php
// tests/Unit/SharedKernel/AggregateRootTest.php
test('aggregate root records and releases events', function () {
    $aggregate = new TestAggregate();
    
    $aggregate->doSomething();
    
    $events = $aggregate->releaseEvents();
    expect($events)->toHaveCount(1);
    expect($events[0])->toBeInstanceOf(SomethingHappened::class);
});

test('releasing events clears them', function () {
    $aggregate = new TestAggregate();
    $aggregate->doSomething();
    
    $aggregate->releaseEvents();
    $events = $aggregate->releaseEvents();
    
    expect($events)->toBeEmpty();
});
```

**Deliverable:** ✅ Shared kernel with tests

**Commit:** `feat: implement shared kernel base classes`

---

### Day 5-7: Authentication & Multi-tenancy

**Tasks:**
1. Setup JWT authentication
2. Implement multi-tenancy
3. Create Organization and User models
4. Global scopes for tenant isolation

**Migrations:**
```php
// database/migrations/2024_01_01_000001_create_organizations_table.php
Schema::create('organizations', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('name');
    $table->string('email')->unique();
    $table->enum('subscription_tier', ['starter', 'pro', 'business']);
    $table->json('settings')->nullable();
    $table->timestamps();
});

// database/migrations/2024_01_01_000002_create_users_table.php
Schema::create('users', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->uuid('organization_id');
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->enum('role', ['owner', 'manager', 'field_worker']);
    $table->timestamps();
    
    $table->foreign('organization_id')
          ->references('id')->on('organizations');
    $table->index(['organization_id', 'role']);
});
```

**Models:**
```php
// app/Models/User.php
class User extends Authenticatable
{
    use HasFactory, Notifiable;
    
    protected $keyType = 'string';
    public $incrementing = false;
    
    protected $fillable = [
        'id', 'organization_id', 'name', 'email', 'password', 'role',
    ];
    
    protected $hidden = ['password', 'remember_token'];
    
    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
    
    public function organization(): BelongsTo
    {
        return $this->belongsTo(Organization::class);
    }
}
```

**Auth Routes:**
```php
// routes/api.php
Route::prefix('auth')->group(function () {
    Route::post('register', [AuthController::class, 'register']);
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout'])->middleware('auth:api');
    Route::post('refresh', [AuthController::class, 'refresh'])->middleware('auth:api');
    Route::get('me', [AuthController::class, 'me'])->middleware('auth:api');
});
```

**Tests:**
```php
// tests/Feature/Auth/LoginTest.php
test('user can login with valid credentials', function () {
    $user = User::factory()->create([
        'email' => 'test@example.com',
        'password' => bcrypt('password'),
    ]);
    
    $response = $this->postJson('/api/auth/login', [
        'email' => 'test@example.com',
        'password' => 'password',
    ]);
    
    $response->assertStatus(200)
             ->assertJsonStructure(['access_token', 'token_type', 'expires_in']);
});

test('user cannot login with invalid credentials', function () {
    $response = $this->postJson('/api/auth/login', [
        'email' => 'wrong@example.com',
        'password' => 'wrong',
    ]);
    
    $response->assertStatus(401);
});
```

**Deliverable:** ✅ Working JWT authentication with multi-tenancy

**Commit:** `feat: implement JWT authentication and multi-tenancy`

---

## 🎯 Phase 1: Core Aggregate - Quote Management (Weeks 2-3)

This is the **showcase** implementation. Full DDD, all patterns.

### Day 8-10: Domain Layer

**Tasks:**
1. Design Quote aggregate
2. Create value objects
3. Implement business rules
4. Domain events

**Value Objects:**
```php
// app/Domains/QuoteManagement/Domain/QuoteId.php
final class QuoteId extends Identifier
{
    public static function generate(): self
    {
        return new self(Str::uuid()->toString());
    }
    
    public static function fromString(string $id): self
    {
        if (!Str::isUuid($id)) {
            throw new InvalidQuoteId($id);
        }
        return new self($id);
    }
}

// app/Domains/QuoteManagement/Domain/QuoteNumber.php
final class QuoteNumber extends ValueObject
{
    private function __construct(private string $value) {}
    
    public static function generate(int $sequence): self
    {
        return new self(sprintf('Q-%06d', $sequence));
    }
    
    public function toString(): string
    {
        return $this->value;
    }
}

// app/Domains/QuoteManagement/Domain/Money.php
final class Money extends ValueObject
{
    private function __construct(
        private int $amount,      // in cents
        private Currency $currency
    ) {
        if ($amount < 0) {
            throw new NegativeMoneyException();
        }
    }
    
    public static function fromCents(int $cents, Currency $currency): self
    {
        return new self($cents, $currency);
    }
    
    public static function fromFloat(float $amount, Currency $currency): self
    {
        return new self((int)($amount * 100), $currency);
    }
    
    public function add(Money $other): self
    {
        $this->ensureSameCurrency($other);
        return new self($this->amount + $other->amount, $this->currency);
    }
    
    public function toFloat(): float
    {
        return $this->amount / 100;
    }
}

// app/Domains/QuoteManagement/Domain/QuoteStatus.php
final class QuoteStatus extends ValueObject
{
    private const DRAFT = 'draft';
    private const SENT = 'sent';
    private const ACCEPTED = 'accepted';
    private const REJECTED = 'rejected';
    private const EXPIRED = 'expired';
    
    private function __construct(private string $value) {}
    
    public static function draft(): self { return new self(self::DRAFT); }
    public static function sent(): self { return new self(self::SENT); }
    public static function accepted(): self { return new self(self::ACCEPTED); }
    
    public function isSent(): bool { return $this->value === self::SENT; }
    public function isDraft(): bool { return $this->value === self::DRAFT; }
    
    public function equals(ValueObject $other): bool
    {
        return $other instanceof self && $this->value === $other->value;
    }
}
```

**Aggregate:**
```php
// app/Domains/QuoteManagement/Domain/Quote.php
class Quote extends AggregateRoot
{
    private function __construct(
        private QuoteId $id,
        private ClientId $clientId,
        private OrganizationId $organizationId,
        private QuoteNumber $number,
        private QuoteStatus $status,
        private string $title,
        private LineItemCollection $lineItems,
        private Money $subtotal,
        private TaxRate $taxRate,
        private Money $discount,
        private Money $total,
        private CarbonImmutable $validUntil,
        private ?CarbonImmutable $sentAt = null,
        private ?CarbonImmutable $acceptedAt = null
    ) {}
    
    public static function createDraft(
        ClientId $clientId,
        OrganizationId $organizationId,
        QuoteNumber $number,
        string $title,
        Currency $currency,
        CarbonImmutable $validUntil
    ): self {
        $quote = new self(
            id: QuoteId::generate(),
            clientId: $clientId,
            organizationId: $organizationId,
            number: $number,
            status: QuoteStatus::draft(),
            title: $title,
            lineItems: LineItemCollection::empty(),
            subtotal: Money::fromCents(0, $currency),
            taxRate: TaxRate::zero(),
            discount: Money::fromCents(0, $currency),
            total: Money::fromCents(0, $currency),
            validUntil: $validUntil
        );
        
        $quote->record(new QuoteDraftCreated(
            quoteId: $quote->id->toString(),
            clientId: $clientId->toString(),
            organizationId: $organizationId->toString(),
            number: $number->toString(),
            title: $title,
            occurredOn: new DateTimeImmutable()
        ));
        
        return $quote;
    }
    
    public function addLineItem(LineItem $item): void
    {
        if (!$this->status->isDraft()) {
            throw new CannotModifyNonDraftQuote();
        }
        
        $this->lineItems->add($item);
        $this->recalculateTotal();
        
        $this->record(new LineItemAdded(
            quoteId: $this->id->toString(),
            lineItemId: $item->id()->toString(),
            description: $item->description(),
            quantity: $item->quantity(),
            unitPrice: $item->unitPrice()->toFloat(),
            occurredOn: new DateTimeImmutable()
        ));
    }
    
    public function send(): void
    {
        if (!$this->status->isDraft()) {
            throw new QuoteAlreadySent();
        }
        
        if ($this->lineItems->isEmpty()) {
            throw new CannotSendEmptyQuote();
        }
        
        if ($this->validUntil->isPast()) {
            throw new InvalidValidUntilDate();
        }
        
        $this->status = QuoteStatus::sent();
        $this->sentAt = CarbonImmutable::now();
        
        $this->record(new QuoteSent(
            quoteId: $this->id->toString(),
            sentAt: new DateTimeImmutable()
        ));
    }
    
    public function accept(): void
    {
        if (!$this->status->isSent()) {
            throw new CanOnlyAcceptSentQuote();
        }
        
        if ($this->validUntil->isPast()) {
            throw new QuoteHasExpired();
        }
        
        $this->status = QuoteStatus::accepted();
        $this->acceptedAt = CarbonImmutable::now();
        
        $this->record(new QuoteAccepted(
            quoteId: $this->id->toString(),
            clientId: $this->clientId->toString(),
            organizationId: $this->organizationId->toString(),
            totalAmount: $this->total->toFloat(),
            currency: $this->total->currency()->code(),
            acceptedAt: new DateTimeImmutable()
        ));
    }
    
    private function recalculateTotal(): void
    {
        $this->subtotal = $this->lineItems->subtotal();
        $tax = $this->subtotal->multiply($this->taxRate->value());
        $this->total = $this->subtotal->add($tax)->subtract($this->discount);
    }
    
    // Getters...
    public function id(): QuoteId { return $this->id; }
    public function status(): QuoteStatus { return $this->status; }
    public function total(): Money { return $this->total; }
}
```

**Tests:**
```php
// tests/Unit/Domains/QuoteManagement/QuoteTest.php
test('can create draft quote', function () {
    $quote = Quote::createDraft(
        clientId: ClientId::generate(),
        organizationId: OrganizationId::generate(),
        number: QuoteNumber::generate(1),
        title: 'Kitchen Renovation',
        currency: Currency::USD(),
        validUntil: CarbonImmutable::now()->addDays(30)
    );
    
    expect($quote->status())->toEqual(QuoteStatus::draft());
    expect($quote->releaseEvents())->toHaveCount(1);
    expect($quote->releaseEvents()[0])->toBeInstanceOf(QuoteDraftCreated::class);
});

test('cannot send empty quote', function () {
    $quote = Quote::createDraft(...);
    
    expect(fn() => $quote->send())
        ->toThrow(CannotSendEmptyQuote::class);
});

test('accepting quote records event', function () {
    $quote = $this->createSentQuote();
    
    $quote->accept();
    
    $events = $quote->releaseEvents();
    expect($events)->toContain(
        fn($e) => $e instanceof QuoteAccepted
    );
});
```

**Deliverable:** ✅ Quote aggregate with full business logic and tests

**Commit:** `feat(quote): implement Quote aggregate with domain logic`

---

### Day 11-13: Application Layer

**Commands & Handlers:**
```php
// app/Domains/QuoteManagement/Application/Commands/CreateQuoteDraft.php
final class CreateQuoteDraft
{
    public function __construct(
        public readonly string $clientId,
        public readonly string $title,
        public readonly string $validUntil
    ) {}
}

// app/Domains/QuoteManagement/Application/Handlers/CreateQuoteDraftHandler.php
class CreateQuoteDraftHandler
{
    public function __construct(
        private QuoteRepository $quotes,
        private ClientRepository $clients
    ) {}
    
    public function handle(CreateQuoteDraft $command): QuoteId
    {
        $client = $this->clients->findById(
            ClientId::fromString($command->clientId)
        );
        
        if (!$client) {
            throw new ClientNotFoundException();
        }
        
        $quote = Quote::createDraft(
            clientId: ClientId::fromString($command->clientId),
            organizationId: OrganizationId::fromString(auth()->user()->organization_id),
            number: $this->quotes->nextNumber(),
            title: $command->title,
            currency: Currency::USD(),
            validUntil: CarbonImmutable::parse($command->validUntil)
        );
        
        $this->quotes->save($quote);
        
        return $quote->id();
    }
}
```

**Tests:**
```php
// tests/Feature/Domains/QuoteManagement/CreateQuoteDraftTest.php
test('can create quote draft', function () {
    $client = Client::factory()->create();
    
    $command = new CreateQuoteDraft(
        clientId: $client->id,
        title: 'Test Quote',
        validUntil: now()->addDays(30)->toDateString()
    );
    
    $quoteId = $this->handler->handle($command);
    
    expect($quoteId)->toBeInstanceOf(QuoteId::class);
    
    $this->assertDatabaseHas('quotes', [
        'id' => $quoteId->toString(),
        'title' => 'Test Quote',
        'status' => 'draft',
    ]);
});
```

**Deliverable:** ✅ Commands and Handlers with tests

**Commit:** `feat(quote): implement application layer with commands and handlers`

---

### Day 14-16: Infrastructure Layer

**Repository:**
```php
// app/Domains/QuoteManagement/Infrastructure/Persistence/EloquentQuoteRepository.php
class EloquentQuoteRepository implements QuoteRepository
{
    public function __construct(
        private EventDispatcher $eventDispatcher
    ) {}
    
    public function nextNumber(): QuoteNumber
    {
        $lastNumber = QuoteEloquentModel::max('number') ?? 0;
        return QuoteNumber::generate($lastNumber + 1);
    }
    
    public function save(Quote $quote): void
    {
        DB::transaction(function () use ($quote) {
            QuoteEloquentModel::updateOrCreate(
                ['id' => $quote->id()->toString()],
                [
                    'client_id' => $quote->clientId()->toString(),
                    'organization_id' => $quote->organizationId()->toString(),
                    'number' => $quote->number()->toString(),
                    'title' => $quote->title(),
                    'status' => $quote->status()->value(),
                    'subtotal' => $quote->subtotal()->toFloat(),
                    // ... map all fields
                ]
            );
            
            // Save line items
            // ...
            
            // Dispatch events
            foreach ($quote->releaseEvents() as $event) {
                $this->eventDispatcher->dispatch($event);
            }
        });
    }
    
    public function findById(QuoteId $id): ?Quote
    {
        $model = QuoteEloquentModel::find($id->toString());
        
        if (!$model) {
            return null;
        }
        
        return $this->mapToDomain($model);
    }
}
```

**Controller:**
```php
// app/Http/Controllers/Api/QuoteController.php
class QuoteController extends Controller
{
    public function __construct(
        private CommandBus $commandBus
    ) {}
    
    public function store(CreateQuoteRequest $request)
    {
        $quoteId = $this->commandBus->dispatch(
            new CreateQuoteDraft(
                clientId: $request->client_id,
                title: $request->title,
                validUntil: $request->valid_until
            )
        );
        
        return response()->json(['id' => $quoteId->toString()], 201);
    }
}
```

**Deliverable:** ✅ Infrastructure layer with HTTP API

**Commit:** `feat(quote): implement infrastructure layer with API endpoints`

---

## 🔄 Phase 2: Event-Driven (Week 4)

**See ADR-012 for details**

### Day 17-19: Event Bus & Listeners

- Implement Event Bus
- Create listeners in other contexts
- Test cross-context communication

**Deliverable:** ✅ QuoteAccepted triggers ProjectCreated

**Commit:** `feat: implement event-driven architecture between contexts`

---

## 📊 Phase 3: CQRS & Read Models (Week 5)

**See ADR-013 for details**

### Day 20-23: Read Models & Projectors

- Create read model tables
- Implement projectors
- Build query services

**Deliverable:** ✅ Fast read operations, dashboard stats

**Commit:** `feat: implement CQRS with read models and projectors`

---

## 🏢 Phase 4: Remaining Contexts (Weeks 6-8)

### Week 6: Project Management
- Simpler than Quote (learning from experience)
- Focus on status workflow
- Link to Quote via events

### Week 7: Invoice & Payment
- Similar to Quote but simpler
- Payment tracking
- Overdue detection

### Week 8: Documents & Client Portal
- File upload/storage
- Portal authentication
- Read-only client access

---

## 🚀 Phase 5: Production Polish (Weeks 9-10)

### Week 9: Testing & Quality
- Architecture tests
- Performance tests
- Security audit
- Code coverage > 80%

### Week 10: Deployment & Monitoring
- CI/CD pipeline
- Monitoring setup
- Documentation
- Demo deployment

---

## 📈 Success Criteria

After 10 weeks, you'll have:

✅ **Enterprise Architecture**
- Full DDD implementation
- Event-Driven Architecture
- CQRS for reads
- Clean separation of concerns

✅ **Production Ready**
- Comprehensive tests
- CI/CD pipeline
- Monitoring & logging
- Security hardened

✅ **Portfolio Showcase**
- Complex domain modeling
- Advanced patterns
- Clean code
- Professional documentation

---

## 📝 Daily Workflow

```bash
# Morning: Plan
1. Review ADR for current task
2. Read relevant pattern documentation
3. Write tests first (TDD)

# During Day: Build
4. Implement feature
5. Run tests frequently
6. Commit when green

# Evening: Reflect
7. Update documentation
8. Write learning notes
9. Plan tomorrow
```

---

## 🎓 Learning Resources Per Phase

**Phase 1 (DDD):**
- Domain-Driven Design by Eric Evans (Ch 5-7)
- Laravel Beyond CRUD (Brent Roose)

**Phase 2 (Events):**
- Domain Events (Martin Fowler)
- Event-Driven Microservices (Chris Richardson)

**Phase 3 (CQRS):**
- CQRS by Martin Fowler
- Microsoft CQRS Journey

---

**Next:** See LARAVEL_DDD_STARTER_GUIDE.md for detailed implementation guide

---

**Last Updated:** 2024-11-12
