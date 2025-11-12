# Contributing to BuildFlow

First off, thank you for considering contributing to BuildFlow! It's people like you that make BuildFlow such a great tool for construction professionals.

## 🎯 Code of Conduct

This project and everyone participating in it is governed by our Code of Conduct. By participating, you are expected to uphold this code.

## 🤔 How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When you create a bug report, include as many details as possible:

- **Use a clear and descriptive title**
- **Describe the exact steps to reproduce the problem**
- **Provide specific examples** - Include links, screenshots, or code samples
- **Describe the behavior you observed** and explain what's wrong
- **Explain the behavior you expected** to see instead
- **Include details about your configuration and environment**

### Suggesting Features

Feature suggestions are tracked as GitHub issues. When creating a feature suggestion:

- **Use a clear and descriptive title**
- **Provide a detailed description** of the suggested feature
- **Explain why this feature would be useful** to most BuildFlow users
- **List some examples** of how the feature would be used
- **Specify which tech stack** it applies to (or if it's stack-agnostic)

### Pull Requests

1. **Fork the repo** and create your branch from `main`
2. **Make your changes** following our coding standards
3. **Add tests** if you've added code that should be tested
4. **Ensure the test suite passes**
5. **Update documentation** if needed
6. **Write a clear commit message** following our commit conventions
7. **Submit the pull request**

## 🏗️ Development Setup

### Prerequisites

Choose your stack and follow the appropriate setup:

#### Next.js Stack
```bash
# Requirements
- Node.js 18+
- pnpm 8+
- PostgreSQL 14+

# Setup
cd apps/nextjs
pnpm install
cp .env.example .env
pnpm dev
```

#### Laravel Stack
```bash
# Requirements
- PHP 8.2+
- Composer 2+
- MySQL 8+ or PostgreSQL 14+

# Setup
cd apps/laravel
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate
php artisan serve
```

#### Symfony Stack
```bash
# Requirements
- PHP 8.2+
- Composer 2+
- Symfony CLI
- PostgreSQL 14+

# Setup
cd apps/symfony
composer install
cp .env.example .env.local
symfony console doctrine:migrations:migrate
symfony serve
```

## 📝 Coding Standards

### General Guidelines

- Write clear, readable code
- Add comments for complex logic
- Keep functions small and focused
- Follow DRY (Don't Repeat Yourself)
- Write meaningful variable names

### Stack-Specific Standards

#### Next.js / TypeScript
- Use TypeScript strict mode
- Follow React best practices
- Use Server Components by default
- Implement proper error boundaries
- Follow the Next.js App Router conventions

```typescript
// Good
interface ClientFormProps {
  client?: Client;
  onSubmit: (data: ClientFormData) => Promise<void>;
}

export function ClientForm({ client, onSubmit }: ClientFormProps) {
  // Implementation
}

// Bad
export function ClientForm(props: any) {
  // Implementation
}
```

#### Laravel / PHP
- Follow PSR-12 coding standard
- Use type hints for parameters and return types
- Write Eloquent models with proper relationships
- Use Form Requests for validation
- Follow Laravel naming conventions

```php
// Good
class Client extends Model
{
    protected $fillable = ['name', 'email', 'phone'];
    
    public function quotes(): HasMany
    {
        return $this->hasMany(Quote::class);
    }
}

// Bad
class Client extends Model
{
    // No type hints, no relationships defined
}
```

#### Symfony / PHP
- Follow Symfony best practices
- Use type hints and return types
- Leverage Symfony's dependency injection
- Use attributes for routing and validation
- Follow Symfony directory structure

```php
// Good
#[Route('/clients', name: 'client_list')]
public function list(ClientRepository $repository): Response
{
    $clients = $repository->findAll();
    return $this->render('client/list.html.twig', [
        'clients' => $clients,
    ]);
}

// Bad
public function list()
{
    // No type hints, no return type
}
```

## 🧪 Testing

### Writing Tests

- Write tests for all new features
- Maintain or improve code coverage
- Test edge cases and error conditions
- Use meaningful test names

#### Next.js Testing
```typescript
// Use Jest + React Testing Library
import { render, screen } from '@testing-library/react';
import { ClientForm } from './ClientForm';

describe('ClientForm', () => {
  it('renders form fields correctly', () => {
    render(<ClientForm />);
    expect(screen.getByLabelText('Name')).toBeInTheDocument();
    expect(screen.getByLabelText('Email')).toBeInTheDocument();
  });
});
```

#### Laravel Testing
```php
// Use PHPUnit + Laravel Testing
public function test_can_create_client()
{
    $response = $this->post('/api/clients', [
        'name' => 'Test Client',
        'email' => 'test@example.com',
    ]);

    $response->assertStatus(201);
    $this->assertDatabaseHas('clients', [
        'email' => 'test@example.com',
    ]);
}
```

#### Symfony Testing
```php
// Use PHPUnit + Symfony Testing
public function testClientCreation(): void
{
    $client = static::createClient();
    $client->request('POST', '/api/clients', [], [], [
        'CONTENT_TYPE' => 'application/json',
    ], json_encode([
        'name' => 'Test Client',
        'email' => 'test@example.com',
    ]));

    $this->assertResponseIsSuccessful();
}
```

### Running Tests

```bash
# Next.js
pnpm test

# Laravel
php artisan test

# Symfony
php bin/phpunit
```

## 📋 Commit Message Guidelines

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

### Examples

```bash
feat(quotes): add PDF generation with watermark

Implemented PDF generation for quotes using Puppeteer.
Free tier includes watermark, Pro tier removes it.

Closes #16

---

fix(clients): prevent duplicate email addresses

Added unique constraint on email field and proper
validation error handling.

Fixes #23

---

docs(readme): update setup instructions for Laravel

Added missing steps for database configuration and
migration commands.
```

## 🎨 UI/UX Guidelines

### Design Principles

1. **Mobile First** - Design for mobile, enhance for desktop
2. **Accessibility** - Follow WCAG 2.1 AA standards
3. **Consistency** - Use the shared UI component library
4. **Performance** - Optimize images, lazy load when possible
5. **User Feedback** - Always show loading states and errors

### Component Library

Use the shared component library from `packages/ui`:

```typescript
// Good - using shared components
import { Button, Input, Card } from '@buildflow/ui';

// Bad - creating custom components for common elements
const MyCustomButton = () => { /* ... */ };
```

## 📚 Documentation

### Code Documentation

- Document complex functions with JSDoc/PHPDoc
- Explain "why" not just "what"
- Update README if you change setup process
- Add examples for new features

### User Documentation

If your feature is user-facing:
- Update the user guide
- Add screenshots or GIFs
- Write a tutorial if needed
- Update FAQ if relevant

## 🏷️ Issue and PR Labels

### Priority Labels
- `priority: critical` - Must be fixed immediately
- `priority: high` - Important, should be done soon
- `priority: medium` - Normal priority
- `priority: low` - Nice to have

### Type Labels
- `type: feature` - New feature
- `type: bug` - Bug fix
- `type: docs` - Documentation
- `type: refactor` - Code refactoring

### Status Labels
- `status: ready` - Ready to be worked on
- `status: in-progress` - Being worked on
- `status: needs-review` - Needs code review
- `status: blocked` - Blocked by dependencies

### Other Labels
- `good first issue` - Good for newcomers
- `help wanted` - We need help with this
- `stack: nextjs` - Next.js specific
- `stack: laravel` - Laravel specific
- `stack: symfony` - Symfony specific
- `stack: agnostic` - Technology agnostic

## 🔍 Code Review Process

### For Reviewers

- Be respectful and constructive
- Test the changes locally
- Check for edge cases
- Verify documentation is updated
- Approve or request changes with clear feedback

### For Authors

- Respond to all review comments
- Make requested changes or explain why not
- Re-request review after changes
- Be patient and open to feedback

## 🎯 Good First Issues

New to the project? Look for issues with the `good first issue` label:

- [Client Management - Tags & Categories](https://github.com/yourusername/buildflow/issues/14)
- [Document Management - Document List](https://github.com/yourusername/buildflow/issues/27)
- [Improved Error Messages](https://github.com/yourusername/buildflow/issues/50)

## 🌍 Translation

Help translate BuildFlow to other languages:

1. Check `docs/translations/` for existing translations
2. Copy `en.json` to your language code (e.g., `pl.json`)
3. Translate all strings
4. Submit a PR with your translation
5. Add yourself to `docs/translations/TRANSLATORS.md`

## 💬 Community

- **Discord:** [Join our server](https://discord.gg/buildflow)
- **GitHub Discussions:** Ask questions and share ideas
- **Twitter:** [@buildflow_app](https://twitter.com/buildflow_app)

## 📜 License

By contributing to BuildFlow, you agree that your contributions will be licensed under the MIT License.

## 🙏 Thank You!

Your contributions make BuildFlow better for everyone. Whether you're fixing a typo, adding a feature, or helping with translations, we appreciate your help!

---

**Questions?** Open an issue!