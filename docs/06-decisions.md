# QueueX Architecture Decision Records (ADR)

**Version:** 1.1  
**Status:** Living Document

---

# ADR-001: FastAPI as Backend Framework

**Status:** Accepted  
**Date:** 2026-08-06  
**Owner:** Aditya Aryan

## Context
QueueX is API-first and requires asynchronous I/O, automatic API documentation, request validation, and high performance.

## Decision
Use **FastAPI**.

## Alternatives
- Django
- Flask
- Express.js
- NestJS

## Rationale
FastAPI provides async support, OpenAPI generation, Pydantic validation, dependency injection, and excellent performance.

## Consequences

**Positive**
- Fast development
- Type-safe APIs
- High performance

**Negative**
- Smaller ecosystem than Django

---

# ADR-002: PostgreSQL as Primary Database

**Status:** Accepted

## Decision
Use PostgreSQL for persistent storage.

## Alternatives
- MongoDB
- MySQL
- SQLite

## Rationale
ACID transactions, JSONB support, indexing, relational integrity.

## Consequences

**Positive**
- Strong consistency
- Mature ecosystem

**Negative**
- Schema migrations required

---

# ADR-003: Redis as Queue Backend

**Status:** Accepted

## Decision
Use Redis as the initial queue backend.

## Alternatives
- RabbitMQ
- Kafka
- AWS SQS

## Rationale
Simple deployment, low latency, excellent Python support.

## Consequences

**Positive**
- Easy local development
- High throughput

**Negative**
- Fewer advanced routing features than RabbitMQ

---

# ADR-004: Celery Workers

**Status:** Accepted

## Decision
Use Celery for background processing.

## Alternatives
- RQ
- Dramatiq
- Custom worker framework

## Consequences

**Positive**
- Mature retry model
- Scheduling support

**Negative**
- More configuration than lightweight libraries

---

# ADR-005: UUID Primary Keys

**Status:** Accepted

## Decision
Use UUIDs across the system.

## Consequences

**Positive**
- Globally unique
- Distributed friendly

**Negative**
- Larger indexes

---

# ADR-006: JWT Authentication

**Status:** Accepted

## Decision
JWT access tokens + refresh tokens.

## Consequences

**Positive**
- Stateless
- Horizontally scalable

**Negative**
- Revocation complexity

---

# ADR-007: WebSockets

**Status:** Accepted

## Decision
Use WebSockets for live updates.

## Alternatives
- Polling
- SSE

## Consequences

**Positive**
- Low latency dashboard

**Negative**
- Persistent connection management

---

# ADR-008: Docker

**Status:** Accepted

## Decision
Containerize every service.

## Consequences

**Positive**
- Reproducible environments

**Negative**
- Extra operational layer

---

# ADR-009: Job Handler Pattern

**Status:** Accepted

## Decision
Each job type implements a common JobHandler interface.

## Consequences

**Positive**
- Open/Closed Principle
- Easy extensibility

**Negative**
- Additional abstraction

---

# ADR-010: Layered Architecture

**Status:** Accepted

## Decision

Presentation → API → Service → Repository → Database

## Consequences

**Positive**
- Separation of concerns
- Easier testing

**Negative**
- More project structure

---

# ADR-011: At-Least-Once Delivery

**Status:** Accepted

## Context
Distributed systems cannot practically guarantee exactly-once execution without significant complexity.

## Decision
QueueX guarantees **At-Least-Once Delivery**.

## Alternatives
- At-most-once
- Exactly-once

## Rationale
Reliability is preferred over avoiding duplicate execution.

## Consequences

**Positive**
- Jobs are not silently lost.

**Negative**
- Duplicate execution is possible.

**Mitigation**
- Idempotent job handlers.

---

# ADR-012: Lease-Based Job Reservation

**Status:** Accepted

## Decision
Workers reserve jobs using leases with expiration timestamps.

## Rationale
If a worker crashes, the lease expires and another worker can reclaim the job.

## Consequences

**Positive**
- Automatic recovery
- No permanently stuck jobs

**Negative**
- Lease renewal logic required

---

# ADR-013: Idempotency Keys

**Status:** Accepted

## Decision
Support an `Idempotency-Key` for job submission.

## Rationale
Network retries and duplicate client requests should not create duplicate jobs.

## Consequences

**Positive**
- Safe retries
- Better client experience

**Negative**
- Additional storage and lookup

---

# ADR-014: Controlled Job State Machine

**Status:** Accepted

## Decision
Allow only predefined state transitions.

Example:

CREATED → QUEUED → RESERVED → RUNNING → COMPLETED

or

RUNNING → FAILED → RETRY_SCHEDULED → QUEUED

## Consequences

**Positive**
- Prevents invalid transitions
- Simplifies debugging

---

# ADR-015: Queue Abstraction Layer

**Status:** Accepted

## Decision
Define a queue interface with a Redis implementation.

## Rationale
Future queue backends should be swappable.

## Future Implementations
- RabbitMQ
- Kafka
- AWS SQS

---

# ADR-016: Soft Delete

**Status:** Accepted

## Decision
Jobs use `deleted_at` instead of permanent deletion.

## Consequences

**Positive**
- Audit history preserved

**Negative**
- Periodic cleanup required

---

# ADR-017: Structured Logging

**Status:** Accepted

## Decision
Every log entry includes:

- timestamp
- trace_id
- job_id
- worker_id
- log_level

## Consequences

**Positive**
- Easier debugging
- Better observability

---

# ADR-018: Repository Pattern

**Status:** Accepted

## Decision
Database access occurs through repository classes.

## Consequences

**Positive**
- Easier testing
- Cleaner business logic

---

# ADR-019: API Boundary Validation

**Status:** Accepted

## Decision
Validate requests with Pydantic before entering the service layer.

## Consequences

**Positive**
- Consistent validation
- Cleaner services

---

# ADR-020: Configuration Management

**Status:** Accepted

## Decision
All runtime configuration comes from environment variables.

Examples

- Database URL
- Redis URL
- JWT Secret
- Retry delay
- Worker concurrency

## Consequences

**Positive**
- Twelve-factor friendly
- Easy deployment across environments

**Negative**
- Environment management required

---

# Future ADR Process

Every new architectural decision should include:

1. Context
2. Decision
3. Alternatives
4. Rationale
5. Consequences
6. Status
7. Date
