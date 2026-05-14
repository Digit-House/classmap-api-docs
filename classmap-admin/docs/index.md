# Class Map – API Documentation

**Class Map** is an Education Management Information System (EMIS) used to support schools and learning centres in Myanmar.

---

## Available APIs

| Module                                                | Endpoints | Description                                    |
| ----------------------------------------------------- | --------- | ---------------------------------------------- |
| [Auth](auth/login.md)                                 | 4         | Login, token refresh, profile, sign-out        |
| [Dashboard](dashboard/overview.md)                    | 2         | Summary stats, daily attendance                |
| [Schools – List](schools/list.md)                     | 1         | Paginated school list with map data            |
| [Schools – General & Analytics](schools/detail.md)    | 5         | School profile, analytics, review, delete      |
| [Schools – Daily Attendance](schools/attendance.md)   | 1         | Attendance logs with filters                   |
| [Schools – Classes](schools/classes.md)               | 4         | Classroom CRUD and export                      |
| [Schools – Plans](schools/plans.md)                   | 7         | Plan submission, review, and export            |
| [Schools – Infrastructure](schools/infrastructure.md) | 2         | Building and facility management               |
| [Schools – Gallery](schools/gallery.md)               | 1         | Photo gallery by category                      |
| [Schools – Daily Logs](schools/daily-logs.md)         | 1         | Daily log entries with review status           |
| [Schools – Teachers](schools/teachers.md)             | 7         | Teacher management, approval, and role control |

**Total: 35 endpoints**

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

Import `classmap-admin.postman_collection.json` from the repo root and set these variables after import:

| Variable        | Description                                 |
| --------------- | ------------------------------------------- |
| `base_url`      | API base URL (e.g. `http://localhost:8080`) |
| `access_token`  | JWT access token from login                 |
| `refresh_token` | Refresh token from login                    |
| `school_id`     | Target school UUID                          |
