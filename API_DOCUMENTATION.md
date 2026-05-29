# API Documentation - MentorSys

RESTful API endpoints for MentorSys mentorship platform.

## Base URL

```
http://localhost:8080/api
```

## Authentication

All endpoints (except `/auth/register` and `/auth/login`) require JWT token in header:

```
Authorization: Bearer {jwt_token}
```

---

## Authentication Endpoints

### 1. Register User

**Endpoint:** `POST /auth/register`

**Request:**
```json
{
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "password": "securePassword123",
  "role": "MENTEE"
}
```

**Response:** `201 Created`
```json
{
  "id": 1,
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "role": "MENTEE",
  "active": true,
  "createdAt": "2026-05-29T21:00:00Z"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid input data
- `409 Conflict` - Email already exists

---

### 2. Login

**Endpoint:** `POST /auth/login`

**Request:**
```json
{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "user": {
    "id": 1,
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "role": "MENTEE"
  }
}
```

**Error Responses:**
- `401 Unauthorized` - Invalid credentials

---

## User Endpoints

### 3. Get User by ID

**Endpoint:** `GET /users/{id}`

**Headers:** `Authorization: Bearer {token}`

**Response:** `200 OK`
```json
{
  "id": 1,
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "role": "MENTEE",
  "active": true,
  "createdAt": "2026-05-29T21:00:00Z",
  "updatedAt": "2026-05-29T21:30:00Z"
}
```

**Error Responses:**
- `404 Not Found` - User not found
- `401 Unauthorized` - Missing or invalid token

---

### 4. Get All Users

**Endpoint:** `GET /users`

**Query Parameters:**
- `role` (optional): Filter by role (MENTOR, MENTEE)
- `page` (optional): Page number (default: 0)
- `size` (optional): Page size (default: 20)

**Response:** `200 OK`
```json
{
  "content": [
    {
      "id": 1,
      "email": "john@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "MENTEE",
      "active": true
    }
  ],
  "totalElements": 1,
  "totalPages": 1,
  "currentPage": 0,
  "pageSize": 20
}
```

---

### 5. Update User

**Endpoint:** `PUT /users/{id}`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "firstName": "Johnny",
  "lastName": "Smith",
  "email": "john.smith@example.com"
}
```

**Response:** `200 OK`
```json
{
  "id": 1,
  "email": "john.smith@example.com",
  "firstName": "Johnny",
  "lastName": "Smith",
  "role": "MENTEE",
  "active": true,
  "updatedAt": "2026-05-29T22:00:00Z"
}
```

**Error Responses:**
- `404 Not Found` - User not found
- `409 Conflict` - Email already exists
- `403 Forbidden` - Unauthorized to update this user

---

### 6. Delete User

**Endpoint:** `DELETE /users/{id}`

**Headers:** `Authorization: Bearer {token}`

**Response:** `204 No Content`

**Error Responses:**
- `404 Not Found` - User not found
- `403 Forbidden` - Unauthorized to delete this user

---

### 7. Deactivate User

**Endpoint:** `PUT /users/{id}/deactivate`

**Headers:** `Authorization: Bearer {token}`

**Response:** `200 OK`
```json
{
  "id": 1,
  "email": "john@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "role": "MENTEE",
  "active": false
}
```

---

## Mentor Endpoints

### 8. Create Mentor Profile

**Endpoint:** `POST /mentors`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "userId": 1,
  "title": "Senior Software Engineer",
  "bio": "10 years of Java experience",
  "expertise": ["Java", "Spring Boot", "Microservices"],
  "hourlyRate": 50,
  "availability": "FULL_TIME"
}
```

**Response:** `201 Created`
```json
{
  "id": 1,
  "userId": 1,
  "title": "Senior Software Engineer",
  "bio": "10 years of Java experience",
  "expertise": ["Java", "Spring Boot", "Microservices"],
  "hourlyRate": 50,
  "availability": "FULL_TIME",
  "rating": 0,
  "createdAt": "2026-05-29T21:00:00Z"
}
```

---

### 9. Get Mentor Profile

**Endpoint:** `GET /mentors/{id}`

**Response:** `200 OK`
```json
{
  "id": 1,
  "userId": 1,
  "user": {
    "id": 1,
    "email": "mentor@example.com",
    "firstName": "Expert",
    "lastName": "Developer"
  },
  "title": "Senior Software Engineer",
  "bio": "10 years of Java experience",
  "expertise": ["Java", "Spring Boot", "Microservices"],
  "hourlyRate": 50,
  "availability": "FULL_TIME",
  "rating": 4.5,
  "sessionsCompleted": 15
}
```

---

### 10. Update Mentor Profile

**Endpoint:** `PUT /mentors/{id}`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "title": "Principal Software Architect",
  "bio": "15 years of enterprise Java experience",
  "expertise": ["Java", "Spring Boot", "Microservices", "Cloud"],
  "hourlyRate": 75,
  "availability": "PART_TIME"
}
```

