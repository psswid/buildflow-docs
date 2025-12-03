# BuildFlow Laravel API - Getting Started with Claude Code

## 📋 Prerequisites Checklist

Before starting, ensure you have:

- [x] ✅ `buildflow-docs` repo created and published
- [x] ✅ OpenAPI contract defined in `api-contract/openapi.yaml`
- [ ] ⏳ PHP 8.2+ installed
- [ ] ⏳ Composer 2+ installed
- [ ] ⏳ MySQL 8+ or PostgreSQL 14+ installed
- [ ] ⏳ Node.js 18+ (for frontend later)
- [ ] ⏳ Claude Code installed and configured

## 🎯 Phase 0: Repository Setup (Day 1)

### Step 1: Create Laravel Repository

```bash
# Create new Laravel project
composer create-project laravel/laravel buildflow-laravel-api
cd buildflow-laravel-api

# Initialize git
git init
git add .
git commit -m "Initial Laravel setup"

# Create GitHub repo and push
gh repo create buildflow-laravel-api --public --source=. --remote=origin
git push -u origin main
```

### Step 2: Link to Documentation

Create comprehensive README.md:

```bash
# Open in Claude Code
code .
```

**Task for Claude Code:**
```
Create a README.md for buildflow-laravel-api that:
1. Links to the API contract: https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml
2. Explains this is a Laravel implementation of BuildFlow
3. Includes setup instructions
4. Links to main documentation
5. Lists tech stack
6. Shows API compliance status
```

### Step 3: Setup Development Environment

**Create `.env` file:**

```bash
# In Claude Code, open .env and configure:
APP_NAME=BuildFlow
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=buildflow
DB_USERNAME=root
DB_PASSWORD=

# API Settings
API_VERSION=v1
API_PREFIX=/api/v1

# JWT Settings (for authentication)
JWT_SECRET=your-secret-key-here
```

### Step 4: Install Dependencies

```bash
# Install additional packages
composer require tymon/jwt-auth
composer require spatie/laravel-permission
composer require barryvdh/laravel-cors
composer require --dev barryvdh/laravel-ide-helper
composer require --dev laravel/pint

# Generate IDE helper (for better autocomplete)
php artisan ide-helper:generate
php artisan ide-helper:models
```

## 🗄️ Phase 0: Database Setup (Day 1-2)

### Step 5: Create Database Migrations

**Task for Claude Code:**

Open Claude Code and use this prompt:

```
I need to create database migrations for BuildFlow based on the data model in:
https://github.com/psswid/buildflow-docs/blob/main/docs/business/requirements.md (Section 8)

Create migrations in this order (respecting foreign key dependencies):
1. organizations table
2. users table
3. clients table
4. quotes table
5. quote_line_items table
6. projects table
7. invoices table
8. invoice_line_items table
9. payments table
10. documents table
11. notes table
12. activity_logs table

For each migration:
- Use proper data types
- Add indexes for foreign keys and commonly queried fields
- Add timestamps (created_at, updated_at)
- Add soft deletes where appropriate
- Follow Laravel naming conventions
```

**Example Migration Structure:**

```php
// database/migrations/2024_01_01_000001_create_organizations_table.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('organizations', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email');
            $table->string('phone')->nullable();
            $table->text('address')->nullable();
            $table->string('logo')->nullable();
            $table->enum('subscription_tier', ['starter', 'pro', 'business'])->default('starter');
            $table->enum('subscription_status', ['active', 'cancelled', 'past_due'])->default('active');
            $table->json('settings')->nullable();
            $table->timestamps();
            
            $table->index('email');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('organizations');
    }
};
```

### Step 6: Create Eloquent Models

**Task for Claude Code:**

```
Create Eloquent models for all entities with:
1. Proper relationships (hasMany, belongsTo, etc.)
2. Fillable fields
3. Casts for JSON and date fields
4. Accessors/Mutators where needed
5. Global scopes for multi-tenancy (organization_id)
6. Model events for activity logging

Start with:
- app/Models/Organization.php
- app/Models/User.php
- app/Models/Client.php
- app/Models/Quote.php
- app/Models/QuoteLineItem.php
```

**Example Model:**

