## Phase 3: CQRS Implementation (Weeks 5-6)

```markdown
# BuildFlow Phase 3: CQRS Implementation

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement read models and projectors for Quote Management, enabling fast queries and dashboard statistics separate from write operations.

**Architecture:** Separate read models (denormalized tables) updated by projectors listening to domain events, with dedicated query services for read operations.

**Tech Stack:** Laravel, PostgreSQL, Domain Events, Pest testing

**Dependencies from Phase 2:**
- ✅ Event Store with domain events persisted
- ✅ Event dispatcher integrated with repositories
- ✅ Cross-context event listeners working
- ✅ All Quote domain events implemented

---

## Week 5: Read Model Infrastructure

---

### Day 32 (Monday): Read Model Migrations

**Goal:** Create denormalized tables optimized for read operations

#### Task 1: Quote List View Migration

**Files:**
- Create: `database/migrations/xxxx_create_quote_list_view_table.php`

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
        Schema::create('quote_list_view', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('organization_id')->index();
            
            // Denormalized data
            $table->string('quote_number', 20);
            $table->uuid('client_id');
            $table->string('client_name');           // From Client context
            $table->string('client_email')->nullable();
            $table->string('title');
            $table->string('status', 20)->default('draft');
            $table->bigInteger('subtotal')->default(0);    // cents
            $table->bigInteger('total')->default(0);       // cents
            $table->string('currency', 3)->default('USD');
            $table->date('valid_until');
            $table->integer('line_items_count')->default(0);
            $table->timestamp('sent_at')->nullable();
            $table->timestamp('accepted_at')->nullable();
            $table->timestamp('created_at');
            $table->timestamp('updated_at');

            // Optimized indexes for common queries
            $table->index(['organization_id', 'status']);
            $table->index(['organization_id', 'created_at']);
            $table->index(['organization_id', 'client_id']);
            $table->index(['client_id', 'status']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('quote_list_view');
    }
};
```

**Step 2: Run migration**

Run: `php artisan migrate`
Expected: Migration successful

**Step 3: Commit**

```bash
git add database/migrations/*_create_quote_list_view_table.php
git commit -m "feat(cqrs): add quote_list_view read model migration"
```

---

#### Task 2: Dashboard Stats Migration

