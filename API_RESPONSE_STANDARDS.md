# API Response Standards & Best Practices

This document outlines the standard API response structure for the Athlink platform, based on industry best practices and production-ready patterns used by major APIs (Stripe, GitHub, Google, etc.).

---

## Table of Contents
- [Core Principles](#core-principles)
- [Standard Response Structure](#standard-response-structure)
- [Success Responses](#success-responses)
- [Error Responses](#error-responses)
- [HTTP Status Codes](#http-status-codes)
- [Pagination](#pagination)
- [Best Practices](#best-practices)
- [References](#references)

---

## Core Principles

### 1. **Consistency is Key**
Every API response should follow the same structure, making it predictable and easier to handle on the client side.

### 2. **Be Explicit**
Always include a `success` boolean to make it crystal clear whether the request succeeded or failed.

### 3. **Provide Context**
Include meaningful messages that help developers understand what happened.

### 4. **Fail Gracefully**
Error responses should be as informative as success responses.

---

## Standard Response Structure

All API responses should follow this base structure:

```json
{
  "success": boolean,
  "message": string,
  "data": object | array | null,
  "errorCode": string (optional, present on errors),
  "errors": array (optional, for validation errors)
}
```

### TypeScript Interface
```typescript
interface ApiResponse<T> {
  success: boolean;
  message: string;
  data?: T;
  errorCode?: string;
  errors?: ValidationError[];
}

interface ValidationError {
  field: string;
  message: string;
}
```

---

## Success Responses

### 1. Single Item Response

**Example: User Profile**
```json
{
  "success": true,
  "message": "Profile retrieved successfully",
  "data": {
    "_id": "673user123",
    "name": "John Doe",
    "email": "john@example.com",
    "athleteProfile": {
      "age": 25,
      "position": "Forward",
      "rating": 4.5
    }
  }
}
```

### 2. List Response (Simple)

**Example: Sports List**
```json
{
  "success": true,
  "message": "Sports retrieved successfully",
  "data": {
    "sports": [
      {
        "_id": "68f22de9",
        "name": "Basketball",
        "icon": "/uploads/sport/icons/basketball.png"
      },
      {
        "_id": "68f22df0",
        "name": "Football",
        "icon": "/uploads/sport/icons/football.png"
      }
    ],
    "count": 2
  }
}
```

### 3. List Response (with Pagination)

**Example: Job Posts**
```json
{
  "success": true,
  "message": "Job posts retrieved successfully",
  "data": {
    "jobs": [
      {
        "_id": "673job123",
        "title": "Looking for Basketball Player",
        "description": "Seeking talented player...",
        "createdAt": "2025-10-23T10:30:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "totalPages": 3,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  }
}
```

### 4. Empty List Response

**Example: No Results**
```json
{
  "success": true,
  "message": "No invitations found",
  "data": {
    "invitations": [],
    "count": 0
  }
}
```

### 5. Creation Success

**Example: Milestone Created**
```json
{
  "success": true,
  "message": "Milestone created successfully",
  "data": {
    "_id": "673milestone456",
    "title": "Complete Training",
    "status": "pending",
    "athleteId": "673athlete789",
    "jobId": "673job123",
    "createdAt": "2025-11-25T10:30:00.000Z"
  }
}
```

### 6. Update Success

**Example: Profile Updated**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": {
    "_id": "673user123",
    "name": "John Updated",
    "updatedAt": "2025-11-25T10:30:00.000Z"
  }
}
```

### 7. Deletion Success

**Example: Item Deleted**
```json
{
  "success": true,
  "message": "Job post deleted successfully",
  "data": null
}
```

---

## Error Responses

### 1. Validation Errors (400 Bad Request)

**Example: Multiple Field Errors**
```json
{
  "success": false,
  "message": "Validation failed",
  "errorCode": "VALIDATION_ERROR",
  "errors": [
    {
      "field": "email",
      "message": "Invalid email format"
    },
    {
      "field": "password",
      "message": "Password must be at least 8 characters"
    }
  ]
}
```

### 2. Resource Not Found (404 Not Found)

**Example: User Not Found**
```json
{
  "success": false,
  "message": "User not found",
  "errorCode": "USER_NOT_FOUND"
}
```

### 3. Duplicate Resource (409 Conflict)

**Example: Duplicate Email**
```json
{
  "success": false,
  "message": "User already exists with this email",
  "errorCode": "USER_ALREADY_EXISTS"
}
```

### 4. Unauthorized (401 Unauthorized)

**Example: Invalid Token**
```json
{
  "success": false,
  "message": "Invalid or expired authentication token",
  "errorCode": "INVALID_TOKEN"
}
```

### 5. Forbidden (403 Forbidden)

**Example: Insufficient Permissions**
```json
{
  "success": false,
  "message": "You do not have permission to perform this action",
  "errorCode": "FORBIDDEN"
}
```

### 6. Server Error (500 Internal Server Error)

**Example: Unexpected Error**
```json
{
  "success": false,
  "message": "An unexpected error occurred. Please try again later.",
  "errorCode": "INTERNAL_SERVER_ERROR"
}
```

### 7. Rate Limit (429 Too Many Requests)

**Example: Too Many Requests**
```json
{
  "success": false,
  "message": "Too many requests. Please try again in 60 seconds.",
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "data": {
    "retryAfter": 60,
    "limit": 100,
    "remaining": 0
  }
}
```

---

## HTTP Status Codes

Use appropriate HTTP status codes along with the response body:

### Success (2xx)
- **200 OK**: Standard success response (GET, PUT, PATCH)
- **201 Created**: Resource created successfully (POST)
- **204 No Content**: Success with no response body (DELETE)

### Client Errors (4xx)
- **400 Bad Request**: Validation errors, malformed request
- **401 Unauthorized**: Missing or invalid authentication
- **403 Forbidden**: Authenticated but not authorized
- **404 Not Found**: Resource doesn't exist
- **409 Conflict**: Resource already exists (e.g., duplicate email)
- **422 Unprocessable Entity**: Semantic validation errors
- **429 Too Many Requests**: Rate limit exceeded

### Server Errors (5xx)
- **500 Internal Server Error**: Unexpected server error
- **503 Service Unavailable**: Server temporarily unavailable

---

## Pagination

### Standard Pagination Response

```json
{
  "success": true,
  "message": "Athletes retrieved successfully",
  "data": {
    "athletes": [...],
    "pagination": {
      "page": 2,
      "limit": 20,
      "total": 156,
      "totalPages": 8,
      "hasNextPage": true,
      "hasPrevPage": true
    }
  }
}
```

### Cursor-Based Pagination (for large datasets)

```json
{
  "success": true,
  "message": "Feed retrieved successfully",
  "data": {
    "posts": [...],
    "pagination": {
      "nextCursor": "eyJpZCI6IjY3M3Bvc3QxMjMiLCJjcmVhdGVkQXQiOiIyMDI1LTExLTI1In0=",
      "hasMore": true,
      "limit": 20
    }
  }
}
```

---

## Best Practices

### 1. **Consistent Field Naming**
- Use `camelCase` for JSON fields (frontend convention)
- Or use `snake_case` consistently (backend convention)
- **Choose one and stick to it across all endpoints**

### 2. **Timestamps**
- Always use ISO 8601 format: `"2025-11-25T10:30:00.000Z"`
- Include timezone information (UTC recommended)

### 3. **Error Codes**
- Use consistent, uppercase error codes: `USER_NOT_FOUND`, `VALIDATION_ERROR`
- Make them descriptive and unique
- Document all error codes

### 4. **Null vs Empty**
- Use `null` for missing/non-existent data
- Use empty array `[]` for empty lists
- Use empty object `{}` sparingly (prefer `null`)

### 5. **Data Wrapping**
```json
// ✅ GOOD: Wrap list in an object with metadata
{
  "success": true,
  "data": {
    "items": [...],
    "count": 10
  }
}

// ❌ BAD: Raw array
{
  "success": true,
  "data": [...]
}
```

### 6. **Message Quality**
```json
// ✅ GOOD: Clear, actionable messages
{
  "message": "Email is required and must be valid"
}

// ❌ BAD: Technical jargon
{
  "message": "Field validation failed on constraint: email"
}
```

### 7. **Nested Resources**
```json
// ✅ GOOD: Include related data when useful
{
  "data": {
    "athlete": {
      "_id": "123",
      "name": "John",
      "sport": {
        "_id": "456",
        "name": "Basketball"
      }
    }
  }
}

// Consider: Provide options to expand/include related resources
// GET /athletes/123?include=sport,team
```

---

## Real-World Examples

### Stripe API
```json
{
  "object": "charge",
  "id": "ch_1234567890",
  "amount": 2000,
  "status": "succeeded"
}
// Note: Stripe doesn't wrap in success/data, but always returns consistent structure
```

### GitHub API
```json
{
  "message": "Not Found",
  "documentation_url": "https://docs.github.com/rest/reference/repos"
}
```

### Google APIs
```json
{
  "error": {
    "code": 404,
    "message": "Resource not found",
    "errors": [
      {
        "domain": "global",
        "reason": "notFound",
        "message": "Resource not found"
      }
    ]
  }
}
```

---

## References

### Industry Standards
- **JSON API Specification**: https://jsonapi.org/
- **REST API Best Practices**: https://restfulapi.net/
- **Microsoft REST API Guidelines**: https://github.com/microsoft/api-guidelines
- **Google API Design Guide**: https://cloud.google.com/apis/design

### Production APIs
- **Stripe API**: https://stripe.com/docs/api
- **GitHub API**: https://docs.github.com/en/rest
- **Twilio API**: https://www.twilio.com/docs/usage/api

---

## Quick Reference

| Scenario | Status Code | Response Structure |
|----------|-------------|-------------------|
| Success with data | 200 | `{ success: true, message: "...", data: {...} }` |
| Resource created | 201 | `{ success: true, message: "...", data: {...} }` |
| Success, no data | 200/204 | `{ success: true, message: "...", data: null }` |
| Validation error | 400 | `{ success: false, message: "...", errorCode: "...", errors: [...] }` |
| Not found | 404 | `{ success: false, message: "...", errorCode: "..." }` |
| Already exists | 409 | `{ success: false, message: "...", errorCode: "..." }` |
| Unauthorized | 401 | `{ success: false, message: "...", errorCode: "..." }` |
| Forbidden | 403 | `{ success: false, message: "...", errorCode: "..." }` |
| Server error | 500 | `{ success: false, message: "...", errorCode: "..." }` |

---

**Last Updated**: 2025-11-25  
**Version**: 1.0  
**Maintained by**: Athlink Development Team
