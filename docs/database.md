# Database Design

Version: 1.0

---

# 1. Overview

QueueX uses PostgreSQL as its primary relational database.

The database stores persistent application data including users, jobs, workers, execution history, and audit logs.

Redis is **not** used as permanent storage. It is used only for queue management, caching, and temporary processing state.

---

# 2. Design Principles

The database is designed to satisfy the following goals:

- Data Integrity
- ACID Transactions
- Scalability
- Normalization
- Easy Querying
- Auditability

---

# 3. Entity Relationship Diagram

Users
│
├────────────┐
│            │
▼            ▼
Jobs      RefreshTokens
│
│
├──────────────┐
│              │
▼              ▼
JobHistory    JobLogs

Workers
│
▼
WorkerHeartbeats

---

# 4. Tables

## 4.1 Users

Purpose

Stores registered users.

Columns

| Column | Type | Constraints |
|---------|------|-------------|
| id | UUID | Primary Key |
| name | VARCHAR(100) | NOT NULL |
| email | VARCHAR(255) | UNIQUE |
| password_hash | TEXT | NOT NULL |
| created_at | TIMESTAMP | NOT NULL |
| updated_at | TIMESTAMP | NOT NULL |

Indexes

- email

---

## 4.2 RefreshTokens

Purpose

Stores refresh tokens for authenticated sessions.

Columns

| Column | Type |
|---------|------|
| id | UUID |
| user_id | UUID |
| token_hash | TEXT |
| expires_at | TIMESTAMP |
| created_at | TIMESTAMP |

Foreign Keys

user_id → Users.id

---

## 4.3 Jobs

Purpose

Stores all submitted jobs.

Columns

| Column | Type | Description |
|---------|------|-------------|
| id | UUID | Primary Key |
| user_id | UUID | Job Owner |
| queue_name | VARCHAR(50) | Queue |
| job_type | VARCHAR(100) | Processing Type |
| payload | JSONB | Request Data |
| priority | SMALLINT | 1-5 |
| status | VARCHAR(30) | Job State |
| retry_count | INTEGER | Current Retry |
| max_retries | INTEGER | Retry Limit |
| scheduled_at | TIMESTAMP | Delayed Execution |
| created_at | TIMESTAMP | Created Time |
| started_at | TIMESTAMP | Execution Started |
| completed_at | TIMESTAMP | Execution Finished |

Indexes

- status
- queue_name
- created_at
- user_id

Foreign Keys

user_id → Users.id

---

## 4.4 Workers

Purpose

Stores registered workers.

Columns

| Column | Type |
|---------|------|
| id | UUID |
| worker_name | VARCHAR(100) |
| hostname | VARCHAR(255) |
| status | VARCHAR(20) |
| current_job_id | UUID |
| started_at | TIMESTAMP |
| last_heartbeat | TIMESTAMP |

Indexes

- status
- last_heartbeat

---

## 4.5 JobHistory

Purpose

Stores every state transition of a job.

Example

Queued

↓

Running

↓

Retrying

↓

Completed

Columns

| Column | Type |
|---------|------|
| id | UUID |
| job_id | UUID |
| previous_status | VARCHAR |
| current_status | VARCHAR |
| changed_at | TIMESTAMP |
| changed_by | VARCHAR |

Foreign Keys

job_id → Jobs.id

---

## 4.6 JobLogs

Purpose

Stores execution logs.

Columns

| Column | Type |
|---------|------|
| id | UUID |
| job_id | UUID |
| worker_id | UUID |
| log_level | VARCHAR |
| message | TEXT |
| created_at | TIMESTAMP |

Foreign Keys

job_id → Jobs.id

worker_id → Workers.id

---

## 4.7 WorkerHeartbeats

Purpose

Stores periodic worker health updates.

Columns

| Column | Type |
|---------|------|
| id | UUID |
| worker_id | UUID |
| cpu_usage | FLOAT |
| memory_usage | FLOAT |
| active_jobs | INTEGER |
| heartbeat_time | TIMESTAMP |

Foreign Keys

worker_id → Workers.id

---

# 5. Job Status

A job can exist in one of the following states.

Pending

Queued

Running

Completed

Failed

Retrying

Cancelled

DeadLetter

---

# 6. Relationships

Users

1

↓

Many

Jobs

Jobs

1

↓

Many

JobHistory

Jobs

1

↓

Many

JobLogs

Workers

1

↓

Many

WorkerHeartbeats

Workers

1

↓

Many

JobLogs

---

# 7. Why JSONB?

The payload column uses PostgreSQL JSONB.

Advantages

- Flexible request structure
- Searchable
- Indexed
- Supports different job types

Example

{
    "image_url": "...",
    "width": 512,
    "height": 512
}

Another job

{
    "email": "abc@gmail.com",
    "subject": "Welcome"
}

Both use the same payload column.

---

# 8. Indexing Strategy

Indexes are created on:

- email
- status
- queue_name
- user_id
- created_at
- last_heartbeat

This improves

- Login
- Dashboard
- Queue Filtering
- Worker Monitoring

---

# 9. Future Tables

Possible future additions

- ScheduledJobs
- DeadLetterQueue
- QueueMetrics
- Notifications
- APIKeys
- Organizations
- Teams

---

# 10. Database Decisions

Why PostgreSQL?

- ACID Compliance
- JSONB Support
- Strong SQL
- Excellent Performance
- Mature Ecosystem

Why UUID?

- Globally Unique
- Better for Distributed Systems
- Harder to Guess
- Easy Horizontal Scaling

Why Separate JobHistory?

- Complete Audit Trail
- Easier Debugging
- Analytics
- Timeline Visualization