**Files:**
- Create: `database/migrations/xxxx_create_quote_dashboard_stats_table.php`

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
        Schema::create('quote_dashboard_stats', function (Blueprint $table) {
            $table->uuid('organization_id')->primary();
            
            // Quote counts by status
            $table->integer('total_quotes')->default(0);
            $table->integer('draft_quotes')->default(0);
            $table->integer('sent_quotes')->default(0);
            $table->integer('accepted_quotes')->default(0);
            $table->integer('rejected_quotes')->default(0);
            $table->integer('expired_quotes')->default(0);
            
            // Financial stats (in cents)
            $table->bigInteger('total_quoted_value')->default(0);
            $table->bigInteger('accepted_value')->default(0);
            $table->bigInteger('pending_value')->default(0);
            $table->bigInteger('rejected_value')->default(0);
            
            // Calculated metrics
            $table->decimal('acceptance_rate', 5, 2)->default(0);
            $table->decimal('average_quote_value', 12, 2)->default(0);
            
            // Time-based stats
            $table->integer('quotes_this_month')->default(0);
            $table->integer('quotes_this_week')->default(0);
            
            $table->timestamp('last_updated')->useCurrent();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('quote_dashboard_stats');
    }
};
```

**Step 2: Run migration**

Run: `php artisan migrate`

**Step 3: Commit**

```bash
git add database/migrations/*_create_quote_dashboard_stats_table.php
git commit -m "feat(cqrs): add quote_dashboard_stats read model migration"
```

---

#### Task 3: Search Index Migration

**Files:**
- Create: `database/migrations/xxxx_create_search_index_table.php`

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
        Schema::create('search_index', function (Blueprint $table) {
            $table->uuid('id')->primary();
            $table->uuid('organization_id')->index();
            $table->string('entity_type', 50);      // quote, project, client, invoice
            $table->uuid('entity_id');
            $table->string('title');
            $table->text('searchable_text');
            $table->json('metadata')->nullable();
            $table->timestamp('created_at');
            $table->timestamp('updated_at');

            $table->index(['organization_id', 'entity_type']);
            $table->index('entity_id');
            $table->fullText('searchable_text');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('search_index');
    }
};
```

**Step 2: Run migration**

Run: `php artisan migrate`

**Step 3: Commit**

```bash
git add database/migrations/*_create_search_index_table.php
git commit -m "feat(cqrs): add search_index read model migration"
```

---

### Day 33 (Tuesday): Read Model Classes

**Goal:** Create Eloquent models for read-only access

#### Task 4: QuoteListView Read Model

**Files:**
- Create: `app/Domains/QuoteManagement/Infrastructure/ReadModels/QuoteListView.php`
- Test: `tests/Unit/Domains/QuoteManagement/ReadModels/QuoteListViewTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Unit\Domains\QuoteManagement\ReadModels;

use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class QuoteListViewTest extends TestCase
{
    use RefreshDatabase;

    /** @test */
    public function can_scope_by_organization(): void
    {
        $this->createQuoteListView(['organization_id' => 'org-1', 'title' => 'Quote 1']);
        $this->createQuoteListView(['organization_id' => 'org-1', 'title' => 'Quote 2']);
        $this->createQuoteListView(['organization_id' => 'org-2', 'title' => 'Quote 3']);

        $quotes = QuoteListView::forOrganization('org-1')->get();

        $this->assertCount(2, $quotes);
    }

    /** @test */
    public function can_filter_by_status(): void
    {
        $this->createQuoteListView(['organization_id' => 'org-1', 'status' => 'draft']);
        $this->createQuoteListView(['organization_id' => 'org-1', 'status' => 'sent']);
        $this->createQuoteListView(['organization_id' => 'org-1', 'status' => 'accepted']);

        $quotes = QuoteListView::forOrganization('org-1')->pending()->get();

        $this->assertCount(1, $quotes);
        $this->assertEquals('sent', $quotes->first()->status);
    }

    /** @test */
    public function can_filter_by_client(): void
    {
        $this->createQuoteListView(['organization_id' => 'org-1', 'client_id' => 'client-1']);
        $this->createQuoteListView(['organization_id' => 'org-1', 'client_id' => 'client-2']);

        $quotes = QuoteListView::forOrganization('org-1')->forClient('client-1')->get();

        $this->assertCount(1, $quotes);
    }

    /** @test */
    public function returns_total_as_float(): void
    {
        $this->createQuoteListView(['total' => 500000]); // $5000.00 in cents

        $quote = QuoteListView::first();

        $this->assertEquals(5000.00, $quote->total_dollars);
    }

    private function createQuoteListView(array $overrides = []): void
    {
        \DB::table('quote_list_view')->insert(array_merge([
            'id' => \Str::uuid()->toString(),
            'organization_id' => 'org-1',
            'quote_number' => 'Q-000001',
            'client_id' => 'client-1',
            'client_name' => 'Test Client',
            'title' => 'Test Quote',
            'status' => 'draft',
            'subtotal' => 100000,
            'total' => 100000,
            'currency' => 'USD',
            'valid_until' => now()->addDays(30),
            'line_items_count' => 1,
            'created_at' => now(),
            'updated_at' => now(),
        ], $overrides));
    }
}
```

**Step 2: Run test to verify it fails**

Run: `./vendor/bin/pest tests/Unit/Domains/QuoteManagement/ReadModels/QuoteListViewTest.php`
Expected: FAIL - class not found

**Step 3: Write QuoteListView model**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\ReadModels;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

/**
 * Read-only model for quote list views.
 * 
 * This model is populated by projectors listening to domain events.
 * DO NOT use save(), update(), or delete() - use projectors instead.
 */
class QuoteListView extends Model
{
    protected $table = 'quote_list_view';
    protected $keyType = 'string';
    public $incrementing = false;

    protected $casts = [
        'subtotal' => 'integer',
        'total' => 'integer',
        'line_items_count' => 'integer',
        'valid_until' => 'date',
        'sent_at' => 'datetime',
        'accepted_at' => 'datetime',
        'created_at' => 'datetime',
        'updated_at' => 'datetime',
    ];

    // ==================
    // Scopes
    // ==================

    public function scopeForOrganization(Builder $query, string $organizationId): Builder
    {
        return $query->where('organization_id', $organizationId);
    }

    public function scopeForClient(Builder $query, string $clientId): Builder
    {
        return $query->where('client_id', $clientId);
    }

    public function scopeDraft(Builder $query): Builder
    {
        return $query->where('status', 'draft');
    }

    public function scopePending(Builder $query): Builder
    {
        return $query->where('status', 'sent');
    }

    public function scopeAccepted(Builder $query): Builder
    {
        return $query->where('status', 'accepted');
    }

    public function scopeRejected(Builder $query): Builder
    {
        return $query->where('status', 'rejected');
    }

    public function scopeExpired(Builder $query): Builder
    {
        return $query->where('status', 'expired');
    }

    public function scopeActive(Builder $query): Builder
    {
        return $query->whereIn('status', ['draft', 'sent']);
    }

    public function scopeCreatedThisMonth(Builder $query): Builder
    {
        return $query->where('created_at', '>=', now()->startOfMonth());
    }

    public function scopeCreatedThisWeek(Builder $query): Builder
    {
        return $query->where('created_at', '>=', now()->startOfWeek());
    }

    // ==================
    // Accessors
    // ==================

    public function getTotalDollarsAttribute(): float
    {
        return $this->total / 100;
    }

    public function getSubtotalDollarsAttribute(): float
    {
        return $this->subtotal / 100;
    }

    public function getFormattedTotalAttribute(): string
    {
        return number_format($this->total_dollars, 2);
    }

    public function getIsExpiredAttribute(): bool
    {
        return $this->valid_until < now() && $this->status === 'sent';
    }

    public function getDaysUntilExpiryAttribute(): ?int
    {
        if ($this->status !== 'sent') {
            return null;
        }
        return now()->diffInDays($this->valid_until, false);
    }
}
```

