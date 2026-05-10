# Login & Authentication – Login

![Login with Phone](../images/login/login-with-phone.png)
![OTP Verify Empty](../images/login/otp-verify.png)
![OTP Filled](../images/login/otp-filled.png)
![OTP Error](../images/login/otp-filled-error.png)
![PIN Setup](../images/login/otp-success-pin-added-login.png)

## Flow

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌─────────────┐     ┌──────┐
│ Enter Phone │────▶│ Request OTP      │────▶│ Enter OTP       │────▶│ Verify OTP  │────▶│ Home │
│             │     │ (SMS)            │     │ (6-digit)       │     │ (Online)    │     │      │
└─────────────┘     └──────────────────┘     └─────────────────┘     └──────┬──────┘     └──────┘
                                                                            │
                                                       New user ◀───────────┘
                                                                            │
                                                       ┌──────────────┐     │
                                                       │ Setup PIN    │─────┘
                                                       │ + Biometric  │
                                                       └──────────────┘
```

## Endpoints

- [POST `/api/v1/auth/otp/request`](#1-request-otp) — Send OTP to phone number
- [POST `/api/v1/auth/otp/verify`](#2-verify-otp) — Verify 6-digit OTP code
- [POST `/api/v1/auth/pin/setup`](#3-setup-pin) — Create PIN and biometric preference

---

### 1. Request OTP
**POST** `/api/v1/auth/otp/request`

Send a 6-digit OTP via SMS to the provided phone number.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| phone_number | string | Yes | E.164 format, e.g. +959123456789 |
| purpose | string | No | `login`, `register_school`, `register_teacher`. Defaults to `login` |

```json
{
  "phone_number": "+959123456789",
  "purpose": "login"
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "ref_code": "F2aR4",
    "expires_in": 300,
    "retry_after": 299
  },
  "meta": null,
  "error": null,
  "message": "OTP sent successfully"
}
```

**Response – 400 Bad Request**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INVALID_PHONE_NUMBER",
    "details": "Phone number must be in E.164 format"
  },
  "message": "Invalid phone number"
}
```

**Response – 429 Too Many Requests**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "OTP_RATE_LIMIT",
    "details": "Please wait 4:59 before requesting a new code"
  },
  "message": "Too many OTP requests"
}
```

---

### 2. Verify OTP
**POST** `/api/v1/auth/otp/verify`

Verify the 6-digit OTP code. Returns tokens for existing users or a temporary token for new users who need to set up PIN.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| phone_number | string | Yes | Same phone used to request OTP |
| otp_code | string | Yes | 6-digit code from SMS |
| ref_code | string | Yes | Reference code from OTP request response |

```json
{
  "phone_number": "+959123456789",
  "otp_code": "123861",
  "ref_code": "F2aR4"
}
```

**Response – 200 OK (Existing User)**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600,
    "is_new_user": false
  },
  "meta": null,
  "error": null,
  "message": "Login successful"
}
```

**Response – 200 OK (New User)**

```json
{
  "success": true,
  "data": {
    "temp_token": "eyJhbGciOiJIUzI1NiIs...",
    "is_new_user": true,
    "expires_in": 900
  },
  "meta": null,
  "error": null,
  "message": "Phone verified. Please set up your PIN."
}
```

**Response – 400 Bad Request**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INVALID_OTP",
    "details": "The code is incorrect. Please try again."
  },
  "message": "Invalid OTP"
}
```

**Response – 401 Unauthorized**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "OTP_EXPIRED",
    "details": "OTP has expired. Please request a new one."
  },
  "message": "OTP expired"
}
```

---

### 3. Setup PIN
**POST** `/api/v1/auth/pin/setup`

Create a 4-6 digit PIN and optionally enable biometric security. Only callable with a valid `temp_token` from OTP verify for new users.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| Authorization | Bearer {temp_token} | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| pin | string | Yes | 4-6 digit numeric PIN |
| enable_biometric | boolean | No | Enable FaceID / Fingerprint. Defaults to `true` |

```json
{
  "pin": "123456",
  "enable_biometric": true
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600,
    "biometric_enabled": true
  },
  "meta": null,
  "error": null,
  "message": "PIN created successfully"
}
```

**Response – 400 Bad Request**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INVALID_PIN",
    "details": "PIN must be 4-6 digits"
  },
  "message": "Invalid PIN format"
}
```

**Response – 401 Unauthorized**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INVALID_TEMP_TOKEN",
    "details": "Temp token expired or invalid"
  },
  "message": "Unauthorized"
}
```

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| INVALID_PHONE_NUMBER | 400 | Phone number not in E.164 format |
| OTP_RATE_LIMIT | 429 | Too many OTP requests |
| INVALID_OTP | 400 | Incorrect OTP code |
| OTP_EXPIRED | 401 | OTP code has expired |
| INVALID_TEMP_TOKEN | 401 | Temp token missing or expired |
| INVALID_PIN | 400 | PIN must be 4-6 digits |
