Understood! I'll provide the content in markdown codeblocks for you to copy. Let me create all four phase plans.

---

## Phase 2: Event-Driven Communication (Weeks 3-4)

```markdown
# BuildFlow Phase 2: Event-Driven Communication

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement Event Bus infrastructure enabling cross-context communication, with QuoteAccepted → ProjectCreated as the showcase flow.

**Architecture:** Laravel's event system with domain event dispatching from repositories, event store for audit/replay, and async listeners via queues for cross-context reactions.

**Tech Stack:** Laravel Events, Laravel Queues, PostgreSQL event_store table, Pest testing

**Dependencies from Phase 1:**
- ✅ SharedKernel/Domain/DomainEvent interface
- ✅ SharedKernel/Domain/AggregateRoot with event recording
- ✅ Quote aggregate with QuoteAccepted event
- ✅ EloquentQuoteRepository dispatching events

---

## Week 3: Event Infrastructure

---

### Day 22 (Monday): Event Dispatcher & Event Store

**Goal:** Create robust event dispatching with persistence for audit trail

#### Task 1: Event Store Migration

**Files:**
- Create: `database/migrations/xxxx_create_event_store_table.php`

**Step 1: Create migration**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('event_store', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->string('event_type', 255)->index();
            $table->string('aggregate_type', 100)->index();
            $table->uuid('aggregate_id')->index();
            $table->json('payload');
            $table->json('metadata')->nullable();
            $table->unsignedBigInteger('sequence');
            $table->timestamp('occurred_at');
            $table->timestamp('stored_at')->useCurrent();
            
            $table->index(['aggregate_id', 'sequence']);
            $table->index(['event_type', 'occurred_at']);
            $table->index('occurred_at');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('event_store');
    }
};
```

**Step 2: Run migration**

Run: `php artisan migrate`
Expected: Migration successful

**Step 3: Commit**

```bash
git add database/migrations/*_create_event_store_table.php
git commit -m "feat: add event store migration"
```

---

#### Task 2: Event Store Repository

**Files:**
- Create: `app/SharedKernel/Infrastructure/EventStore/EventStoreRepository.php`
- Create: `app/SharedKernel/Infrastructure/EventStore/StoredEvent.php`
- Test: `tests/Integration/SharedKernel/EventStoreRepositoryTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Integration\SharedKernel;

use App\SharedKernel\Infrastructure\EventStore\EventStoreRepository;
use App\SharedKernel\Infrastructure\EventStore\StoredEvent;
use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class EventStoreRepositoryTest extends TestCase
{
    use RefreshDatabase;

    private EventStoreRepository $eventStore;

    protected function setUp(): void
    {
        parent::setUp();
        $this->eventStore = new EventStoreRepository();
    }

    /** @test */
    public function can_store_domain_event(): void
    {
        $event = new QuoteDraftCreated(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            quoteNumber: 'Q-000001',
            title: 'Test Quote',
            currency: 'USD',
            occurredOn: new \DateTimeImmutable()
        );

        $this->eventStore->append($event, 'Quote');

        $this->assertDatabaseHas('event_store', [
            'aggregate_id' => 'quote-123',
            'event_type' => QuoteDraftCreated::class,
            'aggregate_type' => 'Quote',
        ]);
    }

    /** @test */
    public function can_retrieve_events_for_aggregate(): void
    {
        $event1 = new QuoteDraftCreated(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            quoteNumber: 'Q-000001',
            title: 'Test Quote',
            currency: 'USD',
            occurredOn: new \DateTimeImmutable()
        );

        $this->eventStore->append($event1, 'Quote');

        $events = $this->eventStore->getStream('quote-123');

        $this->assertCount(1, $events);
        $this->assertInstanceOf(StoredEvent::class, $events[0]);
    }

    /** @test */
    public function events_have_sequential_sequence_numbers(): void
    {
        $event1 = new QuoteDraftCreated(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            quoteNumber: 'Q-000001',
            title: 'First',
            currency: 'USD',
            occurredOn: new \DateTimeImmutable()
        );

        $event2 = new QuoteDraftCreated(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            quoteNumber: 'Q-000001',
            title: 'Second',
            currency: 'USD',
            occurredOn: new \DateTimeImmutable()
        );

        $this->eventStore->append($event1, 'Quote');
        $this->eventStore->append($event2, 'Quote');

        $events = $this->eventStore->getStream('quote-123');

        $this->assertEquals(1, $events[0]->sequence);
        $this->assertEquals(2, $events[1]->sequence);
    }
}
```

**Step 2: Run test to verify it fails**

Run: `./vendor/bin/pest tests/Integration/SharedKernel/EventStoreRepositoryTest.php`
Expected: FAIL - class not found

**Step 3: Write StoredEvent DTO**

