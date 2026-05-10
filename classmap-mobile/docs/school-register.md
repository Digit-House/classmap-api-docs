# School Registration – School Register

![Step 1 School Info](../images/school-register/school-reg-step1.png)
![Step 2 Proof of Document](../images/school-register/school-reg-step2.png)
![Step 2 Filled](../images/school-register/school-reg-step2-filled.png)
![Step 3 Account Setup](../images/school-register/school-reg-step3.png)
![Step 4 Preview & Submit](../images/school-register/school-reg-step4.png)
![Registration Success](../images/school-register/school-reg-success.png)

## Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌─────────┐
│ School Info  │────▶│ Proof of     │────▶│ First Teacher    │────▶│ Preview &        │────▶│ Pending │
│ (Step 1)     │     │ Document     │     │ Account Setup    │     │ Submit (Step 4)  │     │ Review  │
│              │     │ (Step 2)     │     │ (Step 3)         │     │                  │     │         │
└──────────────┘     └──────────────┘     └──────────────────┘     └──────────────────┘     └─────────┘
```

## Endpoints

- [GET `/api/v1/school-systems`](#1-list-school-systems) — List available school systems
- [GET `/api/v1/unesco-partners`](#2-list-unesco-partners) — List UNESCO partners
- [GET `/api/v1/locations/states`](#3-list-states) — List states
- [GET `/api/v1/locations/districts`](#4-list-districts) — List districts by state
- [GET `/api/v1/locations/townships`](#5-list-townships) — List townships by district
- [GET `/api/v1/locations/postcodes`](#6-list-postcodes) — List postcodes by township
- [POST `/api/v1/school-registrations`](#7-create-school-registration) — Step 1: Submit school info
- [PUT `/api/v1/school-registrations/{id}/documents`](#8-upload-documents) — Step 2: Upload proof documents
- [PUT `/api/v1/school-registrations/{id}/admin-account`](#9-setup-admin-account) — Step 3: Setup first teacher account
- [GET `/api/v1/school-registrations/{id}`](#10-get-registration-preview) — Step 4: Preview all data
- [POST `/api/v1/school-registrations/{id}/submit`](#11-submit-registration) — Final submission for review

---

### 1. List School Systems
**GET** `/api/v1/school-systems`

Fetch all available school systems for the dropdown.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "gov", "name": "Government School" },
    { "id": "private", "name": "Private School" },
    { "id": "international", "name": "International School" }
  ],
  "meta": null,
  "error": null,
  "message": "School systems retrieved"
}
```

---

### 2. List UNESCO Partners
**GET** `/api/v1/unesco-partners`

Fetch all UNESCO partners for the optional dropdown.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "rise", "name": "Rural Indigenous Sustainable Education" },
    { "id": "unesco_mm", "name": "UNESCO Myanmar" }
  ],
  "meta": null,
  "error": null,
  "message": "UNESCO partners retrieved"
}
```

---

### 3. List States
**GET** `/api/v1/locations/states`

Fetch all states for the location dropdown.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "ygn", "name": "Yangon" },
    { "id": "mdy", "name": "Mandalay" }
  ],
  "meta": null,
  "error": null,
  "message": "States retrieved"
}
```

---

### 4. List Districts
**GET** `/api/v1/locations/districts?state_id={state_id}`

Fetch districts filtered by state.

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| state_id | string | Yes | State ID from list states |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "ygn_01", "name": "Yangon", "state_id": "ygn" }
  ],
  "meta": null,
  "error": null,
  "message": "Districts retrieved"
}
```

---

### 5. List Townships
**GET** `/api/v1/locations/townships?district_id={district_id}`

Fetch townships filtered by district.

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| district_id | string | Yes | District ID from list districts |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "kamayut", "name": "Kamayut", "district_id": "ygn_01" }
  ],
  "meta": null,
  "error": null,
  "message": "Townships retrieved"
}
```

---

### 6. List Postcodes
**GET** `/api/v1/locations/postcodes?township_id={township_id}`

Fetch postcodes filtered by township.

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| township_id | string | Yes | Township ID from list townships |

**Response – 200 OK**

```json
{
  "success": true,
  "data": [
    { "id": "100293", "code": "100293", "township_id": "kamayut" }
  ],
  "meta": null,
  "error": null,
  "message": "Postcodes retrieved"
}
```

---

### 7. Create School Registration
**POST** `/api/v1/school-registrations`

Step 1: Submit school information and create a registration draft.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| school_system_id | string | Yes | Selected school system ID |
| unesco_partner_id | string | No | Selected UNESCO partner ID |
| full_name_en | string | Yes | School full name in English |
| full_name_mm | string | Yes | School full name in Myanmar |
| abbreviation_en | string | Yes | Abbreviation in English |
| abbreviation_mm | string | Yes | Abbreviation in Myanmar |
| state_id | string | Yes | State ID |
| district_id | string | Yes | District ID |
| township_id | string | Yes | Township ID |
| postcode_id | string | Yes | Postcode ID |

```json
{
  "school_system_id": "gov",
  "unesco_partner_id": "rise",
  "full_name_en": "Basic Education High School No. 2 Kamayut",
  "full_name_mm": "အခြေခံပညာအထက်တန်းကျောင်း (၂) ကမာရွတ်",
  "abbreviation_en": "Kamayut 2",
  "abbreviation_mm": "ကမာရွတ်(၂)",
  "state_id": "ygn",
  "district_id": "ygn_01",
  "township_id": "kamayut",
  "postcode_id": "100293"
}
```

