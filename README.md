# BuildFlow Laravel API

Laravel implementation of [BuildFlow API Contract](https://github.com/yourusername/buildflow-docs/tree/main/api-contract).

## API Compliance

This implementation follows the official OpenAPI specification:
- Spec: https://github.com/yourusername/buildflow-docs/blob/main/api-contract/openapi.yaml
- Version: 1.0.0
- Compliance: 100%

## Testing Contract Compliance
```bash
# Run contract tests
php artisan test --testsuite=Contract
```

**W `buildflow-symfony-api/README.md`:**
```markdown
# BuildFlow Symfony API

Symfony implementation of [BuildFlow API Contract](https://github.com/yourusername/buildflow-docs/tree/main/api-contract).

[Same structure as Laravel]
```

### 3. Frontend konsumuje ten kontrakt

**W `buildflow-react-web/README.md`:**
```markdown
# BuildFlow React Web

React SPA that consumes [BuildFlow API Contract](https://github.com/yourusername/buildflow-docs/tree/main/api-contract).

## Backend Compatibility

This frontend works with ANY backend implementing the BuildFlow API contract:
- ✅ Laravel: http://localhost:8000/api/v1
- ✅ Symfony: http://localhost:8001/api/v1
- ✅ Next.js: http://localhost:3000/api/v1

Configure backend in `.env`:
```
VITE_API_BASE_URL=http://localhost:8000/api/v1