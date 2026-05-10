# Class Map – API Documentation

Welcome to the Class Map API documentation. This site covers both the **Admin Portal** and **Mobile App** REST APIs.

## Platforms

<div class="grid cards" markdown>

-   :material-web:{ .lg .middle } **Admin Portal**

    ---

    Web-based dashboard for school administrators, UNESCO partners, and system operators.

    [View Admin Docs](admin/index.md)

-   :material-cellphone:{ .lg .middle } **Mobile App**

    ---

    Android & iOS app for school principals and teachers. OTP-based login, school registration, and teacher registration.

    [View Mobile Docs](mobile/index.md)

</div>

## Standards

All endpoints across both platforms follow these conventions:

=== "Response Envelope"

    ```json
    {
      "success": true,
      "data": {},
      "meta": null,
      "error": null,
      "message": "..."
    }
    ```

=== "Error Envelope"

    ```json
    {
      "success": false,
      "data": null,
      "error": {
        "code": "ERROR_CODE",
        "details": "Human-readable description"
      },
      "message": "Error message"
    }
    ```

=== "Required Headers"

    | Header | Description |
    |---|---|
    | `Authorization: Bearer <token>` | JWT — required on all authenticated routes |
    | `X-Request-ID` | Trace ID — include on every request |
    | `Content-Type: application/json` | Required for POST / PATCH / PUT |

## Postman Collections

| Platform | File |
|---|---|
| Admin | `classmap-admin.postman_collection.json` |
| Mobile | `classmap-mobile.postman_collection.json` |