```php
// app/Models/Client.php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;

class Client extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'organization_id',
        'name',
        'company_name',
        'email',
        'phone',
        'mobile_phone',
        'address',
        'tags',
        'source',
        'portal_enabled',
        'notes',
        'status',
    ];

    protected $casts = [
        'tags' => 'array',
        'portal_enabled' => 'boolean',
        'portal_last_login' => 'datetime',
    ];

    // Relationships
    public function organization(): BelongsTo
    {
        return $this->belongsTo(Organization::class);
    }

    public function quotes(): HasMany
    {
        return $this->hasMany(Quote::class);
    }

    public function projects(): HasMany
    {
        return $this->hasMany(Project::class);
    }

    public function invoices(): HasMany
    {
        return $this->hasMany(Invoice::class);
    }

    // Scopes
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }

    // Global scope for multi-tenancy
    protected static function booted()
    {
        static::addGlobalScope('organization', function ($query) {
            if (auth()->check()) {
                $query->where('organization_id', auth()->user()->organization_id);
            }
        });
    }
}
```

### Step 7: Create Seeders

**Task for Claude Code:**

```
Create database seeders for development:
1. OrganizationSeeder - create test organization
2. UserSeeder - create admin user
3. ClientSeeder - create 10 sample clients
4. QuoteSeeder - create 5 sample quotes
5. ProjectSeeder - create 3 sample projects

Use factories for realistic fake data.
```

## 🔐 Phase 0: Authentication (Day 2-3)

### Step 8: Setup JWT Authentication

**Task for Claude Code:**

```
Setup JWT authentication using tymon/jwt-auth:

1. Configure JWT in config/jwt.php
2. Create AuthController with:
   - register(Request $request)
   - login(Request $request)
   - logout()
   - refresh()
   - me()
3. Create auth middleware
4. Add auth routes to routes/api.php
5. Follow the OpenAPI spec for request/response format

Reference: https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml
Look for /auth/register, /auth/login endpoints
```

## 📝 Phase 1: First Endpoint - Clients (Day 3-4)

### Step 9: Implement Clients API

**Task for Claude Code:**

```
Implement the Clients API following the OpenAPI contract:
https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml

Create:
1. app/Http/Controllers/Api/ClientController.php with:
   - index() - GET /api/v1/clients
   - store(Request $request) - POST /api/v1/clients
   - show(Client $client) - GET /api/v1/clients/{id}
   - update(Request $request, Client $client) - PUT /api/v1/clients/{id}
   - destroy(Client $client) - DELETE /api/v1/clients/{id}

2. app/Http/Requests/StoreClientRequest.php - validation rules
3. app/Http/Requests/UpdateClientRequest.php - validation rules
4. app/Http/Resources/ClientResource.php - JSON transformation
5. app/Http/Resources/ClientCollection.php - collection with pagination

All responses MUST match the OpenAPI schema exactly.
```

**Example Controller Structure:**

```php
// app/Http/Controllers/Api/ClientController.php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StoreClientRequest;
use App\Http\Requests\UpdateClientRequest;
use App\Http\Resources\ClientResource;
use App\Models\Client;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

class ClientController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:api');
    }

    public function index(): AnonymousResourceCollection
    {
        $clients = Client::query()
            ->when(request('search'), function ($query, $search) {
                $query->where('name', 'like', "%{$search}%")
                      ->orWhere('email', 'like', "%{$search}%");
            })
            ->when(request('status'), function ($query, $status) {
                $query->where('status', $status);
            })
            ->orderBy('created_at', 'desc')
            ->paginate(request('per_page', 20));

        return ClientResource::collection($clients);
    }

    public function store(StoreClientRequest $request): JsonResponse
    {
        $client = Client::create([
            'organization_id' => auth()->user()->organization_id,
            ...$request->validated()
        ]);

        return (new ClientResource($client))
            ->response()
            ->setStatusCode(201);
    }

    public function show(Client $client): ClientResource
    {
        $this->authorize('view', $client);
        
        return new ClientResource($client->load(['quotes', 'projects', 'invoices']));
    }

    public function update(UpdateClientRequest $request, Client $client): ClientResource
    {
        $this->authorize('update', $client);
        
        $client->update($request->validated());

        return new ClientResource($client);
    }

    public function destroy(Client $client): JsonResponse
    {
        $this->authorize('delete', $client);
        
        // Check if client has active projects
        if ($client->projects()->whereIn('status', ['not_started', 'in_progress'])->exists()) {
            return response()->json([
                'message' => 'Cannot delete client with active projects'
            ], 422);
        }

        $client->delete();

        return response()->json(null, 204);
    }
}
```