```php
<?php

namespace App\SharedKernel\Infrastructure\EventStore;

final class StoredEvent
{
    public function __construct(
        public readonly string $id,
        public readonly string $eventType,
        public readonly string $aggregateType,
        public readonly string $aggregateId,
        public readonly array $payload,
        public readonly ?array $metadata,
        public readonly int $sequence,
        public readonly \DateTimeImmutable $occurredAt,
        public readonly \DateTimeImmutable $storedAt
    ) {}

    public static function fromRow(object $row): self
    {
        return new self(
            id: $row->id,
            eventType: $row->event_type,
            aggregateType: $row->aggregate_type,
            aggregateId: $row->aggregate_id,
            payload: json_decode($row->payload, true),
            metadata: $row->metadata ? json_decode($row->metadata, true) : null,
            sequence: (int) $row->sequence,
            occurredAt: new \DateTimeImmutable($row->occurred_at),
            storedAt: new \DateTimeImmutable($row->stored_at)
        );
    }
}
```

**Step 4: Write EventStoreRepository**

```php
<?php

namespace App\SharedKernel\Infrastructure\EventStore;

use App\SharedKernel\Domain\DomainEvent;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Str;

class EventStoreRepository
{
    public function append(DomainEvent $event, string $aggregateType): void
    {
        $sequence = $this->nextSequence($event->aggregateId());

        DB::table('event_store')->insert([
            'id' => Str::uuid()->toString(),
            'event_type' => get_class($event),
            'aggregate_type' => $aggregateType,
            'aggregate_id' => $event->aggregateId(),
            'payload' => json_encode($event->toArray()),
            'metadata' => json_encode([
                'user_id' => auth()->id(),
                'ip_address' => request()?->ip(),
                'user_agent' => request()?->userAgent(),
            ]),
            'sequence' => $sequence,
            'occurred_at' => $event->occurredOn()->format('Y-m-d H:i:s.u'),
        ]);
    }

    public function getStream(string $aggregateId): array
    {
        return DB::table('event_store')
            ->where('aggregate_id', $aggregateId)
            ->orderBy('sequence')
            ->get()
            ->map(fn($row) => StoredEvent::fromRow($row))
            ->toArray();
    }

    public function getEventsByType(string $eventType, ?\DateTimeImmutable $since = null): array
    {
        $query = DB::table('event_store')
            ->where('event_type', $eventType)
            ->orderBy('occurred_at');

        if ($since) {
            $query->where('occurred_at', '>', $since->format('Y-m-d H:i:s.u'));
        }

        return $query->get()
            ->map(fn($row) => StoredEvent::fromRow($row))
            ->toArray();
    }

    private function nextSequence(string $aggregateId): int
    {
        $max = DB::table('event_store')
            ->where('aggregate_id', $aggregateId)
            ->max('sequence');

        return ($max ?? 0) + 1;
    }
}
```

**Step 5: Run tests to verify they pass**

Run: `./vendor/bin/pest tests/Integration/SharedKernel/EventStoreRepositoryTest.php`
Expected: PASS (3 tests)

**Step 6: Commit**

```bash
git add app/SharedKernel/Infrastructure/EventStore/
git add tests/Integration/SharedKernel/EventStoreRepositoryTest.php
git commit -m "feat: implement EventStoreRepository for event persistence"
```

---

#### Task 3: Domain Event Dispatcher

**Files:**
- Create: `app/SharedKernel/Infrastructure/EventBus/DomainEventDispatcher.php`
- Create: `app/SharedKernel/Infrastructure/EventBus/DomainEventDispatcherInterface.php`
- Test: `tests/Unit/SharedKernel/DomainEventDispatcherTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Unit\SharedKernel;

use App\SharedKernel\Infrastructure\EventBus\DomainEventDispatcher;
use App\SharedKernel\Infrastructure\EventStore\EventStoreRepository;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use Illuminate\Support\Facades\Event;
use Mockery;
use Tests\TestCase;

class DomainEventDispatcherTest extends TestCase
{
    /** @test */
    public function dispatches_event_to_laravel_event_system(): void
    {
        Event::fake([QuoteAccepted::class]);
        
        $eventStore = Mockery::mock(EventStoreRepository::class);
        $eventStore->shouldReceive('append')->once();
        
        $dispatcher = new DomainEventDispatcher($eventStore);
        
        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );
        
        $dispatcher->dispatch($event, 'Quote');
        
        Event::assertDispatched(QuoteAccepted::class);
    }

    /** @test */
    public function stores_event_before_dispatching(): void
    {
        Event::fake();
        
        $eventStore = Mockery::mock(EventStoreRepository::class);
        $eventStore->shouldReceive('append')
            ->once()
            ->with(Mockery::type(QuoteAccepted::class), 'Quote');
        
        $dispatcher = new DomainEventDispatcher($eventStore);
        
        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );
        
        $dispatcher->dispatch($event, 'Quote');
    }

    /** @test */
    public function can_dispatch_multiple_events(): void
    {
        Event::fake();
        
        $eventStore = Mockery::mock(EventStoreRepository::class);
        $eventStore->shouldReceive('append')->times(2);
        
        $dispatcher = new DomainEventDispatcher($eventStore);
        
        $events = [
            new QuoteAccepted(
                quoteId: 'quote-123',
                clientId: 'client-456',
                organizationId: 'org-789',
                totalAmount: 5000.00,
                currency: 'USD',
                acceptedAt: new \DateTimeImmutable()
            ),
            new QuoteAccepted(
                quoteId: 'quote-456',
                clientId: 'client-789',
                organizationId: 'org-789',
                totalAmount: 3000.00,
                currency: 'USD',
                acceptedAt: new \DateTimeImmutable()
            ),
        ];
        
        $dispatcher->dispatchAll($events, 'Quote');
        
        Event::assertDispatchedTimes(QuoteAccepted::class, 2);
    }
}
```

