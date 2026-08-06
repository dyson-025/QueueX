# QueueX Database Design

**Version:** 1.1  
**Status:** Draft

---

# 1. Overview

QueueX uses PostgreSQL as its primary persistent datastore.

PostgreSQL stores all durable application data including users, queues, jobs, execution history, worker metadata, metrics, and audit logs.

Redis is **not** a source of truth. It is used only for queue management, scheduling, caching, and temporary processing state.

---

# 2. Design Principles

The schema is designed around:

- ACID transactions
- Data integrity
- Normalization
- Horizontal worker scalability
- Efficient querying
- Auditability
- Fault recovery

---

# 3. Entity Relationship Diagram

```text
Users
 │
 ├──────────────┐
 │              │
RefreshTokens  Jobs ─────────── Queues
                 │
      ┌──────────┼──────────────┐
      │          │              │
 JobEvents   JobLogs     QueueMetrics
      │          │
      └──────────┘
           │
        Workers
           │
           ▼
   WorkerHeartbeats
```

---

# 4. Tables

## 4.1 Users

| Column | Type | Constraints |
|--------|------|-------------|
| id | UUID | PK |
| name | VARCHAR(100) | NOT NULL |
| email | VARCHAR(255) | UNIQUE |
| password_hash | TEXT | NOT NULL |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |

Index: email

---

## 4.2 RefreshTokens

| Column | Type |
|--------|------|
| id | UUID |
| user_id | UUID |
| token_hash | TEXT |
| expires_at | TIMESTAMP |
| created_at | TIMESTAMP |

FK: user_id → Users.id

---

## 4.3 Queues

Stores queue configuration.

| Column | Type |
|--------|------|
| id | UUID |
| name | VARCHAR(50) |
| description | TEXT |
| priority_mode | BOOLEAN |
| max_retries | INTEGER |
| is_paused | BOOLEAN |
| created_at | TIMESTAMP |

Unique: name

---

## 4.4 Jobs

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | PK |
| user_id | UUID | Owner |
| queue_id | UUID | Queue |
| job_type | VARCHAR(100) | Type |
| payload | JSONB | Input |
| result | JSONB | Output |
| priority | SMALLINT | 1-5 |
| status | JobStatus | ENUM |
| retry_count | INTEGER | Current retries |
| max_retries | INTEGER | Limit |
| next_retry_at | TIMESTAMP | Retry schedule |
| scheduled_at | TIMESTAMP | Delayed execution |
| reserved_by_worker_id | UUID | Lease owner |
| lease_expires_at | TIMESTAMP | Lease expiry |
| timeout_seconds | INTEGER | Timeout |
| idempotency_key | UUID | Duplicate prevention |
| error_code | VARCHAR(100) | Last error |
| error_message | TEXT | Last error |
| created_at | TIMESTAMP | Created |
| started_at | TIMESTAMP | Started |
| completed_at | TIMESTAMP | Completed |
| deleted_at | TIMESTAMP | Soft delete |

Indexes

- status
- queue_id
- user_id
- created_at
- idempotency_key

Constraints

- CHECK(priority BETWEEN 1 AND 5)
- CHECK(retry_count >= 0)

FKs

- user_id → Users.id
- queue_id → Queues.id

---

## 4.5 Workers

| Column | Type |
|--------|------|
| id | UUID |
| worker_name | VARCHAR |
| hostname | VARCHAR |
| status | WorkerStatus |
| version | VARCHAR |
| current_queue | UUID |
| max_concurrency | INTEGER |
| current_job_id | UUID |
| started_at | TIMESTAMP |
| last_heartbeat | TIMESTAMP |

---

## 4.6 JobEvents

Stores every lifecycle transition.

| Column | Type |
|--------|------|
| id | UUID |
| job_id | UUID |
| previous_status | JobStatus |
| current_status | JobStatus |
| changed_by | VARCHAR |
| changed_at | TIMESTAMP |

FK: job_id → Jobs.id

---

## 4.7 JobLogs

| Column | Type |
|--------|------|
| id | UUID |
| job_id | UUID |
| worker_id | UUID |
| log_level | VARCHAR |
| message | TEXT |
| created_at | TIMESTAMP |

Future: monthly partitioning.

---

## 4.8 WorkerHeartbeats

| Column | Type |
|--------|------|
| id | UUID |
| worker_id | UUID |
| cpu_usage | FLOAT |
| memory_usage | FLOAT |
| running_jobs | INTEGER |
| queue_length_seen | INTEGER |
| uptime_seconds | BIGINT |
| heartbeat_time | TIMESTAMP |

---

## 4.9 QueueMetrics

| Column | Type |
|--------|------|
| id | UUID |
| queue_id | UUID |
| timestamp | TIMESTAMP |
| pending_jobs | INTEGER |
| running_jobs | INTEGER |
| completed_jobs | INTEGER |
| failed_jobs | INTEGER |
| throughput | FLOAT |
| avg_latency_ms | FLOAT |

---

# 5. ENUM Types

## JobStatus

- CREATED
- QUEUED
- RESERVED
- RUNNING
- FAILED
- RETRY_SCHEDULED
- COMPLETED
- CANCELLED
- DEAD_LETTER

## WorkerStatus

- IDLE
- BUSY
- PAUSED
- OFFLINE

---

# 6. Status Transition Rules

| From | To |
|------|----|
| CREATED | QUEUED |
| QUEUED | RESERVED |
| RESERVED | RUNNING |
| RUNNING | COMPLETED |
| RUNNING | FAILED |
| FAILED | RETRY_SCHEDULED |
| RETRY_SCHEDULED | QUEUED |
| FAILED | DEAD_LETTER |
| QUEUED | CANCELLED |

---

# 7. Why JSONB?

`payload` stores arbitrary job inputs.

`result` stores arbitrary outputs.

Benefits:

- Flexible schemas
- Searchable
- Indexable
- Supports multiple job types

---

# 8. Indexing Strategy

Indexes on:

- email
- queue_id
- status
- created_at
- idempotency_key
- last_heartbeat

---

# 9. Future Tables

- APIKeys
- Organizations
- Teams
- Notifications

---

# 10. Design Decisions

### Why PostgreSQL?

- ACID compliance
- JSONB support
- Mature ecosystem

### Why UUID?

- Globally unique
- Distributed-friendly

### Why JobEvents?

- Full audit trail
- Timeline visualization
- Analytics

### Why Lease Fields?

Workers reserve jobs using leases. If the lease expires because a worker crashes, another worker can safely reclaim the job.

