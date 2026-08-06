# QueueX Queue System Design

**Version:** 1.1  
**Status:** Draft

---

# 1. Overview

The Queue System is the core execution engine of QueueX. It is responsible for accepting jobs, scheduling execution, reserving jobs for workers, coordinating retries, recovering from failures, and ensuring reliable asynchronous processing.

Redis is used as the initial queue backend, while PostgreSQL remains the system of record.

---

# 2. Design Goals

- Asynchronous execution
- FIFO ordering within equal priority
- Priority scheduling
- Delayed execution
- Lease-based job reservation
- Automatic retries
- Dead Letter Queue (DLQ)
- At-least-once delivery
- Horizontal scalability
- Worker fault recovery

---

# 3. Queue Architecture

```text
           Client
              │
              ▼
         QueueX API
              │
     Store Job Metadata
       (PostgreSQL)
              │
              ▼
     Pending Queue (Redis)
              │
      ┌───────┼────────┐
      ▼       ▼        ▼
  Worker1  Worker2  WorkerN
      │
      ▼
Update Database + Events
      │
      ▼
 WebSocket Dashboard
```

---

# 4. Queue Types

## Pending Queue
Ready-to-run jobs.

## Priority Queue
Priority levels:

1. Critical
2. High
3. Normal
4. Low
5. Background

FIFO ordering is preserved within the same priority.

## Delayed Queue
Jobs become visible only after `scheduled_at`.

## Retry Queue
Temporarily stores failed jobs waiting for another attempt.

## Dead Letter Queue (DLQ)
Stores permanently failed jobs for inspection and replay.

---

# 5. Job Lifecycle

```text
Create
   ↓
Validate
   ↓
Persist (PostgreSQL)
   ↓
Pending Queue
   ↓
Reserve
   ↓
Running
 ┌──────────────┐
 ▼              ▼
Completed    Failed
                 │
          Retry Available?
             │        │
            Yes      No
             │        │
             ▼        ▼
       Retry Queue    DLQ
```

---

# 6. Reservation & Leasing

Workers do **not** immediately own a job.

Execution flow:

```text
Worker Polls
      ↓
Atomically Reserve Job
      ↓
Lease Created
      ↓
Heartbeat Renewal
      ↓
Complete or Fail
```

Lease fields:

- reserved_by_worker_id
- lease_expires_at

If the lease expires, another worker may safely reclaim the job.

---

# 7. Worker Lifecycle

```text
Worker Starts
      ↓
Register
      ↓
Heartbeat Thread
      ↓
Poll Queue
      ↓
Reserve Job
      ↓
Execute Handler
      ↓
Acknowledge
      ↓
Next Job
```

Workers shut down gracefully by finishing reserved work before exiting.

---

# 8. Retry Strategy

Default:

- Max retries: 3
- Initial delay: 30 seconds

Retry flow:

```text
Failure
   ↓
retry_count++
   ↓
Calculate next_retry_at
   ↓
Retry Queue
   ↓
Scheduler
   ↓
Pending Queue
```

Future versions may use exponential backoff with jitter.

---

# 9. Failure Recovery

## Worker Crash

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

## API Crash

- Redis retains queued jobs.
- Workers continue processing.
- API resumes after restart.

---

# 10. Delivery Semantics

QueueX guarantees **At-Least-Once Delivery**.

Implications:

- Jobs are never intentionally dropped.
- Duplicate execution is possible after crashes.
- Job handlers should be idempotent.

---

# 11. Idempotency

Clients may submit an `Idempotency-Key`.

Workers verify:

- existing completion
- duplicate requests
- completed side effects

before executing sensitive operations.

---

# 12. Scheduler

Scheduler responsibilities:

- Move delayed jobs into Pending Queue
- Release retry jobs at `next_retry_at`
- Expire stale leases
- Detect abandoned jobs

---

# 13. Worker Heartbeats

Heartbeat interval: 10 seconds

Contains:

- worker_id
- current_job
- status
- uptime
- running_jobs
- cpu_usage
- memory_usage

Missing heartbeats beyond timeout mark the worker offline.

---

# 14. Concurrency Model

- Multiple workers process jobs in parallel.
- Each worker processes up to its configured concurrency.
- A reserved job belongs to only one worker at a time.
- Atomic reservation prevents duplicate claims.

---

# 15. Queue Metrics

Collected metrics:

- Queue depth
- Running jobs
- Throughput
- Success rate
- Failure rate
- Retry count
- Average latency
- Worker utilization
- DLQ size

---

# 16. Error Handling

```text
Worker Exception
      ↓
Structured Log
      ↓
Job Event Recorded
      ↓
Retry or DLQ
      ↓
Dashboard Update
```

Every failure generates:

- JobEvent
- JobLog
- Metrics update

---

# 17. Scalability

QueueX scales by:

- Adding workers
- Running multiple API instances
- Increasing Redis resources
- Queue partitioning (future)
- Multiple queue backends (future)

---

# 18. Design Decisions

## Why Redis?

- Low latency
- Atomic operations
- Mature ecosystem
- Excellent Celery integration

## Why Lease-Based Reservation?

- Prevents lost jobs
- Enables crash recovery

## Why Dead Letter Queue?

- Prevents infinite retries
- Supports debugging and replay

## Why At-Least-Once Delivery?

- Reliability is prioritized over exactly-once complexity.

---

# 19. Future Enhancements

- RabbitMQ backend
- Kafka backend
- Queue abstraction plugins
- Workflow orchestration
- Cron scheduling
- Auto-scaling workers
- Multi-region queues
- Distributed scheduler
