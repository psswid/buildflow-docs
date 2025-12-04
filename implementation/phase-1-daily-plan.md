
# BuildFlow Phase 1: Day-by-Day Implementation Plan

> **Context:** Quote Management with DDD, Event-Driven, CQRS patterns  
> **Duration:** 14 days (Days 8-21 of 10-week roadmap)  
> **Approach:** TDD (Test-Driven Development)  
> **Created:** 2024-12-03

---

## Overview

This plan follows the TDD cycle:
- 🔴 RED: Write failing test
- 🟢 GREEN: Write minimum code to pass
- 🔵 REFACTOR: Clean up code
- ✅ COMMIT: Save progress

---

## Week 2: Domain & Application Layer

---

### Day 8 (Monday): Shared Kernel & First Value Objects

**Goal:** Establish foundation for all DDD patterns

#### Morning (4h)

```
[ ] 🔴 Create tests/Unit/SharedKernel/AggregateRootTest.php
    - test_aggregate_records_domain_events
    - test_releasing_events_clears_them
    - test_aggregate_has_no_events_initially
[ ] 🟢 Implement app/SharedKernel/Domain/AggregateRoot.php
[ ] 🔵 Refactor if needed
[ ] ✅ Commit: "feat: implement AggregateRoot base class"

[ ] 🔴 Create tests/Unit/SharedKernel/IdentifierTest.php
[ ] 🟢 Implement app/SharedKernel/Domain/Identifier.php
[ ] 🔴 Create tests/Unit/SharedKernel/ValueObjectTest.php
[ ] 🟢 Implement app/SharedKernel/Domain/ValueObject.php
[ ] ✅ Commit: "feat: implement base ValueObject and Identifier"
```

#### Afternoon (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteIdTest.php
    - test_can_generate_new_quote_id
    - test_can_create_from_string
    - test_throws_exception_for_invalid_uuid
    - test_two_ids_with_same_value_are_equal
    - test_two_different_ids_are_not_equal
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/QuoteId.php
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/InvalidQuoteIdException.php
[ ] ✅ Commit: "feat(quote): implement QuoteId value object"

[ ] Create folder structure (run these commands):
    mkdir -p app/SharedKernel/{Domain,Infrastructure}
    mkdir -p app/Domains/QuoteManagement/{Domain,Application,Infrastructure}
    mkdir -p app/Domains/QuoteManagement/Domain/{ValueObjects,Events,Exceptions,Entities}
    mkdir -p app/Domains/QuoteManagement/Application/{Commands,Handlers,Queries}
    mkdir -p app/Domains/QuoteManagement/Infrastructure/{Persistence,Http,EventListeners}
    mkdir -p app/Domains/QuoteManagement/Infrastructure/Http/{Controllers,Requests,Resources}
    mkdir -p tests/Unit/Domains/QuoteManagement/{ValueObjects,Events,Entities}
    mkdir -p tests/Feature/Domains/QuoteManagement
    mkdir -p tests/Integration/Api
    mkdir -p tests/Architecture
```

#### EOD Checklist

```
[ ] Run ./vendor/bin/pest - all tests passing
[ ] Run ./vendor/bin/phpstan - no errors
[ ] Commit all work with meaningful messages
[ ] Update personal notes with learnings
```

**Expected Commits:** 3-4

---

### Day 9 (Tuesday): Money & Currency Value Objects

**Goal:** Implement the complex Money value object with full arithmetic

#### Morning (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/CurrencyTest.php
    - test_can_create_usd_currency
    - test_can_create_gbp_currency
    - test_currencies_are_equal_if_same_code
    - test_different_currencies_are_not_equal
    - test_can_get_currency_code
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/Currency.php
[ ] ✅ Commit: "feat(quote): implement Currency value object"

[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/MoneyTest.php
    - test_can_create_money_from_cents
    - test_can_create_money_from_float
    - test_can_create_zero_money
    - test_converts_to_float_correctly
    - test_converts_to_cents_correctly
[ ] 🟢 Implement basic Money (creation only)
[ ] ✅ Commit: "feat(quote): implement Money value object basics"
```

