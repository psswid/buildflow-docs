# Contributing to BuildFlow

Thank you for considering contributing to BuildFlow! This document provides guidelines for contributing to the project.

---

## 🎯 Project Focus

BuildFlow is a **portfolio project** demonstrating **enterprise-grade Laravel architecture** with Domain-Driven Design, Event-Driven Architecture, and CQRS patterns.

**Priority:** One framework (Laravel) implemented deeply with production-ready patterns.

---

## 📚 Before You Start

### Required Reading

Please read these documents before contributing:

1. **[Project Overview](https://github.com/psswid/buildflow-docs/blob/main/PROJECT_OVERVIEW.md)** - Understand the vision
2. **[Architecture](ARCHITECTURE.md)** - Understand the structure
3. **[Domain Analysis](https://github.com/psswid/buildflow-docs/blob/main/DOMAIN_ANALYSIS_EVENT_STORMING.md)** - Understand the domain
4. **Relevant ADRs** - Understand architectural decisions:
   - [ADR-011: Domain-Driven Design](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/011-domain-driven-design.md)
   - [ADR-012: Event-Driven Architecture](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/012-event-driven-architecture.md)
   - [ADR-013: CQRS Basic](https://github.com/psswid/buildflow-docs/blob/main/docs/architecture/decisions/013-cqrs-basic.md)

### Prerequisites

- PHP 8.3+
- PostgreSQL 14+
- Composer
- Understanding of DDD concepts (Aggregates, Value Objects, Domain Events)
- Understanding of Event-Driven Architecture
- Familiarity with CQRS pattern

---

## 🛠️ Development Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub
git clone https://github.com/YOUR_USERNAME/buildflow-laravel-api.git
cd buildflow-laravel-api

# Add upstream remote
git remote add upstream https://github.com/psswid/buildflow-laravel-api.git
```

### 2. Install Dependencies

```bash
composer install
npm install

cp .env.example .env
php artisan key:generate
```

### 3. Configure Database

```bash
# Create database
createdb buildflow_dev

# Update .env
DB_CONNECTION=pgsql
DB_DATABASE=buildflow_dev
DB_USERNAME=your_username
DB_PASSWORD=your_password

# Run migrations
php artisan migrate
php artisan db:seed
```

### 4. Run Tests

```bash
# All tests should pass
./vendor/bin/pest

# Check coverage
./vendor/bin/pest --coverage --min=80
```

### 5. Start Development Server

```bash
php artisan serve
```

---

## 📋 Contribution Guidelines

### Code Organization

BuildFlow follows **modular monolith** architecture with strict layer separation:

```
app/Domains/[Context]/
├── Domain/           # Pure business logic (NO Laravel dependencies)
├── Application/      # Use cases (Commands, Handlers, Queries)
└── Infrastructure/   # Technical details (Laravel, DB, HTTP)
```

**Critical Rules:**
- ✅ Domain layer must be pure PHP (no Laravel imports)
- ✅ Business rules go in Aggregates, not Controllers
- ✅ Use Value Objects for immutable concepts
- ✅ Communicate between contexts via Domain Events
- ✅ Test EVERYTHING (especially domain logic)

### Layer Responsibilities

#### Domain Layer (Pure PHP)
```php
// ✅ Good - Pure business logic
class Quote extends AggregateRoot
{
    public function accept(): void
    {
        if (!$this->status->isSent()) {
            throw new CanOnlyAcceptSentQuote();
        }
        
        $this->status = QuoteStatus::accepted();
        $this->record(new QuoteAccepted($this->id));
    }
}

// ❌ Bad - Laravel dependency in domain
use Illuminate\Support\Facades\DB;  // NO!

class Quote extends Model  // NO! Use AggregateRoot
{
    public function accept()
    {
        DB::transaction(function () {  // NO! Transactions are infrastructure concern
            $this->status = 'accepted';
            $this->save();
        });
    }
}
```

#### Application Layer (Use Cases)
```php
// ✅ Good - Orchestration
class AcceptQuoteHandler
{
    public function handle(AcceptQuote $command): void
    {
        $quote = $this->quotes->findById($command->quoteId);
        
        $quote->accept();  // Business logic in domain
        
        $this->quotes->save($quote);  // Infrastructure handles persistence
    }
}

// ❌ Bad - Business logic in handler
class AcceptQuoteHandler
{
    public function handle(AcceptQuote $command): void
    {
        $quote = Quote::find($command->quoteId);
        
        if ($quote->status !== 'sent') {  // NO! This is business logic
            throw new Exception('Can only accept sent quotes');
        }
        
        $quote->status = 'accepted';
        $quote->save();
    }
}
```

#### Infrastructure Layer (Technical)
```php
// ✅ Good - Laravel concerns here
class EloquentQuoteRepository implements QuoteRepository
{
    public function save(Quote $quote): void
    {
        DB::transaction(function () use ($quote) {
            QuoteEloquentModel::updateOrCreate(
                ['id' => $quote->id()->toString()],
                $this->mapToDatabase($quote)
            );
            
            foreach ($quote->releaseEvents() as $event) {
                Event::dispatch($event);
            }
        });
    }
}
```

---

## 🧪 Testing Requirements

### Test Coverage

| Layer | Minimum Coverage | Priority |
|-------|-----------------|----------|
| Domain | 90% | Critical |
| Application | 80% | High |
| Infrastructure | 60% | Medium |

### Test Types

#### 1. Unit Tests (Domain)
```php
// tests/Unit/Domains/QuoteManagement/QuoteTest.php
test('cannot send empty quote', function () {
    $quote = Quote::createDraft(/* ... */);
    
    expect(fn() => $quote->send())
        ->toThrow(CannotSendEmptyQuote::class);
});
```

#### 2. Integration Tests (Application)
```php
// tests/Integration/Domains/QuoteManagement/AcceptQuoteFlowTest.php
test('accepting quote creates project', function () {
    Event::fake([QuoteAccepted::class, ProjectCreated::class]);
    
    $handler->handle(new AcceptQuote($quoteId));
    
    Event::assertDispatched(QuoteAccepted::class);
    Event::assertDispatched(ProjectCreated::class);
});
```

#### 3. Feature Tests (HTTP)
```php
// tests/Feature/Api/QuoteApiTest.php
test('can accept quote via API', function () {
    $response = $this->actingAs($user, 'api')
        ->postJson("/api/quotes/{$quote->id}/accept");
    
    $response->assertStatus(200);
});
```

#### 4. Architecture Tests
```php
// tests/Architecture/DomainLayerTest.php
test('domain does not depend on infrastructure', function () {
    expect('App\Domains\QuoteManagement\Domain')
        ->not->toUse([
            'Illuminate\Database',
            'Illuminate\Http',
        ]);
});
```

### Running Tests

```bash
# Run all tests
./vendor/bin/pest

# Run specific suite
./vendor/bin/pest tests/Unit
./vendor/bin/pest tests/Feature

# With coverage
./vendor/bin/pest --coverage --min=80

# Architecture tests (must pass!)
./vendor/bin/pest tests/Architecture
```

---

## 📝 Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code refactoring
- `test` - Adding tests
- `docs` - Documentation changes
- `style` - Code style changes (formatting)
- `chore` - Maintenance tasks

**Examples:**
```
feat(quote): implement Quote aggregate with DDD patterns

- Add Quote aggregate with business rules
- Add Value Objects (QuoteId, Money, QuoteStatus)
- Add Domain Events (QuoteDraftCreated, QuoteAccepted)
- Add comprehensive unit tests (95% coverage)

Closes #42
```

```
test(quote): add integration tests for accept quote flow

- Test event dispatching
- Test cross-context communication (Quote → Project)
- Test transaction rollback on failure

Related to #42
```

---

## 🔄 Pull Request Process

### 1. Create Feature Branch

```bash
git checkout -b feature/quote-management
```

### 2. Make Changes

Follow coding standards and layer responsibilities.

### 3. Write Tests

```bash
# Tests must pass
./vendor/bin/pest

# Coverage must be >= 80%
./vendor/bin/pest --coverage --min=80
```

### 4. Update Documentation

If you:
- Add new feature → Update README.md
- Make architectural decision → Create ADR
- Change domain model → Update DOMAIN_ANALYSIS_EVENT_STORMING.md

### 5. Push and Create PR

```bash
git push origin feature/quote-management
```

Create PR on GitHub with:
- Clear description
- Reference to issue (#42)
- Screenshots (if UI changes)
- Testing checklist

### 6. PR Review Checklist

Reviewers will check:

**Architecture:**
- [ ] Follows DDD patterns
- [ ] Domain layer is pure PHP
- [ ] Events used for cross-context communication
- [ ] Proper layer separation

**Code Quality:**
- [ ] Follows PSR-12 coding standards
- [ ] No code duplication
- [ ] Clear naming conventions
- [ ] Adequate comments for complex logic

**Testing:**
- [ ] All tests pass
- [ ] New code is tested
- [ ] Coverage >= 80%
- [ ] Architecture tests pass

**Documentation:**
- [ ] README updated (if needed)
- [ ] ADR created (if architectural change)
- [ ] Code comments for complex logic

---

## 🎨 Code Style

### PSR-12 Standards

```bash
# Check style
./vendor/bin/phpcs

# Auto-fix
./vendor/bin/phpcbf
```

### Naming Conventions

**Aggregates:**
```php
class Quote extends AggregateRoot  // Singular noun
```

**Value Objects:**
```php
final class QuoteId extends Identifier  // Final, specific
final class Money extends ValueObject
```

**Domain Events:**
```php
final class QuoteAccepted implements DomainEvent  // Past tense
final class ProjectCreated implements DomainEvent
```

**Commands:**
```php
final class AcceptQuote  // Imperative, what to do
final class CreateQuoteDraft
```

**Handlers:**
```php
class AcceptQuoteHandler  // Command + Handler suffix
```

**Repositories:**
```php
interface QuoteRepository  // Interface in Domain
class EloquentQuoteRepository implements QuoteRepository  // Implementation in Infrastructure
```

---

## 🐛 Bug Reports

### Creating a Bug Report

Use the [Bug Report Template](.github/ISSUE_TEMPLATE/bug_report.md):

**Include:**
1. Description of the bug
2. Steps to reproduce
3. Expected behavior
4. Actual behavior
5. Environment details (PHP version, PostgreSQL version)
6. Stack trace (if applicable)

---

## 💡 Feature Requests

### Creating a Feature Request

Use the [Feature Request Template](.github/ISSUE_TEMPLATE/feature_request.md):

**Include:**
1. Description of the feature
2. Business value (why is it needed?)
3. Proposed implementation (if you have ideas)
4. Related domain context (Quote, Project, etc.)
5. Impact on architecture

**Important:** Check [GitHub Roadmap](https://github.com/psswid/buildflow-docs/blob/main/BuildFlow_GitHub_Roadmap.md) first to see if feature is already planned.

---

## 📖 Learning Resources

### Domain-Driven Design
- [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)
- [Implementing Domain-Driven Design by Vaughn Vernon](https://vaughnvernon.com/)
- [Laravel Beyond CRUD](https://laravel-beyond-crud.com/)

### Event-Driven Architecture
- [Domain Events (Martin Fowler)](https://martinfowler.com/eaaDev/DomainEvent.html)
- [Event-Driven Microservices](https://microservices.io/patterns/data/event-driven-architecture.html)

### CQRS
- [CQRS by Martin Fowler](https://martinfowler.com/bliki/CQRS.html)
- [Microsoft CQRS Journey](https://docs.microsoft.com/en-us/previous-versions/msp-n-p/jj554200(v=pandp.10))

---

## ❓ Questions?

- **Architecture Questions:** Check [ADRs](https://github.com/psswid/buildflow-docs/tree/main/docs/architecture/decisions)
- **Domain Questions:** Check [Domain Analysis](https://github.com/psswid/buildflow-docs/blob/main/DOMAIN_ANALYSIS_EVENT_STORMING.md)
- **Implementation Questions:** Check [Implementation Roadmap](https://github.com/psswid/buildflow-docs/blob/main/IMPLEMENTATION_ROADMAP.md)
- **Other Questions:** Open a [Discussion](https://github.com/psswid/buildflow-laravel-api/discussions)

---

## 🏆 Recognition

Contributors will be recognized in:
- README.md Contributors section
- Release notes
- Project documentation

---

## 📜 License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for helping make BuildFlow better! 🚀**