**Step 4: Run tests**

Run: `./vendor/bin/pest tests/Unit/Domains/QuoteManagement/ReadModels/QuoteListViewTest.php`
Expected: PASS

**Step 5: Commit**

```bash
git add app/Domains/QuoteManagement/Infrastructure/ReadModels/
git add tests/Unit/Domains/QuoteManagement/ReadModels/
git commit -m "feat(cqrs): implement QuoteListView read model"
```

---

#### Task 5: Dashboard Stats Read Model

**Files:**
- Create: `app/Domains/QuoteManagement/Infrastructure/ReadModels/QuoteDashboardStats.php`

**Step 1: Create model**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\ReadModels;

use Illuminate\Database\Eloquent\Model;

/**
 * Read-only model for dashboard statistics.
 * Updated by QuoteDashboardProjector.
 */
class QuoteDashboardStats extends Model
{
    protected $table = 'quote_dashboard_stats';
    protected $primaryKey = 'organization_id';
    protected $keyType = 'string';
    public $incrementing = false;
    public $timestamps = false;

    protected $casts = [
        'total_quotes' => 'integer',
        'draft_quotes' => 'integer',
        'sent_quotes' => 'integer',
        'accepted_quotes' => 'integer',
        'rejected_quotes' => 'integer',
        'expired_quotes' => 'integer',
        'total_quoted_value' => 'integer',
        'accepted_value' => 'integer',
        'pending_value' => 'integer',
        'rejected_value' => 'integer',
        'acceptance_rate' => 'float',
        'average_quote_value' => 'float',
        'quotes_this_month' => 'integer',
        'quotes_this_week' => 'integer',
        'last_updated' => 'datetime',
    ];

    // ==================
    // Accessors (convert cents to dollars)
    // ==================

    public function getTotalQuotedValueDollarsAttribute(): float
    {
        return $this->total_quoted_value / 100;
    }

    public function getAcceptedValueDollarsAttribute(): float
    {
        return $this->accepted_value / 100;
    }

    public function getPendingValueDollarsAttribute(): float
    {
        return $this->pending_value / 100;
    }

    public function getRejectedValueDollarsAttribute(): float
    {
        return $this->rejected_value / 100;
    }

    public function getAcceptanceRatePercentageAttribute(): string
    {
        return number_format($this->acceptance_rate, 1) . '%';
    }

    // ==================
    // Static Helpers
    // ==================

    public static function forOrganization(string $organizationId): ?self
    {
        return static::find($organizationId);
    }

    public static function getOrCreate(string $organizationId): self
    {
        return static::firstOrCreate(
            ['organization_id' => $organizationId],
            [
                'total_quotes' => 0,
                'draft_quotes' => 0,
                'sent_quotes' => 0,
                'accepted_quotes' => 0,
                'rejected_quotes' => 0,
                'expired_quotes' => 0,
                'total_quoted_value' => 0,
                'accepted_value' => 0,
                'pending_value' => 0,
                'rejected_value' => 0,
                'acceptance_rate' => 0,
                'average_quote_value' => 0,
                'quotes_this_month' => 0,
                'quotes_this_week' => 0,
            ]
        );
    }
}
```

**Step 2: Commit**

```bash
git add app/Domains/QuoteManagement/Infrastructure/ReadModels/QuoteDashboardStats.php
git commit -m "feat(cqrs): implement QuoteDashboardStats read model"
```

---

### Day 34 (Wednesday): Projectors - Part 1

**Goal:** Create projectors that update read models from domain events

#### Task 6: Quote List Projector

**Files:**
- Create: `app/Domains/QuoteManagement/Infrastructure/Projectors/QuoteListProjector.php`
- Test: `tests/Integration/Domains/QuoteManagement/Projectors/QuoteListProjectorTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Integration\Domains\QuoteManagement\Projectors;

