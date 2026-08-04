# QueueX Architecture Decision Records (ADR)

Version: 1.0

---

# ADR-001: FastAPI as Backend Framework

## Status

Accepted

## Decision

Use **FastAPI** as the primary backend framework.

## Context

QueueX is an API-first application that requires:

- High performance
- Asynchronous request handling
- Automatic API documentation
- Strong type validation
- Easy integration with background workers

## Alternatives Considered

- Django
- Flask
- Express.js
- NestJS

## Why FastAPI?

- Excellent async support
- Automatic OpenAPI/Swagger generation
- Pydantic request validation
- High performance
- Clean dependency injection
- Strong Python ecosystem

## Trade-offs

Pros

- Modern architecture
- Easy to maintain
- Excellent developer experience

Cons

- Smaller ecosystem than Django
- Fewer built-in features

---

# ADR-002: PostgreSQL as Primary Database

## Status

Accepted

## Decision

Use PostgreSQL for persistent storage.

## Context

QueueX stores relational data including:

- Users
- Jobs
- Workers
- Logs
- Job History

## Alternatives

- MongoDB
- MySQL
- SQLite

## Why PostgreSQL?

- ACID compliance
- Excellent indexing
- JSONB support
- Mature ecosystem
- Reliable transactions

## Trade-offs

Pros

- Strong consistency
- Excellent query optimizer
- Rich SQL features

Cons

- Schema migrations required
- Slightly more setup than SQLite

---

# ADR-003: Redis as Message Broker

## Status

Accepted

## Decision

Use Redis for queue management and caching.

## Context

Jobs need fast enqueue and dequeue operations.

## Alternatives

- RabbitMQ
- Kafka
- AWS SQS

## Why Redis?

- Extremely fast
- Simple setup
- Celery integration
- Lightweight
- Supports delayed tasks

## Trade-offs

Pros

- Easy development
- Low latency
- Production proven

Cons

- In-memory storage
- Fewer messaging guarantees than RabbitMQ

---

# ADR-004: Celery for Background Workers

## Status

Accepted

## Decision

Use Celery to execute background jobs.

## Context

Jobs should execute independently of API requests.

## Alternatives

- RQ
- Dramatiq
- Custom worker implementation

## Why Celery?

- Mature ecosystem
- Retry support
- Scheduling support
- Distributed workers
- Strong Redis integration

## Trade-offs

Pros

- Reliable
- Feature-rich
- Widely adopted

Cons

- More configuration than lightweight alternatives

---

# ADR-005: UUID as Primary Keys

## Status

Accepted

## Decision

Use UUID instead of auto-increment integers.

## Context

QueueX is designed for distributed systems.

## Alternatives

- Integer IDs

## Why UUID?

- Globally unique
- Difficult to guess
- Better suited for distributed services
- Easier future scaling

## Trade-offs

Pros

- No ID collisions
- Better security

Cons

- Larger storage
- Less human-readable

---

# ADR-006: JWT Authentication

## Status

Accepted

## Decision

Use JWT access tokens with refresh tokens.

## Context

QueueX exposes stateless REST APIs.

## Alternatives

- Server-side sessions
- OAuth only

## Why JWT?

- Stateless authentication
- Easy horizontal scaling
- Standard for REST APIs

## Trade-offs

Pros

- No server session storage
- Efficient authentication

Cons

- Token revocation is more complex
- Requires secure token handling

---

# ADR-007: WebSockets for Live Updates

## Status

Accepted

## Decision

Use WebSockets for real-time dashboard updates.

## Context

Users need immediate visibility into job progress.

## Alternatives

- HTTP polling
- Server-Sent Events (SSE)

## Why WebSockets?

- Full-duplex communication
- Low latency
- Efficient for frequent updates

## Trade-offs

Pros

- Better user experience
- Reduced unnecessary requests

Cons

- More complex connection management

---

# ADR-008: Docker for Containerization

## Status

Accepted

## Decision

Containerize all services with Docker.

## Context

The application consists of multiple services:

- API
- Redis
- PostgreSQL
- Workers
- Frontend

## Alternatives

- Local installation
- Virtual Machines

## Why Docker?

- Consistent environments
- Easy onboarding
- Simplified deployment

## Trade-offs

Pros

- Reproducible builds
- Environment isolation

Cons

- Slight learning curve
- Additional resource usage

---

# ADR-009: Job Handler Pattern

## Status

Accepted

## Decision

Implement a pluggable Job Handler interface instead of conditional logic.

## Context

QueueX supports multiple job types such as:

- Email
- Image Resize
- PDF Generation
- CSV Export

## Alternatives

- Large if/else blocks
- Switch-case dispatch

## Decision

Each job type implements a common interface.

Example

JobHandler

↓

execute(payload)

↓

Specific Handler

## Benefits

- Open/Closed Principle
- Easy extensibility
- Cleaner code
- Better testing

## Trade-offs

Pros

- Modular
- Scalable
- Maintainable

Cons

- More abstraction
- Slightly more initial setup

---

# ADR-010: Layered Architecture

## Status

Accepted

## Decision

Adopt a layered architecture.

Presentation Layer

↓

API Layer

↓

Service Layer

↓

Repository Layer

↓

Database

## Why?

- Separation of concerns
- Easier testing
- Better maintainability
- Cleaner business logic

## Alternatives

- Monolithic controller logic

## Trade-offs

Pros

- Organized codebase
- Easier refactoring

Cons

- More files
- Additional abstraction

---

# Future ADRs

Future architectural decisions should follow the same format:

- Context
- Decision
- Alternatives
- Rationale
- Trade-offs
- Status

This document should evolve alongside the project as new design decisions are made.
