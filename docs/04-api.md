# QueueX REST API Specification

**Version:** 1.1  
**Base URL:** `http://localhost:8000/api/v1`  
**Content-Type:** `application/json`  
**Authentication:** Bearer JWT

---

# 1. API Standards

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
  "error": {
    "code": "JOB_001",
    "message": "Job not found",
    "trace_id": "req_abc123"
  }
}
```

Every response includes:

- `X-Request-ID`
- `API-Version: 1.1`

Supported status codes:

- 200 OK
- 201 Created
- 204 No Content
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 409 Conflict
- 422 Unprocessable Entity
- 429 Too Many Requests
- 500 Internal Server Error

---

# 2. Authentication

## POST /auth/register

Registers a new user.

## POST /auth/login

Returns:

- access_token
- refresh_token

## POST /auth/refresh

Refresh access token.

## POST /auth/logout

Invalidates refresh token.

## POST /auth/change-password

Change current password.

## GET /users/me

Current authenticated user.

---

# 3. Job APIs

## POST /jobs

Creates a new background job.

Headers

```text
Authorization: Bearer <token>
Idempotency-Key: <uuid>
```

Request

```json
{
  "queue":"default",
  "job_type":"email",
  "priority":2,
  "scheduled_at":"2026-08-08T10:30:00Z",
  "payload":{
    "to":"user@example.com",
    "subject":"Welcome"
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

## GET /jobs

Supports

- pagination
- filtering
- sorting
- searching

Examples

```
/jobs?page=1&limit=20
/jobs?status=FAILED
/jobs?priority=1
/jobs?queue_id=<uuid>
/jobs?worker=<uuid>
/jobs?search=email
/jobs?created_after=2026-08-01
/jobs?sort=created_at&order=desc
```

---

## GET /jobs/{jobId}

Job details.

## GET /jobs/{jobId}/events

Lifecycle timeline.

## GET /jobs/{jobId}/logs

Execution logs.

## POST /jobs/{jobId}/cancel

Allowed only for:

- CREATED
- QUEUED

## POST /jobs/{jobId}/retry

Optional body

```json
{
  "override_priority":1,
  "override_queue":"high"
}
```

## DELETE /jobs/{jobId}

Soft delete completed job.

## POST /jobs/bulk

Bulk job submission.

## POST /jobs/bulk/retry

Bulk retry.

---

# 4. Queue APIs

## GET /queues

List queues with statistics.

## GET /queues/{queueId}

Queue details.

## POST /queues

Create queue.

## PATCH /queues/{queueId}

Update configuration.

## POST /queues/{queueId}/pause

Pause processing.

## POST /queues/{queueId}/resume

Resume processing.

---

# 5. Worker APIs

## GET /workers

List workers.

## GET /workers/{workerId}

Worker details.

## GET /workers/{workerId}/metrics

Metrics.

## GET /workers/{workerId}/logs

Worker logs.

## POST /workers/{workerId}/pause

Pause worker.

## POST /workers/{workerId}/resume

Resume worker.

---

# 6. Dashboard

- GET /dashboard/summary
- GET /dashboard/activity
- GET /dashboard/metrics
- GET /dashboard/queues

---

# 7. Metrics

GET /metrics

Supported query parameters

- from
- to
- interval

Returns

- throughput
- latency
- queue depth
- retry count
- worker utilization
- failure rate

---

# 8. Health & System

- GET /health
- GET /system/info
- GET /system/version
- GET /system/readiness
- GET /system/liveness

Health response

```json
{
  "status":"healthy",
  "database":"healthy",
  "redis":"healthy",
  "workers":"healthy"
}
```

---

# 9. WebSocket

Endpoint

```
/ws/jobs
```

Events

- JOB_CREATED
- JOB_STARTED
- JOB_COMPLETED
- JOB_FAILED
- JOB_RETRYING
- WORKER_ONLINE
- WORKER_OFFLINE
- QUEUE_UPDATED

Example

```json
{
  "event":"JOB_COMPLETED",
  "job_id":"uuid",
  "worker_id":"worker-2",
  "status":"COMPLETED",
  "completed_at":"2026-08-06T10:20:00Z"
}
```

---

# 10. Rate Limits

Authenticated

- 100 requests/minute

Anonymous

- 20 requests/minute

---

# 11. Validation Rules

- Valid email
- Password ≥ 8 characters
- Priority between 1 and 5
- Payload must be valid JSON
- Job type must exist
- Queue must exist

---

# 12. Error Codes

Authentication

- AUTH_001 Invalid credentials
- AUTH_002 Token expired

Jobs

- JOB_001 Not found
- JOB_002 Already completed
- JOB_003 Cancelled

Queues

- QUEUE_001 Queue unavailable

Workers

- WORKER_001 Worker offline

System

- SYSTEM_001 Unexpected server error

---

# 13. Job Lifecycle Operations

| Status | Allowed Operations |
|---------|--------------------|
| CREATED | View, Delete |
| QUEUED | View, Cancel |
| RESERVED | View |
| RUNNING | View, Logs |
| FAILED | View, Retry |
| COMPLETED | View, Delete |
| DEAD_LETTER | View, Retry |

---

# 14. Versioning

Current

```
/api/v1
```

Future

```
/api/v2
```

Older versions remain supported for backward compatibility.

---

# 15. Future APIs

- Python SDK
- Node.js SDK
- Go SDK
- Admin APIs
- API Keys
