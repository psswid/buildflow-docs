## Phase 4: Project & Invoice Contexts (Weeks 7-8)

```markdown
# BuildFlow Phase 4: Project & Invoice Contexts

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement Project Management and Invoice & Payment contexts with full aggregates, following patterns established in Quote Management.

**Architecture:** Apply same DDD patterns from Quote context - aggregates with value objects, domain events, event-driven cross-context communication, and CQRS read models.

**Tech Stack:** Laravel, PostgreSQL, Domain Events, CQRS, Pest testing

**Dependencies from Previous Phases:**
- ✅ Phase 1: Quote aggregate, DDD patterns established
- ✅ Phase 2: Event bus, cross-context communication
- ✅ Phase 3: CQRS projectors, query services
- ✅ Project skeleton from Phase 2 (CreateProjectWhenQuoteAccepted)

---

## Week 7: Project Management Context

---

### Day 42 (Monday): Project Aggregate - Domain Layer

**Goal:** Complete the Project aggregate with full business logic

#### Task 1: Project Value Objects

**Files:**
- Create: `app/Domains/ProjectManagement/Domain/ValueObjects/Budget.php`
- Create: `app/Domains/ProjectManagement/Domain/ValueObjects/CompletionPercentage.php`
- Create: `app/Domains/ProjectManagement/Domain/ValueObjects/Priority.php`
- Test: `tests/Unit/Domains/ProjectManagement/ValueObjects/`

**Step 1: Write tests for Budget**

```php
<?php

namespace Tests\Unit\Domains\ProjectManagement\ValueObjects;

use App\Domains\ProjectManagement\Domain\ValueObjects\Budget;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use App\Domains\QuoteManagement\Domain\ValueObjects\Currency;
use Tests\TestCase;

class BudgetTest extends TestCase
{
    /** @test */
    public function can_create_budget(): void
    {
        $estimated = Money::fromFloat(10000.00, Currency::USD());
        $budget = Budget::create($estimated);

        $this->assertEquals(10000.00, $budget->estimated()->toFloat());
        $this->assertEquals(0, $budget->actual()->toFloat());
    }

    /** @test */
    public function can_add_to_actual(): void
    {
        $budget = Budget::create(Money::fromFloat(10000.00, Currency::USD()));
        
        $updated = $budget->addActual(Money::fromFloat(2500.00, Currency::USD()));

        $this->assertEquals(2500.00, $updated->actual()->toFloat());
    }

    /** @test */
    public function calculates_remaining(): void
    {
        $budget = Budget::create(Money::fromFloat(10000.00, Currency::USD()))
            ->addActual(Money::fromFloat(3000.00, Currency::USD()));

        $this->assertEquals(7000.00, $budget->remaining()->toFloat());
    }

    /** @test */
    public function calculates_percentage_used(): void
    {
        $budget = Budget::create(Money::fromFloat(10000.00, Currency::USD()))
            ->addActual(Money::fromFloat(2500.00, Currency::USD()));

        $this->assertEquals(25.0, $budget->percentageUsed());
    }

    /** @test */
    public function detects_over_budget(): void
    {
        $budget = Budget::create(Money::fromFloat(10000.00, Currency::USD()))
            ->addActual(Money::fromFloat(12000.00, Currency::USD()));

        $this->assertTrue($budget->isOverBudget());
        $this->assertEquals(-2000.00, $budget->remaining()->toFloat());
    }
}
```

**Step 2: Implement Budget**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;

final class Budget extends ValueObject
{
    private function __construct(
        private Money $estimated,
        private Money $actual
    ) {}

    public static function create(Money $estimated): self
    {
        return new self(
            $estimated,
            Money::fromCents(0, $estimated->currency())
        );
    }

    public function addActual(Money $amount): self
    {
        return new self(
            $this->estimated,
            $this->actual->add($amount)
        );
    }

    public function estimated(): Money
    {
        return $this->estimated;
    }

    public function actual(): Money
    {
        return $this->actual;
    }

    public function remaining(): Money
    {
        return $this->estimated->subtract($this->actual);
    }

    public function percentageUsed(): float
    {
        if ($this->estimated->toCents() === 0) {
            return 0;
        }
        return round(($this->actual->toCents() / $this->estimated->toCents()) * 100, 1);
    }

    public function isOverBudget(): bool
    {
        return $this->actual->toCents() > $this->estimated->toCents();
    }

    public function equals(ValueObject $other): bool
    {
        return $other instanceof self
            && $this->estimated->equals($other->estimated)
            && $this->actual->equals($other->actual);
    }
}
```

