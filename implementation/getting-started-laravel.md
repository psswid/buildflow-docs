# Laravel DDD Starter Guide

## 🎯 Cel

Ten przewodnik pokazuje **krok po kroku** jak zaimplementować pierwszy aggregate w Laravel używając Domain-Driven Design. Będziemy budować **Quote aggregate** jako przykład.

---

## 📚 Przed rozpoczęciem

**Przeczytaj:**
- [ADR-011: Domain-Driven Design](docs-architecture-decisions-011-domain-driven-design.md)
- [DOMAIN_ANALYSIS_EVENT_STORMING.md](DOMAIN_ANALYSIS_EVENT_STORMING.md)
- [IMPLEMENTATION_ROADMAP.md](IMPLEMENTATION_ROADMAP.md)

**Zrozum:**
- Czym są Aggregates, Value Objects, Domain Events
- Różnicę między Domain, Application, Infrastructure
- Test-Driven Development (TDD)

---

## 🏗️ Step-by-Step Implementation

### Step 1: Zaplanuj Aggregate

**Pytania do zadania:**

1. **Co jest aggregate root?** 
   - Quote (główna encja)

2. **Jakie są granice aggregate?**
   - Quote + LineItems (są częścią Quote)
   - ❌ Client NIE jest częścią aggregate (tylko referencja przez ClientId)

3. **Jakie są invarianty (business rules)?**
   - Quote nie może być pusty gdy wysyłany
   - Sent quote nie może być edytowany
   - Accepted quote nie może być rejected
   - Total = sum(lineItems) + tax - discount

4. **Jakie domain events?**
   - QuoteDraftCreated
   - LineItemAdded
   - QuoteCalculated
   - QuoteSent
   - QuoteAccepted
   - QuoteRejected

**Narysuj na papierze lub w narzędziu:**
```
┌─────────────────────────────────┐
│         Quote (Root)            │
├─────────────────────────────────┤
│ - id: QuoteId                   │
│ - clientId: ClientId            │
│ - number: QuoteNumber           │
│ - status: QuoteStatus           │
│ - lineItems: LineItemCollection │
│ - total: Money                  │
├─────────────────────────────────┤
│ + createDraft()                 │
│ + addLineItem()                 │
│ + send()                        │
│ + accept()                      │
└─────────────────────────────────┘
```

---

### Step 2: Utwórz folder structure

```bash
mkdir -p app/Domains/QuoteManagement/Domain
mkdir -p app/Domains/QuoteManagement/Domain/Events
mkdir -p app/Domains/QuoteManagement/Domain/Exceptions
mkdir -p app/Domains/QuoteManagement/Domain/ValueObjects
mkdir -p app/Domains/QuoteManagement/Application/Commands
mkdir -p app/Domains/QuoteManagement/Application/Handlers
mkdir -p app/Domains/QuoteManagement/Infrastructure/Persistence
mkdir -p app/Domains/QuoteManagement/Infrastructure/Http
mkdir -p tests/Unit/Domains/QuoteManagement
mkdir -p tests/Feature/Domains/QuoteManagement
```

---

### Step 3: Zaimplementuj Value Objects (TDD)

**Dlaczego Value Objects najpierw?**
- Są podstawowymi blokami
- Nie mają dependencies
- Łatwe do przetestowania
- Pokazują język domeny

#### 3.1 QuoteId

**Test najpierw:**
```php
// tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteIdTest.php
<?php

use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;

test('can generate new quote id', function () {
    $id = QuoteId::generate();
    
    expect($id)->toBeInstanceOf(QuoteId::class);
    expect($id->toString())->toMatch('/^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i');
});

test('can create from string', function () {
    $uuid = '123e4567-e89b-12d3-a456-426614174000';
    $id = QuoteId::fromString($uuid);
    
    expect($id->toString())->toBe($uuid);
});

test('throws exception for invalid uuid', function () {
    QuoteId::fromString('not-a-uuid');
})->throws(InvalidQuoteIdException::class);

test('two ids with same value are equal', function () {
    $uuid = '123e4567-e89b-12d3-a456-426614174000';
    $id1 = QuoteId::fromString($uuid);
    $id2 = QuoteId::fromString($uuid);
    
    expect($id1->equals($id2))->toBeTrue();
});

test('two different ids are not equal', function () {
    $id1 = QuoteId::generate();
    $id2 = QuoteId::generate();
    
    expect($id1->equals($id2))->toBeFalse();
});
```

