---
description: Convert UI screenshots into API docs markdown and a Postman collection. Usage: /api-doc <image-folder-path>
---

# /api-doc — UI Screenshot → API Docs + Postman Collection

Convert UI screenshots into API documentation markdown and a Postman collection.

**Usage:** `/api-doc <image-folder-path>`

**Example:**
```
/api-doc better-mobile-mock-apidocs/image/user-level2/personalInfo
/api-doc better-admin-mock-api-docs/images/loan-management
```

---

## Your job — follow every step in order, do not skip any

### STEP 0 — Read the rules

Before anything else, read the backend rules file:

```
better-mobile-mock-apidocs/yoma-api-skill.md
```

Every API doc you generate MUST follow those rules: response envelope, HTTP status codes, error codes, RBAC roles, middleware chain, and request headers.

---

### STEP 1 — Detect app type and resolve paths

From `$ARGUMENTS` (the image folder path), determine:

| Signal in path | App type | Docs root | Postman output |
|---|---|---|---|
| `better-mobile-mock-apidocs` | Mobile | `better-mobile-mock-apidocs/docs/` | `better-mobile-mock-apidocs/postman-collection.json` |
| `better-admin-mock-api-docs` | Admin Web | `better-admin-mock-api-docs/docs/` | `better-admin-mock-api-docs/<module>.postman_collection.json` |

Extract the **module name** from the last folder segment of the image path.
Examples:
- `image/user-level2/personalInfo` → module = `personalInfo`, level = `user-level2`
- `images/loan-management` → module = `loan-management`

**Docs output path rules:**
- Mobile: `better-mobile-mock-apidocs/docs/<level>/<module>.md`  
  e.g. `docs/user-level2/personalInfo.md`
- Admin: `better-admin-mock-api-docs/docs/<module>/<action>.md` (one file per endpoint action, matching the existing pattern)

---

### STEP 2 — Read every image in the folder

Use the Read tool to open every image file in `$ARGUMENTS`. Read them all before writing anything.

For each image, identify:
- Screen title / page name
- All form fields (label, type, required/optional)
- All buttons and their actions
- Any displayed data fields (for detail/view screens)
- Any status labels, dropdowns, toggles, file upload areas
- Navigation elements (Back, Next, Submit, tabs)

Build a complete mental model of the full user flow across all screens.

---

### STEP 3 — Design the API contract

From your screen analysis, derive every API endpoint needed. For each endpoint fill in:

| Field | Value |
|---|---|
| Method | GET / POST / PUT / PATCH / DELETE |
| Path | `/api/v1/{resource}/...` |
| Auth | Bearer token required? Which role? |
| Request fields | list all fields, types, required/optional |
| Success response | status code + body shape |
| Error responses | all expected error cases |

Apply the HTTP method selection table from the skill file:
- Fetch data → GET 200
- Create → POST 201
- Full replace → PUT 200
- Partial update → PATCH 200
- Remove → DELETE 204

Apply the full status code checklist from the skill file for each method.

---

### STEP 4 — Check existing docs to avoid duplication

Before writing, check if a docs file already exists at the output path. If it does, read it and extend it rather than overwriting.

Also check if related endpoints (same resource, different action) are already documented nearby and use consistent naming.

---

### STEP 5 — Write the API docs markdown

Write the markdown file to the output path determined in Step 1.

**Required structure** (match the existing docs style):

```markdown
# <Screen Title> – <Module Name>

![<Screen 1 label>](<relative path to image>)
![<Screen 2 label>](<relative path to image>)
...

## Flow

[ASCII flow diagram]

## Endpoints

- [METHOD `/path`](#anchor) — short description
...

---

### 1. <Endpoint Name>
**METHOD** `/path`

**Headers**
**Request Body** (table of fields, then JSON example)
**Response – 2xx**
**Response – 4xx / 5xx**
...

## Error Codes

| Code | HTTP Status | Description |
```

Image paths must be relative from the docs file location. Calculate the correct `../` depth.

Response bodies must use the standard envelope from the skill file:
```json
{
  "success": true/false,
  "data": {},
  "meta": null,
  "error": null,
  "message": "..."
}
```

---

### STEP 6 — Rebuild the full Postman collection from ALL docs

**Do not just append the new module.** After writing the new docs file, rebuild the entire Postman collection from scratch by reading every `.md` file under the docs root. This keeps the collection permanently in sync — no endpoint documented in any `.md` file can be missing.

#### 6a — Discover all docs files

Find every `.md` file under the docs root (recursive, exclude `index.md`):
- Mobile: all files under `better-mobile-mock-apidocs/docs/`
- Admin: all files under `better-admin-mock-api-docs/docs/`

