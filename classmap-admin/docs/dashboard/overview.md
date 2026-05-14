# Dashboard Overview – Dashboard

![Dashboard](../images/dashboard/Dashboard.png)

## Flow

```
Admin logs in
     |
     v
GET /api/v1/admin/dashboard/overview   <-- summary stats + action-required items
     |
     v
GET /api/v1/admin/dashboard/attendance  <-- daily attendance cards
```

## Endpoints

- [GET `/api/v1/admin/dashboard/overview`](#1-get-dashboard-overview) — Summary stats and action-required items
- [GET `/api/v1/admin/dashboard/attendance`](#2-get-dashboard-daily-attendance) — Daily attendance list across all schools

---

### 1. Get Dashboard Overview

**GET** `/api/v1/admin/dashboard/overview`

**Headers**

| Key             | Value                     | Required |
| --------------- | ------------------------- | -------- |
| `Authorization` | `Bearer {{access_token}}` | Yes      |
| `Content-Type`  | `application/json`        | Yes      |
| `X-Request-ID`  | `<uuid>`                  | Yes      |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "summary": {
      "totalSchool": 1248,
      "totalSchoolChange": -12,
      "activePlans": 4892,
      "activePlansChange": 1050,
      "totalSurvey": 342,
      "surveyPending": 125,
      "surveyComplete": 214,
      "totalTeacher": 3456,
      "teacherMale": 1343,
      "teacherFemale": 1374,
      "teacherSwd": 14
    },
    "actionRequired": [
      {
        "id": "act_001",
        "type": "SCHOOL_DEVELOPMENT_PLAN",
        "priority": "high",
        "title": "School Development Plan Request",
        "schoolName": "Lincoln Elementary School",
        "daysAgo": 3,
        "actionUrl": "/schools/sch_001/plans/plan_001"
      },
      {
        "id": "act_002",
        "type": "NEW_SCHOOL_REVIEW",
        "priority": "medium",
        "title": "New School Review",
        "schoolName": "Oakridge Elementary",
        "daysAgo": 1,
        "actionUrl": "/schools/sch_002"
      },
      {
        "id": "act_003",
        "type": "TEACHER_VERIFICATION",
        "priority": "low",
        "title": "Teacher Verification Pending",
        "description": "New teacher profile requires profile verification",
        "actionUrl": "/schools/sch_003/teachers/tch_001"
      }
    ]
  },
  "meta": null,
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description              |
| ------ | ----------------------- | ------------------------ |
| `401`  | `UNAUTHORIZED`          | Missing or invalid token |
| `403`  | `FORBIDDEN`             | Insufficient role        |
| `429`  | `RATE_LIMIT_EXCEEDED`   | Rate limit exceeded      |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault  |

---

### 2. Get Dashboard Daily Attendance

**GET** `/api/v1/admin/dashboard/attendance`

**Headers**

| Key             | Value                     | Required |
| --------------- | ------------------------- | -------- |
| `Authorization` | `Bearer {{access_token}}` | Yes      |
| `Content-Type`  | `application/json`        | Yes      |
| `X-Request-ID`  | `<uuid>`                  | Yes      |

**Query Parameters**

| Parameter  | Type    | Required | Description                  |
| ---------- | ------- | -------- | ---------------------------- |
| `page`     | integer | No       | Page number (default: 1)     |
| `limit` | integer | No       | Items per page (default: 10) |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    {
      "id": "att_001",
      "schoolName": "Ngat Pyae Taw or Dhamma Lin Kar Ya",
      "teacherName": "Thu Zarkhin",
      "classTime": "Day School (9:00 AM - 3:00 PM)",
      "customClassStartTime": null,
      "customClassEndTime": null,
      "date": "2026-05-05",
      "isReviewed": true,
      "note": null,
      "maleCount": 45,
      "femaleCount": 52,
      "swdCount": 8,
      "schoolMaleCount": 48,
      "schoolFemaleCount": 55,
      "schoolSwdCount": 9
    }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 42,
    "totalPages": 5
  },
  "error": null,
  "message": "Successfully"
}
```

**Response – 4xx / 5xx**

| Status | Error Code              | Description              |
| ------ | ----------------------- | ------------------------ |
| `401`  | `UNAUTHORIZED`          | Missing or invalid token |
| `403`  | `FORBIDDEN`             | Insufficient role        |
| `429`  | `RATE_LIMIT_EXCEEDED`   | Rate limit exceeded      |
| `500`  | `INTERNAL_SERVER_ERROR` | Unexpected server fault  |

## Error Codes

| Code                    | HTTP Status | Description              |
| ----------------------- | ----------- | ------------------------ |
| `UNAUTHORIZED`          | 401         | Missing or invalid token |
| `FORBIDDEN`             | 403         | Insufficient role        |
| `RATE_LIMIT_EXCEEDED`   | 429         | Too many requests        |
| `INTERNAL_SERVER_ERROR` | 500         | Unexpected server error  |