**Step 2: Run test to verify it fails**

Run: `./vendor/bin/pest tests/Unit/SharedKernel/DomainEventDispatcherTest.php`
Expected: FAIL - class not found

**Step 3: Write the interface**

```php
<?php

namespace App\SharedKernel\Infrastructure\EventBus;

use App\SharedKernel\Domain\DomainEvent;

interface DomainEventDispatcherInterface
{
    public function dispatch(DomainEvent $event, string $aggregateType): void;
    
    public function dispatchAll(array $events, string $aggregateType): void;
}
```

**Step 4: Write the implementation**

```php
<?php

namespace App\SharedKernel\Infrastructure\EventBus;

use App\SharedKernel\Domain\DomainEvent;
use App\SharedKernel\Infrastructure\EventStore\EventStoreRepository;
use Illuminate\Support\Facades\Event;

class DomainEventDispatcher implements DomainEventDispatcherInterface
{
    public function __construct(
        private EventStoreRepository $eventStore
    ) {}

    public function dispatch(DomainEvent $event, string $aggregateType): void
    {
        // Store first (for audit trail)
        $this->eventStore->append($event, $aggregateType);
        
        // Then dispatch to Laravel's event system
        Event::dispatch($event);
    }

    public function dispatchAll(array $events, string $aggregateType): void
    {
        foreach ($events as $event) {
            $this->dispatch($event, $aggregateType);
        }
    }
}
```

**Step 5: Run tests to verify they pass**

Run: `./vendor/bin/pest tests/Unit/SharedKernel/DomainEventDispatcherTest.php`
Expected: PASS (3 tests)

**Step 6: Commit**

```bash
git add app/SharedKernel/Infrastructure/EventBus/
git commit -m "feat: implement DomainEventDispatcher with event store integration"
```

---

### Day 23 (Tuesday): Update Repository & Register Services

**Goal:** Integrate event dispatcher into existing repository, register services in container

#### Task 4: Update EloquentQuoteRepository

**Files:**
- Modify: `app/Domains/QuoteManagement/Infrastructure/Persistence/EloquentQuoteRepository.php`
- Test: `tests/Integration/Domains/QuoteManagement/EloquentQuoteRepositoryTest.php`

**Step 1: Write/update integration test**

```php
<?php

namespace Tests\Integration\Domains\QuoteManagement;

use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Infrastructure\Persistence\EloquentQuoteRepository;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class EloquentQuoteRepositoryTest extends TestCase
{
    use RefreshDatabase;

    private EloquentQuoteRepository $repository;

    protected function setUp(): void
    {
        parent::setUp();
        $this->repository = app(EloquentQuoteRepository::class);
    }

    /** @test */
    public function dispatches_domain_events_on_save(): void
    {
        Event::fake([QuoteDraftCreated::class]);

        $quote = $this->createDraftQuote();

        $this->repository->save($quote);

        Event::assertDispatched(QuoteDraftCreated::class, function ($event) use ($quote) {
            return $event->quoteId === $quote->id()->toString();
        });
    }

    /** @test */
    public function stores_events_in_event_store(): void
    {
        $quote = $this->createDraftQuote();

        $this->repository->save($quote);

        $this->assertDatabaseHas('event_store', [
            'aggregate_id' => $quote->id()->toString(),
            'event_type' => QuoteDraftCreated::class,
            'aggregate_type' => 'Quote',
        ]);
    }

    /** @test */
    public function dispatches_multiple_events_in_order(): void
    {
        Event::fake([QuoteDraftCreated::class, QuoteAccepted::class]);

        $quote = $this->createSentQuoteWithLineItems();
        $quote->accept();

        $this->repository->save($quote);

        // Should dispatch QuoteAccepted (QuoteDraftCreated already released)
        Event::assertDispatched(QuoteAccepted::class);
    }

    private function createDraftQuote(): Quote
    {
        // Use factory or builder pattern
        return Quote::createDraft(
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            number: $this->repository->nextNumber(),
            title: 'Test Quote',
            currency: Currency::USD(),
            validUntil: CarbonImmutable::now()->addDays(30)
        );
    }
}
```

**Step 2: Update EloquentQuoteRepository**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\Persistence;

use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\QuoteRepository;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteNumber;
use App\SharedKernel\Infrastructure\EventBus\DomainEventDispatcherInterface;
use Illuminate\Support\Facades\DB;

class EloquentQuoteRepository implements QuoteRepository
{
    public function __construct(
        private DomainEventDispatcherInterface $eventDispatcher
    ) {}

    public function nextIdentity(): QuoteId
    {
        return QuoteId::generate();
    }

    public function nextNumber(): QuoteNumber
    {
        $lastSequence = QuoteEloquentModel::max('sequence') ?? 0;
        return QuoteNumber::generate($lastSequence + 1);
    }

