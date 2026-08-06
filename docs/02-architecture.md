# QueueX Architecture

**Version:** 1.1  
**Status:** Draft

---

# 1. Overview

QueueX is a distributed asynchronous job processing platform built around a clear separation between API services and background workers. The API accepts requests and manages metadata, while workers execute long-running jobs independently.

Core architectural goals:

- Separation of concerns
- Horizontal scalability
- Fault tolerance
- Reliability
- Real-time observability
- Modular service design

---

# 2. High-Level Architecture

```text
               AudioPilot          Trackathon
                    │                   │
                    └────── REST API ───┘
                            │
                    +-------------------+
                    |   QueueX API      |
                    |     (FastAPI)     |
                    +-------------------+
                     │        │
                     │        ├──────────────┐
                     │        │              │
                     ▼        ▼              ▼
               PostgreSQL   Redis      WebSocket Gateway
                              │
                    ┌─────────┴─────────┐
                    │ Pending Queue      │
                    │ Delayed Queue      │
                    │ Retry Queue        │
                    │ Dead Letter Queue  │
                    └─────────┬─────────┘
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
             Worker 1     Worker 2     Worker N
```

---

# 3. Component Architecture

## 3.1 Client Applications

Examples

- AudioPilot
- Trackathon
- QueueX Dashboard

Responsibilities

- Authentication
- Submit jobs
- View status
- Receive live updates

Not Responsible For

- Executing jobs
- Scheduling retries

---

## 3.2 FastAPI Service

Internal Modules

```text
FastAPI
├── Auth Service
├── Job Service
├── Queue Service
├── Scheduler Service
├── Worker Service
├── Metrics Service
├── Notification Service
└── WebSocket Gateway
```

Responsibilities

- Authentication
- Authorization
- Request validation
- Job creation
- Metadata management
- Queue publishing
- Dashboard APIs

Not Responsible For

- Executing business logic
- Long-running tasks

---

## 3.3 PostgreSQL

Stores

- Users
- Jobs
- Job history
- Worker metadata
- Retry history
- Execution results
- Metrics

Reason

- ACID transactions
- Strong consistency
- Relational integrity

---

## 3.4 Redis

Redis is used as the messaging backbone.

Internal structures

```text
Redis
├── Pending Queue
├── Delayed Queue
├── Retry Queue
├── Dead Letter Queue
├── Worker Heartbeats
└── Cache
```

Reason

- High throughput
- Low latency
- Atomic queue operations

---

## 3.5 Worker Service

Worker Pipeline

```text
Poll Queue
      ↓
Reserve Job
      ↓
Start Heartbeat
      ↓
Execute Job
      ↓
Store Result
      ↓
Acknowledge
      ↓
Complete / Retry / DLQ
```

Responsibilities

- Reserve jobs atomically
- Execute jobs
- Send heartbeat
- Update database
- Retry failures
- Graceful shutdown

---

## 3.6 Scheduler

Responsibilities

- Monitor delayed jobs
- Move scheduled jobs into pending queue
- Trigger retries after backoff

---

## 3.7 WebSocket Gateway

Responsibilities

- Live job updates
- Queue metrics
- Worker status
- Dashboard synchronization

---

## 3.8 Metrics Service

Collects

- Queue depth
- Throughput
- Processing latency
- Failure rate
- Worker utilization

---

# 4. Data Flow

## Job Submission

```text
Client
   ↓
QueueX API
   ↓
Validate Request
   ↓
Store Metadata (PostgreSQL)
   ↓
Publish Job (Redis)
   ↓
Worker Reserves Job
   ↓
Execute
   ↓
Store Result
   ↓
WebSocket Notification
   ↓
Client
```

---

## Retry Flow

```text
Worker
   ↓
Job Failed
   ↓
Increment Retry Count
   ↓
Retry Queue
   ↓
Scheduler Delay
   ↓
Pending Queue
   ↓
Worker
```

---

## Worker Failure Recovery

```text
Worker Crash
      ↓
Heartbeat Timeout
      ↓
Lease Expires
      ↓
Job Returns to Queue
      ↓
Another Worker Reserves Job
```

---

# 5. Job State Machine

```text
Created
   ↓
Queued
   ↓
Reserved
   ↓
Running
   ├──────────────► Completed
   │
   └──────────────► Failed
                      ↓
              Retry Scheduled
                      ↓
                   Queued
                      ↓
                 Max Retries?
                 │         │
                No        Yes
                 │         │
                 ▼         ▼
              Running     Dead Letter Queue
```

---

# 6. Communication

| Component | Protocol |
|-----------|----------|
| Client ↔ API | HTTP/REST |
| Dashboard ↔ API | WebSocket |
| API ↔ Redis | Redis Commands |
| Workers ↔ Redis | Queue Operations |
| API ↔ PostgreSQL | SQL |
| Workers ↔ PostgreSQL | SQL |

---

# 7. Deployment

```text
Docker Compose

        │
     Nginx
        │
 ┌──────┴──────┐
 │             │
React      FastAPI
               │
      ┌────────┴────────┐
      │                 │
 PostgreSQL          Redis
                          │
                    Worker Pool
```

Every component runs in an isolated container.

---

# 8. Fault Tolerance

Worker Crash

- Heartbeat expires
- Job lease expires
- Job becomes available
- Another worker resumes processing

API Crash

- Redis retains queued jobs
- Workers continue processing
- API resumes normally after restart

Redis Restart

- Persistence enabled
- Queues restored

---

# 9. Architecture Decisions

## Why Redis?

Pros

- Fast
- Simple
- Excellent ecosystem
- Easy local deployment

Trade-off

- Fewer advanced routing capabilities than RabbitMQ

Decision

Redis is sufficient for V1.

---

## Why PostgreSQL?

- Strong consistency
- ACID transactions
- Excellent relational support

---

## Why Separate Workers?

Benefits

- Independent scaling
- Better reliability
- Responsive APIs

---

## Why WebSockets?

Benefits

- Live dashboard
- Instant status updates
- Better user experience

---

# 10. Scalability

QueueX scales by

- Adding workers
- Running multiple API instances
- Increasing Redis capacity
- PostgreSQL read replicas (future)

No application logic changes are required.

---

# 11. Future Architecture

- RabbitMQ backend
- Kafka backend
- Plugin SDK
- Multi-tenancy
- Kubernetes
- Distributed scheduler
- Auto-scaling workers
- Cloud deployment
