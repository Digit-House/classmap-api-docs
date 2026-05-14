# School Lists – Schools

![School List](../images/school/list.png)

## Flow

```
Admin navigates to Schools
        |
        v
GET /api/v1/admin/schools   <-- paginated list with search + status filter
        |
        v
Map pins + sidebar list rendered
```

## Endpoints

- [GET `/api/v1/admin/schools`](#1-list-schools) — Paginated list of all schools with map data

---

### 1. List Schools

**GET** `/api/v1/admin/schools`

**Headers**

| Key             | Value                     | Required |
| --------------- | ------------------------- | -------- |
| `Authorization` | `Bearer {{access_token}}` | Yes      |
| `Content-Type`  | `application/json`        | Yes      |
| `X-Request-ID`  | `<uuid>`                  | Yes      |

**Query Parameters**

| Parameter         | Type    | Required | Description                                         |
| ----------------- | ------- | -------- | --------------------------------------------------- |
| `page`            | integer | No       | Page number (default: 1)                            |
| `limit`           | integer | No       | Items per page (default: 10)                        |
| `search`          | string  | No       | Search by school name                               |
| `schoolStatus`    | string  | No       | Filter by status: `requesting`, `rejected`, `valid` |
| `stateId`         | string  | No       | Filter by state UUID                                |
| `districtId`      | string  | No       | Filter by district UUID                             |
| `townshipId`      | string  | No       | Filter by township UUID                             |
| `includeTeachers` | boolean | No       | Include teacher list (default: false)               |
| `includeDrafts`   | boolean | No       | Include draft schools (default: false)              |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "id": "sch_001",
      "name": "Lincoln High School",
      "nameMm": "Lincoln High School",
      "abbreviation": "LHS",
      "abbreviationMm": null,
      "schoolStatus": "valid",
      "schoolSystem": { "id": "gov", "name": "Government School" },
      "state": { "id": "ayey", "name": "Ayeyarwady Region" },
      "district": { "id": "hin", "name": "Hinthada District" },
      "township": { "id": "hin_01", "name": "Hinthada" },
      "postcode": "1234"
    },
    {
      "id": "sch_002",
      "name": "Oakridge Elementary",
      "nameMm": "Oakridge Elementary",
      "abbreviation": "OE",
      "abbreviationMm": null,
      "schoolStatus": "valid",
      "schoolSystem": { "id": "gov", "name": "Government School" },
      "state": { "id": "ygn", "name": "Yangon Region" },
      "district": { "id": "ygn_01", "name": "Yangon District" },
      "township": { "id": "kamayut", "name": "Kamayut" },
      "postcode": "11001"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 1248,
    "totalPages": 125
  },
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description              |
| ------ | ----------------------- | ------------------------ |
| `400`  | `VALIDATION_ERROR`      | Invalid query parameters |
| `401`  | `UNAUTHORIZED`          | Missing or invalid token |
| `403`  | `FORBIDDEN`             | Insufficient role        |
| `429`  | `RATE_LIMIT_EXCEEDED`   | Rate limit exceeded      |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault  |

## Error Codes

| Code                    | HTTP Status | Description                   |
| ----------------------- | ----------- | ----------------------------- |
| `VALIDATION_ERROR`      | 400         | Invalid query parameter value |
| `UNAUTHORIZED`          | 401         | Missing or invalid token      |
| `FORBIDDEN`             | 403         | Insufficient role             |
| `RATE_LIMIT_EXCEEDED`   | 429         | Too many requests             |
| `INTERNAL_SERVER_ERROR` | 500         | Unexpected server error       |