    public function save(Quote $quote): void
    {
        DB::transaction(function () use ($quote) {
            // Save quote
            QuoteEloquentModel::updateOrCreate(
                ['id' => $quote->id()->toString()],
                $this->mapToDatabase($quote)
            );

            // Save line items
            $this->saveLineItems($quote);

            // Dispatch all recorded events
            $this->eventDispatcher->dispatchAll(
                $quote->releaseEvents(),
                'Quote'
            );
        });
    }

    public function findById(QuoteId $id): ?Quote
    {
        $model = QuoteEloquentModel::with('lineItems')
            ->find($id->toString());

        if (!$model) {
            return null;
        }

        return $this->mapToDomain($model);
    }

    private function mapToDatabase(Quote $quote): array
    {
        return [
            'organization_id' => $quote->organizationId()->toString(),
            'client_id' => $quote->clientId()->toString(),
            'number' => $quote->number()->toString(),
            'sequence' => $quote->number()->sequence(),
            'title' => $quote->title(),
            'status' => $quote->status()->value(),
            'subtotal' => $quote->subtotal()->toCents(),
            'tax_rate' => $quote->taxRate()->value(),
            'discount' => $quote->discount()->toCents(),
            'total' => $quote->total()->toCents(),
            'currency' => $quote->currency()->code(),
            'valid_until' => $quote->validUntil(),
            'sent_at' => $quote->sentAt(),
            'accepted_at' => $quote->acceptedAt(),
        ];
    }

    private function saveLineItems(Quote $quote): void
    {
        // Delete existing and re-insert (simple approach)
        LineItemEloquentModel::where('quote_id', $quote->id()->toString())->delete();

        foreach ($quote->lineItems() as $index => $item) {
            LineItemEloquentModel::create([
                'id' => $item->id()->toString(),
                'quote_id' => $quote->id()->toString(),
                'description' => $item->description(),
                'quantity' => $item->quantity(),
                'unit_price' => $item->unitPrice()->toCents(),
                'total' => $item->total()->toCents(),
                'sort_order' => $index,
            ]);
        }
    }

    private function mapToDomain(QuoteEloquentModel $model): Quote
    {
        // Reconstruct aggregate from database
        return Quote::reconstitute(
            id: QuoteId::fromString($model->id),
            // ... map all fields
        );
    }
}
```

**Step 3: Run tests**

Run: `./vendor/bin/pest tests/Integration/Domains/QuoteManagement/EloquentQuoteRepositoryTest.php`
Expected: PASS

**Step 4: Commit**

```bash
git add app/Domains/QuoteManagement/Infrastructure/Persistence/EloquentQuoteRepository.php
git add tests/Integration/Domains/QuoteManagement/EloquentQuoteRepositoryTest.php
git commit -m "feat(quote): integrate event dispatcher into repository"
```

---

#### Task 5: Register Services in Container

**Files:**
- Modify: `app/Providers/AppServiceProvider.php`
- Create: `app/Providers/EventServiceProvider.php` (update)

**Step 1: Update AppServiceProvider**

```php
<?php

namespace App\Providers;

use App\SharedKernel\Infrastructure\EventBus\DomainEventDispatcher;
use App\SharedKernel\Infrastructure\EventBus\DomainEventDispatcherInterface;
use App\SharedKernel\Infrastructure\EventStore\EventStoreRepository;
use App\Domains\QuoteManagement\Domain\QuoteRepository;
use App\Domains\QuoteManagement\Infrastructure\Persistence\EloquentQuoteRepository;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        // Event Infrastructure
        $this->app->singleton(EventStoreRepository::class);
        
        $this->app->singleton(
            DomainEventDispatcherInterface::class,
            DomainEventDispatcher::class
        );

        // Repositories
        $this->app->bind(
            QuoteRepository::class,
            EloquentQuoteRepository::class
        );
    }

    public function boot(): void
    {
        //
    }
}
```

**Step 2: Commit**

```bash
git add app/Providers/AppServiceProvider.php
git commit -m "feat: register event infrastructure in service container"
```

---

### Day 24 (Wednesday): Cross-Context Event Listeners

**Goal:** Implement ProjectManagement listener that reacts to QuoteAccepted

#### Task 6: Create Project Context Skeleton

**Files:**
- Create: `app/Domains/ProjectManagement/Domain/Project.php`
- Create: `app/Domains/ProjectManagement/Domain/ValueObjects/ProjectId.php`
- Create: `app/Domains/ProjectManagement/Domain/ValueObjects/ProjectStatus.php`
- Create: `app/Domains/ProjectManagement/Domain/Events/ProjectCreated.php`
- Create: `app/Domains/ProjectManagement/Domain/ProjectRepository.php`

**Step 1: Create ProjectId value object**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\Identifier;
use Illuminate\Support\Str;

final class ProjectId extends Identifier
{
    public static function generate(): self
    {
        return new self(Str::uuid()->toString());
    }

    public static function fromString(string $id): self
    {
        if (!Str::isUuid($id)) {
            throw new \InvalidArgumentException("Invalid project ID: {$id}");
        }
        return new self($id);
    }
}
```