**Step 3: Write CompletionPercentage**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class CompletionPercentage extends ValueObject
{
    private function __construct(private int $value) {}

    public static function zero(): self
    {
        return new self(0);
    }

    public static function fromInt(int $value): self
    {
        if ($value < 0 || $value > 100) {
            throw new \InvalidArgumentException(
                "Completion percentage must be between 0 and 100, got: {$value}"
            );
        }
        return new self($value);
    }

    public function value(): int
    {
        return $this->value;
    }

    public function isComplete(): bool
    {
        return $this->value === 100;
    }

    public function equals(ValueObject $other): bool
    {
        return $other instanceof self && $this->value === $other->value;
    }
}
```

**Step 4: Write Priority**

```php
<?php

namespace App\Domains\ProjectManagement\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class Priority extends ValueObject
{
    private const LOW = 'low';
    private const MEDIUM = 'medium';
    private const HIGH = 'high';
    private const URGENT = 'urgent';

    private function __construct(private string $value) {}

    public static function low(): self { return new self(self::LOW); }
    public static function medium(): self { return new self(self::MEDIUM); }
    public static function high(): self { return new self(self::HIGH); }
    public static function urgent(): self { return new self(self::URGENT); }

    public static function fromString(string $value): self
    {
        return match ($value) {
            self::LOW => self::low(),
            self::MEDIUM => self::medium(),
            self::HIGH => self::high(),
            self::URGENT => self::urgent(),
            default => throw new \InvalidArgumentException("Invalid priority: {$value}"),
        };
    }

    public function value(): string { return $this->value; }

    public function equals(ValueObject $other): bool
    {
        return $other instanceof self && $this->value === $other->value;
    }
}
```

**Step 5: Commit**

```bash
git add app/Domains/ProjectManagement/Domain/ValueObjects/
git add tests/Unit/Domains/ProjectManagement/ValueObjects/
git commit -m "feat(project): implement Project value objects"
```

---

#### Task 2: Project Domain Events

**Files:**
- Create: `app/Domains/ProjectManagement/Domain/Events/ProjectStarted.php`
- Create: `app/Domains/ProjectManagement/Domain/Events/ProjectCompleted.php`
- Create: `app/Domains/ProjectManagement/Domain/Events/ProjectPaused.php`
- Create: `app/Domains/ProjectManagement/Domain/Events/ProgressUpdated.php`
- Create: `app/Domains/ProjectManagement/Domain/Events/WorkLogged.php`

**Step 1: Create events**

```php
<?php
// app/Domains/ProjectManagement/Domain/Events/ProjectStarted.php
namespace App\Domains\ProjectManagement\Domain\Events;

use App\SharedKernel\Domain\DomainEvent;

final class ProjectStarted implements DomainEvent
{
    public function __construct(
        public readonly string $projectId,
        public readonly string $organizationId,
        public readonly \DateTimeImmutable $startedAt
    ) {}

    public function eventName(): string { return 'project.started'; }
    public function occurredOn(): \DateTimeImmutable { return $this->startedAt; }
    public function aggregateId(): string { return $this->projectId; }

