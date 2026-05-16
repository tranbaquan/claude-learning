# Authentication & Authorization

## 1. Authentication

### 1.1 Register New User
- **Endpoint**: `POST /auth/register`

- **Request Body**:

    | Field     | Type   | Description                |
    |-----------|--------|----------------------------|
    | email     | string | User's email address       |
    | password  | string | User's password            |

- **Expected Response**:
  - Status: `201 Created` (or `200 OK`)
  - Body contains at least:
    - `user.id`
    - `user.email`
    - Optional: `accessToken`, `refreshToken`
  - `user.email` equals submitted email
  - Password is never returned

### 1.2 Login User
- **Endpoint**: `POST /auth/login`
- **Request Body**:

    | Field     | Type   | Description                |
    |-----------|--------|----------------------------|
    | email     | string | User's email address       |
    | password  | string | User's password            |
- **Expected Response**:
  - Status: `200 OK`
  - Body contains:
    - `accessToken`
    - `refreshToken`
    - `user.id`
    - `user.email`
  - `user.email` equals submitted email
  - Password is never returned