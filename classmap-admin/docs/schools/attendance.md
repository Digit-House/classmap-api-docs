# Daily Attendance – Schools

![Daily Attendance Tab](../images/school/detail/daily-attendance.png)

## Flow

```
Admin opens Daily Attendance tab
        |
        v
GET /api/v1/admin/schools/{id}/attendance   <-- default list (no filters)
        |
Admin applies filters (teacher name, class time, education level, date range)
        |
        v
GET /api/v1/admin/schools/{id}/attendance?teacher_name=...&from_date=...  <-- filtered
```

## Endpoints

- [GET `/api/v1/admin/schools/{id}/attendance`](#1-list-school-daily-attendance) — Paginated attendance logs with optional filters

---

### 1. List School Daily Attendance
**GET** `/api/v1/admin/schools/{id}/attendance`

**Headers**

| Key | Value | Required |
|---|---|---|
| `Authorization` | `Bearer {{access_token}}` | Yes |
| `Content-Type` | `application/json` | Yes |
| `X-Request-ID` | `<uuid>` | Yes |

**Path Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | School UUID |

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `teacher_name` | string | No | Filter by teacher name (partial match) |
| `class_time` | string | No | Filter by class time slot: `day`, `evening`, `morning` |
| `education_level` | string | No | Filter by education level |
| `from_date` | string | No | Start date (ISO 8601: `YYYY-MM-DD`) |
| `to_date` | string | No | End date (ISO 8601: `YYYY-MM-DD`) |
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Items per page (default: 10) |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "id": "att_001",
      "teacher_name": "Thu Zarkhin",
      "class_time": "Day School (9:00 AM - 3:00 PM)",
      "date": "2026-05-05T12:00:00Z",
      "present_count": {
        "male": 45,
        "female": 52,
        "swd": 8,
        "total": 105
      },
      "absent_count": {
        "male": 3,
        "female": 2,
        "swd": 1,
        "total": 6
      }
    },
    {
      "id": "att_002",
      "teacher_name": "Aung Min",
      "class_time": "Day School (9:00 AM - 3:00 PM)",
      "date": "2026-06-05T12:00:00Z",
      "present_count": {
        "male": 42,
        "female": 56,
        "swd": 7,
        "total": 98
      },
      "absent_count": {
        "male": 4,
        "female": 3,
        "swd": 1,
        "total": 8
      }
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 10,
    "total": 24
  },
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Invalid query parameters or date format |
| `401` | `UNAUTHORIZED` | Missing or invalid token |
| `403` | `FORBIDDEN` | Insufficient role |
| `404` | `SCHOOL_NOT_FOUND` | School ID does not exist |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid query param or date format |
| `UNAUTHORIZED` | 401 | Missing or invalid token |
| `FORBIDDEN` | 403 | Insufficient role |
| `SCHOOL_NOT_FOUND` | 404 | School not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