**Response:** `200 OK` (Same structure as Get Mentor)

---

### 11. Get All Mentors

**Endpoint:** `GET /mentors`

**Query Parameters:**
- `expertise` (optional): Filter by expertise
- `minRating` (optional): Minimum rating (0-5)
- `availability` (optional): FULL_TIME or PART_TIME
- `page` (optional): Page number
- `size` (optional): Page size

**Response:** `200 OK`
```json
{
  "content": [
    {
      "id": 1,
      "user": { ... },
      "title": "Senior Software Engineer",
      "expertise": ["Java", "Spring Boot"],
      "hourlyRate": 50,
      "rating": 4.5
    }
  ],
  "totalElements": 10,
  "totalPages": 1,
  "currentPage": 0
}
```

---

## Session Endpoints

### 12. Schedule Session

**Endpoint:** `POST /sessions`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "mentorId": 1,
  "menteeId": 2,
  "title": "Java Spring Boot Fundamentals",
  "description": "Learn Spring Boot basics",
  "scheduledAt": "2026-06-05T14:00:00Z",
  "duration": 60,
  "sessionType": "MENTORING"
}
```

**Response:** `201 Created`
```json
{
  "id": 1,
  "mentorId": 1,
  "menteeId": 2,
  "title": "Java Spring Boot Fundamentals",
  "description": "Learn Spring Boot basics",
  "scheduledAt": "2026-06-05T14:00:00Z",
  "duration": 60,
  "sessionType": "MENTORING",
  "status": "SCHEDULED",
  "createdAt": "2026-05-29T21:00:00Z"
}
```

**Error Responses:**
- `404 Not Found` - Mentor or Mentee not found
- `409 Conflict` - Time slot already booked

---

### 13. Get Session

**Endpoint:** `GET /sessions/{id}`

**Response:** `200 OK`
```json
{
  "id": 1,
  "mentor": { ... },
  "mentee": { ... },
  "title": "Java Spring Boot Fundamentals",
  "description": "Learn Spring Boot basics",
  "scheduledAt": "2026-06-05T14:00:00Z",
  "duration": 60,
  "sessionType": "MENTORING",
  "status": "SCHEDULED",
  "meetingLink": "https://zoom.us/j/123456789",
  "notes": "",
  "rating": null,
  "createdAt": "2026-05-29T21:00:00Z"
}
```

---

### 14. Get User Sessions

**Endpoint:** `GET /sessions/user/{userId}`

**Query Parameters:**
- `status` (optional): Filter by status (SCHEDULED, COMPLETED, CANCELLED)
- `from` (optional): Start date (ISO format)
- `to` (optional): End date (ISO format)

**Response:** `200 OK`
```json
{
  "content": [
    {
      "id": 1,
      "mentor": { ... },
      "mentee": { ... },
      "title": "Java Spring Boot Fundamentals",
      "scheduledAt": "2026-06-05T14:00:00Z",
      "status": "SCHEDULED"
    }
  ],
  "totalElements": 5,
  "totalPages": 1
}
```

---

### 15. Update Session

**Endpoint:** `PUT /sessions/{id}`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "title": "Advanced Spring Boot Patterns",
  "scheduledAt": "2026-06-06T15:00:00Z",
  "duration": 90
}
```

**Response:** `200 OK`

---

### 16. Complete Session

