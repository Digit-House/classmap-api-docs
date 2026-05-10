# Class Map – API Documentation

**Class Map** is an Education Management Information System (EMIS) used to support schools and learning centres in Myanmar.

---

## Available APIs

| Module | Endpoints | Description |
|---|---|---|
| [Auth](auth/login.md) | 2 | Login, token refresh |
| [Dashboard](dashboard/overview.md) | 2 | Summary stats, daily attendance |
| [Schools – List](schools/list.md) | 1 | Paginated school list with map data |
| [Schools – General & Analytics](schools/detail.md) | 3 | School profile and analytics charts |
| [Schools – Daily Attendance](schools/attendance.md) | 1 | Attendance logs with filters |
| [Schools – Classes](schools/classes.md) | 4 | Classroom CRUD and export |
| [Schools – Plans](schools/plans.md) | 7 | Plan submission, review, and export |
| [Schools – Infrastructure](schools/infrastructure.md) | 2 | Building and facility management |
| [Schools – Gallery](schools/gallery.md) | 1 | Photo gallery by category |
| [Schools – Daily Logs](schools/daily-logs.md) | 1 | Activity audit log |
| [Schools – Teachers](schools/teachers.md) | 6 | Teacher management and role control |

**Total: 30 endpoints**

---

## Standards

All endpoints follow these conventions:

=== "Response Envelope"

    ```json
    {
      "success": true,
      "data": {},
      "meta": { "page": 1, "per_page": 20, "total": 150 },
      "error": null,
      "message": "Successfully"
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
        "request_id": "a1b2c3d4-..."
      }
    }
    ```

=== "Required Headers"

    | Header | Description |
    |---|---|
    | `Authorization: Bearer <token>` | JWT — required on all authenticated routes |
    | `X-Request-ID` | Trace ID — include on every request |
    | `Content-Type: application/json` | Required for POST / PATCH / PUT |

## Postman Collection

Import `classmap-admin.postman_collection.json` from the repo root and set these variables after import:

| Variable | Description |
|---|---|
| `base_url` | API base URL (e.g. `http://localhost:8080`) |
| `access_token` | JWT access token from login |
| `refresh_token` | Refresh token from login |
| `school_id` | Target school UUID |