use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\LineItemAdded;
use App\Domains\QuoteManagement\Domain\Events\QuoteSent;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Infrastructure\Projectors\QuoteListProjector;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class QuoteListProjectorTest extends TestCase
{
    use RefreshDatabase;

    private QuoteListProjector $projector;

    protected function setUp(): void
    {
        parent::setUp();
        $this->projector = app(QuoteListProjector::class);
        
        // Create a mock client for denormalization
        \DB::table('clients')->insert([
            'id' => 'client-123',
            'organization_id' => 'org-456',
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }

    /** @test */
    public function creates_list_view_on_quote_draft_created(): void
    {
        $event = new QuoteDraftCreated(
            quoteId: 'quote-789',
            clientId: 'client-123',
            organizationId: 'org-456',
            quoteNumber: 'Q-000001',
            title: 'Kitchen Renovation',
            currency: 'USD',
            occurredOn: new \DateTimeImmutable()
        );

        $this->projector->onQuoteDraftCreated($event);

        $this->assertDatabaseHas('quote_list_view', [
            'id' => 'quote-789',
            'quote_number' => 'Q-000001',
            'client_name' => 'John Doe',
            'status' => 'draft',
            'line_items_count' => 0,
        ]);
    }

    /** @test */
    public function updates_totals_on_line_item_added(): void
    {
        $this->createQuoteListView('quote-789');

        $event = new LineItemAdded(
            quoteId: 'quote-789',
            lineItemId: 'item-001',
            description: 'Labor',
            quantity: 10,
            unitPrice: 7500, // $75.00 in cents
            lineItemTotal: 75000, // $750.00 in cents
            newQuoteTotal: 75000,
            occurredOn: new \DateTimeImmutable()
        );

        $this->projector->onLineItemAdded($event);

        $quote = QuoteListView::find('quote-789');
        $this->assertEquals(75000, $quote->total);
        $this->assertEquals(1, $quote->line_items_count);
    }

    /** @test */
    public function updates_status_on_quote_sent(): void
    {
        $this->createQuoteListView('quote-789');

        $event = new QuoteSent(
            quoteId: 'quote-789',
            sentAt: new \DateTimeImmutable('2024-01-15 10:00:00')
        );

        $this->projector->onQuoteSent($event);

        $quote = QuoteListView::find('quote-789');
        $this->assertEquals('sent', $quote->status);
        $this->assertNotNull($quote->sent_at);
    }

    /** @test */
    public function updates_status_on_quote_accepted(): void
    {
        $this->createQuoteListView('quote-789', ['status' => 'sent']);

        $event = new QuoteAccepted(
            quoteId: 'quote-789',
            clientId: 'client-123',
            organizationId: 'org-456',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable('2024-01-20 14:30:00')
        );

        $this->projector->onQuoteAccepted($event);

        $quote = QuoteListView::find('quote-789');
        $this->assertEquals('accepted', $quote->status);
        $this->assertNotNull($quote->accepted_at);
    }

    private function createQuoteListView(string $id, array $overrides = []): void
    {
        \DB::table('quote_list_view')->insert(array_merge([
            'id' => $id,
            'organization_id' => 'org-456',
            'quote_number' => 'Q-000001',
            'client_id' => 'client-123',
            'client_name' => 'John Doe',
            'title' => 'Test Quote',
            'status' => 'draft',
            'subtotal' => 0,
            'total' => 0,
            'currency' => 'USD',
            'valid_until' => now()->addDays(30),
            'line_items_count' => 0,
            'created_at' => now(),
            'updated_at' => now(),
        ], $overrides));
    }
}
```

**Step 2: Run test to verify it fails**

Run: `./vendor/bin/pest tests/Integration/Domains/QuoteManagement/Projectors/QuoteListProjectorTest.php`
Expected: FAIL - class not found

**Step 3: Write QuoteListProjector**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\Projectors;

use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\LineItemAdded;
use App\Domains\QuoteManagement\Domain\Events\LineItemRemoved;
use App\Domains\QuoteManagement\Domain\Events\QuoteSent;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Domain\Events\QuoteRejected;
use Illuminate\Support\Facades\DB;

class QuoteListProjector
{
    public function onQuoteDraftCreated(QuoteDraftCreated $event): void
    {
        $client = $this->getClient($event->clientId);

        DB::table('quote_list_view')->insert([
            'id' => $event->quoteId,
            'organization_id' => $event->organizationId,
            'quote_number' => $event->quoteNumber,
            'client_id' => $event->clientId,
            'client_name' => $client?->name ?? 'Unknown Client',
            'client_email' => $client?->email,
            'title' => $event->title,
            'status' => 'draft',
            'subtotal' => 0,
            'total' => 0,
            'currency' => $event->currency,
            'valid_until' => now()->addDays(30), // Default, should come from event
            'line_items_count' => 0,
            'created_at' => $event->occurredOn(),
            'updated_at' => $event->occurredOn(),
        ]);
    }

    public function onLineItemAdded(LineItemAdded $event): void
    {
        $current = DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->first();

        if (!$current) {
            return; // Quote not found, skip
        }

        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'total' => $event->newQuoteTotal,
                'subtotal' => $event->newQuoteTotal, // Simplified, would need tax calc
                'line_items_count' => $current->line_items_count + 1,
                'updated_at' => $event->occurredOn(),
            ]);
    }

    public function onLineItemRemoved(LineItemRemoved $event): void
    {
        $current = DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->first();

        if (!$current) {
            return;
        }

        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'total' => $event->newQuoteTotal,
                'subtotal' => $event->newQuoteTotal,
                'line_items_count' => max(0, $current->line_items_count - 1),
                'updated_at' => $event->occurredOn(),
            ]);
    }

    public function onQuoteSent(QuoteSent $event): void
    {
        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'status' => 'sent',
                'sent_at' => $event->sentAt,
                'updated_at' => $event->sentAt,
            ]);
    }

    public function onQuoteAccepted(QuoteAccepted $event): void
    {
        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'status' => 'accepted',
                'accepted_at' => $event->acceptedAt,
                'updated_at' => $event->acceptedAt,
            ]);
    }

    public function onQuoteRejected(QuoteRejected $event): void
    {
        DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->update([
                'status' => 'rejected',
                'updated_at' => $event->occurredOn(),
            ]);
    }

    private function getClient(string $clientId): ?object
    {
        return DB::table('clients')
            ->where('id', $clientId)
            ->first();
    }
}
```

**Step 4: Run tests**

Run: `./vendor/bin/pest tests/Integration/Domains/QuoteManagement/Projectors/QuoteListProjectorTest.php`
Expected: PASS

**Step 5: Commit**

```bash
git add app/Domains/QuoteManagement/Infrastructure/Projectors/QuoteListProjector.php
git add tests/Integration/Domains/QuoteManagement/Projectors/
git commit -m "feat(cqrs): implement QuoteListProjector"
```

---

### Day 35 (Thursday): Projectors - Part 2

#### Task 7: Dashboard Stats Projector

**Files:**
- Create: `app/Domains/QuoteManagement/Infrastructure/Projectors/QuoteDashboardProjector.php`
- Test: `tests/Integration/Domains/QuoteManagement/Projectors/QuoteDashboardProjectorTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Integration\Domains\QuoteManagement\Projectors;

use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\QuoteSent;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Infrastructure\Projectors\QuoteDashboardProjector;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteDashboardStats;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class QuoteDashboardProjectorTest extends TestCase
{
    use RefreshDatabase;

    private QuoteDashboardProjector $projector;

    protected function setUp(): void
    {
        parent::setUp();
        $this->projector = app(QuoteDashboardProjector::class);
    }

    /** @test */
    public function increments_total_and_draft_on_quote_created(): void
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

        $this->projector->onQuoteDraftCreated($event);

        $stats = QuoteDashboardStats::find('org-789');
        $this->assertEquals(1, $stats->total_quotes);
        $this->assertEquals(1, $stats->draft_quotes);
    }

    /** @test */
    public function moves_draft_to_sent_and_adds_pending_value(): void
    {
        // Setup: org has 1 draft quote
        $this->createStats('org-789', [
            'total_quotes' => 1,
            'draft_quotes' => 1,
        ]);

        // Create list view for the quote (needed for total)
        $this->createQuoteListView('quote-123', 'org-789', 500000); // $5000

        $event = new QuoteSent(
            quoteId: 'quote-123',
            sentAt: new \DateTimeImmutable()
        );

        $this->projector->onQuoteSent($event);

        $stats = QuoteDashboardStats::find('org-789');
        $this->assertEquals(0, $stats->draft_quotes);
        $this->assertEquals(1, $stats->sent_quotes);
        $this->assertEquals(500000, $stats->pending_value);
    }

    /** @test */
    public function moves_sent_to_accepted_and_calculates_rate(): void
    {
        $this->createStats('org-789', [
            'total_quotes' => 5,
            'sent_quotes' => 2,
            'accepted_quotes' => 2,
            'pending_value' => 800000,
        ]);

        $event = new QuoteAccepted(
            quoteId: 'quote-123',
            clientId: 'client-456',
            organizationId: 'org-789',
            totalAmount: 5000.00,
            currency: 'USD',
            acceptedAt: new \DateTimeImmutable()
        );

        $this->projector->onQuoteAccepted($event);

        $stats = QuoteDashboardStats::find('org-789');
        $this->assertEquals(1, $stats->sent_quotes);
        $this->assertEquals(3, $stats->accepted_quotes);
        $this->assertEquals(500000, $stats->accepted_value);
        $this->assertEquals(300000, $stats->pending_value); // 800000 - 500000
        $this->assertEquals(60.0, $stats->acceptance_rate); // 3 accepted / 5 total
    }

    private function createStats(string $orgId, array $data = []): void
    {
        \DB::table('quote_dashboard_stats')->insert(array_merge([
            'organization_id' => $orgId,
            'total_quotes' => 0,
            'draft_quotes' => 0,
            'sent_quotes' => 0,
            'accepted_quotes' => 0,
            'rejected_quotes' => 0,
            'expired_quotes' => 0,
            'total_quoted_value' => 0,
            'accepted_value' => 0,
            'pending_value' => 0,
            'rejected_value' => 0,
            'acceptance_rate' => 0,
            'average_quote_value' => 0,
            'quotes_this_month' => 0,
            'quotes_this_week' => 0,
        ], $data));
    }

    private function createQuoteListView(string $id, string $orgId, int $total): void
    {
        \DB::table('quote_list_view')->insert([
            'id' => $id,
            'organization_id' => $orgId,
            'quote_number' => 'Q-000001',
            'client_id' => 'client-123',
            'client_name' => 'Test',
            'title' => 'Test',
            'status' => 'draft',
            'subtotal' => $total,
            'total' => $total,
            'currency' => 'USD',
            'valid_until' => now()->addDays(30),
            'line_items_count' => 1,
            'created_at' => now(),
            'updated_at' => now(),
        ]);
    }
}
```

**Step 2: Write QuoteDashboardProjector**

```php
<?php

namespace App\Domains\QuoteManagement\Infrastructure\Projectors;

use App\Domains\QuoteManagement\Domain\Events\QuoteDraftCreated;
use App\Domains\QuoteManagement\Domain\Events\QuoteSent;
use App\Domains\QuoteManagement\Domain\Events\QuoteAccepted;
use App\Domains\QuoteManagement\Domain\Events\QuoteRejected;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteDashboardStats;
use Illuminate\Support\Facades\DB;

class QuoteDashboardProjector
{
    public function onQuoteDraftCreated(QuoteDraftCreated $event): void
    {
        $stats = QuoteDashboardStats::getOrCreate($event->organizationId);

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $event->organizationId)
            ->update([
                'total_quotes' => $stats->total_quotes + 1,
                'draft_quotes' => $stats->draft_quotes + 1,
                'quotes_this_month' => $stats->quotes_this_month + 1,
                'quotes_this_week' => $stats->quotes_this_week + 1,
                'last_updated' => now(),
            ]);
    }

    public function onQuoteSent(QuoteSent $event): void
    {
        $quote = DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->first();

        if (!$quote) {
            return;
        }

        $stats = QuoteDashboardStats::find($quote->organization_id);
        if (!$stats) {
            return;
        }

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $quote->organization_id)
            ->update([
                'draft_quotes' => max(0, $stats->draft_quotes - 1),
                'sent_quotes' => $stats->sent_quotes + 1,
                'pending_value' => $stats->pending_value + $quote->total,
                'total_quoted_value' => $stats->total_quoted_value + $quote->total,
                'last_updated' => now(),
            ]);

        $this->recalculateAverageQuoteValue($quote->organization_id);
    }

    public function onQuoteAccepted(QuoteAccepted $event): void
    {
        $stats = QuoteDashboardStats::find($event->organizationId);
        if (!$stats) {
            return;
        }

        $totalCents = (int)($event->totalAmount * 100);

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $event->organizationId)
            ->update([
                'sent_quotes' => max(0, $stats->sent_quotes - 1),
                'accepted_quotes' => $stats->accepted_quotes + 1,
                'pending_value' => max(0, $stats->pending_value - $totalCents),
                'accepted_value' => $stats->accepted_value + $totalCents,
                'last_updated' => now(),
            ]);

        $this->recalculateAcceptanceRate($event->organizationId);
    }

    public function onQuoteRejected(QuoteRejected $event): void
    {
        $quote = DB::table('quote_list_view')
            ->where('id', $event->quoteId)
            ->first();

        if (!$quote) {
            return;
        }

        $stats = QuoteDashboardStats::find($quote->organization_id);
        if (!$stats) {
            return;
        }

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $quote->organization_id)
            ->update([
                'sent_quotes' => max(0, $stats->sent_quotes - 1),
                'rejected_quotes' => $stats->rejected_quotes + 1,
                'pending_value' => max(0, $stats->pending_value - $quote->total),
                'rejected_value' => $stats->rejected_value + $quote->total,
                'last_updated' => now(),
            ]);

        $this->recalculateAcceptanceRate($quote->organization_id);
    }

    private function recalculateAcceptanceRate(string $organizationId): void
    {
        $stats = QuoteDashboardStats::find($organizationId);
        if (!$stats || $stats->total_quotes === 0) {
            return;
        }

        $rate = ($stats->accepted_quotes / $stats->total_quotes) * 100;

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $organizationId)
            ->update(['acceptance_rate' => round($rate, 2)]);
    }

    private function recalculateAverageQuoteValue(string $organizationId): void
    {
        $stats = QuoteDashboardStats::find($organizationId);
        $sentAndAccepted = $stats->sent_quotes + $stats->accepted_quotes + $stats->rejected_quotes;

        if ($sentAndAccepted === 0) {
            return;
        }

        $average = $stats->total_quoted_value / $sentAndAccepted;

        DB::table('quote_dashboard_stats')
            ->where('organization_id', $organizationId)
            ->update(['average_quote_value' => round($average / 100, 2)]);
    }
}
```

**Step 3: Run tests**

Run: `./vendor/bin/pest tests/Integration/Domains/QuoteManagement/Projectors/QuoteDashboardProjectorTest.php`
Expected: PASS

**Step 4: Commit**

```bash
git add app/Domains/QuoteManagement/Infrastructure/Projectors/QuoteDashboardProjector.php
git add tests/Integration/Domains/QuoteManagement/Projectors/QuoteDashboardProjectorTest.php
git commit -m "feat(cqrs): implement QuoteDashboardProjector"
```

---

### Day 36 (Friday): Query Service

#### Task 8: Quote Query Service

**Files:**
- Create: `app/Domains/QuoteManagement/Application/Queries/QuoteQueryService.php`
- Test: `tests/Integration/Domains/QuoteManagement/Queries/QuoteQueryServiceTest.php`

**Step 1: Write the failing test**

```php
<?php

namespace Tests\Integration\Domains\QuoteManagement\Queries;

use App\Domains\QuoteManagement\Application\Queries\QuoteQueryService;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class QuoteQueryServiceTest extends TestCase
{
    use RefreshDatabase;

    private QuoteQueryService $queryService;

    protected function setUp(): void
    {
        parent::setUp();
        $this->queryService = app(QuoteQueryService::class);
    }

    /** @test */
    public function lists_quotes_for_organization_paginated(): void
    {
        $this->createQuotes('org-1', 25);
        $this->createQuotes('org-2', 5);

        $result = $this->queryService->list('org-1', perPage: 10);

        $this->assertCount(10, $result->items());
        $this->assertEquals(25, $result->total());
    }

    /** @test */
    public function filters_by_status(): void
    {
        $this->createQuotes('org-1', 3, ['status' => 'draft']);
        $this->createQuotes('org-1', 2, ['status' => 'sent']);
        $this->createQuotes('org-1', 1, ['status' => 'accepted']);

        $result = $this->queryService->list('org-1', status: 'sent');

        $this->assertCount(2, $result->items());
    }

    /** @test */
    public function filters_by_client(): void
    {
        $this->createQuotes('org-1', 3, ['client_id' => 'client-1']);
        $this->createQuotes('org-1', 2, ['client_id' => 'client-2']);

        $result = $this->queryService->list('org-1', clientId: 'client-1');

        $this->assertCount(3, $result->items());
    }

    /** @test */
    public function sorts_by_created_at_desc_by_default(): void
    {
        $this->createQuote('org-1', ['id' => 'q-1', 'created_at' => now()->subDays(2)]);
        $this->createQuote('org-1', ['id' => 'q-2', 'created_at' => now()->subDay()]);
        $this->createQuote('org-1', ['id' => 'q-3', 'created_at' => now()]);

        $result = $this->queryService->list('org-1');

        $ids = collect($result->items())->pluck('id')->toArray();
        $this->assertEquals(['q-3', 'q-2', 'q-1'], $ids);
    }

    /** @test */
    public function gets_dashboard_stats(): void
    {
        $this->createDashboardStats('org-1', [
            'total_quotes' => 50,
            'accepted_quotes' => 30,
            'accepted_value' => 15000000, // $150,000
        ]);

        $stats = $this->queryService->getDashboardStats('org-1');

        $this->assertEquals(50, $stats['total_quotes']);
        $this->assertEquals(30, $stats['accepted_quotes']);
        $this->assertEquals(150000.00, $stats['accepted_value_dollars']);
    }

    /** @test */
    public function searches_quotes_by_title(): void
    {
        $this->createQuote('org-1', ['title' => 'Kitchen Renovation']);
        $this->createQuote('org-1', ['title' => 'Bathroom Remodel']);
        $this->createQuote('org-1', ['title' => 'Kitchen Cabinet Install']);

        $result = $this->queryService->search('org-1', 'Kitchen');

        $this->assertCount(2, $result);
    }

    private function createQuotes(string $orgId, int $count, array $overrides = []): void
    {
        for ($i = 0; $i < $count; $i++) {
            $this->createQuote($orgId, $overrides);
        }
    }

    private function createQuote(string $orgId, array $overrides = []): void
    {
        \DB::table('quote_list_view')->insert(array_merge([
            'id' => $overrides['id'] ?? \Str::uuid()->toString(),
            'organization_id' => $orgId,
            'quote_number' => 'Q-' . str_pad(rand(1, 999999), 6, '0', STR_PAD_LEFT),
            'client_id' => 'client-1',
            'client_name' => 'Test Client',
            'title' => 'Test Quote',
            'status' => 'draft',
            'subtotal' => 100000,
            'total' => 100000,
            'currency' => 'USD',
            'valid_until' => now()->addDays(30),
            'line_items_count' => 1,
            'created_at' => now(),
            'updated_at' => now(),
        ], $overrides));
    }

    private function createDashboardStats(string $orgId, array $overrides = []): void
    {
        \DB::table('quote_dashboard_stats')->insert(array_merge([
            'organization_id' => $orgId,
            'total_quotes' => 0,
            'draft_quotes' => 0,
            'sent_quotes' => 0,
            'accepted_quotes' => 0,
            'rejected_quotes' => 0,
            'expired_quotes' => 0,
            'total_quoted_value' => 0,
            'accepted_value' => 0,
            'pending_value' => 0,
            'rejected_value' => 0,
            'acceptance_rate' => 0,
            'average_quote_value' => 0,
            'quotes_this_month' => 0,
            'quotes_this_week' => 0,
        ], $overrides));
    }
}
```

**Step 2: Write QuoteQueryService**

```php
<?php

namespace App\Domains\QuoteManagement\Application\Queries;

use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteListView;
use App\Domains\QuoteManagement\Infrastructure\ReadModels\QuoteDashboardStats;
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

class QuoteQueryService
{
    public function list(
        string $organizationId,
        ?string $status = null,
        ?string $clientId = null,
        ?string $sortBy = 'created_at',
        ?string $sortDir = 'desc',
        int $perPage = 20
    ): LengthAwarePaginator {
        return QuoteListView::forOrganization($organizationId)
            ->when($status, fn($q) => $q->where('status', $status))
            ->when($clientId, fn($q) => $q->where('client_id', $clientId))
            ->orderBy($sortBy, $sortDir)
            ->paginate($perPage);
    }

    public function findById(string $quoteId): ?QuoteListView
    {
        return QuoteListView::find($quoteId);
    }

    public function getDashboardStats(string $organizationId): array
    {
        $stats = QuoteDashboardStats::find($organizationId);

        if (!$stats) {
            return $this->emptyStats();
        }

        return [
            'total_quotes' => $stats->total_quotes,
            'draft_quotes' => $stats->draft_quotes,
            'sent_quotes' => $stats->sent_quotes,
            'accepted_quotes' => $stats->accepted_quotes,
            'rejected_quotes' => $stats->rejected_quotes,
            'expired_quotes' => $stats->expired_quotes,
            'total_quoted_value_dollars' => $stats->total_quoted_value / 100,
            'accepted_value_dollars' => $stats->accepted_value / 100,
            'pending_value_dollars' => $stats->pending_value / 100,
            'rejected_value_dollars' => $stats->rejected_value / 100,
            'acceptance_rate' => $stats->acceptance_rate,
            'average_quote_value' => $stats->average_quote_value,
            'quotes_this_month' => $stats->quotes_this_month,
            'quotes_this_week' => $stats->quotes_this_week,
        ];
    }

    public function search(string $organizationId, string $term): Collection
    {
        return QuoteListView::forOrganization($organizationId)
            ->where(function ($query) use ($term) {
                $query->where('title', 'like', "%{$term}%")
                    ->orWhere('quote_number', 'like', "%{$term}%")
                    ->orWhere('client_name', 'like', "%{$term}%");
            })
            ->orderByDesc('created_at')
            ->limit(50)
            ->get();
    }

    public function getRecentQuotes(string $organizationId, int $limit = 5): Collection
    {
        return QuoteListView::forOrganization($organizationId)
            ->orderByDesc('created_at')
            ->limit($limit)
            ->get();
    }

    public function getPendingQuotes(string $organizationId): Collection
    {
        return QuoteListView::forOrganization($organizationId)
            ->pending()
            ->orderBy('valid_until')
            ->get();
    }

    public function getExpiringQuotes(string $organizationId, int $daysAhead = 7): Collection
    {
        return QuoteListView::forOrganization($organizationId)
            ->pending()
            ->where('valid_until', '<=', now()->addDays($daysAhead))
            ->orderBy('valid_until')
            ->get();
    }

    private function emptyStats(): array
    {
        return [
            'total_quotes' => 0,
            'draft_quotes' => 0,
            'sent_quotes' => 0,
            'accepted_quotes' => 0,
            'rejected_quotes' => 0,
            'expired_quotes' => 0,
            'total_quoted_value_dollars' => 0,
            'accepted_value_dollars' => 0,
            'pending_value_dollars' => 0,
            'rejected_value_dollars' => 0,
            'acceptance_rate' => 0,
            'average_quote_value' => 0,
            'quotes_this_month' => 0,
            'quotes_this_week' => 0,
        ];
    }
}
```

**Step 3: Run tests**

Run: `./vendor/bin/pest tests/Integration/Domains/QuoteManagement/Queries/QuoteQueryServiceTest.php`
Expected: PASS

**Step 4: Commit**

```bash
git add app/Domains/QuoteManagement/Application/Queries/
git add tests/Integration/Domains/QuoteManagement/Queries/
git commit -m "feat(cqrs): implement QuoteQueryService"
```

---

## Week 6: Integration & Optimization

### Day 37-38: Register Projectors, Update Controllers

#### Task 9: Register Projectors in EventServiceProvider

#### Task 10: Update QuoteController to use Query Service

#### Task 11: Add Dashboard Endpoint

---

### Day 39-40: Rebuild Projections Command

#### Task 12: Create RebuildProjectionsCommand

---

### Day 41: Performance Testing & Documentation

#### Task 13: Performance benchmarks

#### Task 14: Documentation

---

## Phase 3 Success Criteria

```
[ ] Read model tables created (quote_list_view, quote_dashboard_stats, search_index)
[ ] Projectors update read models on all Quote events
[ ] QuoteQueryService provides fast reads
[ ] Dashboard endpoint returns stats
[ ] Search functionality working
[ ] Rebuild projections command working
[ ] Performance: list queries < 50ms
[ ] All integration tests passing
```
```

---