# Auth API Test Guide (Manual)

## 1) Goal
This document provides a practical checklist to manually test core authentication APIs with fake data.

## 2) Scope
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `GET /auth/profile`
- `POST /auth/logout`

## 3) Assumptions
- Base URL: `https://api.example.local/v1`
- Auth scheme: Bearer JWT for access token
- Refresh token is sent in request body (if your API uses cookies, adapt accordingly)
- Content type: `application/json`

## 4) Fake Test Data
Use these users during testing:

| Label              | Email                         | Password       | Full name       | Notes                       |
|--------------------|-------------------------------|----------------|-----------------|-----------------------------|
| Valid User A       | `alice.tester+01@example.com` | `P@ssw0rd!234` | Alice Tester    | New user for register/login |
| Valid User B       | `bob.tester+01@example.com`   | `P@ssw0rd!234` | Bob Tester      | Secondary account           |
| Invalid Email      | `alice.tester`                | `P@ssw0rd!234` | Alice Bad Email | Email format negative test  |
| Weak Password User | `weak.pass+01@example.com`    | `123456`       | Weak Pass       | Password policy test        |

Reusable invalid token examples:
- Expired access token: `eyJhbGciOi...expired`
- Malformed token: `not-a-jwt`

## 5) Environment Variables (Optional)
For terminal testing:

```bash
BASE_URL="https://api.example.local/v1"
EMAIL="alice.tester+01@example.com"
PASSWORD="P@ssw0rd!234"
FULL_NAME="Alice Tester"
ACCESS_TOKEN=""
REFRESH_TOKEN=""
```

## 6) Test Cases

### 6.1 Register - Happy Path
**Endpoint**: `POST /auth/register`

**Request**
```bash
curl -i -X POST "$BASE_URL/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice.tester+01@example.com",
    "password": "P@ssw0rd!234",
    "fullName": "Alice Tester"
  }'
```

**Expected**
- Status: `201 Created` (or `200 OK` based on implementation)
- Body contains at least:
  - `user.id`
  - `user.email`
  - Optional: `accessToken`, `refreshToken`
- `user.email` equals submitted email
- Password is never returned

---

### 6.2 Register - Duplicate Email
**Endpoint**: `POST /auth/register`

**Request**: Repeat the same request as 6.1.

**Expected**
- Status: `409 Conflict` (or `400 Bad Request`)
- Error shape example:
  ```json
  {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "Email is already registered"
  }
  ```

---

### 6.3 Register - Validation Errors
#### Case A: Invalid email format
```bash
curl -i -X POST "$BASE_URL/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice.tester",
    "password": "P@ssw0rd!234",
    "fullName": "Alice Bad Email"
  }'
```

#### Case B: Weak password
```bash
curl -i -X POST "$BASE_URL/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "weak.pass+01@example.com",
    "password": "123456",
    "fullName": "Weak Pass"
  }'
```

**Expected**
- Status: `400 Bad Request`
- Clear field-level validation messages

---

### 6.4 Login - Happy Path
**Endpoint**: `POST /auth/login`

```bash
curl -i -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice.tester+01@example.com",
    "password": "P@ssw0rd!234"
  }'
```

**Expected**
- Status: `200 OK`
- Response includes:
  - `accessToken`
  - `refreshToken`
  - `user.email`
- Save tokens for next tests

---

### 6.5 Login - Wrong Password
```bash
curl -i -X POST "$BASE_URL/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "alice.tester+01@example.com",
    "password": "WrongPassword!"
  }'
```

**Expected**
- Status: `401 Unauthorized`
- Generic error message (should not reveal whether account exists)

---

### 6.6 Profile - Authorized
**Endpoint**: `GET /auth/profile`

```bash
curl -i -X GET "$BASE_URL/auth/profile" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

**Expected**
- Status: `200 OK`
- Returns current user profile:
  - `id`
  - `email`
  - `fullName`
- Data belongs to token owner

---

### 6.7 Profile - Missing/Invalid Token
#### Case A: No authorization header
```bash
curl -i -X GET "$BASE_URL/auth/profile"
```

#### Case B: Malformed token
```bash
curl -i -X GET "$BASE_URL/auth/profile" \
  -H "Authorization: Bearer not-a-jwt"
```

#### Case C: Expired token
```bash
curl -i -X GET "$BASE_URL/auth/profile" \
  -H "Authorization: Bearer eyJhbGciOi...expired"
```

**Expected**
- Status: `401 Unauthorized`
- Error code indicates missing/invalid/expired token

---

### 6.8 Refresh Token - Happy Path
**Endpoint**: `POST /auth/refresh`

```bash
curl -i -X POST "$BASE_URL/auth/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "REPLACE_WITH_VALID_REFRESH_TOKEN"
  }'
```

**Expected**
- Status: `200 OK`
- Returns new `accessToken`
- Depending on implementation, may rotate and return new `refreshToken`

---

### 6.9 Refresh Token - Invalid/Revoked
```bash
curl -i -X POST "$BASE_URL/auth/refresh" \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "invalid-or-revoked-token"
  }'
```

**Expected**
- Status: `401 Unauthorized` (or `403 Forbidden`)
- Old/invalid refresh token cannot be reused

---

### 6.10 Logout
**Endpoint**: `POST /auth/logout`

```bash
curl -i -X POST "$BASE_URL/auth/logout" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -d '{
    "refreshToken": "REPLACE_WITH_VALID_REFRESH_TOKEN"
  }'
```

**Expected**
- Status: `200 OK` (or `204 No Content`)
- Refresh token is invalidated
- After logout, calling `/auth/refresh` with same token should fail

## 7) Security & Negative Checklist
- SQL injection-like input in email/password should not break API
- Very long strings should return validation errors, not server errors
- Error responses should not expose stack traces or sensitive internals
- Brute-force behavior:
  - Optional lockout/rate-limit after repeated failed login attempts
  - Verify rate-limit response (e.g., `429 Too Many Requests`) if implemented
- Token replay checks:
  - Reusing revoked refresh token must fail

## 8) Recommended Execution Order
1. Register new user
2. Login
3. Profile with valid access token
4. Refresh token
5. Logout
6. Retry refresh (should fail)
7. Run all negative cases

## 9) Test Result Template
Use this table to record outcomes:

| Test ID | Endpoint              | Scenario             | Expected Status | Actual Status | Pass/Fail | Notes |
|---------|-----------------------|----------------------|-----------------|---------------|-----------|-------|
| AUTH-01 | `POST /auth/register` | Valid input          | 201/200         |               |           |       |
| AUTH-02 | `POST /auth/register` | Duplicate email      | 409/400         |               |           |       |
| AUTH-03 | `POST /auth/login`    | Valid credentials    | 200             |               |           |       |
| AUTH-04 | `POST /auth/login`    | Wrong password       | 401             |               |           |       |
| AUTH-05 | `GET /auth/profile`   | Valid bearer token   | 200             |               |           |       |
| AUTH-06 | `GET /auth/profile`   | Missing token        | 401             |               |           |       |
| AUTH-07 | `POST /auth/refresh`  | Valid refresh token  | 200             |               |           |       |
| AUTH-08 | `POST /auth/logout`   | Valid logout         | 200/204         |               |           |       |
| AUTH-09 | `POST /auth/refresh`  | Reused revoked token | 401/403         |               |           |       |