**Step 2: Create ProjectStatus**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class ProjectStatus extends ValueObject
{
    private const NOT_STARTED = 'not_started';
    private const IN_PROGRESS = 'in_progress';
    private const ON_HOLD = 'on_hold';
    private const COMPLETED = 'completed';
    private const CANCELLED = 'cancelled';

    private function __construct(private string $value) {}

    public static function notStarted(): self
    {
        return new self(self::NOT_STARTED);
    }

    public static function inProgress(): self
    {
        return new self(self::IN_PROGRESS);
    }

    public static function onHold(): self
    {
        return new self(self::ON_HOLD);
    }

    public static function completed(): self
    {
        return new self(self::COMPLETED);
    }

    public static function cancelled(): self
    {
        return new self(self::CANCELLED);
    }

    public function value(): string
    {
        return $this->value;
    }

    public function isActive(): bool
    {
        return in_array($this->value, [self::NOT_STARTED, self::IN_PROGRESS, self::ON_HOLD]);
    }

    public function equals(ValueObject $other): bool
    {
        return $other instanceof self && $this->value === $other->value;
    }
}
```

**Step 3: Create ProjectCreated event**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\Events;

use App\SharedKernel\Domain\DomainEvent;

final class ProjectCreated implements DomainEvent
{
    public function __construct(
        public readonly string $projectId,
        public readonly string $quoteId,
        public readonly string $clientId,
        public readonly string $organizationId,
        public readonly string $title,
        public readonly float $budget,
        public readonly string $currency,
        public readonly \DateTimeImmutable $createdAt
    ) {}

    public function eventName(): string
    {
        return 'project.created';
    }

    public function occurredOn(): \DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function aggregateId(): string
    {
        return $this->projectId;
    }

    public function toArray(): array
    {
        return [
            'project_id' => $this->projectId,
            'quote_id' => $this->quoteId,
            'client_id' => $this->clientId,
            'organization_id' => $this->organizationId,
            'title' => $this->title,
            'budget' => $this->budget,
            'currency' => $this->currency,
            'created_at' => $this->createdAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

**Step 4: Create minimal Project aggregate**

```php
<?php

namespace App\Domains\ProjectManagement\Domain;

use App\SharedKernel\Domain\AggregateRoot;
use App\Domains\ProjectManagement\Domain\ValueObjects\ProjectId;
use App\Domains\ProjectManagement\Domain\ValueObjects\ProjectStatus;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;

class Project extends AggregateRoot
{
    private function __construct(
        private ProjectId $id,
        private QuoteId $originQuoteId,
        private ClientId $clientId,
        private OrganizationId $organizationId,
        private string $title,
        private ProjectStatus $status,
        private Money $budget,
        private \DateTimeImmutable $createdAt
    ) {}

    public static function createFromQuote(
        QuoteId $quoteId,
        ClientId $clientId,
        OrganizationId $organizationId,
        string $title,
        Money $budget
    ): self {
        $project = new self(
            id: ProjectId::generate(),
            originQuoteId: $quoteId,
            clientId: $clientId,
            organizationId: $organizationId,
            title: $title,
            status: ProjectStatus::notStarted(),
            budget: $budget,
            createdAt: new \DateTimeImmutable()
        );

        $project->record(new ProjectCreated(
            projectId: $project->id->toString(),
            quoteId: $quoteId->toString(),
            clientId: $clientId->toString(),
            organizationId: $organizationId->toString(),
            title: $title,
            budget: $budget->toFloat(),
            currency: $budget->currency()->code(),
            createdAt: new \DateTimeImmutable()
        ));

        return $project;
    }

    // Getters
    public function id(): ProjectId { return $this->id; }
    public function originQuoteId(): QuoteId { return $this->originQuoteId; }
    public function clientId(): ClientId { return $this->clientId; }
    public function organizationId(): OrganizationId { return $this->organizationId; }
    public function title(): string { return $this->title; }
    public function status(): ProjectStatus { return $this->status; }
    public function budget(): Money { return $this->budget; }
}
```

**Step 5: Commit**

```bash
git add app/Domains/ProjectManagement/
git commit -m "feat(project): create Project aggregate skeleton"
```

---

#### Task 7: Create QuoteAccepted Listener

**Files:**
- Create: `app/Domains/ProjectManagement/Infrastructure/EventListeners/CreateProjectWhenQuoteAccepted.php`
- Create: `app/Domains/ProjectManagement/Application/Commands/CreateProjectFromQuote.php`
- Create: `app/Domains/ProjectManagement/Application/Handlers/CreateProjectFromQuoteHandler.php`
- Test: `tests/Integration/Domains/ProjectManagement/CreateProjectWhenQuoteAcceptedTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Integration\Domains\ProjectManagement;

use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;
use App\Domains\ProjectManagement\Infrastructure\EventListeners\CreateProjectWhenQuoteAccepted;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class CreateProjectWhenQuoteAcceptedTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function creates_project_when_quote_accepted(): void
    {
        Event::fake([ProjectCreated::class]);

        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );

        $listener = app(CreateProjectWhenQuoteAccepted::class);
        $listener->handle($event);

        $this->assertDatabaseHas('projects', [
            'origin_quote_id' => 'quote-123',
            'client_id' => 'client-456',
            'organization_id' => 'org-789',
        ]);
    }

    /** @test */
    public function project_has_budget_from_quote_total(): void
    {
        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 7500.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );

        $listener = app(CreateProjectWhenQuoteAccepted::class);
        $listener->handle($event);

        $this->assertDatabaseHas('projects', [
            'origin_quote_id' => 'quote-123',
            'budget' => 750000, // cents
        ]);
    }

    /** @test */
    public function project_created_event_is_dispatched(): void
    {
        Event::fake([ProjectCreated::class]);

        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );

        $listener = app(CreateProjectWhenQuoteAccepted::class);
        $listener->handle($event);

        Event::assertDispatched(ProjectCreated::class, function ($event) {
            return $event->quoteId === 'quote-123';
        });
    }
}
```

**Step 2: Create Command DTO**

```php
<?php