    public function toArray(): array
    {
        return [
            'project_id' => $this->projectId,
            'organization_id' => $this->organizationId,
            'started_at' => $this->startedAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

```php
<?php
// app/Domains/ProjectManagement/Domain/Events/ProjectCompleted.php
namespace App\Domains\ProjectManagement\Domain\Events;

use App\SharedKernel\Domain\DomainEvent;

final class ProjectCompleted implements DomainEvent
{
    public function __construct(
        public readonly string $projectId,
        public readonly string $clientId,
        public readonly string $organizationId,
        public readonly float $finalCost,
        public readonly string $currency,
        public readonly \DateTimeImmutable $completedAt
    ) {}

    public function eventName(): string { return 'project.completed'; }
    public function occurredOn(): \DateTimeImmutable { return $this->completedAt; }
    public function aggregateId(): string { return $this->projectId; }

    public function toArray(): array
    {
        return [
            'project_id' => $this->projectId,
            'client_id' => $this->clientId,
            'organization_id' => $this->organizationId,
            'final_cost' => $this->finalCost,
            'currency' => $this->currency,
            'completed_at' => $this->completedAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

```php
<?php
// app/Domains/ProjectManagement/Domain/Events/WorkLogged.php
namespace App\Domains\ProjectManagement\Domain\Events;

use App\SharedKernel\Domain\DomainEvent;

final class WorkLogged implements DomainEvent
{
    public function __construct(
        public readonly string $projectId,
        public readonly string $workLogId,
        public readonly string $description,
        public readonly float $hours,
        public readonly float $costAmount,
        public readonly \DateTimeImmutable $loggedAt
    ) {}

    public function eventName(): string { return 'project.work_logged'; }
    public function occurredOn(): \DateTimeImmutable { return $this->loggedAt; }
    public function aggregateId(): string { return $this->projectId; }

    public function toArray(): array
    {
        return [
            'project_id' => $this->projectId,
            'work_log_id' => $this->workLogId,
            'description' => $this->description,
            'hours' => $this->hours,
            'cost_amount' => $this->costAmount,
            'logged_at' => $this->loggedAt->format('Y-m-d H:i:s'),
        ];
    }
}
```

**Step 2: Commit**

```bash
git add app/Domains/ProjectManagement/Domain/Events/
git commit -m "feat(project): implement Project domain events"
```

---

#### Task 3: Complete Project Aggregate

**Files:**
- Modify: `app/Domains/ProjectManagement/Domain/Project.php`
- Test: `tests/Unit/Domains/ProjectManagement/ProjectTest.php`

**Step 1: Write tests**

```php
<?php

namespace Tests\Unit\Domains\ProjectManagement;

use App\Domains\ProjectManagement\Domain\Project;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;
use App\Domains\ProjectManagement\Domain\Events\ProjectStarted;
use App\Domains\ProjectManagement\Domain\Events\ProjectCompleted;
use App\Domains\ProjectManagement\Domain\Events\WorkLogged;
use App\Domains\ProjectManagement\Domain\Exceptions\CannotCompleteProjectWithOpenIssues;
use App\Domains\ProjectManagement\Domain\Exceptions\ProjectAlreadyStarted;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use App\Domains\QuoteManagement\Domain\ValueObjects\Currency;
use Tests\TestCase;

class ProjectTest extends TestCase
{
    /** @test */
    public function can_create_project_from_quote(): void
    {
        $project = Project::createFromQuote(
            quoteId: QuoteId::fromString('quote-123'),
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            title: 'Kitchen Renovation',
            budget: Money::fromFloat(10000.00, Currency::USD())
        );

        $this->assertEquals('not_started', $project->status()->value());
        $this->assertEquals(0, $project->completionPercentage()->value());
    }

    /** @test */
    public function creating_project_records_event(): void
    {
        $project = Project::createFromQuote(
            quoteId: QuoteId::fromString('quote-123'),
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            title: 'Test',
            budget: Money::fromFloat(5000.00, Currency::USD())
        );

        $events = $project->releaseEvents();
        $this->assertCount(1, $events);
        $this->assertInstanceOf(ProjectCreated::class, $events[0]);
    }

    /** @test */
    public function can_start_project(): void
    {
        $project = $this->createProject();
        
        $project->start();

        $this->assertEquals('in_progress', $project->status()->value());
    }

    /** @test */
    public function starting_project_records_event(): void
    {
        $project = $this->createProject();
        $project->releaseEvents(); // Clear creation event
        
        $project->start();

        $events = $project->releaseEvents();
        $this->assertCount(1, $events);
        $this->assertInstanceOf(ProjectStarted::class, $events[0]);
    }

    /** @test */
    public function cannot_start_already_started_project(): void
    {
        $project = $this->createProject();
        $project->start();

        $this->expectException(ProjectAlreadyStarted::class);
        $project->start();
    }

    /** @test */
    public function can_log_work(): void
    {
        $project = $this->createProject();
        $project->start();
        $project->releaseEvents();
        
        $project->logWork(
            description: 'Cabinet installation',
            hours: 8,
            cost: Money::fromFloat(600.00, Currency::USD())
        );

        $this->assertEquals(600.00, $project->budget()->actual()->toFloat());
    }

    /** @test */
    public function logging_work_records_event(): void
    {
        $project = $this->createProject();
        $project->start();
        $project->releaseEvents();
        
        $project->logWork(
            description: 'Demo work',
            hours: 4,
            cost: Money::fromFloat(300.00, Currency::USD())
        );

        $events = $project->releaseEvents();
        $this->assertCount(1, $events);
        $this->assertInstanceOf(WorkLogged::class, $events[0]);
    }

    /** @test */
    public function can_update_progress(): void
    {
        $project = $this->createProject();
        $project->start();
        
        $project->updateProgress(50);

        $this->assertEquals(50, $project->completionPercentage()->value());
    }

    /** @test */
    public function can_complete_project(): void
    {
        $project = $this->createProject();
        $project->start();
        $project->updateProgress(100);
        $project->releaseEvents();
        
        $project->complete();

        $this->assertEquals('completed', $project->status()->value());
    }

    /** @test */
    public function completing_project_records_event(): void
    {
        $project = $this->createProject();
        $project->start();
        $project->updateProgress(100);
        $project->releaseEvents();
        
        $project->complete();

        $events = $project->releaseEvents();
        $this->assertInstanceOf(ProjectCompleted::class, $events[0]);
    }

    private function createProject(): Project
    {
        return Project::createFromQuote(
            quoteId: QuoteId::fromString('quote-123'),
            clientId: ClientId::generate(),
            organizationId: OrganizationId::generate(),
            title: 'Test Project',
            budget: Money::fromFloat(10000.00, Currency::USD())
        );
    }
}
```

**Step 2: Complete Project aggregate**

```php
<?php

namespace App\Domains\ProjectManagement\Domain;

use App\SharedKernel\Domain\AggregateRoot;
use App\Domains\ProjectManagement\Domain\ValueObjects\ProjectId;
use App\Domains\ProjectManagement\Domain\ValueObjects\ProjectStatus;
use App\Domains\ProjectManagement\Domain\ValueObjects\Budget;
use App\Domains\ProjectManagement\Domain\ValueObjects\CompletionPercentage;
use App\Domains\ProjectManagement\Domain\ValueObjects\Priority;
use App\Domains\ProjectManagement\Domain\Events\ProjectCreated;
use App\Domains\ProjectManagement\Domain\Events\ProjectStarted;
use App\Domains\ProjectManagement\Domain\Events\ProjectCompleted;
use App\Domains\ProjectManagement\Domain\Events\ProjectPaused;
use App\Domains\ProjectManagement\Domain\Events\ProgressUpdated;
use App\Domains\ProjectManagement\Domain\Events\WorkLogged;
use App\Domains\ProjectManagement\Domain\Exceptions\ProjectAlreadyStarted;
use App\Domains\ProjectManagement\Domain\Exceptions\ProjectNotStarted;
use App\Domains\ProjectManagement\Domain\Exceptions\CannotCompleteIncompleteProject;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;
use Illuminate\Support\Str;

class Project extends AggregateRoot
{
    private function __construct(
        private ProjectId $id,
        private QuoteId $originQuoteId,
        private ClientId $clientId,
        private OrganizationId $organizationId,
        private string $title,
        private ProjectStatus $status,
        private Budget $budget,
        private CompletionPercentage $completionPercentage,
        private Priority $priority,
        private ?\DateTimeImmutable $startedAt,
        private ?\DateTimeImmutable $completedAt,
        private \DateTimeImmutable $createdAt
    ) {}

    public static function createFromQuote(
        QuoteId $quoteId,
        ClientId $clientId,
        OrganizationId $organizationId,
        string $title,
        Money $budgetAmount
    ): self {
        $project = new self(
            id: ProjectId::generate(),
            originQuoteId: $quoteId,
            clientId: $clientId,
            organizationId: $organizationId,
            title: $title,
            status: ProjectStatus::notStarted(),
            budget: Budget::create($budgetAmount),
            completionPercentage: CompletionPercentage::zero(),
            priority: Priority::medium(),
            startedAt: null,
            completedAt: null,
            createdAt: new \DateTimeImmutable()
        );

        $project->record(new ProjectCreated(
            projectId: $project->id->toString(),
            quoteId: $quoteId->toString(),
            clientId: $clientId->toString(),
            organizationId: $organizationId->toString(),
            title: $title,
            budget: $budgetAmount->toFloat(),
            currency: $budgetAmount->currency()->code(),
            createdAt: new \DateTimeImmutable()
        ));

        return $project;
    }

    public function start(): void
    {
        if (!$this->status->equals(ProjectStatus::notStarted())) {
            throw new ProjectAlreadyStarted();
        }

        $this->status = ProjectStatus::inProgress();
        $this->startedAt = new \DateTimeImmutable();

        $this->record(new ProjectStarted(
            projectId: $this->id->toString(),
            organizationId: $this->organizationId->toString(),
            startedAt: $this->startedAt
        ));
    }

    public function logWork(string $description, float $hours, Money $cost): void
    {
        if (!$this->status->equals(ProjectStatus::inProgress())) {
            throw new ProjectNotStarted();
        }

        $this->budget = $this->budget->addActual($cost);

        $this->record(new WorkLogged(
            projectId: $this->id->toString(),
            workLogId: Str::uuid()->toString(),
            description: $description,
            hours: $hours,
            costAmount: $cost->toFloat(),
            loggedAt: new \DateTimeImmutable()
        ));
    }

    public function updateProgress(int $percentage): void
    {
        $this->completionPercentage = CompletionPercentage::fromInt($percentage);

        $this->record(new ProgressUpdated(
            projectId: $this->id->toString(),
            percentage: $percentage,
            updatedAt: new \DateTimeImmutable()
        ));
    }

    public function pause(): void
    {
        if (!$this->status->equals(ProjectStatus::inProgress())) {
            throw new ProjectNotStarted();
        }

        $this->status = ProjectStatus::onHold();

        $this->record(new ProjectPaused(
            projectId: $this->id->toString(),
            pausedAt: new \DateTimeImmutable()
        ));
    }

    public function resume(): void
    {
        if (!$this->status->equals(ProjectStatus::onHold())) {
            throw new \DomainException('Can only resume paused projects');
        }

        $this->status = ProjectStatus::inProgress();
    }

    public function complete(): void
    {
        if (!$this->completionPercentage->isComplete()) {
            throw new CannotCompleteIncompleteProject();
        }

        $this->status = ProjectStatus::completed();
        $this->completedAt = new \DateTimeImmutable();

        $this->record(new ProjectCompleted(
            projectId: $this->id->toString(),
            clientId: $this->clientId->toString(),
            organizationId: $this->organizationId->toString(),
            finalCost: $this->budget->actual()->toFloat(),
            currency: $this->budget->actual()->currency()->code(),
            completedAt: $this->completedAt
        ));
    }

    // Getters
    public function id(): ProjectId { return $this->id; }
    public function originQuoteId(): QuoteId { return $this->originQuoteId; }
    public function clientId(): ClientId { return $this->clientId; }
    public function organizationId(): OrganizationId { return $this->organizationId; }
    public function title(): string { return $this->title; }
    public function status(): ProjectStatus { return $this->status; }
    public function budget(): Budget { return $this->budget; }
    public function completionPercentage(): CompletionPercentage { return $this->completionPercentage; }
    public function priority(): Priority { return $this->priority; }
    public function startedAt(): ?\DateTimeImmutable { return $this->startedAt; }
    public function completedAt(): ?\DateTimeImmutable { return $this->completedAt; }
}
```

**Step 3: Run tests and commit**

```bash
./vendor/bin/pest tests/Unit/Domains/ProjectManagement/ProjectTest.php
git add app/Domains/ProjectManagement/Domain/
git add tests/Unit/Domains/ProjectManagement/
git commit -m "feat(project): complete Project aggregate with business logic"
```

---

### Day 43-44: Project Infrastructure & API

#### Task 4: Project Repository & Migration
#### Task 5: Project Command Handlers
#### Task 6: Project API Controller

---

### Day 45: Project CQRS (Read Models)

#### Task 7: ProjectListView & ProjectDashboardStats
#### Task 8: Project Projectors

---

## Week 8: Invoice & Payment Context

---

### Day 46 (Monday): Invoice Aggregate

#### Task 9: Invoice Value Objects

**Files:**
- Create: `app/Domains/InvoicePayment/Domain/ValueObjects/InvoiceId.php`
- Create: `app/Domains/InvoicePayment/Domain/ValueObjects/InvoiceNumber.php`
- Create: `app/Domains/InvoicePayment/Domain/ValueObjects/InvoiceStatus.php`
- Create: `app/Domains/InvoicePayment/Domain/ValueObjects/PaymentMethod.php`

```php
<?php
// app/Domains/InvoicePayment/Domain/ValueObjects/InvoiceStatus.php
namespace App\Domains\InvoicePayment\Domain\ValueObjects;

use App\SharedKernel\Domain\ValueObject;

final class InvoiceStatus extends ValueObject
{
    private const DRAFT = 'draft';
    private const SENT = 'sent';
    private const PAID = 'paid';
    private const PARTIALLY_PAID = 'partially_paid';
    private const OVERDUE = 'overdue';
    private const CANCELLED = 'cancelled';

    private function __construct(private string $value) {}

    public static function draft(): self { return new self(self::DRAFT); }
    public static function sent(): self { return new self(self::SENT); }
    public static function paid(): self { return new self(self::PAID); }
    public static function partiallyPaid(): self { return new self(self::PARTIALLY_PAID); }
    public static function overdue(): self { return new self(self::OVERDUE); }
    public static function cancelled(): self { return new self(self::CANCELLED); }

    public function value(): string { return $this->value; }

    public function isPaid(): bool { return $this->value === self::PAID; }
    public function isOverdue(): bool { return $this->value === self::OVERDUE; }
    public function canRecordPayment(): bool
    {
        return in_array($this->value, [self::SENT, self::PARTIALLY_PAID, self::OVERDUE]);
    }

    public function equals(ValueObject $other): bool
    {
        return $other instanceof self && $this->value === $other->value;
    }
}
```

---

#### Task 10: Invoice Aggregate & Events

**Files:**
- Create: `app/Domains/InvoicePayment/Domain/Invoice.php`
- Create: `app/Domains/InvoicePayment/Domain/Entities/Payment.php`
- Create: `app/Domains/InvoicePayment/Domain/Events/InvoiceGenerated.php`
- Create: `app/Domains/InvoicePayment/Domain/Events/InvoiceSent.php`
- Create: `app/Domains/InvoicePayment/Domain/Events/PaymentReceived.php`
- Create: `app/Domains/InvoicePayment/Domain/Events/InvoiceMarkedAsPaid.php`

```php
<?php

namespace App\Domains\InvoicePayment\Domain;

use App\SharedKernel\Domain\AggregateRoot;
use App\Domains\InvoicePayment\Domain\ValueObjects\InvoiceId;
use App\Domains\InvoicePayment\Domain\ValueObjects\InvoiceNumber;
use App\Domains\InvoicePayment\Domain\ValueObjects\InvoiceStatus;
use App\Domains\InvoicePayment\Domain\Entities\Payment;
use App\Domains\InvoicePayment\Domain\Events\InvoiceGenerated;
use App\Domains\InvoicePayment\Domain\Events\InvoiceSent;
use App\Domains\InvoicePayment\Domain\Events\PaymentReceived;
use App\Domains\InvoicePayment\Domain\Events\InvoiceMarkedAsPaid;
use App\Domains\InvoicePayment\Domain\Exceptions\CannotRecordPaymentOnPaidInvoice;
use App\Domains\InvoicePayment\Domain\Exceptions\PaymentExceedsBalance;
use App\Domains\QuoteManagement\Domain\ValueObjects\QuoteId;
use App\Domains\QuoteManagement\Domain\ValueObjects\ClientId;
use App\Domains\QuoteManagement\Domain\ValueObjects\OrganizationId;
use App\Domains\QuoteManagement\Domain\ValueObjects\Money;

class Invoice extends AggregateRoot
{
    private array $payments = [];

    private function __construct(
        private InvoiceId $id,
        private ?QuoteId $originQuoteId,
        private ClientId $clientId,
        private OrganizationId $organizationId,
        private InvoiceNumber $number,
        private InvoiceStatus $status,
        private Money $total,
        private Money $amountPaid,
        private \DateTimeImmutable $dueDate,
        private ?\DateTimeImmutable $sentAt,
        private ?\DateTimeImmutable $paidAt,
        private \DateTimeImmutable $createdAt
    ) {}

    public static function generateFromQuote(
        QuoteId $quoteId,
        ClientId $clientId,
        OrganizationId $organizationId,
        InvoiceNumber $number,
        Money $total,
        \DateTimeImmutable $dueDate
    ): self {
        $invoice = new self(
            id: InvoiceId::generate(),
            originQuoteId: $quoteId,
            clientId: $clientId,
            organizationId: $organizationId,
            number: $number,
            status: InvoiceStatus::draft(),
            total: $total,
            amountPaid: Money::fromCents(0, $total->currency()),
            dueDate: $dueDate,
            sentAt: null,
            paidAt: null,
            createdAt: new \DateTimeImmutable()
        );

        $invoice->record(new InvoiceGenerated(
            invoiceId: $invoice->id->toString(),
            quoteId: $quoteId->toString(),
            clientId: $clientId->toString(),
            organizationId: $organizationId->toString(),
            invoiceNumber: $number->toString(),
            total: $total->toFloat(),
            currency: $total->currency()->code(),
            dueDate: $dueDate,
            generatedAt: new \DateTimeImmutable()
        ));

        return $invoice;
    }

    public function send(): void
    {
        if (!$this->status->equals(InvoiceStatus::draft())) {
            throw new \DomainException('Can only send draft invoices');
        }

        $this->status = InvoiceStatus::sent();
        $this->sentAt = new \DateTimeImmutable();

        $this->record(new InvoiceSent(
            invoiceId: $this->id->toString(),
            clientId: $this->clientId->toString(),
            sentAt: $this->sentAt
        ));
    }

    public function recordPayment(Money $amount, string $method, ?string $reference = null): void
    {
        if (!$this->status->canRecordPayment()) {
            throw new CannotRecordPaymentOnPaidInvoice();
        }

        $balance = $this->balance();
        if ($amount->toCents() > $balance->toCents()) {
            throw new PaymentExceedsBalance($amount->toFloat(), $balance->toFloat());
        }

        $payment = Payment::create($amount, $method, $reference);
        $this->payments[] = $payment;
        $this->amountPaid = $this->amountPaid->add($amount);

        $this->record(new PaymentReceived(
            invoiceId: $this->id->toString(),
            paymentId: $payment->id(),
            amount: $amount->toFloat(),
            method: $method,
            reference: $reference,
            receivedAt: new \DateTimeImmutable()
        ));

        // Check if fully paid
        if ($this->amountPaid->toCents() >= $this->total->toCents()) {
            $this->markAsPaid();
        } else {
            $this->status = InvoiceStatus::partiallyPaid();
        }
    }

    private function markAsPaid(): void
    {
        $this->status = InvoiceStatus::paid();
        $this->paidAt = new \DateTimeImmutable();

        $this->record(new InvoiceMarkedAsPaid(
            invoiceId: $this->id->toString(),
            clientId: $this->clientId->toString(),
            organizationId: $this->organizationId->toString(),
            total: $this->total->toFloat(),
            paidAt: $this->paidAt
        ));
    }

    public function markAsOverdue(): void
    {
        if ($this->status->equals(InvoiceStatus::sent()) || 
            $this->status->equals(InvoiceStatus::partiallyPaid())) {
            $this->status = InvoiceStatus::overdue();
        }
    }

    public function balance(): Money
    {
        return $this->total->subtract($this->amountPaid);
    }

    public function isOverdue(): bool
    {
        return $this->dueDate < new \DateTimeImmutable() 
            && !$this->status->isPaid();
    }

    // Getters
    public function id(): InvoiceId { return $this->id; }
    public function clientId(): ClientId { return $this->clientId; }
    public function organizationId(): OrganizationId { return $this->organizationId; }
    public function number(): InvoiceNumber { return $this->number; }
    public function status(): InvoiceStatus { return $this->status; }
    public function total(): Money { return $this->total; }
    public function amountPaid(): Money { return $this->amountPaid; }
    public function dueDate(): \DateTimeImmutable { return $this->dueDate; }
}
```

---

### Day 47-48: Invoice Infrastructure & Cross-Context Integration

#### Task 11: Generate Invoice When Quote Accepted
#### Task 12: Invoice API Endpoints
#### Task 13: Payment Recording Flow

---

### Day 49: Invoice CQRS & Overdue Detection

#### Task 14: Invoice Read Models
#### Task 15: Overdue Detection Command (Scheduled)

---

## Phase 4 Success Criteria

```
[ ] Project aggregate complete with start/pause/complete workflow
[ ] Work logging with budget tracking
[ ] Project API endpoints working
[ ] Invoice aggregate with payment tracking
[ ] Partial payment support
[ ] Invoice generated from accepted quote (via event)
[ ] Overdue detection scheduled task
[ ] CQRS read models for both contexts
[ ] 30+ new tests passing
```
```

---