**Uruchom test (powinien failować):**
```bash
./vendor/bin/pest tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteIdTest.php
```

**Implementacja:**
```php
// app/SharedKernel/Domain/Identifier.php
<?php

namespace App\SharedKernel\Domain;

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

// app/Domains/QuoteManagement/Domain/ValueObjects/QuoteId.php
<?php

namespace App\Domains\QuoteManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\Identifier;
use App\Domains\QuoteManagement\Domain\Exceptions\InvalidQuoteIdException;
use Illuminate\Support\Str;

final class QuoteId extends Identifier
{
    public static function generate(): self
    {
        return new self(Str::uuid()->toString());
    }
    
    public static function fromString(string $id): self
    {
        if (!Str::isUuid($id)) {
            throw new InvalidQuoteIdException("Invalid UUID: {$id}");
        }
        
        return new self($id);
    }
}

// app/Domains/QuoteManagement/Domain/Exceptions/InvalidQuoteIdException.php
<?php

namespace App\Domains\QuoteManagement\Domain\Exceptions;

use InvalidArgumentException;

class InvalidQuoteIdException extends InvalidArgumentException
{
}
```

**Uruchom test ponownie (powinien być zielony):**
```bash
./vendor/bin/pest tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteIdTest.php
```

✅ **Commit:** `test: add QuoteId value object tests`
✅ **Commit:** `feat(quote): implement QuoteId value object`

---

#### 3.2 Money Value Object

**Test:**
```php
// tests/Unit/Domains/QuoteManagement/ValueObjects/MoneyTest.php
<?php

use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use App\Domains\QuoteManagement\Domain\ValueObjects\Currency;

test('can create money from cents', function () {
    $money = Money::fromCents(1000, Currency::USD());
    
    expect($money->toFloat())->toBe(10.00);
});

test('can create money from float', function () {
    $money = Money::fromFloat(10.50, Currency::USD());
    
    expect($money->toFloat())->toBe(10.50);
    expect($money->toCents())->toBe(1050);
});

test('can add money', function () {
    $money1 = Money::fromFloat(10.00, Currency::USD());
    $money2 = Money::fromFloat(5.50, Currency::USD());
    
    $result = $money1->add($money2);
    
    expect($result->toFloat())->toBe(15.50);
});

test('can subtract money', function () {
    $money1 = Money::fromFloat(10.00, Currency::USD());
    $money2 = Money::fromFloat(3.50, Currency::USD());
    
    $result = $money1->subtract($money2);
    
    expect($result->toFloat())->toBe(6.50);
});

test('can multiply money', function () {
    $money = Money::fromFloat(10.00, Currency::USD());
    
    $result = $money->multiply(2.5);
    
    expect($result->toFloat())->toBe(25.00);
});

test('throws exception for negative amount', function () {
    Money::fromCents(-100, Currency::USD());
})->throws(NegativeMoneyException::class);

test('throws exception when adding different currencies', function () {
    $usd = Money::fromFloat(10, Currency::USD());
    $eur = Money::fromFloat(10, Currency::EUR());
    
    $usd->add($eur);
})->throws(CurrencyMismatchException::class);

test('two money objects with same value are equal', function () {
    $money1 = Money::fromFloat(10.50, Currency::USD());
    $money2 = Money::fromFloat(10.50, Currency::USD());
    
    expect($money1->equals($money2))->toBeTrue();
});
```