namespace App\Domains\ProjectManagement\Application\Commands;

final class CreateProjectFromQuote
{
    public function __construct(
        public readonly string $quoteId,
        public readonly string $clientId,
        public readonly string $organizationId,
        public readonly float $budget,
        public readonly string $currency
    ) {}
}
```

**Step 3: Create Handler**

```php
<?php

namespace App\Domains\ProjectManagement\Application\Handlers;

use App\Domains\ProjectManagement\Application\Commands\CreateProjectFromQuote;
use App\Domains\ProjectManagement\Domain\Project;
use App\Domains\ProjectManagement\Domain\ProjectRepository;
use App\Domains\ProjectManagement\Domain\ValueObjects\ProjectId;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use App\Domains\QuoteManagement\Domain\ValueObjects\Currency;

class CreateProjectFromQuoteHandler
{
    public function __construct(
        private ProjectRepository $projects
    ) {}

    public function handle(CreateProjectFromQuote $command): ProjectId
    {
        $project = Project::createFromQuote(
            quoteId: QuoteId::fromString($command->quoteId),
            clientId: ClientId::fromString($command->clientId),
            organizationId: OrganizationId::fromString($command->organizationId),
            title: $this->generateProjectTitle($command->quoteId),
            budget: Money::fromFloat($command->budget, Currency::fromCode($command->currency))
        );

        $this->projects->save($project);

        return $project->id();
    }

    private function generateProjectTitle(string $quoteId): string
    {
        // Could fetch quote title, for now use simple format
        return "Project from Quote";
    }
}
```

**Step 4: Create Listener**

```php
<?php

namespace App\Domains\ProjectManagement\Infrastructure\EventListeners;

use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\ProjectManagement\Application\Commands\CreateProjectFromQuote;
use App\Domains\ProjectManagement\Application\Handlers\CreateProjectFromQuoteHandler;
use Illuminate\Contracts\Queue\ShouldQueue;

class CreateProjectWhenQuoteAccepted implements ShouldQueue
{
    public $queue = 'default';
    public $tries = 3;
    public $backoff = [60, 300, 900];

    public function __construct(
        private CreateProjectFromQuoteHandler $handler
    ) {}

    public function handle(QuoteAccepted $event): void
    {
        $this->handler->handle(
            new CreateProjectFromQuote(
                quoteId: $event->quoteId,
                clientId: $event->clientId,
                organizationId: $event->organizationId,
                budget: $event->totalAmount,
                currency: $event->currency
            )
        );
    }