### Step 10: API Routes

**Task for Claude Code:**

```
Update routes/api.php to include client routes following REST conventions:

GET    /api/v1/clients          -> index
POST   /api/v1/clients          -> store
GET    /api/v1/clients/{id}     -> show
PUT    /api/v1/clients/{id}     -> update
DELETE /api/v1/clients/{id}     -> destroy

Use route resource: Route::apiResource('clients', ClientController::class);
Wrap in auth:api middleware
Add API versioning prefix
```

## 🧪 Phase 1: Contract Testing (Day 4-5)

### Step 11: Setup Contract Testing

**Task for Claude Code:**

```
Create contract tests to verify API compliance with OpenAPI spec:

1. Install spectator for OpenAPI testing:
   composer require --dev spectator/spectator

2. Copy OpenAPI spec to tests/api-contract/openapi.yaml
   (from buildflow-docs repo)

3. Create tests/Feature/Contract/ClientContractTest.php with tests for:
   - test_list_clients_matches_contract()
   - test_create_client_matches_contract()
   - test_show_client_matches_contract()
   - test_update_client_matches_contract()
   - test_delete_client_matches_contract()

Each test should:
- Make API request
- Assert status code
- Validate response against OpenAPI schema using Spectator
```

**Example Contract Test:**

```php
// tests/Feature/Contract/ClientContractTest.php
<?php

namespace Tests\Feature\Contract;

use App\Models\Client;
use App\Models\Organization;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Spectator\Spectator;
use Tests\TestCase;

class ClientContractTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();
        
        // Load OpenAPI spec
        Spectator::using('tests/api-contract/openapi.yaml');
        
        // Create test organization and user
        $organization = Organization::factory()->create();
        $this->user = User::factory()->create([
            'organization_id' => $organization->id
        ]);
    }

    public function test_list_clients_matches_contract(): void
    {
        Client::factory()->count(5)->create([
            'organization_id' => $this->user->organization_id
        ]);

        $response = $this->actingAs($this->user, 'api')
            ->getJson('/api/v1/clients');

        $response->assertStatus(200)
            ->assertValidRequest()
            ->assertValidResponse();
    }

    public function test_create_client_matches_contract(): void
    {
        $clientData = [
            'name' => 'Test Client',
            'email' => 'test@example.com',
            'phone' => '+48 123 456 789',
            'address' => 'Test Address'
        ];

        $response = $this->actingAs($this->user, 'api')
            ->postJson('/api/v1/clients', $clientData);

        $response->assertStatus(201)
            ->assertValidRequest()
            ->assertValidResponse();
    }

    // More contract tests...
}
```

## 🚀 Working with Claude Code - Best Practices

### How to Use Claude Code Effectively

#### 1. **Project Context**

When opening Claude Code, always provide context:

```
I'm building buildflow-laravel-api, a Laravel implementation of the BuildFlow API.

Key references:
- API Contract: https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml
- Business Requirements: https://github.com/psswid/buildflow-docs/blob/main/docs/business/requirements.md
- Data Model: Section 8 of business requirements

Current task: [describe what you're working on]
```

#### 2. **Specific Prompts for Claude Code**

**Good Prompt:**
```
Create a ClientController that implements the /clients endpoints from our OpenAPI spec.
Requirements:
- Follow Laravel best practices
- Use resource classes for JSON responses
- Implement proper validation
- Add authorization checks
- Include pagination on index
- Match OpenAPI response schemas exactly

Reference OpenAPI spec at: [link to openapi.yaml]
```

**Bad Prompt:**
```
Make a controller for clients
```

#### 3. **Iterative Development**

Work in small iterations:

```
Step 1: Create Client model with relationships
[wait for output, review]

Step 2: Create migration for clients table
[wait for output, review]

Step 3: Create ClientController with index and store methods
[wait for output, review]

Step 4: Create contract tests for those endpoints
[wait for output, test]
```

#### 4. **Testing Workflow**

After each feature:

```bash
# Run all tests
php artisan test

# Run specific test suite
php artisan test --testsuite=Feature

# Run contract tests only
php artisan test tests/Feature/Contract

# Check code style
./vendor/bin/pint

# Static analysis
./vendor/bin/phpstan analyse
```

## 📋 Daily Workflow with Claude Code