**Implementacja:**
```php
// app/Domains/QuoteManagement/Domain/ValueObjects/Currency.php
<?php

namespace App\Domains\QuoteManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class Currency extends ValueObject
{
    private const USD = 'USD';
    private const EUR = 'EUR';
    private const GBP = 'GBP';
    private const PLN = 'PLN';
    
    private function __construct(private string $code) {}
    
    public static function USD(): self { return new self(self::USD); }
    public static function EUR(): self { return new self(self::EUR); }
    public static function GBP(): self { return new self(self::GBP); }
    public static function PLN(): self { return new self(self::PLN); }
    
    public function code(): string
    {
        return $this->code;
    }
    
    public function equals(ValueObject $other): bool
    {
        return $other instanceof self 
            && $this->code === $other->code;
    }
}

// app/Domains/QuoteManagement/Domain/ValueObjects/Money.php
<?php

namespace App\Domains\QuoteManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;
use App\Domains\QuoteManagement\Domain\Exceptions\NegativeMoneyException;
use App\Domains\QuoteManagement\Domain\Exceptions\CurrencyMismatchException;

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
        return new self((int)round($amount * 100), $currency);
    }
    
    public static function zero(Currency $currency): self
    {
        return new self(0, $currency);
    }
    
    public function add(Money $other): self
    {
        $this->ensureSameCurrency($other);
        return new self($this->amount + $other->amount, $this->currency);
    }
    
    public function subtract(Money $other): self
    {
        $this->ensureSameCurrency($other);
        return new self($this->amount - $other->amount, $this->currency);
    }
    
    public function multiply(float $factor): self
    {
        return new self((int)round($this->amount * $factor), $this->currency);
    }
    
    public function toFloat(): float
    {
        return $this->amount / 100;
    }
    
    public function toCents(): int
    {
        return $this->amount;
    }
    
    public function currency(): Currency
    {
        return $this->currency;
    }
    
    public function equals(ValueObject $other): bool
    {
        return $other instanceof self 
            && $this->amount === $other->amount
            && $this->currency->equals($other->currency);
    }
    
    private function ensureSameCurrency(Money $other): void
    {
        if (!$this->currency->equals($other->currency)) {
            throw new CurrencyMismatchException(
                "Cannot operate on different currencies: {$this->currency->code()} and {$other->currency->code()}"
            );
        }
    }
}
```

✅ **Commit:** `test: add Money value object tests`
✅ **Commit:** `feat(quote): implement Money value object`

---

#### 3.3 QuoteStatus Value Object

**Test:**
```php
// tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteStatusTest.php
<?php

use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteStatus;

test('can create draft status', function () {
    $status = QuoteStatus::draft();
    
    expect($status->isDraft())->toBeTrue();
    expect($status->isSent())->toBeFalse();
});

test('can create sent status', function () {
    $status = QuoteStatus::sent();
    
    expect($status->isSent())->toBeTrue();
    expect($status->isDraft())->toBeFalse();
});

test('can check if status allows editing', function () {
    expect(QuoteStatus::draft()->canBeEdited())->toBeTrue();
    expect(QuoteStatus::sent()->canBeEdited())->toBeFalse();
    expect(QuoteStatus::accepted()->canBeEdited())->toBeFalse();
});

test('statuses are equal if same value', function () {
    $draft1 = QuoteStatus::draft();
    $draft2 = QuoteStatus::draft();
    
    expect($draft1->equals($draft2))->toBeTrue();
});

test('different statuses are not equal', function () {
    $draft = QuoteStatus::draft();
    $sent = QuoteStatus::sent();
    
    expect($draft->equals($sent))->toBeFalse();
});
```