**Response – 201 Created**

```json
{
  "success": true,
  "data": {
    "id": "sch_reg_001",
    "status": "draft",
    "current_step": 1,
    "created_at": "2026-05-10T06:30:00Z"
  },
  "meta": null,
  "error": null,
  "message": "School registration created"
}
```

**Response – 400 Bad Request**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "details": { "full_name_en": "This field is required" }
  },
  "message": "Validation failed"
}
```

---

### 8. Upload Documents
**PUT** `/api/v1/school-registrations/{id}/documents`

Step 2: Upload proof of document photos and notes. Uses `multipart/form-data`.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | multipart/form-data | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Path Variables**

| Variable | Description |
|---|---|
| id | School registration ID from step 1 |

**Request Body (multipart)**

| Field | Type | Required | Description |
|---|---|---|---|
| photos | file[] | Yes | JPEG/PNG, max 5MB each, up to 5 photos |
| notes | string | Yes | Notes to UNESCO reviewer |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_reg_001",
    "status": "draft",
    "current_step": 2,
    "documents": [
      { "id": "doc_1", "url": "https://cdn.example.com/doc_1.jpg" },
      { "id": "doc_2", "url": "https://cdn.example.com/doc_2.jpg" }
    ],
    "notes": "This is an example notes."
  },
  "meta": null,
  "error": null,
  "message": "Documents uploaded"
}
```

**Response – 400 Bad Request**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INVALID_FILE",
    "details": "Photos must be JPEG or PNG, max 5MB each"
  },
  "message": "Invalid file format"
}
```

**Response – 404 Not Found**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "REGISTRATION_NOT_FOUND",
    "details": "School registration not found"
  },
  "message": "Registration not found"
}
```

---

### 9. Setup Admin Account
**PUT** `/api/v1/school-registrations/{id}/admin-account`

Step 3: Create the first teacher/admin account for the school.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Path Variables**

| Variable | Description |
|---|---|
| id | School registration ID |

**Request Body**

| Field | Type | Required | Description |
|---|---|---|---|
| displayed_name | string | Yes | Name shown in the app |
| phone_number | string | Yes | E.164 format for SMS login |

```json
{
  "displayed_name": "Daw Mya Mya",
  "phone_number": "+959123456789"
}
```

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_reg_001",
    "status": "draft",
    "current_step": 3,
    "admin_account": {
      "displayed_name": "Daw Mya Mya",
      "phone_number": "+959123456789"
    }
  },
  "meta": null,
  "error": null,
  "message": "Admin account configured"
}
```

**Response – 409 Conflict**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "PHONE_ALREADY_USED",
    "details": "This phone number is already associated with another account"
  },
  "message": "Phone number already in use"
}
```

---

### 10. Get Registration Preview
**GET** `/api/v1/school-registrations/{id}`

Step 4: Retrieve the full registration data for preview before submission.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Path Variables**

| Variable | Description |
|---|---|
| id | School registration ID |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_reg_001",
    "status": "draft",
    "current_step": 4,
    "school_info": {
      "school_system": "Government School",
      "unesco_partner": "Rural Indigenous Sustainable Education",
      "full_name_en": "Basic Education High School No. 2 Kamayut",
      "full_name_mm": "အခြေခံပညာအထက်တန်းကျောင်း (၂) ကမာရွတ်",
      "abbreviation_en": "Kamayut 2",
      "abbreviation_mm": "ကမာရွတ်(၂)",
      "state": "Yangon",
      "district": "Yangon",
      "township": "Kamayut",
      "postcode": "100293"
    },
    "documents": {
      "photos": [
        { "id": "doc_1", "url": "https://cdn.example.com/doc_1.jpg" }
      ],
      "notes": "This is an example notes."
    },
    "admin_account": {
      "displayed_name": "Daw Mya Mya",
      "phone_number": "+959123456789"
    }
  },
  "meta": null,
  "error": null,
  "message": "Registration preview loaded"
}
```

---

### 11. Submit Registration
**POST** `/api/v1/school-registrations/{id}/submit`

Final submission. Status changes to `pending_review`. An SMS notification will be sent once approved.

**Headers**

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| X-Request-ID | {{$guid}} | Yes |

**Path Variables**

| Variable | Description |
|---|---|
| id | School registration ID |

**Response – 200 OK**

```json
{
  "success": true,
  "data": {
    "id": "sch_reg_001",
    "status": "pending_review",
    "submitted_at": "2026-05-10T07:00:00Z"
  },
  "meta": null,
  "error": null,
  "message": "Registration submitted for review"
}
```

**Response – 422 Unprocessable Entity**

```json
{
  "success": false,
  "data": null,
  "meta": null,
  "error": {
    "code": "INCOMPLETE_REGISTRATION",
    "details": "All 4 steps must be completed before submission"
  },
  "message": "Incomplete registration"
}
```

## Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| VALIDATION_ERROR | 400 | Required field missing or invalid |
| INVALID_FILE | 400 | File type or size not allowed |
| REGISTRATION_NOT_FOUND | 404 | School registration ID not found |
| PHONE_ALREADY_USED | 409 | Phone number already registered |
| INCOMPLETE_REGISTRATION | 422 | Missing steps before submission |
