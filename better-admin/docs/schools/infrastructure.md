# School Infrastructure – Schools

![Infrastructure Tab](../../images/school/infrastructure-detail.png)

## Flow

```
Admin opens Infrastructure tab
        |
        v
GET /api/v1/schools/{id}/infrastructure    <-- building list + facility counts + related plans
        |
Admin selects a building type (dropdown: School, etc.)
        |
        v
GET /api/v1/schools/{id}/infrastructure?building_type=school
        |
Admin clicks "Manage Facility"
        |
        v
PATCH /api/v1/schools/{id}/infrastructure/{building_id}
```

## Endpoints

- [GET `/api/v1/schools/{id}/infrastructure`](#1-get-school-infrastructure) — Buildings, facility counts, and related plans
- [PATCH `/api/v1/schools/{id}/infrastructure/{building_id}`](#2-update-building-facility) — Update facility counts for a building

---

### 1. Get School Infrastructure
**GET** `/api/v1/schools/{id}/infrastructure`

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
| `building_type` | string | No | Filter by type: `school`, `hostel`, etc. (default: `school`) |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "buildings": [
      {
        "id": "bld_001",
        "building_number": 1,
        "building_name": "Availability of School Infrastructure",
        "building_type": "school",
        "facilities": [
          { "name": "Classroom", "count": 3 },
          { "name": "Library", "count": 1 },
          { "name": "Restrooms", "count": 4 },
          { "name": "Computer Lab", "count": 0 },
          { "name": "Principal's office", "count": 1 },
          { "name": "Staff/Teacher's room", "count": 2 },
          { "name": "Administrative office", "count": 1 }
        ]
      }
    ],
    "related_plans": {
      "summary": {
        "total": 8,
        "pending": 2,
        "approved": 2,
        "funding_unavailable": 4
      },
      "plans": [
        {
          "id": "plan_001",
          "title": "School Development Plan",
          "review_result": "Please revised"
        },
        {
          "id": "plan_002",
          "title": "Annual Budget Report",
          "review_result": "Pending approval"
        },
        {
          "id": "plan_003",
          "title": "Teacher Training Schedule",
          "review_result": "To be finalized"
        }
      ]
    }
  },
  "meta": null,
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

---

### 2. Update Building Facility
**PATCH** `/api/v1/schools/{id}/infrastructure/{building_id}`

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
| `building_id` | string | Yes | Building UUID |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| `facilities` | array | Yes | Array of facility objects with name and count |

```json
{
  "facilities": [
    { "name": "Classroom", "count": 4 },
    { "name": "Library", "count": 1 },
    { "name": "Restrooms", "count": 5 },
    { "name": "Computer Lab", "count": 1 },
    { "name": "Principal's office", "count": 1 },
    { "name": "Staff/Teacher's room", "count": 2 },
    { "name": "Administrative office", "count": 1 }
  ]
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "building_id": "bld_001",
    "building_name": "Availability of School Infrastructure",
    "facilities": [
      { "name": "Classroom", "count": 4 },
      { "name": "Library", "count": 1 }
    ],
    "updated_at": "2026-05-08T10:00:00Z"
  },
  "meta": null,
  "error": null,
  "message": "Facility updated successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Invalid facility data |
| `401` | `UNAUTHORIZED` | Missing or invalid token |
| `403` | `FORBIDDEN` | Insufficient role |
| `404` | `BUILDING_NOT_FOUND` | Building ID does not exist |
| `409` | `CONFLICT` | Concurrent update conflict |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid facility data |
| `UNAUTHORIZED` | 401 | Missing or invalid token |
| `FORBIDDEN` | 403 | Insufficient role |
| `SCHOOL_NOT_FOUND` | 404 | School not found |
| `BUILDING_NOT_FOUND` | 404 | Building not found |
| `CONFLICT` | 409 | Concurrent update conflict |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