**Endpoint:** `POST /sessions/{id}/complete`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "notes": "Great session! Covered dependency injection and autoconfiguration",
  "rating": 5
}
```

**Response:** `200 OK`
```json
{
  "id": 1,
  "status": "COMPLETED",
  "completedAt": "2026-06-05T15:00:00Z",
  "notes": "Great session! Covered dependency injection and autoconfiguration",
  "rating": 5
}
```

---

### 17. Cancel Session

**Endpoint:** `POST /sessions/{id}/cancel`

**Headers:** `Authorization: Bearer {token}`

**Request:**
```json
{
  "reason": "Unable to attend due to emergency"
}
```

**Response:** `200 OK`
```json
{
  "id": 1,
  "status": "CANCELLED",
  "cancelledAt": "2026-05-29T21:30:00Z",
  "cancellationReason": "Unable to attend due to emergency"
}
```

---

## Notification Endpoints

### 18. Get User Notifications

**Endpoint:** `GET /notifications`

**Headers:** `Authorization: Bearer {token}`

**Query Parameters:**
- `unreadOnly` (optional): true/false (default: false)
- `page` (optional): Page number
- `size` (optional): Page size

**Response:** `200 OK`
```json
{
  "content": [
    {
      "id": 1,
      "userId": 2,
      "type": "SESSION_SCHEDULED",
      "title": "New session scheduled",
      "message": "Mentor John has scheduled a session with you",
      "read": false,
      "data": {
        "sessionId": 1
      },
      "createdAt": "2026-05-29T21:00:00Z"
    }
  ],
  "totalElements": 5,
  "totalPages": 1,
  "unreadCount": 3
}
```

---

### 19. Mark Notification as Read

**Endpoint:** `PUT /notifications/{id}/read`

**Headers:** `Authorization: Bearer {token}`

**Response:** `200 OK`
```json
{
  "id": 1,
  "read": true,
  "readAt": "2026-05-29T21:35:00Z"
}
```

---

### 20. Delete Notification

**Endpoint:** `DELETE /notifications/{id}`

**Headers:** `Authorization: Bearer {token}`

**Response:** `204 No Content`

---

### 21. Mark All Notifications as Read

**Endpoint:** `PUT /notifications/mark-all-read`

**Headers:** `Authorization: Bearer {token}`

**Response:** `200 OK`
```json
{
  "message": "All notifications marked as read",
  "count": 3
}
```

---

## WebSocket Endpoints

Real-time communication via WebSocket (STOMP protocol).

### Connection

**WebSocket URL:** `ws://localhost:8080/ws`

**STOMP Protocol:**
```
CONNECT
accept-version:1.2
```

---

### Subscribe to Notifications

**Subscribe to:**
```
/user/queue/notifications
```

**Message Format:**
```json
{
  "type": "NOTIFICATION",
  "notification": {
    "id": 1,
    "type": "SESSION_SCHEDULED",
    "title": "New session",
    "message": "...",
    "createdAt": "2026-05-29T21:00:00Z"
  }
}
```

---

### Subscribe to Session Messages

**Subscribe to:**
```
/topic/messages/{sessionId}
```

**Send Message:**
```
POST /app/session/{sessionId}/message
```

**Body:**
```json
{
  "content": "Hi, let's discuss Spring Boot configuration",
  "senderId": 1
}
```

---

### Subscribe to User Presence

**Subscribe to:**
```
/topic/presence
```

**Message Format:**
```json
{
  "userId": 1,
  "status": "ONLINE",
  "timestamp": "2026-05-29T21:00:00Z"
}
```

---

## Error Responses

### 400 Bad Request
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Invalid input data",
  "path": "/api/users"
}
```

### 401 Unauthorized
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid or expired token",
  "path": "/api/sessions"
}
```

### 403 Forbidden
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 403,
  "error": "Forbidden",
  "message": "You don't have permission to perform this action",
  "path": "/api/users/1"
}
```

### 404 Not Found
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 404,
  "error": "Not Found",
  "message": "User not found with id: 999",
  "path": "/api/users/999"
}
```

### 409 Conflict
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 409,
  "error": "Conflict",
  "message": "Email already exists",
  "path": "/api/auth/register"
}
```

### 500 Internal Server Error
```json
{
  "timestamp": "2026-05-29T21:00:00Z",
  "status": 500,
  "error": "Internal Server Error",
  "message": "An unexpected error occurred",
  "path": "/api/sessions"
}
```

---

## Enum Values

### UserRole
- `MENTOR`
- `MENTEE`

### SessionType
- `MENTORING`
- `TRAINING`
- `COACHING`

### SessionStatus
- `SCHEDULED`
- `IN_PROGRESS`
- `COMPLETED`
- `CANCELLED`

### Availability
- `FULL_TIME`
- `PART_TIME`
- `WEEKENDS_ONLY`

### NotificationType
- `SESSION_SCHEDULED`
- `SESSION_REMINDER`
- `SESSION_COMPLETED`
- `SESSION_CANCELLED`
- `MENTOR_RATING`
- `MESSAGE_RECEIVED`

---

## Rate Limiting

- **Limit:** 1000 requests per hour per IP
- **Headers:**
  ```
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1622308800
  ```

---

## Pagination

All list endpoints support pagination:

**Parameters:**
- `page`: 0-indexed page number (default: 0)
- `size`: Number of items per page (default: 20, max: 100)
- `sort`: Sort field and direction (e.g., `sort=createdAt,desc`)

**Response Metadata:**
```json
{
  "content": [...],
  "totalElements": 100,
  "totalPages": 5,
  "currentPage": 0,
  "pageSize": 20,
  "hasNext": true,
  "hasPrevious": false
}
```

---

**Last Updated:** 2026-05-29  
**Version:** 1.0.0