    public function failed(QuoteAccepted $event, \Throwable $exception): void
    {
        // Log failure, send alert
        \Log::error('Failed to create project from quote', [
            'quote_id' => $event->quoteId,
            'error' => $exception->getMessage(),
        ]);
    }
}
```

**Step 5: Create Project migration and repository (needed for test)**

```php
// database/migrations/xxxx_create_projects_table.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('projects', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('origin_quote_id')->index();
            $table->uuid('client_id')->index();
            $table->uuid('organization_id')->index();
            $table->string('title');
            $table->string('status', 20)->default('not_started');
            $table->bigInteger('budget'); // cents
            $table->string('currency', 3);
            $table->timestamp('started_at')->nullable();
            $table->timestamp('completed_at')->nullable();
            $table->timestamps();

            $table->index(['organization_id', 'status']);
            $table->foreign('origin_quote_id')->references('id')->on('quotes');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('projects');
    }
};
```

**Step 6: Run tests**

Run: `./vendor/bin/pest tests/Integration/Domains/ProjectManagement/CreateProjectWhenQuoteAcceptedTest.php`
Expected: PASS

**Step 7: Commit**

```bash
git add app/Domains/ProjectManagement/
git add database/migrations/*_create_projects_table.php
git add tests/Integration/Domains/ProjectManagement/
git commit -m "feat(project): create project when quote accepted via event listener"
```

---

### Day 25 (Thursday): Register Listeners & Integration Test

**Goal:** Wire up all event listeners, test full flow

#### Task 8: Register Event Listeners

**Files:**
- Modify: `app/Providers/EventServiceProvider.php`

**Step 1: Update EventServiceProvider**

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

// Quote Events
use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\LineItemAdded;
use App\Domains\QuoteManagement\Domain\Events\LineItemRemoved;
use App\Domains\QuoteManagement\Domain\Events\QuoteSent;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Domain\Events\QuoteRejected;

// Project Events
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;

// Listeners
use App\Domains\ProjectManagement\Infrastructure\EventListeners\CreateProjectWhenQuoteAccepted;

class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        // Quote Management Context Events
        QuoteDraftCreated::class => [
            // Future: UpdateDashboardStats, AddToSearchIndex
        ],

        LineItemAdded::class => [
            // Future: RecalculateQuoteTotal projector
        ],

        QuoteSent::class => [
            // Future: SendQuoteEmail, UpdateStats
        ],

        QuoteAccepted::class => [
            // Cross-context: Project Management reacts
            CreateProjectWhenQuoteAccepted::class,
            // Future: SendAcceptanceEmail, GenerateInvoice
        ],

        QuoteRejected::class => [
            // Future: ScheduleFollowUp, NotifyTeam
        ],

        // Project Management Context Events
        ProjectCreated::class => [
            // Future: SendProjectStartEmail, CreateInitialMilestone
        ],
    ];

    public function boot(): void
    {
        //
    }

    public function shouldDiscoverEvents(): bool
    {
        return false;
    }
}
```

**Step 2: Commit**

```bash
git add app/Providers/EventServiceProvider.php
git commit -m "feat: register cross-context event listeners"
```

---

#### Task 9: End-to-End Integration Test

**Files:**
- Create: `tests/Integration/CrossContext/QuoteToProjectFlowTest.php`

**Step 1: Write the integration test**

```php
<?php

namespace Tests\Integration\CrossContext;

use App\Domains\QuoteManagement\Domain\Quote;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Currency;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use App\Domains\QuoteManagement\Domain\Entities\LineItem;
use App\Domains\QuoteManagement\Domain\QuoteRepository;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;
use Carbon\CarbonImmutable;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class QuoteToProjectFlowTest extends TestCase
{
    use RefreshDatabase;

    private QuoteRepository $quoteRepository;

    protected function setUp(): void
    {
        parent::setUp();
        $this->quoteRepository = app(QuoteRepository::class);
    }

    /** @test */
    public function accepting_quote_creates_project_via_event(): void
    {
        // Arrange: Create a quote with line items
        $quote = Quote::createDraft(
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            number: $this->quoteRepository->nextNumber(),
            title: 'Kitchen Renovation',
            currency: Currency::USD(),
            validUntil: CarbonImmutable::now()->addDays(30)
        );

        $quote->addLineItem(LineItem::create(
            description: 'Labor',
            quantity: 40,
            unitPrice: Money::fromFloat(75.00, Currency::USD())
        ));

        $quote->addLineItem(LineItem::create(
            description: 'Materials',
            quantity: 1,
            unitPrice: Money::fromFloat(2000.00, Currency::USD())
        ));

        // Release creation events before saving
        $quote->releaseEvents();

        // Save the quote (now in draft)
        $this->quoteRepository->save($quote);

        // Reload to ensure clean state
        $quote = $this->quoteRepository->findById($quote->id());

        // Send the quote
        $quote->send();
        $this->quoteRepository->save($quote);

        // Reload again
        $quote = $this->quoteRepository->findById($quote->id());

        // Act: Accept the quote (this should trigger project creation)
        Queue::fake(); // Capture queued listeners
        Event::fake([ProjectCreated::class]);

        $quote->accept();
        $this->quoteRepository->save($quote);

        // Assert: Event was stored
        $this->assertDatabaseHas('event_store', [
            'aggregate_id' => $quote->id()->toString(),
            'event_type' => \App\Domains\QuoteManagement\Domain\Events\QuoteAccepted::class,
        ]);
    }

    /** @test */
    public function full_flow_creates_project_in_database(): void
    {
        // Disable queue to run synchronously
        $this->withoutQueuedJobs();

        // Arrange
        $clientId = ClientId::generate();
        $orgId = OrganizationId::generate();

        $quote = Quote::createDraft(
            clientId: $clientId,
            organizationId: $orgId,
            number: $this->quoteRepository->nextNumber(),
            title: 'Bathroom Remodel',
            currency: Currency::USD(),
            validUntil: CarbonImmutable::now()->addDays(30)
        );

        $quote->addLineItem(LineItem::create(
            description: 'Complete remodel',
            quantity: 1,
            unitPrice: Money::fromFloat(8500.00, Currency::USD())
        ));

        $quote->releaseEvents();
        $this->quoteRepository->save($quote);

        $quote = $this->quoteRepository->findById($quote->id());
        $quote->send();
        $this->quoteRepository->save($quote);

        $quote = $this->quoteRepository->findById($quote->id());

        // Act
        $quote->accept();
        $this->quoteRepository->save($quote);

        // Process queued jobs synchronously
        $this->processQueuedJobs();

        // Assert
        $this->assertDatabaseHas('projects', [
            'origin_quote_id' => $quote->id()->toString(),
            'client_id' => $clientId->toString(),
            'organization_id' => $orgId->toString(),
            'status' => 'not_started',
        ]);
    }

    private function withoutQueuedJobs(): void
    {
        config(['queue.default' => 'sync']);
    }

    private function processQueuedJobs(): void
    {
        // If using fake, process manually
        // Otherwise sync queue handles it
    }
}
```

**Step 2: Run test**

Run: `./vendor/bin/pest tests/Integration/CrossContext/QuoteToProjectFlowTest.php`
Expected: PASS

**Step 3: Commit**

```bash
git add tests/Integration/CrossContext/
git commit -m "test: add end-to-end quote to project flow integration test"
```

---

### Day 26 (Friday): Event Replay & Debugging Tools

**Goal:** Create tools for debugging event flow

#### Task 10: Event Replay Command

**Files:**
- Create: `app/Console/Commands/ReplayEventsCommand.php`

**Step 1: Create command**

```php
<?php