**Implementacja:**
```php
// app/Domains/QuoteManagement/Domain/ValueObjects/QuoteStatus.php
<?php

namespace App\Domains\QuoteManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class QuoteStatus extends ValueObject
{
    private const DRAFT = 'draft';
    private const SENT = 'sent';
    private const ACCEPTED = 'accepted';
    private const REJECTED = 'rejected';
    private const EXPIRED = 'expired';
    
    private function __construct(private string $value) {}
    
    public static function draft(): self
    {
        return new self(self::DRAFT);
    }
    
    public static function sent(): self
    {
        return new self(self::SENT);
    }
    
    public static function accepted(): self
    {
        return new self(self::ACCEPTED);
    }
    
    public static function rejected(): self
    {
        return new self(self::REJECTED);
    }
    
    public static function expired(): self
    {
        return new self(self::EXPIRED);
    }
    
    public function isDraft(): bool
    {
        return $this->value === self::DRAFT;
    }
    
    public function isSent(): bool
    {
        return $this->value === self::SENT;
    }
    
    public function isAccepted(): bool
    {
        return $this->value === self::ACCEPTED;
    }
    
    public function canBeEdited(): bool
    {
        return $this->value === self::DRAFT;
    }
    
    public function value(): string
    {
        return $this->value;
    }
    
    public function equals(ValueObject $other): bool
    {
        return $other instanceof self 
            && $this->value === $other->value;
    }
}
```

✅ **Commit:** `test: add QuoteStatus value object tests`
✅ **Commit:** `feat(quote): implement QuoteStatus value object`

---

### Step 4: Zaimplementuj Domain Events

**Test:**
```php
// tests/Unit/Domains/QuoteManagement/Events/QuoteDraftCreatedTest.php
<?php

use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;

test('can create quote draft created event', function () {
    $event = new QuoteDraftCreated(
        quoteId: 'quote-123',
        clientId: 'client-456',
        organizationId: 'org-789',
        number: 'Q-000001',
        title: 'Kitchen Renovation',
        occurredOn: new DateTimeImmutable()
    );
    
    expect($event->quoteId)->toBe('quote-123');
    expect($event->eventName())->toBe('quote.draft_created');
    expect($event->aggregateId())->toBe('quote-123');
});

test('event can be serialized to array', function () {
    $occurredOn = new DateTimeImmutable();
    $event = new QuoteDraftCreated(
        quoteId: 'quote-123',
        clientId: 'client-456',
        organizationId: 'org-789',
        number: 'Q-000001',
        title: 'Kitchen Renovation',
        occurredOn: $occurredOn
    );
    
    $array = $event->toArray();
    
    expect($array)->toHaveKey('quote_id', 'quote-123');
    expect($array)->toHaveKey('client_id', 'client-456');
});
```

**Implementacja:**
```php
// app/Domains/QuoteManagement/Domain/Events/QuoteDraftCreated.php
<?php

namespace App\Domains\QuoteManagement\Domain\Events;

use App\SharedKernel\Domain\DomainEvent;
use DateTimeImmutable;

final class QuoteDraftCreated implements DomainEvent
{
    public function __construct(
        public readonly string $quoteId,
        public readonly string $clientId,
        public readonly string $organizationId,
        public readonly string $number,
        public readonly string $title,
        public readonly DateTimeImmutable $occurredOn
    ) {}
    
    public function eventName(): string
    {
        return 'quote.draft_created';
    }
    
    public function occurredOn(): DateTimeImmutable
    {
        return $this->occurredOn;
    }
    
    public function aggregateId(): string
    {
        return $this->quoteId;
    }
    
    public function toArray(): array
    {
        return [
            'quote_id' => $this->quoteId,
            'client_id' => $this->clientId,
            'organization_id' => $this->organizationId,
            'number' => $this->number,
            'title' => $this->title,
            'occurred_on' => $this->occurredOn->format('Y-m-d H:i:s'),
        ];
    }
}
```

Analogicznie zaimplementuj:
- `QuoteSent`
- `QuoteAccepted`
- `LineItemAdded`

✅ **Commit:** `test: add domain event tests`
✅ **Commit:** `feat(quote): implement domain events`

---

### Step 5: Zaimplementuj Aggregate (TDD)

