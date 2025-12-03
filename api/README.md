# BuildFlow API Documentation

## Overview

BuildFlow provides a **RESTful API** for managing construction business operations. The API follows an **API-first approach** where the OpenAPI specification serves as the single source of truth.

## 📄 OpenAPI Specification

The complete API contract is defined in:
- **[openapi.yaml](../api-contract/openapi.yaml)** - Full OpenAPI 3.0.3 specification

## 🎯 API-First Development

BuildFlow uses an **API-first methodology** (see [ADR-002](../architecture/decisions/002-api-first-approach.md)):

1. **Design First** - API endpoints designed in OpenAPI spec before implementation
2. **Contract as Truth** - Spec is authoritative, implementations must match
3. **Automated Testing** - Contract tests validate compliance (see [ADR-008](../architecture/decisions/008-contract-testing-strategy.md))
4. **Frontend Independence** - React frontend works with any compliant backend

## 🔐 Authentication

### JWT Bearer Tokens

BuildFlow uses **JWT (JSON Web Tokens)** for authentication (see [ADR-003](../architecture/decisions/003-jwt-authentication.md)):

```bash
# 1. Login to get token
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "contractor@example.com",
    "password": "SecurePassword123!"
  }'

# Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "contractor@example.com",
    "name": "John Doe",
    "role": "owner"
  }
}

# 2. Use token in subsequent requests
curl -X GET http://localhost:8000/api/v1/clients \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Token Lifecycle

- **Expiration:** Tokens expire after configured time (default: 24 hours)
- **Refresh:** Use `/auth/refresh` to get new token before expiration
- **Logout:** Use `/auth/logout` to invalidate token

## 🏢 Multi-Tenancy

BuildFlow uses **row-level multi-tenancy** (see [ADR-004](../architecture/decisions/004-multi-tenancy-row-level.md)):

- **Tenant Isolation:** Each organization's data is completely isolated
- **Automatic Scoping:** Tenant ID from JWT token automatically scopes all queries
- **No Tenant in URLs:** Tenant determined from authentication, not URL parameters
- **Security:** Users can only access their organization's data

## 📡 Base URLs

### Development

```
Laravel: http://localhost:8000/api/v1
```

### Production (TBD)

```
Production: https://api.buildflow.example.com/v1
```

## 📊 API Structure

### Resource Groups

**Authentication** (`/auth/*`)
- `/auth/login` - User login
- `/auth/logout` - User logout  
- `/auth/refresh` - Token refresh

**Clients** (`/clients/*`)
- `/clients` - List, create clients
- `/clients/{id}` - Get, update, delete client

**Quotes** (`/quotes/*`)
- `/quotes` - List, create quotes
- `/quotes/{id}` - Get, update quote
- `/quotes/{id}/send` - Send quote to client
- `/quotes/{id}/accept` - Accept quote (creates project)

**Projects** (`/projects/*`) - TBD
**Invoices** (`/invoices/*`) - TBD
**Documents** (`/documents/*`) - TBD

## 🧪 Testing

### Contract Tests

All API endpoints must pass contract tests that validate responses against the OpenAPI specification:

```php
// Laravel Example - spectator/spectator package
public function test_list_clients_matches_contract(): void
{
    $response = $this->getJson('/api/v1/clients');
    
    $response->assertValidResponse();  // Validates against OpenAPI spec
}
```

**See:** [ADR-008: Contract Testing Strategy](../architecture/decisions/008-contract-testing-strategy.md)

### Manual Testing

**Postman Collection:** Import `openapi.yaml` into Postman for manual testing

**Swagger UI:** (To be added) Interactive API documentation

## 📝 Request/Response Format

### Standard Request Headers

```
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
```

### Standard Response Format

**Success Response:**
```json
{
  "data": { ... },  // or array for lists
  "meta": { ... }   // pagination, etc.
}
```

**Error Response:**
```json
{
  "message": "Error description",
  "errors": {
    "field": ["Validation error message"]
  }
}
```

### HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 401 | Unauthorized | Missing/invalid token |
| 403 | Forbidden | Valid token, insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Validation errors |
| 500 | Internal Server Error | Server error |

## 🔄 Pagination

List endpoints return paginated results:

**Request:**
```bash
GET /api/v1/clients?page=2&per_page=20
```

**Response:**
```json
{
  "data": [ ... ],
  "meta": {
    "current_page": 2,
    "per_page": 20,
    "total": 150,
    "last_page": 8,
    "from": 21,
    "to": 40
  }
}
```

**Parameters:**
- `page` - Page number (default: 1)
- `per_page` - Items per page (default: 20, max: 100)

## 🔍 Filtering & Searching

Many endpoints support filtering and search:

**Clients:**
```bash
# Search by name, email, or phone
GET /api/v1/clients?search=John

# Filter by tag
GET /api/v1/clients?tag=commercial
```

**Quotes:**
```bash
# Filter by status
GET /api/v1/quotes?status=sent

# Filter by client
GET /api/v1/quotes?client_id=123
```

## 📖 Example Workflows

### Create Quote → Send → Accept → Project

```bash
# 1. Create client
POST /api/v1/clients
{
  "name": "John Smith",
  "email": "john@example.com",
  "phone": "+1-555-123-4567"
}
# Response: { "id": 123, ... }

# 2. Create quote
POST /api/v1/quotes
{
  "client_id": 123,
  "title": "Kitchen Renovation",
  "line_items": [
    {
      "description": "Custom cabinets",
      "quantity": 10,
      "unit": "linear feet",
      "unit_price": 150.00
    }
  ]
}
# Response: { "id": 456, "status": "draft", ... }

# 3. Send quote to client
POST /api/v1/quotes/456/send
# Response: { "id": 456, "status": "sent", ... }

# 4. Accept quote (creates project)
POST /api/v1/quotes/456/accept
{
  "signature": "data:image/png;base64,..."
}
# Response: {
#   "quote": { "id": 456, "status": "accepted", ... },
#   "project": { "id": 789, "status": "active", ... }
# }
```

## 🛠️ Development Tools

### Recommended Tools

- **Postman** - API testing and exploration
- **Swagger UI** - Interactive documentation (coming soon)
- **HTTPie** - CLI HTTP client
- **Bruno** - Open-source API client

### Code Generation

The OpenAPI spec can generate:
- **TypeScript types** for frontend
- **API clients** for various languages
- **Mock servers** for frontend development
- **Documentation** automatically

## 🔗 Related Documentation

### Architecture
- [ADR-002: API-First Approach](../architecture/decisions/002-api-first-approach.md) - Why API-first
- [ADR-003: JWT Authentication](../architecture/decisions/003-jwt-authentication.md) - Authentication strategy
- [ADR-004: Multi-Tenancy](../architecture/decisions/004-multi-tenancy-row-level.md) - Data isolation
- [ADR-008: Contract Testing](../architecture/decisions/008-contract-testing-strategy.md) - API validation
- [ADR-010: Frontend-Backend Separation](../architecture/decisions/010-frontend-backend-separation.md) - API design

### Implementation
- [Getting Started Guide](../implementation/getting-started-laravel.md) - Laravel implementation
- [Testing Strategy](../implementation/testing-strategy.md) - Test approach
- [Implementation Roadmap](../implementation/roadmap.md) - Development plan

### Domain
- [Event Storming](../domain/event-storming.md) - Domain events
- [Bounded Contexts](../domain/event-storming.md#bounded-contexts) - Domain structure

## 📊 API Versioning

**Current Version:** v1

**Strategy:**
- Version in URL path (`/api/v1/*`)
- Breaking changes require major version bump
- Non-breaking changes (new fields, endpoints) can be added to current version
- All versions supported for minimum 6 months after new version release

## 🚀 Future Endpoints (Planned)

### Phase 2-5
- Projects CRUD
- Invoices CRUD
- Documents upload/download
- Client Portal endpoints
- Team management
- Notifications
- Reports & analytics

**See:** [Implementation Roadmap](../implementation/roadmap.md)

## 💡 Best Practices

### For API Consumers

1. **Always check status codes** - Don't assume 200
2. **Handle errors gracefully** - Parse error messages
3. **Use pagination** - Don't load all data at once
4. **Cache when appropriate** - Reduce API calls
5. **Include proper headers** - Content-Type, Accept, Authorization
6. **Validate input** - Client-side validation before API call

### For API Implementers

1. **Follow OpenAPI spec exactly** - Contract tests enforce this
2. **Return proper status codes** - Match HTTP semantics
3. **Include helpful error messages** - Aid debugging
4. **Validate input thoroughly** - Security and data integrity
5. **Test contract compliance** - Run spectator tests
6. **Document deviations** - If you must deviate, document why

## 🐛 Troubleshooting

### Common Issues

**401 Unauthorized**
- Check token is valid and not expired
- Verify `Authorization: Bearer <token>` header format
- Try refreshing token

**422 Validation Error**
- Check request body against OpenAPI schema
- Validate required fields are present
- Check field types and formats

**404 Not Found**
- Verify resource ID exists
- Check you have access (multi-tenancy)
- Verify URL path is correct

### Debug Mode

Laravel development includes detailed error messages. In production, errors are sanitized.

## 📞 Support

- **Documentation Issues:** [GitHub Issues](https://github.com/psswid/buildflow-docs/issues)
- **API Questions:** See [Contributing Guide](../guides/contributing.md)

---

**Quick Links:**
- [OpenAPI Spec](../api-contract/openapi.yaml)
- [Architecture Overview](../architecture/ARCHITECTURE.md)
- [Getting Started](../implementation/getting-started-laravel.md)