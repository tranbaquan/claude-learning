---
paths:
  - "**/*.py"
---

# Coding Conventions for API Tests

## 1. Core Principles

- Tests are executable specifications: optimize for clarity over cleverness.
- Keep tests deterministic and isolated; each test should pass independently.
- Prefer small, focused tests with one primary behavior per test.
- Use consistent Arrange-Act-Assert structure.
- Avoid refactoring unrelated production code while adding tests.

## 2. Python Style

- Follow PEP 8 for formatting.
- Target line length: 88-100 chars (choose one value project-wide and keep consistent).
- Use explicit imports; avoid wildcard imports.
- Use type hints for shared helpers, fixtures returning structured data, and utility modules.
- Keep helper functions pure when possible (no hidden global state).
- Raise clear exceptions in helper code; avoid bare `except`.

## 3. Docstrings and Comments

- Add docstrings for reusable helpers and non-obvious fixtures.
- Keep docstrings concise: what the helper does and key inputs/outputs.
- Use comments sparingly; only explain non-trivial intent.
- Do not comment obvious code.

## 4. Test File and Function Naming

- Test file: `test_<feature>.py`
- Test function: `test_<endpoint_or_unit>_<condition>_<expected_result>`
- Examples:
    - `test_login_with_valid_credentials_returns_tokens`
    - `test_profile_without_token_returns_401`

## 5. Test Structure (AAA Pattern)

Use this order in each test:

1. Arrange: build payloads, headers, and fixtures
2. Act: call API/function once
3. Assert: status code first, then contract, then business fields

Example skeleton:

```python
def test_login_with_valid_credentials_returns_tokens(client, registered_user):
    # Arrange
    payload = {"email": registered_user["email"], "password": registered_user["password"]}

    # Act
    response = client.post("/auth/login", json=payload)
    body = response.json()

    # Assert
    assert response.status_code == 200
    assert "accessToken" in body
    assert "refreshToken" in body
    assert body["user"]["email"] == registered_user["email"]
```

## 6. Pytest Conventions

- Prefer plain `assert` statements with explicit checks.
- Group tests by domain under `tests/api`, `tests/unit`, etc.
- Keep fixtures in `tests/conftest.py` unless domain-specific.
- Fixture scope defaults to `function`; use broader scope only when safe and justified.
- Use `pytest.mark.parametrize` for validation matrices.
- Avoid inter-test dependencies and shared mutable state.

## 7. Fixture Guidelines

Recommended fixtures:

- `base_url`: loaded from `BASE_URL`
- `auth_client`: authorized client/request helper
- `test_user`: deterministic fake user data
- `registered_user`: user prepared for auth scenarios
- `tokens`: access + refresh token pair

Fixture rules:

- Return structured, minimal data needed by tests.
- Avoid network calls in unrelated fixtures.
- Clean up created test data when API supports deletion.
- Never embed real credentials.

## 8. Assertion Standards

Always assert:

- HTTP status code
- Response schema keys and expected types
- Business-critical values
- Security expectations (no password leakage, invalid token rejected)

Prefer:

- `assert response.status_code == 401`
- `assert body["code"] == "INVALID_CREDENTIALS"`

Avoid:

- Only `assert response.ok`
- Overly broad assertions that hide failures

## 9. API Testing Style

For auth endpoints (`register`, `login`, `refresh`, `profile`, `logout`):

- Validate happy path and negative path for each endpoint.
- Verify token lifecycle behavior (issue, refresh, revoke).
- Confirm protected endpoints reject missing/malformed/expired tokens.
- Verify error response format is stable (`code`, `message`).

## 10. Determinism and Flakiness Rules

- No random sleeps; use polling with bounded retries only if unavoidable.
- Freeze or control time when testing expiry logic.
- Use deterministic fake data (predictable emails/usernames).
- Avoid dependence on test execution order.
- Keep retry logic explicit and minimal.

## 11. Security and Logging

- Never print secrets/tokens in plain text logs.
- Sanitize request/response debug output.
- Use fake, disposable test accounts only.
- Ensure failures do not expose stack traces from server responses as accepted behavior.

## 12. Recommended Project Layout

```text
tests/
  conftest.py
  api/
    test_auth_register.py
    test_auth_login.py
    test_auth_refresh.py
    test_auth_profile.py
    test_auth_logout.py
```


