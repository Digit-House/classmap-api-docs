# School Detail – General & Analytics

![School Analytics Tab](../images/school/analytics.png)
![School General Tab](../images/school/detail/general.png)

## Flow

```
Admin clicks school from list
        |
        v
GET /api/v1/admin/schools/{id}           <-- General tab (default)
        |
        v
GET /api/v1/admin/schools/{id}/analytics <-- Analytics tab
        |
Admin clicks "Edit Profile"
        |
        v
PATCH /api/v1/admin/schools/{id}         <-- Save school profile changes
```

## Endpoints

- [GET `/api/v1/admin/schools/{id}`](#1-get-school-detail) — Full school profile (General tab)
- [GET `/api/v1/admin/schools/{id}/analytics`](#2-get-school-analytics) — Analytics charts and stats
- [PATCH `/api/v1/admin/schools/{id}`](#3-update-school-profile) — Update school profile

---

### 1. Get School Detail
**GET** `/api/v1/admin/schools/{id}`

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

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_001",
    "identity": {
      "name_en": "Lincoln High School",
      "name_mm": "Lincoln High School",
      "abbreviation_en": "LHS",
      "abbreviation_mm": "LHS",
      "school_system": "Government School",
      "review_status": "verified",
      "unesco_partner": "CH019320",
      "school_code": "O1938DJ&",
      "registration_date": "2023-08-15T09:30:00Z",
      "last_updated": "2024-05-20T14:45:00Z"
    },
    "location": {
      "state": "Ayeyarwady Region",
      "district": "Hinthada District",
      "postcode": "1234"
    },
    "contact": {
      "primary_email": "contact@classmap.edu"
    },
    "education_levels": {
      "offering": ["kindergarten_plus5", "primary", "middle", "high_school"],
      "non_offering": ["monastic_school", "teacher_training", "other"]
    },
    "active_classes": [
      {
        "id": "cls_001",
        "name": "Kindergarten",
        "grade_level": "Grade 1",
        "section": "A",
        "teacher": "Daw Hla Hla",
        "academic_year": "2026 - 2027",
        "demographic": {
          "total": 35,
          "male": 18,
          "female": 15,
          "swd": 2
        }
      }
    ]
  },
  "meta": null,
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Invalid school ID format |
| `401` | `UNAUTHORIZED` | Missing or invalid token |
| `403` | `FORBIDDEN` | Insufficient role |
| `404` | `SCHOOL_NOT_FOUND` | School ID does not exist |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

---

### 2. Get School Analytics
**GET** `/api/v1/admin/schools/{id}/analytics`

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

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "summary": {
      "total_teachers": 66,
      "total_students": 1044,
      "students_with_disabilities": 47,
      "teacher_student_ratio": "1:16"
    },
    "enrollment_growth": [
      { "year": 2020, "students": 900, "teachers": 55 },
      { "year": 2021, "students": 950, "teachers": 58 },
      { "year": 2022, "students": 980, "teachers": 60 },
      { "year": 2023, "students": 1010, "teachers": 63 },
      { "year": 2024, "students": 1044, "teachers": 66 }
    ],
    "performance_metrics": [
      { "month": "Jan", "attendance": 98, "performance": 95, "satisfaction": 99 },
      { "month": "Feb", "attendance": 97, "performance": 95, "satisfaction": 100 },
      { "month": "Mar", "attendance": 98, "performance": 96, "satisfaction": 99 },
      { "month": "Apr", "attendance": 97, "performance": 95, "satisfaction": 100 },
      { "month": "May", "attendance": 98, "performance": 96, "satisfaction": 100 }
    ],
    "teacher_qualifications": {
      "masters_phd": 61,
      "bachelors": 33,
      "diploma": 6
    },
    "student_diversity": {
      "male": { "count": 485, "percentage": 46.5 },
      "female": { "count": 512, "percentage": 49.0 },
      "swd": { "count": 47, "percentage": 4.5 }
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

### 3. Update School Profile
**PATCH** `/api/v1/admin/schools/{id}`

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

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| `name_en` | string | No | English school name |
| `name_mm` | string | No | Myanmar school name |
| `abbreviation_en` | string | No | English abbreviation |
| `abbreviation_mm` | string | No | Myanmar abbreviation |
| `school_system` | string | No | School system type |
| `unesco_partner` | string | No | UNESCO partner code |
| `state` | string | No | State/Region |
| `district` | string | No | District |
| `postcode` | string | No | Postal code |
| `primary_email` | string | No | Contact email |
| `education_levels_offering` | array | No | List of offering education levels |
| `education_levels_non_offering` | array | No | List of non-offering education levels |

```json
{
  "name_en": "Lincoln High School",
  "name_mm": "Lincoln High School",
  "abbreviation_en": "LHS",
  "state": "Ayeyarwady Region",
  "district": "Hinthada District",
  "postcode": "1234",
  "primary_email": "contact@classmap.edu",
  "education_levels_offering": ["kindergarten_plus5", "primary", "middle", "high_school"]
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_001",
    "name_en": "Lincoln High School",
    "last_updated": "2026-05-08T10:00:00Z"
  },
  "meta": null,
  "error": null,
  "message": "School profile updated successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code | Description |
|---|---|---|
| `400` | `VALIDATION_ERROR` | Invalid input fields |
| `401` | `UNAUTHORIZED` | Missing or invalid token |
| `403` | `FORBIDDEN` | Insufficient role |
| `404` | `SCHOOL_NOT_FOUND` | School ID does not exist |
| `409` | `CONFLICT` | Concurrent update conflict |
| `422` | `BUSINESS_RULE_VIOLATION` | Business rule validation failed |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded |
| `500` | `INTERNAL_SERVER_ERROR` | Unexpected server fault |

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `VALIDATION_ERROR` | 400 | Invalid or missing fields |
| `UNAUTHORIZED` | 401 | Missing or invalid token |
| `FORBIDDEN` | 403 | Insufficient role |
| `SCHOOL_NOT_FOUND` | 404 | School not found |
| `CONFLICT` | 409 | Concurrent update conflict |
| `BUSINESS_RULE_VIOLATION` | 422 | Business rule validation failed |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_SERVER_ERROR` | 500 | Unexpected server error |