**Najpierw najmniejszy test:**
```php
// tests/Unit/Domains/QuoteManagement/QuoteTest.php
<?php

use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\ValueObjects\*;

test('can create draft quote', function () {
    $quote = Quote::createDraft(
        clientId: ClientId::generate(),
        organizationId: OrganizationId::generate(),
        number: QuoteNumber::generate(1),
        title: 'Kitchen Renovation',
        currency: Currency::USD(),
        validUntil: CarbonImmutable::now()->addDays(30)
    );
    
    expect($quote)->toBeInstanceOf(Quote::class);
    expect($quote->status()->isDraft())->toBeTrue();
});

test('creating draft records event', function () {
    $quote = Quote::createDraft(
        clientId: ClientId::generate(),
        organizationId: OrganizationId::generate(),
        number: QuoteNumber::generate(1),
        title: 'Kitchen Renovation',
        currency: Currency::USD(),
        validUntil: CarbonImmutable::now()->addDays(30)
    );
    
    $events = $quote->releaseEvents();
    
    expect($events)->toHaveCount(1);
    expect($events[0])->toBeInstanceOf(QuoteDraftCreated::class);
});
```

**Implementacja (minimalna, żeby test przeszedł):**
```php
// app/Domains/QuoteManagement/Domain/Quote.php
<?php

namespace App\Domains\QuoteManagement\Domain;

use App\SharedKernel\Domain\AggregateRoot;
use App\Domains\QuoteManagement\Domain\ValueObjects\*;
use App\Domains\QuoteManagement\Domain\Events\*;
use Carbon\CarbonImmutable;

class Quote extends AggregateRoot
{
    private function __construct(
        private QuoteId $id,
        private ClientId $clientId,
        private OrganizationId $organizationId,
        private QuoteNumber $number,
        private QuoteStatus $status,
        private string $title,
        private Currency $currency,
        private CarbonImmutable $validUntil,
        private Money $total
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
            currency: $currency,
            validUntil: $validUntil,
            total: Money::zero($currency)
        );
        
        $quote->record(new QuoteDraftCreated(
            quoteId: $quote->id->toString(),
            clientId: $clientId->toString(),
            organizationId: $organizationId->toString(),
            number: $number->toString(),
            title: $title,
            occurredOn: new \DateTimeImmutable()
        ));
        
        return $quote;
    }
    
    // Getters
    public function id(): QuoteId { return $this->id; }
    public function status(): QuoteStatus { return $this->status; }
    public function total(): Money { return $this->total; }
}
```

**Kolejny test - business rule:**
```php
test('cannot send empty quote', function () {
    $quote = Quote::createDraft(
        clientId: ClientId::generate(),
        organizationId: OrganizationId::generate(),
        number: QuoteNumber::generate(1),
        title: 'Kitchen Renovation',
        currency: Currency::USD(),
        validUntil: CarbonImmutable::now()->addDays(30)
    );
    
    $quote->send();
})->throws(CannotSendEmptyQuoteException::class);
```

**Implementacja:**
```php
// app/Domains/QuoteManagement/Domain/Quote.php

public function send(): void
{
    if ($this->lineItems->isEmpty()) {
        throw new CannotSendEmptyQuoteException();
    }
    
    if (!$this->status->isDraft()) {
        throw new QuoteAlreadySentException();
    }
    
    $this->status = QuoteStatus::sent();
    $this->sentAt = CarbonImmutable::now();
    
    $this->record(new QuoteSent(
        quoteId: $this->id->toString(),
        sentAt: new \DateTimeImmutable()
    ));
}
```

**Kontynuuj TDD dla każdej business rule:**
- Cannot edit sent quote
- Cannot accept expired quote
- Total calculation is correct
- etc.

✅ **Commit:** `test: add Quote aggregate business rule tests`
✅ **Commit:** `feat(quote): implement Quote aggregate`

---

### Step 6: Repository Interface

**Domain definiuje interface:**
```php
// app/Domains/QuoteManagement/Domain/QuoteRepository.php
<?php

namespace App\Domains\QuoteManagement\Domain;

use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteNumber;

interface QuoteRepository
{
    public function nextIdentity(): QuoteId;
    public function nextNumber(): QuoteNumber;
    public function save(Quote $quote): void;
    public function findById(QuoteId $id): ?Quote;
}
```

---

### Step 7: Application Layer (Commands & Handlers)