Read each file. Process them in a consistent order (alphabetical by path).

#### 6b — Extract endpoints from each file

For each `.md` file parse:

1. **Folder name** — derive from the file path + `# Title` heading.
   Use this mapping pattern (file path → Postman folder name):
   - `docs/login.md` → `"Auth / Login"`
   - `docs/user-level1/register.md` → `"Auth / Register"`
   - `docs/user-level1/forgot.md` → `"Auth / Forgot Password"`
   - `docs/user-level2/personalInfo.md` → `"User / KYC Level 2"`
   - `docs/home-screen.md` → `"Retail / Home Screen"`
   - `docs/product-detail.md` → `"Retail / Product Detail"`
   - For any new file: derive the folder name from the `# Title – Module` heading at the top of the file.

2. **Each endpoint** — every `### N. <Name>` section becomes one Postman request:
   - **Method + path**: read from the `**METHOD** \`/path\`` line immediately after the heading
   - **Auth required**: present if the Headers block contains `Authorization`; omit the Authorization header for public/optional-auth endpoints (or include it as optional)
   - **Request body**: read from the `**Request Body**` table + JSON code block; use `"mode": "raw"` for JSON, `"mode": "formdata"` for multipart
   - **Path variables**: any `{id}` or `{param}` in the path becomes a `{{variable}}` reference (e.g. `{id}` → `{{product_id}}`)
   - **Query parameters**: read from the **Query Parameters** table; add each as a `query` array entry in `url`
   - **Saved responses**: every `**Response – 2xx**` and `**Response – 4xx / 5xx**` table row with a JSON block becomes one entry in the `response` array

#### 6c — Build and write the collection

Assemble the full collection JSON and **overwrite** the existing file:

```json
{
  "info": {
    "name": "Better Mobile – Full API",
    "description": "Auto-rebuilt from all docs under better-mobile-mock-apidocs/docs/. Covers: <comma-separated list of all modules>.",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "<Folder name derived in 6b>",
      "item": [
        {
          "name": "<Endpoint Name>",
          "request": {
            "method": "POST",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "Authorization", "value": "Bearer {{access_token}}" },
              { "key": "X-Request-ID", "value": "{{$guid}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "<JSON body as string>",
              "options": { "raw": { "language": "json" } }
            },
            "url": {
              "raw": "{{base_url}}/path",
              "host": ["{{base_url}}"],
              "path": ["path", "segments"]
            }
          },
          "response": [
            {
              "name": "<status> – <label>",
              "originalRequest": { },
              "status": "OK",
              "code": 200,
              "header": [{ "key": "Content-Type", "value": "application/json" }],
              "body": "<response JSON as string>",
              "cookie": []
            }
          ]
        }
      ]
    }
  ],
  "variable": [
    { "key": "base_url",    "value": "http://localhost:8080", "type": "string" },
    { "key": "access_token","value": "",                     "type": "string" }
  ]
}
```

**Variable rules** — scan all paths across all docs and add a collection variable for every `{param}` placeholder found:

| Path token | Variable key | Default value |
|---|---|---|
| `{id}` (generic) | `resource_id` | `""` |
| `{id}` under `/products/` | `product_id` | `"prod_002"` |
| Any other `{param}` | `<param>` | `""` |
| Always include | `base_url` | `"http://localhost:8080"` |
| Always include | `access_token` | `""` |
| Always include | `refresh_token` | `""` |
| Present if used | `registration_token`, `reset_token`, `kyc_session_token` | `""` |

**Additional rules:**
- One Postman folder per docs file (not per `###` section) — use the folder name from 6b
- One Postman request per `###` endpoint section within that folder
- Every `**Response –`  block in the markdown becomes a saved example in `response[]`
- Multipart endpoints (`**multipart/form-data**` or file upload fields in the table): use `"mode": "formdata"`; file fields use `{ "key": "field", "value": "", "type": "file", "src": "" }`
- Use `{{base_url}}` and `{{access_token}}` variables throughout
- Always add `{ "key": "X-Request-ID", "value": "{{$guid}}" }` to every request header

---

### STEP 7 — Update the index

For **mobile**: update `better-mobile-mock-apidocs/docs/index.md` — add the new module to the Available APIs table if it is not already listed.

For **admin**: check if there is an index or README at the admin docs root and update it similarly.

---

### STEP 8 — Report back

When done, output a summary table:

| Output | Path | Endpoints |
|---|---|---|
| API Docs | `path/to/file.md` | N endpoints |
| Postman | `path/to/file.json` | N requests |

List each endpoint: `METHOD /path — description`
