# Class Map Admin – API Documentation

Education Management Information System (EMIS) — Admin Web Portal

## Available APIs

| Module | File | Endpoints |
|---|---|---|
| Auth / Login | [auth/login.md](auth/login.md) | 2 |
| Dashboard / Overview | [dashboard/overview.md](dashboard/overview.md) | 2 |
| Schools / List | [schools/list.md](schools/list.md) | 1 |
| Schools / Detail & Analytics | [schools/detail.md](schools/detail.md) | 3 |
| Schools / Daily Attendance | [schools/attendance.md](schools/attendance.md) | 1 |
| Schools / Classes | [schools/classes.md](schools/classes.md) | 4 |
| Schools / Plans | [schools/plans.md](schools/plans.md) | 7 |
| Schools / Infrastructure | [schools/infrastructure.md](schools/infrastructure.md) | 2 |
| Schools / Gallery | [schools/gallery.md](schools/gallery.md) | 1 |
| Schools / Daily Logs | [schools/daily-logs.md](schools/daily-logs.md) | 1 |
| Schools / Teachers | [schools/teachers.md](schools/teachers.md) | 6 |

**Total: 30 endpoints**

## Postman Collection

`../classmap-admin.postman_collection.json` — import into Postman and set `base_url` + `access_token`.

## Standards

All endpoints follow the rules in `../../yoma-api-skill.md`:
- Standard response envelope `{ success, data, meta, error, message }`
- JWT Bearer auth via `Authorization` header
- `X-Request-ID` header on every request
- RBAC roles: `superadmin`, `admin`, `user`, `service`, `readonly-svc`