#### Afternoon (4h)

```
[ ] 🔴 Add arithmetic tests to MoneyTest.php
    - test_can_add_money
    - test_can_subtract_money
    - test_can_multiply_money
    - test_throws_exception_for_negative_amount
    - test_throws_exception_when_adding_different_currencies
    - test_throws_exception_when_subtracting_different_currencies
    - test_two_money_objects_with_same_value_are_equal
    - test_different_amounts_are_not_equal
[ ] 🟢 Implement arithmetic operations in Money
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/NegativeMoneyException.php
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/CurrencyMismatchException.php
[ ] 🔵 Refactor Money for cleaner code
[ ] ✅ Commit: "feat(quote): implement Money arithmetic operations"
```

#### EOD Checklist

```
[ ] Money has 8+ passing tests
[ ] All edge cases covered (negative, currency mismatch)
[ ] Run full test suite
[ ] Code reviewed against DDD principles
```

**Expected Commits:** 3-4

---

### Day 10 (Wednesday): Status, Number & Supporting VOs

**Goal:** Complete all value objects needed for Quote aggregate

#### Morning (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteStatusTest.php
    - test_can_create_draft_status
    - test_can_create_sent_status
    - test_can_create_accepted_status
    - test_can_create_rejected_status
    - test_can_create_expired_status
    - test_is_draft_returns_true_for_draft
    - test_is_sent_returns_true_for_sent
    - test_can_be_edited_returns_true_only_for_draft
    - test_statuses_are_equal_if_same_value
    - test_can_get_string_value
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/QuoteStatus.php
[ ] ✅ Commit: "feat(quote): implement QuoteStatus value object"

