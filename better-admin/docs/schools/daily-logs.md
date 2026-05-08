# Daily Logs – Schools

![Daily Logs Tab](../../images/school/log-history.png)

## Flow

```
Admin opens Daily Logs tab
        |
        v
GET /api/v1/schools/{id}/daily-logs   <-- paginated activity log
```

## Endpoints

- [GET `/api/v1/schools/{id}/daily-logs`](#1-list-daily-logs) — Paginated list of activity logs for the school

---

### 1. List Daily Logs
**GET** `/api/v1/schools/{id}/daily-logs`

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
| `page` | integer | No | Page number (default: 1) |
| `per_page` | integer | No | Items per page (default: 10) |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "id": "log_001",
      "time": "09:15:23",
      "date": "2026-05-07",
      "account": {
        "name": "Daw Hla Hla",
        "account_id": "T-2041"
      },
      "action": "Updated Profile Information",
      "changes": "Changed phone number and emergency contact",
      "status": "success"
    },
    {
      "id": "log_002",
      "time": "10:45:12",
      "date": "2026-05-08",
      "account": {
        "name": "Ko Ko Naing",
        "account_id": "T-2042"
      },
      "action": "Submitted Leave Request",
      "changes": "Requesting leave for personal reasons",
      "status": "pending"
    },
    {
      "id": "log_003",
      "time": "11:20:45",
      "date": "2026-05-09",
      "account": {
        "name": "Aye Aye Win",
        "account_id": "T-2043"
      },
      "action": "Completed Training Module",
      "changes": "Finished the advanced marketing strategies course",
      "status": "success"
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 10,
    "total": 8
  },
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `401` | `UNAUTHORIZED` | Missing or invalid token |
| `403` | `FORBIDDEN` | Insufficient role |
| `404` | `SCHOOL_NOT_FOUND` | School ID does not exist |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `UNAUTHORIZED` | 401 | Missing or invalid token |
| `FORBIDDEN` | 403 | Insufficient role |
| `SCHOOL_NOT_FOUND` | 404 | School not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
