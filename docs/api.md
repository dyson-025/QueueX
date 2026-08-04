# QueueX REST API Specification

Version: 1.0

Base URL

http://localhost:8000/api/v1

Content-Type

application/json

Authentication

Bearer JWT Token

---

# API Standards

## Success Response

```json
{
    "success": true,
    "message": "Job created successfully",
    "data": {}
}
```

## Error Response

```json
{
    "success": false,
    "message": "Validation failed",
    "errors": [
        {
            "field": "priority",
            "message": "Priority must be between 1 and 5"
        }
    ]
}
```

HTTP Status Codes

200 OK

201 Created

204 No Content

400 Bad Request

401 Unauthorized

403 Forbidden

404 Not Found

409 Conflict

422 Validation Error

500 Internal Server Error

---

# Authentication APIs

## Register

POST /auth/register

Request

```json
{
    "name":"John Doe",
    "email":"john@gmail.com",
    "password":"StrongPassword123"
}
```

Response

```json
{
    "success": true,
    "message": "User registered successfully"
}
```

---

## Login

POST /auth/login

Request

```json
{
    "email":"john@gmail.com",
    "password":"StrongPassword123"
}
```

Response

```json
{
    "success": true,
    "data":{
        "access_token":"...",
        "refresh_token":"..."
    }
}
```

---

## Refresh Token

POST /auth/refresh

---

## Logout

POST /auth/logout

---

## Current User

GET /users/me

---

# Job APIs

## Create Job

POST /jobs

Request

```json
{
    "job_type":"email",
    "priority":2,
    "payload":{
        "to":"abc@gmail.com",
        "subject":"Welcome",
        "body":"Hello"
    }
}
```

Response

```json
{
    "success":true,
    "data":{
        "job_id":"uuid",
        "status":"QUEUED"
    }
}
```

---

## Get All Jobs

GET /jobs

Query Parameters

?page=1

?limit=20

?status=RUNNING

?priority=1

?queue=default

?sort=created_at

---

## Get Job

GET /jobs/{jobId}

---

## Cancel Job

POST /jobs/{jobId}/cancel

---

## Retry Job

POST /jobs/{jobId}/retry

---

## Delete Job

DELETE /jobs/{jobId}

---

# Queue APIs

## Queue Statistics

GET /queues

Response

```json
{
    "waiting":25,
    "running":5,
    "failed":3,
    "completed":142
}
```

---

## Queue Details

GET /queues/{queueName}

---

# Worker APIs

## Get Workers

GET /workers

---

## Worker Details

GET /workers/{workerId}

---

## Worker Metrics

GET /workers/{workerId}/metrics

---

# Dashboard APIs

## Dashboard Summary

GET /dashboard

Response

```json
{
    "jobs_today":120,
    "running_jobs":8,
    "workers":5,
    "success_rate":98.4
}
```

---

# Metrics APIs

GET /metrics

Returns

- Throughput
- Average Execution Time
- Retry Count
- Queue Length
- Failed Jobs
- Worker Utilization

---

# Health Check

GET /health

Response

```json
{
    "status":"healthy"
}
```

---

# WebSocket API

/ws/jobs

Events

JOB_CREATED

JOB_STARTED

JOB_COMPLETED

JOB_FAILED

JOB_RETRYING

WORKER_ONLINE

WORKER_OFFLINE

QUEUE_UPDATED

---

# Pagination

GET /jobs?page=2&limit=20

Response

```json
{
    "success":true,
    "data":[...],
    "pagination":{
        "page":2,
        "limit":20,
        "total":320,
        "pages":16
    }
}
```

---

# Filtering

Examples

/jobs?status=FAILED

/jobs?priority=1

/jobs?queue=default

/jobs?worker=worker-3

/jobs?created_after=2026-08-01

---

# Sorting

/jobs?sort=created_at

/jobs?sort=status

/jobs?sort=priority

Ascending

?order=asc

Descending

?order=desc

---

# Rate Limits

Authenticated User

100 requests/minute

Anonymous User

20 requests/minute

---

# Validation Rules

Email

Valid email format

Password

Minimum 8 characters

Priority

1–5

Payload

Must be valid JSON

Job Type

Must exist

---

# Error Codes

AUTH_001

Invalid credentials

AUTH_002

Token expired

JOB_001

Job not found

JOB_002

Job already completed

JOB_003

Job cancelled

QUEUE_001

Queue unavailable

WORKER_001

Worker offline

SYSTEM_001

Unexpected server error

---

# Versioning

Current Version

v1

Future

/api/v2

Older versions remain supported for backward compatibility.
