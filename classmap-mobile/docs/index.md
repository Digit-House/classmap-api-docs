# Class Map Mobile – API Documentation

**Class Map Mobile** is the Android & iOS app for school principals and teachers. It supports OTP-based authentication, school registration, and teacher registration.

---

## Available APIs

| Module                                      | Endpoints | Description                                                             |
| ------------------------------------------- | --------- | ----------------------------------------------------------------------- |
| [Login & Authentication](login.md)          | 7         | Phone OTP login, PIN setup, biometric, refresh, me, sign-out, reset PIN |
| [School Registration](school-register.md)   | 13        | Multi-step wizard: school info, OTP, documents, admin account, submit   |
| [Teacher Registration](teacher-register.md) | 5         | School code lookup, account info, OTP verification, submit              |

**Total: 25 endpoints**

---

## Login Flow

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌─────────────┐     ┌──────┐
│ App Launch  │────▶│ Has PIN?         │────▶│ PIN Login UI    │────▶│ Verify PIN  │────▶│ Home │
└─────────────┘     │                  │     │                 │     │ (Local)     │     │      │
                    │ Yes ─────────────┘     └─────────────────┘     └──────┬──────┘     └──────┘
                    │                                                       │
                    │ No                                                    │ Invalid
                    │                                                       │
                    ▼                                                       ▼
            ┌──────────────────┐                                    ┌─────────────┐
            │ Mobile Login UI  │                                    │ PIN Login UI│
            └──────────────────┘                                    └─────────────┘
                    │
                    ▼
            ┌──────────────────┐     ┌─────────────────┐     ┌─────────────┐
            │ Enter OTP        │────▶│ Verify OTP      │────▶│ Set PIN     │────▶ Home
            └──────────────────┘     │ (Online)        │     └─────────────┘
                    ▲                └──────┬──────────┘
                    │                       │ Invalid
                    └───────────────────────┘
```

---

## Standards

All endpoints follow these conventions:

=== "Response Envelope"

    ```json
    {
      "success": true,
      "data": {},
      "meta": { "page": 1, "limit": 20, "total": 150, "totalPages": 8 },
      "error": null,
      "message": "Successfully",
      "timestamp": "2026-05-08T10:00:00.000Z"
    }
    ```

=== "Error Envelope"

    ```json
    {
      "success": false,
      "data": null,
      "error": {
        "code": "RESOURCE_NOT_FOUND",
        "message": "The requested resource does not exist.",
        "details": null,
        "statusCode": 404
      },
      "message": "Resource not found",
      "timestamp": "2026-05-08T10:00:00.000Z"
    }
    ```

=== "Required Headers"

    | Header | Description |
    |---|---|
    | `Authorization: Bearer <token>` | JWT — required on all authenticated routes |
    | `X-Request-ID` | Trace ID — recommended on every request |
    | `Content-Type: application/json` | Required for POST / PATCH / PUT |

## Postman Collection

Import `classmap-mobile.postman_collection.json` from the repo root and set these variables after import:

| Variable                  | Description                                 |
| ------------------------- | ------------------------------------------- |
| `base_url`                | API base URL (e.g. `http://localhost:8080`) |
| `access_token`            | JWT access token from login                 |
| `refresh_token`           | Refresh token from login                    |
| `temp_token`              | Temporary token from OTP verify (new users) |
| `school_registration_id`  | Draft school registration ID                |
| `teacher_registration_id` | Draft teacher registration ID               |
| `school_id`               | Target school UUID                          |
| `school_code`             | School register code (e.g. `SCH001`)        |