namespace App\Console\Commands;

use App\SharedKernel\Infrastructure\EventStore\EventStoreRepository;
use Illuminate\Console\Command;
use Illuminate\Support\Facades\Event;

class ReplayEventsCommand extends Command
{
    protected $signature = 'events:replay 
        {--aggregate= : Replay events for specific aggregate ID}
        {--type= : Replay events of specific type}
        {--since= : Replay events since date (Y-m-d H:i:s)}
        {--dry-run : Show events without dispatching}';

    protected $description = 'Replay domain events from event store';

    public function __construct(
        private EventStoreRepository $eventStore
    ) {
        parent::__construct();
    }

    public function handle(): int
    {
        $aggregateId = $this->option('aggregate');
        $eventType = $this->option('type');
        $since = $this->option('since');
        $dryRun = $this->option('dry-run');

        $events = $this->fetchEvents($aggregateId, $eventType, $since);

        $this->info(sprintf('Found %d events to replay', count($events)));

        if ($dryRun) {
            $this->displayEvents($events);
            return Command::SUCCESS;
        }

        if (!$this->confirm('Replay these events?')) {
            return Command::SUCCESS;
        }

        $bar = $this->output->createProgressBar(count($events));

        foreach ($events as $storedEvent) {
            $event = $this->deserializeEvent($storedEvent);
            Event::dispatch($event);
            $bar->advance();
        }

        $bar->finish();
        $this->newLine();
        $this->info('Events replayed successfully');

        return Command::SUCCESS;
    }

    private function fetchEvents(?string $aggregateId, ?string $eventType, ?string $since): array
    {
        if ($aggregateId) {
            return $this->eventStore->getStream($aggregateId);
        }

        if ($eventType) {
            $sinceDate = $since ? new \DateTimeImmutable($since) : null;
            return $this->eventStore->getEventsByType($eventType, $sinceDate);
        }

        $this->error('Specify --aggregate or --type');
        return [];
    }

    private function displayEvents(array $events): void
    {
        $this->table(
            ['ID', 'Type', 'Aggregate', 'Occurred At'],
            collect($events)->map(fn($e) => [
                substr($e->id, 0, 8) . '...',
                class_basename($e->eventType),
                substr($e->aggregateId, 0, 8) . '...',
                $e->occurredAt->format('Y-m-d H:i:s'),
            ])
        );
    }

    private function deserializeEvent($storedEvent): object
    {
        $class = $storedEvent->eventType;
        
        // Reconstruct event from payload
        // This requires events to have a static fromArray method
        if (method_exists($class, 'fromArray')) {
            return $class::fromArray($storedEvent->payload);
        }

        throw new \RuntimeException("Event {$class} does not support deserialization");
    }
}
```

**Step 2: Add fromArray to domain events**

Update `QuoteAccepted.php`:

```php
public static function fromArray(array $data): self
{
    return new self(
        quoteId: $data['quote_id'],
        clientId: $data['client_id'],
        organizationId: $data['organization_id'],
        totalAmount: (float) $data['total_amount'],
        currency: $data['currency'],
        acceptedAt: new \DateTimeImmutable($data['accepted_at'])
    );
}
```

**Step 3: Commit**

```bash
git add app/Console/Commands/ReplayEventsCommand.php
git commit -m "feat: add event replay command for debugging"
```

---

## Week 4: Advanced Event Patterns & Polish

---

### Day 27-28: Notification Listener & Additional Listeners

#### Task 11: Send Email on Quote Accepted

**Files:**
- Create: `app/Domains/Notifications/Infrastructure/EventListeners/SendQuoteAcceptedEmail.php`
- Create: `app/Mail/QuoteAcceptedMail.php`

(Similar pattern to CreateProjectWhenQuoteAccepted)

---

### Day 29-30: Event Failure Handling & Monitoring

#### Task 12: Failed Event Handler

**Files:**
- Create: `app/SharedKernel/Infrastructure/EventBus/FailedEventHandler.php`
- Create: `database/migrations/xxxx_create_failed_events_table.php`

---

### Day 31: Documentation & Phase Completion

#### Task 13: Update Documentation

**Files:**
- Update: `README.md`
- Create: `docs/architecture/event-driven.md`

---

## Phase 2 Success Criteria

```
[ ] Event Store table created and populated
[ ] Domain events persisted on aggregate save
[ ] QuoteAccepted triggers ProjectCreated
[ ] Project created in database with correct data
[ ] Async processing via queues
[ ] Event replay command working
[ ] All integration tests passing
[ ] Event flow documented
```

## Phase 2 Completion Checklist

```
[ ] 10+ new tests passing
[ ] Event Store with 3+ event types stored
[ ] Cross-context communication working
[ ] Queue-based async listeners
[ ] Error handling for failed listeners
[ ] Event replay capability
[ ] Documentation updated
```
```

---

Now let me continue with Phase 3: