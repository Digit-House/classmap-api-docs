# 🚀 Yoma Fleet — API Endpoint Creation Skill

> Follow this skill file top-to-bottom every time you create a new API endpoint.  
> Each section is a checkpoint. Do not skip any step.

---

## STEP 1 — Define the Contract First

Before writing any code, define the contract.

### 1.1 Fill in the Endpoint Card

| Field | Value |
|---|---|
| **Method** | `GET` / `POST` / `PUT` / `PATCH` / `DELETE` |
| **Path** | `/api/v1/{resource}/{:id}` |
| **Resource Name** | e.g. `User`, `Vehicle`, `Booking` |
| **Summary** | One-line description |
| **Tag / Group** | e.g. `users`, `vehicles` |
| **Layer** | `Public REST` / `Private gRPC` |
| **Allowed Roles** | `superadmin` / `admin` / `user` / `service` / `readonly-svc` |

### 1.2 Choose the Correct HTTP Method

| Intent | Method | Success Code |
|---|---|---|
| Fetch resource(s) | `GET` | `200 OK` |
| Create new resource | `POST` | `201 Created` |
| Full replace | `PUT` | `200 OK` |
| Partial update | `PATCH` | `200 OK` |
| Remove resource | `DELETE` | `204 No Content` |

### 1.3 Write the OpenAPI Annotation (before coding)

```go
// @Summary      <one-line summary>
// @Description  <detail>
// @Tags         <tag>
// @Accept       json
// @Produce      json
// @Param        id   path      string  true  "Resource UUID"  format(uuid)
// @Success      200  {object}  response.XxxResponse
// @Failure      400  {object}  response.ErrorResponse
// @Failure      401  {object}  response.ErrorResponse
// @Failure      403  {object}  response.ErrorResponse
// @Failure      404  {object}  response.ErrorResponse
// @Failure      429  {object}  response.ErrorResponse
// @Failure      500  {object}  response.ErrorResponse
// @Security     BearerAuth
// @Router       /api/v1/{resource}/{id} [get]
```

> **Rule:** The annotation is the source of truth. Code must match it.

---

## STEP 2 — HTTP Status Code Checklist

Pick every code your endpoint must handle. Check off as you implement.

### By Method

**GET**
- [ ] `200` — resource returned
- [ ] `400` — bad query param / invalid UUID
- [ ] `401` — missing / invalid token
- [ ] `403` — valid token, wrong role
- [ ] `404` — ID not found
- [ ] `429` — rate limit exceeded
- [ ] `500` — unexpected server fault

**POST**
- [ ] `201` — resource created
- [ ] `400` — validation failure / missing fields
- [ ] `401` — missing / invalid token
- [ ] `403` — insufficient role
- [ ] `409` — duplicate / conflict
- [ ] `422` — business rule violation
- [ ] `429` — rate limit exceeded
- [ ] `500` — unexpected server fault

**PUT / PATCH**
- [ ] `200` — updated resource returned
- [ ] `400` — invalid input
- [ ] `401` — missing / invalid token
- [ ] `403` — insufficient role
- [ ] `404` — resource not found
- [ ] `409` — concurrent update conflict
- [ ] `422` — business rule violation
- [ ] `429` — rate limit exceeded
- [ ] `500` — unexpected server fault

**DELETE**
- [ ] `204` — deleted, no body
- [ ] `401` — missing / invalid token
- [ ] `403` — insufficient role
- [ ] `404` — resource not found
- [ ] `429` — rate limit exceeded
- [ ] `500` — unexpected server fault

---

## STEP 3 — Response Envelope

Every response **must** use the standard envelope. No raw objects.

### Success Response

```json
{
  "success": true,
  "data":    { },
  "meta":    { "page": 1, "per_page": 20, "total": 150 },
  "error":   null,
  "message": "Successfully"
}
```

> `meta` is `null` for non-paginated responses.  
> `data` is `null` for `204 No Content`.

### Error Response

```json
{
  "success": false,
  "data":    null,
  "error": {
    "code":       "RESOURCE_NOT_FOUND",
    "message":    "The requested resource does not exist.",
    "request_id": "a1b2c3d4-..."
  }
}
```

### 429 Rate Limit Response

