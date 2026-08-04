# Queue System Design

Version: 1.0

---

# 1. Overview

The Queue System is responsible for accepting submitted jobs, storing them temporarily, distributing them to available workers, handling retries, and ensuring reliable execution.

Redis is used as the message broker because of its high performance, simplicity, and excellent integration with Celery.

---

# 2. Objectives

The queue system should:

- Execute jobs asynchronously
- Process jobs in FIFO order by default
- Support priority-based execution
- Retry failed jobs automatically
- Support delayed and scheduled execution
- Prevent duplicate processing
- Recover from worker failures
- Scale horizontally by adding workers

---

# 3. Job Lifecycle

                    Create Job
                         │
                         ▼
                     Validate
                         │
                         ▼
               Save Metadata (PostgreSQL)
                         │
                         ▼
                  Push Job to Redis
                         │
                         ▼
                  Waiting Queue
                         │
             Worker Fetches Job
                         │
                         ▼
                     Running
                 ┌───────────────┐
                 │               │
                 ▼               ▼
           Completed         Failed
                                  │
                           Retry Allowed?
                           ┌────────────┐
                           │            │
                           ▼            ▼
                      Retry Queue   Dead Letter Queue

---

# 4. Queue Types

## Default Queue

Purpose

Processes all standard jobs.

Examples

- Email
- Report Generation
- AI Processing

---

## Priority Queue

High-priority jobs execute before lower-priority jobs.

Priority Levels

1 - Critical

2 - High

3 - Normal

4 - Low

5 - Background

---

## Delayed Queue

Jobs remain inactive until a scheduled timestamp.

Examples

- Send email tomorrow
- Generate report after 1 hour

---

## Dead Letter Queue

Stores jobs that permanently failed.

Reasons

- Maximum retries exceeded
- Invalid payload
- Unexpected exceptions

These jobs require manual inspection.

---

# 5. Worker Lifecycle

Worker Starts

↓

Register Worker

↓

Heartbeat Begins

↓

Listen for Jobs

↓

Receive Job

↓

Lock Job

↓

Execute

↓

Update Status

↓

Wait for Next Job

Workers continuously poll Redis for new jobs.

---

# 6. Retry Strategy

If a job fails

↓

Increment Retry Count

↓

Retry Count < Max Retry?

↓

Yes

↓

Wait Retry Delay

↓

Push Back to Queue

↓

Retry

↓

No

↓

Move to Dead Letter Queue

Default

Maximum Retries = 3

Retry Delay = 30 seconds

Future versions may use exponential backoff.

---

# 7. Worker Failure Recovery

Scenario

Worker crashes while processing.

Recovery

- Job lock expires.
- Job becomes available again.
- Another worker picks it up.
- Processing resumes.

This prevents permanent job loss.

---

# 8. Idempotency

Some operations should never execute twice.

Example

Charging a credit card twice is incorrect.

QueueX assigns every job a unique Job ID.

Workers verify whether a job has already completed before executing it.

This ensures safe retries.

---

# 9. Queue States

Waiting

Job is in Redis.

Running

Worker is executing.

Completed

Execution finished successfully.

Failed

Execution failed.

Retrying

Waiting for another attempt.

Cancelled

Cancelled before execution.

Dead Letter

Maximum retries exceeded.

---

# 10. Worker Heartbeats

Every worker periodically sends a heartbeat.

Heartbeat contains

- Worker ID
- Timestamp
- Current Job
- CPU Usage (future)
- Memory Usage (future)

If no heartbeat is received within a configurable timeout, the worker is marked Offline.

---

# 11. Horizontal Scaling

QueueX scales horizontally.

Current

API

↓

Redis

↓

Worker 1

Add More Workers

API

↓

Redis

↓

Worker 1

Worker 2

Worker 3

Worker 4

No API changes are required.

---

# 12. Queue Metrics

The system tracks:

- Waiting Jobs
- Running Jobs
- Completed Jobs
- Failed Jobs
- Average Execution Time
- Retry Count
- Queue Throughput
- Active Workers

These metrics are displayed on the dashboard.

---

# 13. Error Handling

Worker Exception

↓

Log Error

↓

Increment Retry Count

↓

Retry if allowed

↓

Otherwise move to Dead Letter Queue

Every failure is logged for debugging.

---

# 14. Future Improvements

- RabbitMQ Support
- Kafka Support
- Priority Scheduling Algorithms
- Distributed Scheduling
- Worker Auto Scaling
- Rate Limiting
- Workflow Chaining
- Cron Expressions
- Queue Persistence Across Regions

---

# 15. Design Decisions

Why Redis?

- Extremely fast
- Simple architecture
- Excellent Celery support
- In-memory performance
- Widely used in production

Why Separate Workers?

- Keeps API responsive
- Independent scaling
- Better fault isolation

Why Dead Letter Queue?

- Prevents infinite retries
- Makes debugging easier
- Improves system reliability

Why Heartbeats?

- Detect worker failures
- Enable recovery
- Provide monitoring information
