![IAB Tech Lab](https://drive.google.com/uc?id=10yoBoG5uRETSXRrnJPUDuONujvADrSG1)
# Live Event Ad Playbook (LEAP)
# Common API Conventions Reference

## Table of Contents

- [Overview](#overview)
- [Cross-API Identity Correlation](#cross-api_identity_correlation)
- [Naming Conventions](#naming-conventions)
- [HTTP Methods](#http-methods)
- [Authentication](#authentication)
- [Request and Response Formats](#request-and-response-formats)
- [Status Codes](#status-codes)
- [Error Handling](#error-handling)
- [Pagination](#pagination)
- [Versioning](#versioning)
- [Rate Limiting](#rate-limiting)
- [Sorting](#sorting)

---

## Overview

This document establishes common conventions and standards applicable to all Live Event Ad Playbook (LEAP) APIs. API-specific details, including endpoints, query parameters, and examples, are documented in separate API specification files.

---

## Cross-API Identity Correlation

### Purpose

The Forecast API supports pre-event planning, and the Concurrent Streams API reports near-real-time stream counts. Implementers may correlate the same live event across these APIs, OpenRTB transactions, and downstream reporting by preserving a **consistent, agreed identifier**.

### Equally Valid Correlation Methods

Either of the following may serve as the cross-API join key; neither is inherently preferred:

1. **Content ID — `content.id`:** Match the same publisher-provided Content ID across the Forecast API, Concurrent Streams API, and OpenRTB when its assignment and scope are understood and the value is consistently preserved. A Content ID is not automatically universal across unrelated publishers or sellers.

2. **Extended Content ID — `content.data[].cids[]`:** Match an Extended Content ID together with the associated `data[].name` identifying its source. Comparing only the CID string without its originating source can create false matches. This approach is useful when participants have different local `content.id` values but recognize the same external content identifier.

Both methods may appear in a response; neither requires the other to be present. When both are supplied but identify different content, implementers should resolve the discrepancy rather than silently treating the records as equivalent.

### Example Field Paths

| Context | Content ID | Extended Content ID |
|---|---|---|
| Forecast API | `events[].content.id` | `events[].content.data[].cids[]` |
| Concurrent Streams API | `streamsdata[].mediastreams[].content.id` | `streamsdata[].mediastreams[].content.data[].cids[]` |
| OpenRTB app | `app.content.id` | `app.content.data[].cids[]` |
| OpenRTB site | `site.content.id` | `site.content.data[].cids[]` |

For Extended Content IDs, the corresponding `data[].name` must also be used to identify the source. The paths above describe existing content objects, not new LEAP fields.

### Illustrative Matching Values

```json
{
  "id": "event-content-12345",
  "data": [
    {
      "name": "id-provider.example",
      "cids": ["shared-content-id-12345"]
    }
  ]
}
```

This represents the content object portion of an illustrative API response or bid request. Either identifier may support correlation independently, provided the participating systems preserve it consistently.

### Additional Implementation Considerations

- The Forecast API's event-level `id` identifies its event record and is recommended to equal `content.id` where possible, but equality is not guaranteed.
- An OpenRTB bid request's top-level `id` identifies the auction request, not the live event.
- Source-qualified Extended Content IDs should be preserved through intermediaries where permitted.
- Event timing and stable content metadata may help validate an uncertain match but are not guaranteed unique identifiers.
- This guidance clarifies existing identifier usage and does not introduce new v1.1 API requirements.

## Naming Conventions

Establishes foundational naming rules for resources, fields, and parameters across all APIs.

### Resource Naming

- Use plural nouns for collections (e.g., `/eventforecasts`, `/concurrentstreams`).
- Use lowercase for resource paths.
- Avoid verbs in resource names. Actions are expressed through HTTP methods.

### Field Naming

- Use lowercase for JSON fields to align with existing OpenRTB naming conventions and the current LEAP Forecast API and Concurrent Streams API specifications (e.g., `eventid`, `contentid`, `expectedpeak`).
- Use lowercase for query parameters to align with the JSON field naming convention (e.g., `scheduledstart`, `scheduledend`, `lastmodifieddate`).
- Boolean fields use descriptive names where possible (e.g., `tentative`, `unplanned`).

---

## HTTP Methods

Defines how HTTP verbs map to operations on API resources.

### GET - Retrieve resources (idempotent, safe)

- Collection: `GET /resources` with filtering, sorting, and pagination.
- Individual: `GET /resources/{id}`.

### POST - Create resources (non-idempotent)

- Returns `201 Created` with `Location` header pointing to the new resource.

### PUT - Replace entire resource (idempotent)

- Returns `200 OK` or `204 No Content`.

### PATCH - Partial update (idempotent)

- Modifies specified fields only.

### DELETE - Remove resource (idempotent)

- Returns `204 No Content`.

---

## Authentication

Specifies security mechanisms for API access and authorization.

Authentication protocol is left to the discretion of the event creator or API provider and should be discussed with API users before implementation. OAuth 2.0 is a common way to support secure API access and is recommended for consideration, but it is not required or prescribed by this guidance.

When OAuth 2.0 is used, include the access token in the `Authorization` header:

```http
Authorization: Bearer {token}
```

### Mechanism

OAuth 2.0 may be implemented using Client Credentials, Authorization Code grant, or another mutually agreed approach.

### Required Headers

```http
Authorization: Bearer {access_token}
Content-Type: application/json
Accept: application/json
```

### Optional Headers

```http
Idempotency-Key: {uuid}
X-Request-ID: {uuid}
```

### Security Requirements

- HTTPS only (TLS 1.2+).
- Token expiration, rotation, and refresh policies should be defined by the API provider.
- Role-based access control may be defined per API.

### Error Responses

- `401 Unauthorized` - Missing or invalid credentials.
- `403 Forbidden` - Insufficient permissions.

---

## Request and Response Formats

Defines data structure, encoding, and format conventions for API communication.

### Content Type

```http
application/json; charset=utf-8
```

### Date/Time Format

ISO 8601 UTC (`YYYY-MM-DDTHH:MM:SSZ`).

### Field Conventions

- Use lowercase for JSON fields, unless an API-specific specification defines a different convention.
- Optional fields may be omitted.
- `null` indicates explicit removal for `PATCH` operations.
- Omitted fields in `PATCH` operations are not modified.

### Response Headers

```http
Content-Type: application/json; charset=utf-8
X-Request-ID: {uuid}
X-RateLimit-Limit: {number}
X-RateLimit-Remaining: {number}
X-RateLimit-Reset: {timestamp}
```

### Collection Response Structure

```json
{
  "data": [ ... ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 1234,
    "hasmore": true
  }
}
```

---

## Status Codes

Follow standard HTTP status codes per RFC 7231.

### Success (2xx)

- `200 OK` - Successful `GET`, `PUT`, or `PATCH`.
- `201 Created` - Successful `POST` with `Location` header.
- `204 No Content` - Successful `DELETE`.

### Client Errors (4xx)

- `400 Bad Request` - Invalid JSON or missing required fields.
- `401 Unauthorized` - Missing or invalid authentication.
- `403 Forbidden` - Insufficient permissions.
- `404 Not Found` - Resource does not exist.
- `409 Conflict` - Duplicate resource identifier.
- `422 Unprocessable Entity` - Invalid field values.
- `429 Too Many Requests` - Rate limit exceeded.

### Server Errors (5xx)

- `500 Internal Server Error` - Unexpected error.
- `503 Service Unavailable` - Temporary outage.

---

## Error Handling

Defines standardized error response structure and common error codes.

### Standard Error Response

```json
{
  "error": {
    "code": "string",
    "message": "string",
    "details": [
      {
        "field": "string",
        "issue": "string"
      }
    ],
    "requestid": "string"
  }
}
```

### Common Error Codes

- `INVALID_JSON`, `MISSING_REQUIRED_FIELD`, `INVALID_FIELD_VALUE`, `INVALID_DATE_FORMAT` (400)
- `MISSING_CREDENTIALS`, `INVALID_TOKEN`, `EXPIRED_TOKEN` (401)
- `INSUFFICIENT_PERMISSIONS` (403)
- `RESOURCE_NOT_FOUND` (404)
- `DUPLICATE_RESOURCE`, `RESOURCE_CONFLICT` (409)
- `RATE_LIMIT_EXCEEDED` (429)
- `INTERNAL_ERROR`, `SERVICE_UNAVAILABLE` (500/503)

### Best Practices

- Include `X-Request-ID` for tracing.
- Provide actionable error messages.
- Include field-level details for validation errors.
- Never expose internal system details.

---

## Pagination

Defines standard approach for handling large result sets.

### Query Parameters

- `limit={number}` - Maximum results per page (default: 50, max: 100).
- `offset={number}` - Starting position (default: 0).

### Response Structure

```json
{
  "data": [ ... ],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 1234,
    "hasmore": true
  }
}
```

### Pagination Fields

- `limit` - Number of results in current page.
- `offset` - Starting position of current page.
- `total` - Total resources matching query.
- `hasmore` - Boolean indicating more results exist.

---

## Versioning

Defines API evolution and backward compatibility strategy.

### Strategy

URI-based versioning with major version in base path (e.g., `/v1/`, `/v2/`).

### Version Policy

- Increment for breaking changes, such as field removal, type changes, or behavior changes.
- Maintain backward compatibility within the same major version.
- Support previous versions for 12 months minimum.
- Provide 6-month deprecation notice.

---

## Rate Limiting

Defines request throttling policies to protect API resources and ensure fair usage.

### Default Thresholds

- 1,000 requests/hour per client.
- 100 requests/minute burst limit.

### Response Headers

```http
X-RateLimit-Limit: {number}
X-RateLimit-Remaining: {number}
X-RateLimit-Reset: {timestamp}
```

### 429 Response

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Retry after {seconds} seconds.",
    "requestid": "..."
  }
}
```

Includes `Retry-After: {seconds}` header.

### Best Practices

- Monitor `X-RateLimit-Remaining` header.
- Implement exponential backoff for `429` responses.
- Cache responses to reduce API calls.

---

## Sorting

Defines standard approach for ordering result sets.

### Query Parameters

- `sort={field}` - Field to sort by.
- `order=asc|desc` - Sort direction (default: `asc`).

### Example

```http
GET /resources?sort=createddate&order=desc
```