```json
{
  "success": false,
  "data":    null,
  "error": {
    "code":        "RATE_LIMIT_EXCEEDED",
    "message":     "Too many requests. Retry after 60 seconds.",
    "retry_after": 60
  }
}
```

> ⚠️ **Required:** `Retry-After` and `X-RateLimit-Reset` headers must be present on **all** `429` responses.

### Standard Error Codes

| HTTP | Error Code |
|---|---|
| `400` | `VALIDATION_ERROR` |
| `401` | `UNAUTHORIZED` |
| `403` | `FORBIDDEN` |
| `404` | `{RESOURCE}_NOT_FOUND` |
| `409` | `CONFLICT` |
| `422` | `BUSINESS_RULE_VIOLATION` |
| `429` | `RATE_LIMIT_EXCEEDED` |
| `500` | `INTERNAL_SERVER_ERROR` |

---

## STEP 4 — Middleware Chain

Every endpoint automatically passes through this chain (do not re-implement manually).

```
[1] Request ID        → generate / propagate X-Request-ID
[2] Logging           → entry log: method, path, request_id
[3] Recovery          → panic → 500, log stack trace
[4] CORS              → public layer only
[5] JWT Auth          → extract & validate claims          (timeout: 500ms)
[6] RBAC              → check role × resource × action
[7] Rate Limiter      → per-user token-bucket (Redis)
[8] Timeout           → enforce context deadline
[9] Your Handler      → business logic here
[10] Response Logger  → exit log: status, latency
```

> You only write **Step 9**. Steps 1–8 and 10 are middleware.

---

## STEP 5 — RBAC Configuration

### 5.1 Identify Roles

Check the role matrix and confirm your endpoint's role assignments:

| Role | Typical Access |
|---|---|
| `superadmin` | Full CRUD on all resources |
| `admin` | GET, POST, PUT (no DELETE) |
| `user` | Own resource only (GET, PUT) |
| `service` | Internal gRPC Read + Write |
| `readonly-svc` | Internal gRPC Read only |

### 5.2 Casbin Policy Entry

Add a policy line for each role that needs access:

```
p, superadmin, /api/v1/{resource}/*, GET|POST|PUT|DELETE
p, admin,      /api/v1/{resource}/*, GET|POST|PUT
p, user,       /api/v1/{resource}/:self, GET|PUT
```

---

## STEP 6 — Timeout Budgets

Set context deadlines on every external call inside your handler.

| Operation | Timeout | On Exceed |
|---|---|---|
| REST handler (total) | `10s` | cancel context → `503` |
| DB query — read | `3s` | cancel context → `500` |
| DB query — write | `5s` | cancel context → `500` |
| gRPC client call | `5s` | `DEADLINE_EXCEEDED` propagated |
| Outbound HTTP call | `3s` | cancel context → `502` |
| JWT validation | `500ms` | short-circuit → `401` |

```go
// Example: DB read with timeout
ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
defer cancel()
result := db.WithContext(ctx).Find(&record)
```

---

## STEP 7 — Request Headers

Your handler must respect and propagate these headers.

| Header | Direction | Description |
|---|---|---|
| `Authorization: Bearer <token>` | Inbound | JWT — validated by middleware |
| `X-Request-ID` | In + Out | Trace ID — propagate to all downstream calls |
| `Content-Type: application/json` | In + Out | Always JSON |
| `Retry-After` | Outbound (429 only) | Seconds until retry allowed |
| `X-RateLimit-Reset` | Outbound (429 only) | Unix timestamp of reset |

### For gRPC Private Layer — Required Metadata

| Key | Format |
|---|---|
| `authorization` | `Bearer <service-token>` |
| `x-request-id` | UUID |
| `x-caller-id` | `service-name@version` |

---

## STEP 8 — Logging

Use the correct log level. Do not use `fmt.Println` in handlers.

| Level | When to Use |
|---|---|
| `DEBUG` | Detailed flow — dev only, disabled in prod |
| `INFO` | Successful request completion, lifecycle events |
| `WARN` | Recoverable issues: retry, cache miss, rate limit |
| `ERROR` | Failures that need attention, service still running |
| `FATAL` | Unrecoverable startup failures → `os.Exit(1)` |

```go
// Good
log.Info().Str("request_id", reqID).Str("user_id", uid).Msg("user retrieved")
log.Error().Err(err).Str("request_id", reqID).Msg("failed to fetch user")

// Bad
fmt.Println("got user", user)
```

