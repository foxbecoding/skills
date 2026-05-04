## Documentation References
Consult `~/.claude/skills/django-backend/docs/django/docs` when unsure about a Django behavior or pattern.
Consult `~/.claude/skills/django-backend/docs/django-rest-framework/docs` when unsure about a DRF behavior or pattern.

## Project Structure

Views live in `apps/<app>/views.py` — keep them thin, no business logic.
All business logic belongs in service classes under `apps/<app>/services/`.
Tests live in `apps/<app>/tests/test_<name>.py`. Factories live in `apps/<app>/tests/factories.py`. Register factories in the root `conftest.py`.

## Service Layer

Services are classes, not module-level functions. Services raise exceptions for error cases; views catch and return the appropriate response.

```python
class UserService:
    def create_user(self, data: dict) -> User:
        """..."""
        ...
```

## Testing

Use **pytest-django** and **pytest-factoryboy** for all tests.
Follow TDD: write the failing test first, implement the minimum code to pass, then refactor.

Run tests with:
```bash
pytest
```

## Security

Apply the following to every endpoint:

- **Permissions** — every view must explicitly declare `permission_classes`. Never rely on global defaults.
- **Authentication** — sensitive endpoints must require authentication. Unauthenticated access returns 401, not leaked data.
- **Throttling** — auth-related endpoints (login, password reset, OTP) must use `ScopedRateThrottle` with a named scope in `DEFAULT_THROTTLE_RATES`.
- **Input validation** — validate all input at the serializer level. Never use raw request data in views or services.
- **Sensitive data** — never log or expose passwords, tokens, or PII in responses or error messages.

## Response Envelope

All API responses must use `ApiResponseMixin` from `main_core`:

```python
class MyView(ApiResponseMixin, APIView):
    def get(self, request):
        return self.success_response(data=..., message="...")
```

Standardised envelope:

```json
{
  "success": true,
  "message": "...",
  "data": ...
}
```

## Code Standards

- Follow Django and DRF best practices and idiomatic patterns in all implementations.
- All service methods, model managers, and public utility functions must have docblocks and return type annotations.