### Day 1: Setup
- [ ] Create Laravel project
- [ ] Setup database
- [ ] Create migrations
- [ ] Create models
- [ ] Run migrations and seeders

### Day 2-3: Authentication
- [ ] Setup JWT
- [ ] Create auth endpoints
- [ ] Test authentication flow
- [ ] Create auth contract tests

### Day 4-5: Clients Module
- [ ] Create ClientController
- [ ] Create request validation
- [ ] Create resource classes
- [ ] Test manually with Postman
- [ ] Write contract tests
- [ ] Document any deviations from spec

### Day 6-7: Quotes Module
- [ ] Create Quote model and migration
- [ ] Create QuoteController
- [ ] Implement line items logic
- [ ] Test quote creation
- [ ] Contract tests

### Week 2: Continue with other modules
- Projects
- Invoices
- Documents
- Dashboard

## 🔧 Useful Commands

```bash
# Create migration
php artisan make:migration create_clients_table

# Create model with migration, factory, seeder, controller
php artisan make:model Client -mfsc

# Create API resource
php artisan make:resource ClientResource

# Create request validation
php artisan make:request StoreClientRequest

# Create controller
php artisan make:controller Api/ClientController --api

# Create test
php artisan make:test ClientContractTest

# Run specific test
php artisan test --filter=ClientContractTest

# Clear cache
php artisan optimize:clear

# Generate API documentation
php artisan l5-swagger:generate
```

## 📚 Claude Code Prompt Templates

### Template 1: Creating a New Module

```
I need to implement the [MODULE_NAME] module following our OpenAPI contract.

Context:
- API Contract: https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml
- Section: [SECTION_NAME] endpoints
- Data Model: [LINK_TO_DATA_MODEL]

Create:
1. Migration for [table_name] with all required fields
2. Eloquent model with relationships
3. Controller with CRUD operations
4. Request validation classes
5. Resource classes for JSON responses
6. Contract tests

All responses must match OpenAPI schema exactly.
```

### Template 2: Debugging Issues

```
I'm getting this error: [ERROR_MESSAGE]

Context:
- Working on: [FEATURE]
- File: [FILE_PATH]
- Expected: [EXPECTED_BEHAVIOR]
- Actual: [ACTUAL_BEHAVIOR]

Stack trace:
[PASTE_STACK_TRACE]

Help me:
1. Understand what's causing this
2. Fix the issue
3. Add tests to prevent it
```

### Template 3: Code Review

```
Review this code for:
- Laravel best practices
- Security issues
- Performance concerns
- OpenAPI contract compliance
- Test coverage

[PASTE_CODE]
```

## 🎯 Next Steps After Phase 0

Once you complete Phase 0 (Foundation), you'll have:
- ✅ Laravel project setup
- ✅ Database schema
- ✅ Authentication working
- ✅ First API module (Clients) complete
- ✅ Contract tests passing

Then proceed to:
1. **Phase 1 remaining modules:** Quotes, Projects, Invoices
2. **Frontend setup:** Create buildflow-react-web
3. **Integration testing:** Connect React to Laravel API
4. **Symfony implementation:** Start buildflow-symfony-api

## 💡 Pro Tips

1. **Commit Often:** After each working feature
2. **Test First:** Write contract tests before or alongside code
3. **Document Deviations:** If you need to deviate from OpenAPI spec, document why
4. **Use Postman:** Import OpenAPI spec to Postman for manual testing
5. **Keep Docs Updated:** Update buildflow-docs if you discover issues with spec

## 🆘 Getting Help

If stuck:
1. Check buildflow-docs for business logic
2. Check OpenAPI spec for API contracts
3. Ask Claude Code with specific context
4. Open issue in buildflow-docs repo
5. Check Laravel documentation

## 📞 Quick Links

- **Repo:** https://github.com/psswid/buildflow-laravel-api
- **Docs:** https://github.com/psswid/buildflow-docs
- **API Contract:** https://github.com/psswid/buildflow-docs/blob/main/api-contract/openapi.yaml
- **Business Reqs:** https://github.com/psswid/buildflow-docs/blob/main/docs/business/requirements.md
- **Roadmap:** https://github.com/psswid/buildflow-docs/blob/main/docs/roadmap/github-roadmap.md

---

**Ready to start? Open Claude Code and use the prompts above! 🚀**

**Last Updated:** November 12, 2025