---

## STEP 9 — Unit Test Checklist

Every handler must have unit tests before merging.

- [ ] Test file created: `handler_test.go` (same package for white-box)
- [ ] All DB / gRPC / HTTP dependencies mocked via interfaces
- [ ] No real network, filesystem, or DB connections
- [ ] Each test uses `t.Parallel()`
- [ ] Table-driven tests with fixtures in `testdata/*.json`
- [ ] Happy path covered
- [ ] All error paths covered (not found, validation, auth, etc.)
- [ ] Coverage ≥ 80% for this package

```go
func TestGetUser_NotFound(t *testing.T) {
    t.Parallel()
    mockRepo := mocks.NewUserRepository(t)
    mockRepo.On("FindByID", mock.Anything, "bad-id").Return(nil, ErrNotFound)
    // ... assert 404 envelope response
}
```

---

## STEP 10 — Health & Observability

Ensure your service dependency is reflected in the readiness probe.

```json
// GET /health/ready
{
  "status": "ok",
  "checks": {
    "postgres":          { "status": "ok", "latency_ms": 2 },
    "redis":             { "status": "ok", "latency_ms": 1 },
    "grpc_auth_service": { "status": "ok" }
  }
}
```

- [ ] New external dependency added to `/health/ready` check
- [ ] Metrics exposed via `/metrics` endpoint

---

## STEP 11 — Versioning Check

Before merging, answer these questions:

| Question | If YES → Action |
|---|---|
| Did I add an optional field? | Non-breaking. Ship in current version. |
| Did I rename or remove a field? | Create new proto version (v2), deprecate old for 90 days. |
| Did I change the HTTP method? | Add new route, return `301` from old route. |
| Did I make a breaking gRPC change? | **BLOCKED** — `buf breaking` check will fail CI. |

---

## STEP 12 — Pre-Merge Final Checklist

Run through this before opening a PR.

### Contract
- [ ] OpenAPI annotation matches actual handler signature
- [ ] Proto doc comment includes authorization, rate limit, and error codes
- [ ] Response envelope used on all responses (success and error)

### Security
- [ ] JWT middleware active on this route
- [ ] RBAC Casbin policy entry added
- [ ] No sensitive data (password, token) returned in response body

### Reliability
- [ ] Context timeouts set on all DB and external calls
- [ ] `X-Request-ID` propagated to all downstream calls
- [ ] `429` response includes `Retry-After` + `X-RateLimit-Reset` headers

### Quality
- [ ] Unit tests written and passing
- [ ] Coverage ≥ 80% for this package
- [ ] No `fmt.Println` — all logging via structured logger
- [ ] No hardcoded secrets or config values

### Documentation
- [ ] `docs/openapi/` updated (or `swag init` run)
- [ ] `/health/ready` updated if new dependency added

---

## Quick Reference — Error Code Template

Copy-paste when writing error responses:

```go
// 400
return c.JSON(400, ErrorResponse{Code: "VALIDATION_ERROR", Message: "Invalid input or missing required fields."})

// 401
return c.JSON(401, ErrorResponse{Code: "UNAUTHORIZED", Message: "Missing or invalid authentication token."})

// 403
return c.JSON(403, ErrorResponse{Code: "FORBIDDEN", Message: "You do not have permission to perform this action."})

// 404
return c.JSON(404, ErrorResponse{Code: "USER_NOT_FOUND", Message: "The requested user does not exist."})

// 409
return c.JSON(409, ErrorResponse{Code: "CONFLICT", Message: "A resource with this value already exists."})

// 422
return c.JSON(422, ErrorResponse{Code: "BUSINESS_RULE_VIOLATION", Message: "Business rule validation failed."})

// 429 — always add headers
c.Response().Header().Set("Retry-After", "60")
c.Response().Header().Set("X-RateLimit-Reset", strconv.FormatInt(resetTime.Unix(), 10))
return c.JSON(429, RateLimitErrorResponse{Code: "RATE_LIMIT_EXCEEDED", Message: "Too many requests. Retry after 60 seconds.", RetryAfter: 60})

// 500
return c.JSON(500, ErrorResponse{Code: "INTERNAL_SERVER_ERROR", Message: "An unexpected server error occurred."})
```

---

*Yoma Fleet Backend Guidelines — Last updated April 2026*