**Test:**
```php
// tests/Feature/Domains/QuoteManagement/CreateQuoteDraftTest.php
<?php

use App\Domains\QuoteManagement\Application\Commands\CreateQuoteDraft;
use App\Domains\QuoteManagement\Application\Handlers\CreateQuoteDraftHandler;

test('can create quote draft', function () {
    $client = Client::factory()->create();
    
    $command = new CreateQuoteDraft(
        clientId: $client->id,
        title: 'Kitchen Renovation',
        validUntil: now()->addDays(30)->toDateString()
    );
    
    $handler = app(CreateQuoteDraftHandler::class);
    $quoteId = $handler->handle($command);
    
    expect($quoteId)->toBeInstanceOf(QuoteId::class);
    
    $this->assertDatabaseHas('quotes', [
        'id' => $quoteId->toString(),
        'title' => 'Kitchen Renovation',
        'status' => 'draft',
    ]);
});
```

**Implementacja:**
```php
// app/Domains/QuoteManagement/Application/Commands/CreateQuoteDraft.php
<?php

namespace App\Domains\QuoteManagement\Application\Commands;

final class CreateQuoteDraft
{
    public function __construct(
        public readonly string $clientId,
        public readonly string $title,
        public readonly string $validUntil
    ) {}
}

// app/Domains/QuoteManagement/Application/Handlers/CreateQuoteDraftHandler.php
<?php

namespace App\Domains\QuoteManagement\Application\Handlers;

use App\Domains\QuoteManagement\Application\Commands\CreateQuoteDraft;
use App\Domains\QuoteManagement\Domain\QuoteRepository;
use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\ValueObjects\*;
use Carbon\CarbonImmutable;

class CreateQuoteDraftHandler
{
    public function __construct(
        private QuoteRepository $quotes
    ) {}
    
    public function handle(CreateQuoteDraft $command): QuoteId
    {
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

✅ **Commit:** `test: add CreateQuoteDraft handler tests`
✅ **Commit:** `feat(quote): implement CreateQuoteDraft command and handler`

---

### Step 8: Infrastructure Layer

**Repository Implementation:**
```php
// app/Domains/QuoteManagement/Infrastructure/Persistence/EloquentQuoteRepository.php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\Persistence;

use App\Domains\QuoteManagement\Domain\QuoteRepository;
use App\Domains\QuoteManagement\Domain\Quote;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Event;

class EloquentQuoteRepository implements QuoteRepository
{
    public function nextNumber(): QuoteNumber
    {
        $lastNumber = QuoteEloquentModel::where('organization_id', auth()->user()->organization_id)
            ->max('sequence') ?? 0;
        
        return QuoteNumber::generate($lastNumber + 1);
    }
    
