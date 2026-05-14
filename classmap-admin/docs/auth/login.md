# Login – Auth

![Login Screen](../images/login/login.png)

## Flow

```
User enters username/email + password
        |
        v
POST /api/v1/admin/auth/login
        |
   +---------+
   | success |---> return access_token + refresh_token
   | failure |---> 401 UNAUTHORIZED
   +---------+
```

## Endpoints

- [POST `/api/v1/admin/auth/login`](#1-login) — Authenticate with username/email and password
- [POST `/api/v1/admin/auth/refresh`](#2-refresh-token) — Issue a new access token using a valid refresh token
- [GET `/api/v1/admin/auth/me`](#3-get-current-user) — Get current user profile
- [POST `/api/v1/admin/auth/sign-out`](#4-sign-out) — Revoke current session

---

### 1. Login

**POST** `/api/v1/admin/auth/login`

**Headers**

| Key            | Value              | Required |
| -------------- | ------------------ | -------- |
| `Content-Type` | `application/json` | Yes      |
| `X-Request-ID` | `<uuid>`           | Yes      |

**Request Body**

| Field             | Type   | Required | Description               |
| ----------------- | ------ | -------- | ------------------------- |
| `emailOrUsername` | string | Yes      | Username or email address |
| `password`        | string | Yes      | User password             |

```json
{
  "emailOrUsername": "taylor@classmap.edu",
  "password": "secret123"
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "accessTokenExpiresAt": 1715601600,
    "refreshTokenExpiresAt": 1716206400,
    "requirePinSetup": false,
    "user": {
      "id": "usr_001",
      "username": "taylor",
      "displayedName": "Taylor",
      "phoneNumber": null,
      "email": "taylor@classmap.edu",
      "status": true,
      "roles": ["admin"]
    },
    "teacher": null
  },
  "meta": null,
  "error": null,
  "message": "Login successful"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description                 |
| ------ | ----------------------- | --------------------------- |
| `400`  | `VALIDATION_ERROR`      | Missing or malformed fields |
| `401`  | `UNAUTHORIZED`          | Invalid credentials         |
| `429`  | `RATE_LIMIT_EXCEEDED`   | Too many login attempts     |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault     |

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid username/email or password.",
    "request_id": "a1b2c3d4-0000-0000-0000-000000000001"
  }
}
```

---

### 2. Refresh Token

**POST** `/api/v1/admin/auth/refresh`

Exchange a valid refresh token for a new access token. Call this when the access token has expired (client receives `401 UNAUTHORIZED`) rather than forcing the user to log in again.

**Headers**

| Key            | Value              | Required |
| -------------- | ------------------ | -------- |
| `Content-Type` | `application/json` | Yes      |
| `X-Request-ID` | `<uuid>`           | Yes      |

**Request Body**

| Field          | Type   | Required | Description                   |
| -------------- | ------ | -------- | ----------------------------- |
| `refreshToken` | string | Yes      | Refresh token issued at login |

```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...(new)",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...(rotated)",
    "accessTokenExpiresAt": 1715601600,
    "refreshTokenExpiresAt": 1716206400,
    "requirePinSetup": false,
    "user": {
      "id": "usr_001",
      "username": "taylor",
      "displayedName": "Taylor",
      "phoneNumber": null,
      "email": "taylor@classmap.edu",
      "status": true,
      "roles": ["admin"]
    },
    "teacher": null
  },
  "meta": null,
  "error": null,
  "message": "Token refreshed successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description                         |
| ------ | ----------------------- | ----------------------------------- |
| `400`  | `VALIDATION_ERROR`      | Missing `refresh_token` field       |
| `401`  | `UNAUTHORIZED`          | Refresh token is invalid or expired |
| `429`  | `RATE_LIMIT_EXCEEDED`   | Too many refresh attempts           |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault             |

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Refresh token is invalid or has expired.",
    "request_id": "a1b2c3d4-0000-0000-0000-000000000002"
  }
}
```

---

### 3. Get Current User

**GET** `/api/v1/admin/auth/me`

Retrieve the current authenticated user profile.

**Headers**

| Key             | Value                     | Required |
| --------------- | ------------------------- | -------- |
| `Authorization` | `Bearer {{access_token}}` | Yes      |
| `Content-Type`  | `application/json`        | Yes      |
| `X-Request-ID`  | `<uuid>`                  | Yes      |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": "usr_001",
      "username": "taylor",
      "displayedName": "Taylor",
      "phoneNumber": null,
      "email": "taylor@classmap.edu",
      "status": true,
      "roles": ["admin"]
    },
    "teacher": null
  },
  "meta": null,
  "error": null,
  "message": "User fetched successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description              |
| ------ | ----------------------- | ------------------------ |
| `401`  | `UNAUTHORIZED`          | Missing or invalid token |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault  |

---

### 4. Sign Out

**POST** `/api/v1/admin/auth/sign-out`

Revoke the current session and invalidate the refresh token.

**Headers**

| Key             | Value                     | Required |
| --------------- | ------------------------- | -------- |
| `Authorization` | `Bearer {{access_token}}` | Yes      |
| `Content-Type`  | `application/json`        | Yes      |
| `X-Request-ID`  | `<uuid>`                  | Yes      |

**Response – 200 OK**

```json
{
  "success": true,
  "data": "Signed out successfully",
  "meta": null,
  "error": null,
  "message": "Signed out successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description              |
| ------ | ----------------------- | ------------------------ |
| `401`  | `UNAUTHORIZED`          | Missing or invalid token |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault  |

## Error Codes

| Code                    | HTTP Status | Description                                          |
| ----------------------- | ----------- | ---------------------------------------------------- |
| `VALIDATION_ERROR`      | 400         | Missing or malformed request fields                  |
| `UNAUTHORIZED`          | 401         | Invalid credentials or expired/invalid refresh token |
| `RATE_LIMIT_EXCEEDED`   | 429         | Login or refresh attempts exceeded                   |
| `INTERNAL_SERVER_ERROR` | 500         | Unexpected server error                              |
