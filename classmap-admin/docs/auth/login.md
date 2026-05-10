# Login – Auth

![Login Screen](../images/login/login.png)

## Flow

```
User enters username/email + password
        |
        v
POST /api/v1/auth/login
        |
   +---------+
   | success |---> return access_token + refresh_token
   | failure |---> 401 UNAUTHORIZED
   +---------+
```

## Endpoints

- [POST `/api/v1/auth/login`](#1-login) — Authenticate with username/email and password
- [POST `/api/v1/auth/refresh`](#2-refresh-token) — Issue a new access token using a valid refresh token

---

### 1. Login
**POST** `/api/v1/auth/login`

**Headers**

| Key | Value | Required |
|---|---|---|
| `Content-Type` | `application/json` | Yes |
| `X-Request-ID` | `<uuid>` | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | string | Yes | Username or email address |
| `password` | string | Yes | User password |
| `remember_me` | boolean | No | Extend session lifetime |

```json
{
  "email": "taylor@classmap.edu",
  "password": "secret123",
  "remember_me": false
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "expires_in": 3600,
    "token_type": "Bearer",
    "user": {
      "id": "usr_001",
      "display_name": "Taylor",
      "email": "taylor@classmap.edu",
      "role": "admin"
    }
  },
  "meta": null,
  "error": null,
  "message": "Login successful"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Missing or malformed fields |
| `401` | `UNAUTHORIZED` | Invalid credentials |
| `429` | `RATE_LIMIT_EXCEEDED` | Too many login attempts |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

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
**POST** `/api/v1/auth/refresh`

Exchange a valid refresh token for a new access token. Call this when the access token has expired (client receives `401 UNAUTHORIZED`) rather than forcing the user to log in again.

**Headers**

| Key | Value | Required |
|---|---|---|
| `Content-Type` | `application/json` | Yes |
| `X-Request-ID` | `<uuid>` | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| `refresh_token` | string | Yes | Refresh token issued at login |

```json
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...(new)",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...(rotated)",
    "expires_in": 3600,
    "token_type": "Bearer"
  },
  "meta": null,
  "error": null,
  "message": "Token refreshed successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Missing `refresh_token` field |
| `401` | `UNAUTHORIZED` | Refresh token is invalid or expired |
| `429` | `RATE_LIMIT_EXCEEDED` | Too many refresh attempts |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

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

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Missing or malformed request fields |
| `UNAUTHORIZED` | 401 | Invalid credentials or expired/invalid refresh token |
| `RATE_LIMIT_EXCEEDED` | 429 | Login or refresh attempts exceeded |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