    public function save(Quote $quote): void
    {
        DB::transaction(function () use ($quote) {
            QuoteEloquentModel::updateOrCreate(
                ['id' => $quote->id()->toString()],
                [
                    'organization_id' => $quote->organizationId()->toString(),
                    'client_id' => $quote->clientId()->toString(),
                    'number' => $quote->number()->toString(),
                    'title' => $quote->title(),
                    'status' => $quote->status()->value(),
                    'total' => $quote->total()->toFloat(),
                    'currency' => $quote->total()->currency()->code(),
                    // ... more fields
                ]
            );
            
            // Dispatch domain events
            foreach ($quote->releaseEvents() as $event) {
                Event::dispatch($event);
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
    
    private function mapToDomain(QuoteEloquentModel $model): Quote
    {
        // Reconstruction from database
        // This is complex - need to use reflection or 
        // add reconstruction method to aggregate
    }
}
```

**Service Provider Binding:**
```php
// app/Providers/AppServiceProvider.php

public function register(): void
{
    $this->app->bind(
        \App\Domains\QuoteManagement\Domain\QuoteRepository::class,
        \App\Domains\QuoteManagement\Infrastructure\Persistence\EloquentQuoteRepository::class
    );
}
```

✅ **Commit:** `feat(quote): implement Eloquent repository`

---

### Step 9: HTTP Layer (API)

**Controller:**
```php
// app/Http/Controllers/Api/QuoteController.php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Domains\QuoteManagement\Application\Commands\CreateQuoteDraft;
use App\Domains\QuoteManagement\Application\Handlers\CreateQuoteDraftHandler;

class QuoteController extends Controller
{
    public function __construct(
        private CreateQuoteDraftHandler $createQuoteDraftHandler
    ) {}
    
    public function store(CreateQuoteRequest $request)
    {
        $quoteId = $this->createQuoteDraftHandler->handle(
            new CreateQuoteDraft(
                clientId: $request->client_id,
                title: $request->title,
                validUntil: $request->valid_until
            )
        );
        
        return response()->json([
            'id' => $quoteId->toString(),
            'message' => 'Quote draft created successfully'
        ], 201);
    }
}
```

**Routes:**
```php
// routes/api.php
Route::middleware('auth:api')->group(function () {
    Route::prefix('quotes')->group(function () {
        Route::post('/', [QuoteController::class, 'store']);
        Route::get('/', [QuoteController::class, 'index']);
        Route::get('/{id}', [QuoteController::class, 'show']);
        Route::post('/{id}/send', [QuoteController::class, 'send']);
        Route::post('/{id}/accept', [QuoteController::class, 'accept']);
    });
});
```

**API Test:**
```php
// tests/Feature/Api/QuoteControllerTest.php
test('can create quote via API', function () {
    $user = User::factory()->create();
    $client = Client::factory()->create(['organization_id' => $user->organization_id]);
    
    $response = $this->actingAs($user)
        ->postJson('/api/quotes', [
            'client_id' => $client->id,
            'title' => 'Kitchen Renovation',
            'valid_until' => now()->addDays(30)->toDateString(),
        ]);
    
    $response->assertStatus(201)
             ->assertJsonStructure(['id', 'message']);
});
```

✅ **Commit:** `feat(quote): add HTTP API endpoints`

---

## 🎓 Lessons Learned

### Co działa dobrze:
1. **TDD** - Tests først wymusza przemyślenie designu
2. **Value Objects** - Encapsulation i immutability
3. **Domain Events** - Loose coupling między kontekstami
4. **Layering** - Jasna separacja odpowiedzialności

### Częste pułapki:
1. **Anemic Domain Model** - Nie wrzucaj logiki do serwisów
2. **Too Big Aggregates** - Aggregate = granica transakcji
3. **Premature Optimization** - Zacznij prosto, refaktoruj później
4. **Skipping Tests** - Tests są dokumentacją

---

## 📚 Następne kroki

Po zaimplementowaniu Quote:

1. **Event-Driven Architecture** (ADR-012)
   - Setup Event Bus
   - Implement listeners in other contexts
   - Test cross-context communication

2. **CQRS** (ADR-013)
   - Create read models
   - Implement projectors
   - Optimize queries

3. **Remaining Contexts**
   - Project Management (simpler than Quote)
   - Invoice & Payment
   - Document Management

---

## 🔍 Architecture Tests

```php
// tests/Architecture/DomainLayerTest.php
test('domain layer does not depend on infrastructure', function () {
    expect('App\Domains\QuoteManagement\Domain')
        ->not->toUse([
            'Illuminate\Database',
            'Illuminate\Http',
        ]);
});

test('application handlers use domain interfaces', function () {
    expect('App\Domains\QuoteManagement\Application\Handlers')
        ->toUseNothing()
        ->ignoring([
            'App\Domains\QuoteManagement\Domain',
            'App\Domains\QuoteManagement\Application',
        ]);
});
```

---

## 💡 Tips

**When stuck:**
1. Draw on paper
2. Write test first
3. Implement minimum to pass
4. Refactor
5. Repeat

**Remember:**
- Domain = Business Logic (pure PHP)
- Application = Use Cases (orchestration)
- Infrastructure = Technical Details (Laravel/DB)

**Ask yourself:**
- "Is this a business rule?" → Domain
- "Is this a use case?" → Application
- "Is this a technical detail?" → Infrastructure

---

**You're ready to start! Good luck! 🚀**

---

**Last Updated:** 2024-11-12
