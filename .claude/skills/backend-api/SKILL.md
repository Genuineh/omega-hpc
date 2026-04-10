---
name: backend-api
description: Backend API Design Guide. Provides guidance for RESTful API design, including naming, versioning, and error handling.
version: 1.0.0
---

# API Design Guide

Guide for designing backend APIs.

---

## RESTful Design

### Resource Naming

| Resource | Path | Methods |
|----------|------|---------|
| Users | /users | GET, POST |
| User | /users/{id} | GET, PUT, DELETE |
| User's orders | /users/{id}/orders | GET |
| Create order for user | /users/{id}/orders | POST |

### Naming Rules

- Use nouns, not verbs
- Plural for collections
- kebab-case for paths
- snake_case for parameters

### HTTP Methods

| Method | Meaning | Idempotent |
|--------|---------|------------|
| GET | Read | Yes |
| POST | Create | No |
| PUT | Replace | Yes |
| PATCH | Update | No |
| DELETE | Remove | Yes |

---

## Response Formats

### Success Response

```json
{
  "data": {
    "id": "123",
    "name": "John",
    "email": "john@example.com"
  },
  "meta": {
    "request_id": "abc-123"
  }
}
```

### Collection Response

```json
{
  "data": [
    { "id": "1", "name": "A" },
    { "id": "2", "name": "B" }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "pages": 5
  },
  "links": {
    "self": "/users?page=1",
    "next": "/users?page=2"
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "request_id": "abc-123"
  }
}
```

---

## Status Codes

| Code | Meaning | Use For |
|------|---------|----------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Missing auth |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate, state conflict |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Error | Server error |

---

## Versioning

### URL Versioning

```
/api/v1/users
/api/v2/users
```

### Header Versioning

```
Accept: application/vnd.myapp.v2+json
```

### When to Version

- Breaking changes
- Remove deprecated fields
- Change response format

---

## Pagination

### Cursor-based (Recommended)

```json
{
  "data": [...],
  "pagination": {
    "cursor": "abc123",
    "next_cursor": "def456"
  }
}
```

### Offset-based

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 100
  }
}
```

---

## API Design Checklist

- [ ] Resources named correctly
- [ ] HTTP methods used properly
- [ ] Status codes appropriate
- [ ] Error responses consistent
- [ ] Pagination implemented
- [ ] Versioning strategy defined
- [ ] Authentication documented
- [ ] Rate limiting defined