[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/QuoteNumberTest.php
    - test_generates_formatted_number_Q_000001
    - test_generates_sequential_numbers
    - test_can_get_string_representation
    - test_quote_numbers_are_equal_if_same_value
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/QuoteNumber.php
[ ] ✅ Commit: "feat(quote): implement QuoteNumber value object"
```

#### Afternoon (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/TaxRateTest.php
    - test_can_create_tax_rate
    - test_can_create_zero_tax_rate
    - test_throws_exception_for_negative_rate
    - test_can_get_value
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/TaxRate.php
[ ] ✅ Commit: "feat(quote): implement TaxRate value object"

[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/ValueObjects/DiscountTest.php
    - test_can_create_discount
    - test_can_create_zero_discount
    - test_cannot_be_negative
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/ValueObjects/Discount.php
[ ] ✅ Commit: "feat(quote): implement Discount value object"

[ ] Create shared identifier VOs (copy pattern from QuoteId):
    - app/Domains/QuoteManagement/Domain/ValueObjects/ClientId.php
    - app/Domains/QuoteManagement/Domain/ValueObjects/OrganizationId.php
[ ] ✅ Commit: "feat(quote): add ClientId and OrganizationId value objects"
```

#### EOD Checklist

```
[ ] All 7 Value Objects implemented with tests
[ ] Run: ./vendor/bin/pest tests/Unit/Domains/QuoteManagement/ValueObjects/
[ ] All VOs are final classes
[ ] All VOs are immutable (no setters)
```

**Expected Commits:** 5-6

---

### Day 11 (Thursday): Domain Events & LineItem Entity

**Goal:** Implement all domain events and LineItem entity

#### Morning (4h)

```
[ ] Create app/SharedKernel/Domain/DomainEvent.php interface:
    - eventName(): string
    - occurredOn(): DateTimeImmutable
    - aggregateId(): string
    - toArray(): array

[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/Events/QuoteDraftCreatedTest.php
    - test_can_create_event
    - test_event_has_correct_name
    - test_can_get_aggregate_id
    - test_can_serialize_to_array
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Events/QuoteDraftCreated.php
[ ] ✅ Commit: "feat(quote): implement QuoteDraftCreated event"

[ ] 🔴 Create tests for remaining events (same pattern)
[ ] 🟢 Implement all events:
    - LineItemAdded.php
    - LineItemRemoved.php
    - QuoteSent.php
    - QuoteAccepted.php
    - QuoteRejected.php
[ ] ✅ Commit: "feat(quote): implement all domain events"
```

#### Afternoon (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/Entities/LineItemTest.php
    - test_can_create_line_item
    - test_calculates_total_correctly
    - test_quantity_must_be_positive
    - test_unit_price_must_be_positive
    - test_can_update_quantity
    - test_can_update_description
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Entities/LineItem.php
[ ] ✅ Commit: "feat(quote): implement LineItem entity"

[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/Entities/LineItemCollectionTest.php
    - test_collection_starts_empty
    - test_can_add_item
    - test_can_remove_item
    - test_can_check_if_empty
    - test_calculates_subtotal
    - test_can_count_items
    - test_is_iterable
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Entities/LineItemCollection.php
[ ] ✅ Commit: "feat(quote): implement LineItemCollection"
```

#### EOD Checklist

```
[ ] 6 Domain Events implemented
[ ] LineItem entity with validation
[ ] LineItemCollection with iteration
[ ] All tests passing
```

**Expected Commits:** 4-5

---

### Day 12 (Friday): Quote Aggregate - Part 1

**Goal:** Implement Quote creation and line item management

#### Morning (4h)

```
[ ] 🔴 Create tests/Unit/Domains/QuoteManagement/QuoteTest.php

    Creation tests:
    - test_can_create_draft_quote
    - test_creating_draft_records_event
    - test_new_quote_has_draft_status
    - test_new_quote_has_zero_total
    - test_new_quote_has_empty_line_items

[ ] 🟢 Implement Quote::createDraft() static factory method
[ ] ✅ Commit: "feat(quote): implement Quote aggregate creation"
```

#### Afternoon (4h)

```
[ ] 🔴 Add line item tests to QuoteTest.php

    Line item tests:
    - test_can_add_line_item_to_draft_quote
    - test_adding_line_item_records_event
    - test_adding_line_item_recalculates_total
    - test_can_add_multiple_line_items
    - test_can_remove_line_item
    - test_removing_line_item_recalculates_total
    - test_cannot_add_line_item_to_sent_quote

[ ] 🟢 Implement Quote::addLineItem()
[ ] 🟢 Implement Quote::removeLineItem()
[ ] 🟢 Implement Quote::recalculateTotal() (private)
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/CannotModifySentQuote.php
[ ] ✅ Commit: "feat(quote): implement Quote line item management"
```

#### EOD Checklist

```
[ ] Quote can be created
[ ] Line items can be added/removed
[ ] Total recalculates automatically
[ ] Events recorded correctly
[ ] Business rules enforced
```

**Expected Commits:** 2-3

---

### Day 13 (Saturday - Optional): Quote Aggregate - Part 2

**Goal:** Implement Quote sending and acceptance

#### Morning (4h)

```
[ ] 🔴 Add sending tests to QuoteTest.php

    Sending tests:
    - test_can_send_quote_with_line_items
    - test_sending_records_quote_sent_event
    - test_cannot_send_empty_quote
    - test_cannot_send_already_sent_quote
    - test_cannot_edit_sent_quote
    - test_sending_sets_sent_at_timestamp

[ ] 🟢 Implement Quote::send()
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/CannotSendEmptyQuote.php
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/QuoteAlreadySentException.php
[ ] ✅ Commit: "feat(quote): implement Quote sending"
```

#### Afternoon (4h)

```
[ ] 🔴 Add acceptance tests to QuoteTest.php

    Acceptance tests:
    - test_can_accept_sent_quote
    - test_accepting_records_quote_accepted_event
    - test_cannot_accept_draft_quote
    - test_cannot_accept_expired_quote
    - test_accepted_quote_cannot_be_edited
    - test_can_reject_sent_quote
    - test_rejecting_records_quote_rejected_event

[ ] 🟢 Implement Quote::accept()
[ ] 🟢 Implement Quote::reject()
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/CanOnlyAcceptSentQuote.php
[ ] 🟢 Implement app/Domains/QuoteManagement/Domain/Exceptions/QuoteHasExpired.php
[ ] ✅ Commit: "feat(quote): implement Quote acceptance and rejection"
```

#### Evening (2h)

```
[ ] 🔴 Add total calculation tests
    - test_total_equals_subtotal_plus_tax_minus_discount
    - test_total_with_zero_tax
    - test_total_with_zero_discount
[ ] 🔵 Refactor Quote for cleaner code
[ ] ✅ Commit: "refactor(quote): clean up Quote aggregate"
```

#### EOD Checklist

```
[ ] Quote aggregate complete with 15+ tests
[ ] All business rules enforced
[ ] All state transitions working
[ ] Events recorded for all actions
```

**Expected Commits:** 3-4

---

### Day 14 (Sunday - Optional): Repository Interface & Application Commands

**Goal:** Define interfaces and create command DTOs

#### Morning (3h)

```
[ ] Create app/Domains/QuoteManagement/Domain/QuoteRepository.php interface:
    - nextIdentity(): QuoteId
    - nextNumber(): QuoteNumber
    - save(Quote $quote): void
    - findById(QuoteId $id): ?Quote
[ ] ✅ Commit: "feat(quote): define QuoteRepository interface"

[ ] Create all Command DTOs in app/Domains/QuoteManagement/Application/Commands/:
    - CreateQuoteDraft.php
    - AddLineItem.php
    - RemoveLineItem.php
    - SendQuote.php
    - AcceptQuote.php
    - RejectQuote.php
[ ] ✅ Commit: "feat(quote): add command DTOs"
```

#### Afternoon (3h)

```
[ ] 🔴 Create tests/Feature/Domains/QuoteManagement/CreateQuoteDraftTest.php
    - test_can_create_quote_draft_via_handler
    - test_quote_is_persisted
    - test_returns_quote_id
[ ] 🟢 Implement app/Domains/QuoteManagement/Application/Handlers/CreateQuoteDraftHandler.php
[ ] ✅ Commit: "feat(quote): implement CreateQuoteDraftHandler"
```

#### EOD Checklist

```
[ ] Repository interface defined
[ ] All 6 commands created
[ ] First handler implemented with test
```

**Expected Commits:** 3-4

---

## Week 3: Application & Infrastructure Layer

---

### Day 15 (Monday): Application Handlers - Part 1

**Goal:** Implement remaining command handlers

#### Morning (4h)

```
[ ] 🔴 Create tests/Feature/Domains/QuoteManagement/AddLineItemTest.php
    - test_can_add_line_item_via_handler
    - test_throws_exception_if_quote_not_found
    - test_total_is_recalculated
[ ] 🟢 Implement app/Domains/QuoteManagement/Application/Handlers/AddLineItemHandler.php
[ ] ✅ Commit: "feat(quote): implement AddLineItemHandler"

[ ] 🔴 Create tests/Feature/Domains/QuoteManagement/RemoveLineItemTest.php
[ ] 🟢 Implement app/Domains/QuoteManagement/Application/Handlers/RemoveLineItemHandler.php
[ ] ✅ Commit: "feat(quote): implement RemoveLineItemHandler"
```

#### Afternoon (4h)

```
[ ] 🔴 Create tests/Feature/Domains/QuoteManagement/SendQuoteTest.php
    - test_can_send_quote_via_handler
    - test_dispatches_quote_sent_event
    - test_throws_exception_for_empty_quote
[ ] 🟢 Implement app/Domains/QuoteManagement/Application/Handlers/SendQuoteHandler.php
[ ] ✅ Commit: "feat(quote): implement SendQuoteHandler"

[ ] 🔴 Create tests/Feature/Domains/QuoteManagement/AcceptQuoteTest.php
    - test_can_accept_quote_via_handler
    - test_dispatches_quote_accepted_event
    - test_throws_exception_for_expired_quote
[ ] 🟢 Implement app/Domains/QuoteManagement/Application/Handlers/AcceptQuoteHandler.php
[ ] ✅ Commit: "feat(quote): implement AcceptQuoteHandler"
```

#### EOD Checklist

```
[ ] 5 handlers implemented
[ ] All handlers have tests
[ ] Events properly dispatched
```

**Expected Commits:** 4-5

---

### Day 16 (Tuesday): Infrastructure - Migrations & Models

**Goal:** Create database schema and Eloquent models

#### Morning (4h)

```
[ ] Create migration: database/migrations/xxxx_create_quotes_table.php

    Schema::create('quotes', function (Blueprint $table) {
        $table->uuid('id')->primary();
        $table->uuid('organization_id')->index();
        $table->uuid('client_id')->index();
        $table->string('number', 20)->unique();
        $table->integer('sequence');
        $table->string('title');
        $table->string('status', 20)->default('draft');
        $table->decimal('subtotal', 12, 2)->default(0);
        $table->decimal('tax_rate', 5, 2)->default(0);
        $table->decimal('discount', 12, 2)->default(0);
        $table->decimal('total', 12, 2)->default(0);
        $table->string('currency', 3)->default('USD');
        $table->date('valid_until');
        $table->timestamp('sent_at')->nullable();
        $table->timestamp('accepted_at')->nullable();
        $table->timestamps();
        
        $table->index(['organization_id', 'status']);
        $table->index(['organization_id', 'created_at']);
    });

[ ] ✅ Commit: "feat(quote): add quotes migration"

[ ] Create migration: database/migrations/xxxx_create_quote_line_items_table.php

    Schema::create('quote_line_items', function (Blueprint $table) {
        $table->uuid('id')->primary();
        $table->uuid('quote_id');
        $table->text('description');
        $table->decimal('quantity', 10, 2);
        $table->decimal('unit_price', 12, 2);
        $table->decimal('total', 12, 2);
        $table->integer('sort_order')->default(0);
        $table->timestamps();
        
        $table->foreign('quote_id')
              ->references('id')
              ->on('quotes')
              ->onDelete('cascade');
    });

[ ] ✅ Commit: "feat(quote): add quote_line_items migration"
```

#### Afternoon (4h)

```
[ ] Create app/Domains/QuoteManagement/Infrastructure/Persistence/QuoteEloquentModel.php
[ ] Create app/Domains/QuoteManagement/Infrastructure/Persistence/LineItemEloquentModel.php
[ ] Run: php artisan migrate
[ ] ✅ Commit: "feat(quote): add Eloquent models"

[ ] Create database/factories/QuoteFactory.php
[ ] Create database/factories/LineItemFactory.php
[ ] ✅ Commit: "feat(quote): add model factories"
```

#### EOD Checklist

```
[ ] Migrations run without error
[ ] Models have proper relationships
[ ] Factories create valid data
```

**Expected Commits:** 4-5

---

### Day 17 (Wednesday): Eloquent Repository Implementation

**Goal:** Implement the repository that connects Domain to Database

#### Morning (4h)

```
[ ] 🔴 Create tests/Integration/Domains/QuoteManagement/EloquentQuoteRepositoryTest.php
    - test_can_save_quote
    - test_can_find_quote_by_id
    - test_returns_null_for_non_existent_quote
    - test_generates_next_identity
    - test_generates_sequential_numbers
    - test_saves_line_items
    - test_dispatches_domain_events_on_save
[ ] 🟢 Implement EloquentQuoteRepository::save()
[ ] ✅ Commit: "feat(quote): implement EloquentQuoteRepository save"
```

#### Afternoon (4h)

```
[ ] 🟢 Implement EloquentQuoteRepository::findById()
[ ] 🟢 Implement domain-to-eloquent mapping
[ ] 🟢 Implement eloquent-to-domain mapping (reconstruct aggregate)
[ ] 🔵 Add Quote::reconstitute() static method for reconstruction
[ ] ✅ Commit: "feat(quote): implement EloquentQuoteRepository findById"

[ ] Register in app/Providers/AppServiceProvider.php:
    $this->app->bind(
        QuoteRepository::class,
        EloquentQuoteRepository::class
    );
[ ] ✅ Commit: "feat(quote): register repository in ServiceProvider"
```

#### EOD Checklist

```
[ ] Repository fully functional
[ ] Aggregate reconstructs correctly from DB
[ ] Events dispatched on save
[ ] Multi-tenancy scope applied
```

**Expected Commits:** 3-4

---

### Day 18 (Thursday): HTTP Layer - Controllers & Routes

**Goal:** Create REST API endpoints

#### Morning (4h)

```
[ ] Create request classes in app/Domains/QuoteManagement/Infrastructure/Http/Requests/:
    - CreateQuoteRequest.php
    - AddLineItemRequest.php
    - UpdateLineItemRequest.php
[ ] ✅ Commit: "feat(quote): add HTTP request validation"

[ ] Create resource classes in app/Domains/QuoteManagement/Infrastructure/Http/Resources/:
    - QuoteResource.php
    - QuoteListResource.php
    - LineItemResource.php
[ ] ✅ Commit: "feat(quote): add API resources"
```

#### Afternoon (4h)

```
[ ] Create app/Domains/QuoteManagement/Infrastructure/Http/Controllers/QuoteController.php
    Methods:
    - store()        -> CreateQuoteDraft
    - show()         -> Get single quote
    - index()        -> List quotes
    - addLineItem()  -> Add line item
    - removeLineItem() -> Remove line item
    - send()         -> Send quote
    - accept()       -> Accept quote
    - reject()       -> Reject quote
[ ] ✅ Commit: "feat(quote): implement QuoteController"

[ ] Add routes to routes/api.php:

    Route::middleware('auth:api')->prefix('quotes')->group(function () {
        Route::post('/', [QuoteController::class, 'store']);
        Route::get('/', [QuoteController::class, 'index']);
        Route::get('/{id}', [QuoteController::class, 'show']);
        Route::post('/{id}/line-items', [QuoteController::class, 'addLineItem']);
        Route::delete('/{id}/line-items/{itemId}', [QuoteController::class, 'removeLineItem']);
        Route::post('/{id}/send', [QuoteController::class, 'send']);
        Route::post('/{id}/accept', [QuoteController::class, 'accept']);
        Route::post('/{id}/reject', [QuoteController::class, 'reject']);
    });

[ ] ✅ Commit: "feat(quote): add API routes"
```

#### EOD Checklist

```
[ ] All endpoints defined
[ ] Request validation working
[ ] Resources format responses correctly
```

**Expected Commits:** 4-5

---

### Day 19 (Friday): API Feature Tests

**Goal:** Full API integration testing

#### Morning (4h)

```
[ ] 🔴 Create tests/Feature/Api/QuoteControllerTest.php
    - test_can_create_quote_via_api
    - test_returns_201_with_quote_id
    - test_requires_authentication
    - test_validates_required_fields
    - test_can_get_quote_by_id
    - test_can_list_quotes
    - test_list_only_shows_own_organization_quotes
[ ] 🟢 Fix any issues found
[ ] ✅ Commit: "test: add Quote API feature tests - creation & listing"
```

#### Afternoon (4h)

```
[ ] 🔴 Continue API tests:
    - test_can_add_line_item_via_api
    - test_can_remove_line_item_via_api
    - test_can_send_quote_via_api
    - test_cannot_send_empty_quote
    - test_can_accept_quote_via_api
    - test_cannot_accept_expired_quote
[ ] 🟢 Fix any issues
[ ] ✅ Commit: "test: add Quote API feature tests - actions"
```

#### EOD Checklist

```
[ ] All API endpoints tested
[ ] Authentication working
[ ] Validation working
[ ] Happy path and error cases covered
```

**Expected Commits:** 2-3

---

### Day 20 (Saturday - Optional): Architecture Tests & Polish

**Goal:** Ensure DDD rules are enforced, polish code

#### Morning (3h)

```
[ ] 🔴 Create tests/Architecture/DomainLayerTest.php

    test('domain layer does not depend on infrastructure')
        ->expect('App\Domains\QuoteManagement\Domain')
        ->not->toUse([
            'Illuminate\Database',
            'Illuminate\Http',
            'Illuminate\Support\Facades',
        ]);
    
    test('value objects are final')
        ->expect('App\Domains\QuoteManagement\Domain\ValueObjects')
        ->toBeFinal();
    
    test('domain events implement interface')
        ->expect('App\Domains\QuoteManagement\Domain\Events')
        ->toImplement('App\SharedKernel\Domain\DomainEvent');

[ ] 🟢 Fix any violations
[ ] ✅ Commit: "test: add architecture tests for DDD enforcement"
```

#### Afternoon (3h)

```
[ ] Run full test suite with coverage:
    ./vendor/bin/pest --coverage --min=80
[ ] Run static analysis:
    ./vendor/bin/phpstan analyse --level=8
[ ] Fix any issues
[ ] ✅ Commit: "fix: address code quality issues"

[ ] Document public methods with PHPDoc
[ ] ✅ Commit: "docs: add PHPDoc to public API"
```

#### EOD Checklist

```
[ ] Architecture tests passing
[ ] 80%+ code coverage
[ ] PHPStan level 8 passing
[ ] Code documented
```

**Expected Commits:** 3-4

---

### Day 21 (Sunday - Optional): Documentation & Demo

**Goal:** Create documentation and verify everything works

#### Morning (3h)

```
[ ] Update README.md with:
    - Setup instructions
    - Running tests
    - API documentation
    - Architecture overview
[ ] ✅ Commit: "docs: update README with setup and usage"

[ ] Create Postman collection or update OpenAPI:
    - All endpoints documented
    - Example requests/responses
[ ] ✅ Commit: "docs: add API documentation"
```

#### Afternoon (3h)

```
[ ] Create database/seeders/QuoteSeeder.php
[ ] ✅ Commit: "feat: add quote seed data for demos"

[ ] Manual Demo - Full Happy Path:
    1. Start server: php artisan serve
    2. Login and get token
    3. Create quote
    4. Add line items
    5. Send quote
    6. Accept quote
    7. Verify events dispatched
    8. Check database state

[ ] Update implementation progress in buildflow-docs
```

#### EOD Checklist

```
[ ] Demo scenario works end-to-end
[ ] Documentation complete
[ ] Seed data creates realistic quotes
[ ] Phase 1 SUCCESS CRITERIA met
```

**Expected Commits:** 3-4

---

## Summary Statistics

| Metric              | Target | Checkpoint |
|---------------------|--------|------------|
| Unit Tests          | 45+    | Day 13     |
| Integration Tests   | 10+    | Day 17     |
| Feature Tests       | 10+    | Day 19     |
| Architecture Tests  | 3+     | Day 20     |
| Code Coverage       | 80%+   | Day 20     |
| PHPStan Level       | 8      | Day 20     |
| Total Commits       | 40-50  | Day 21     |

---

## Risk Mitigation

### If Behind Schedule

- Day 13-14: Can be combined (aggregate completion)
- Day 20-21: Can be compressed (polish & docs)
- Weekend days: Optional, use as catch-up time

### Common Blockers

- Aggregate reconstruction: Allow extra time on Day 17
- Multi-tenancy issues: Test with scopes early
- Event dispatching: Debug with Event::fake()

### Escalation

- If >2 days behind: Reduce scope to core 4 endpoints
- If tests failing: Pause features to fix

---

## Phase 1 Completion Checklist

Before marking Phase 1 complete:

```
[ ] All 60+ tests passing
[ ] 80%+ code coverage (90% domain)
[ ] PHPStan level 8 clean
[ ] Architecture tests passing
[ ] All 7 API endpoints working
[ ] Demo scenario successful
[ ] README updated
[ ] Code reviewed for DDD compliance
```

**After completion:** Move to Phase 2 (Event-Driven Architecture)

---

## Quick Reference: File Locations

### Domain Layer (Pure PHP - NO Laravel)
```
app/Domains/QuoteManagement/Domain/
├── Quote.php                           # Aggregate Root
├── QuoteRepository.php                 # Interface
├── ValueObjects/
│   ├── QuoteId.php
│   ├── QuoteNumber.php
│   ├── QuoteStatus.php
│   ├── Money.php
│   ├── Currency.php
│   ├── TaxRate.php
│   ├── Discount.php
│   ├── ClientId.php
│   └── OrganizationId.php
├── Entities/
│   ├── LineItem.php
│   └── LineItemCollection.php
├── Events/
│   ├── QuoteDraftCreated.php
│   ├── LineItemAdded.php
│   ├── LineItemRemoved.php
│   ├── QuoteSent.php
│   ├── QuoteAccepted.php
│   └── QuoteRejected.php
└── Exceptions/
    ├── InvalidQuoteIdException.php
    ├── CannotSendEmptyQuote.php
    ├── CannotModifySentQuote.php
    ├── QuoteAlreadySentException.php
    ├── CanOnlyAcceptSentQuote.php
    ├── QuoteHasExpired.php
    ├── NegativeMoneyException.php
    └── CurrencyMismatchException.php
```

### Application Layer (Use Cases)
```
app/Domains/QuoteManagement/Application/
├── Commands/
│   ├── CreateQuoteDraft.php
│   ├── AddLineItem.php
│   ├── RemoveLineItem.php
│   ├── SendQuote.php
│   ├── AcceptQuote.php
│   └── RejectQuote.php
├── Handlers/
│   ├── CreateQuoteDraftHandler.php
│   ├── AddLineItemHandler.php
│   ├── RemoveLineItemHandler.php
│   ├── SendQuoteHandler.php
│   ├── AcceptQuoteHandler.php
│   └── RejectQuoteHandler.php
└── Queries/
    └── QuoteQueryService.php
```

### Infrastructure Layer (Laravel)
```
app/Domains/QuoteManagement/Infrastructure/
├── Persistence/
│   ├── EloquentQuoteRepository.php
│   ├── QuoteEloquentModel.php
│   └── LineItemEloquentModel.php
└── Http/
    ├── Controllers/
    │   └── QuoteController.php
    ├── Requests/
    │   ├── CreateQuoteRequest.php
    │   └── AddLineItemRequest.php
    └── Resources/
        ├── QuoteResource.php
        └── LineItemResource.php
```

### Tests
```
tests/
├── Unit/Domains/QuoteManagement/
│   ├── QuoteTest.php
│   ├── ValueObjects/
│   ├── Entities/
│   └── Events/
├── Feature/Domains/QuoteManagement/
│   ├── CreateQuoteDraftTest.php
│   ├── AddLineItemTest.php
│   ├── SendQuoteTest.php
│   └── AcceptQuoteTest.php
├── Integration/Api/
│   └── QuoteControllerTest.php
└── Architecture/
    └── DomainLayerTest.php
```

---

**Good luck! Follow the TDD discipline and you'll have an enterprise-grade Quote aggregate!**
````
