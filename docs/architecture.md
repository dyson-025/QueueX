# QueueX Architecture

Version: 1.0

---

# 1. Overview

QueueX follows a distributed service architecture where the API server is responsible for accepting client requests while dedicated worker processes execute long-running tasks asynchronously.

The architecture separates request handling from task execution, improving scalability, responsiveness, and fault tolerance.

---

# 2. High Level Architecture

                           +----------------+
                           |     Client     |
                           +----------------+
                                    |
                             HTTP / WebSocket
                                    |
                                    ▼
                     +---------------------------+
                     |      FastAPI Server       |
                     +---------------------------+
                      |        |          |
                      |        |          |
                      ▼        ▼          ▼
               PostgreSQL   Redis      WebSocket
                              |
                        Redis Queue
                              |
          ------------------------------------------
          |                |                |
          ▼                ▼                ▼
     Worker 1         Worker 2         Worker 3
          |                |                |
          -------------------------------
                      |
                      ▼
                 PostgreSQL

---

# 3. Components

## 3.1 Client

Responsibilities

- User Authentication
- Submit Jobs
- View Job Status
- Cancel Jobs
- Retry Jobs
- Dashboard
- Receive Real-time Updates

Technology

- React
- TypeScript

---

## 3.2 API Server

Responsibilities

- Authentication
- Authorization
- Validate Requests
- Create Jobs
- Read Jobs
- Cancel Jobs
- Push Jobs to Redis
- Notify Clients

Technology

- FastAPI

---

## 3.3 PostgreSQL

Stores

- Users
- Jobs
- Worker Information
- Job History
- Retry Information

Chosen because

- ACID Transactions
- Relational Data
- Strong Consistency
- Mature Ecosystem

---

## 3.4 Redis

Responsibilities

- Message Queue
- Job Scheduling
- Cache
- Temporary State

Chosen because

- Extremely Fast
- In-memory
- Reliable Queue Operations
- Widely Used

---

## 3.5 Workers

Responsibilities

- Pull Jobs
- Execute Jobs
- Retry Failed Jobs
- Update Status
- Send Heartbeats

Workers are completely independent from the API server.

Multiple workers can execute simultaneously.

---

## 3.6 WebSocket Server

Responsibilities

- Notify Job Status
- Worker Status
- Queue Statistics

Without WebSockets users would need to refresh repeatedly.

---

# 4. Data Flow

## Job Submission

User

↓

POST /jobs

↓

FastAPI

↓

Validate Request

↓

Store Metadata in PostgreSQL

↓

Push Job ID into Redis Queue

↓

Return Response

↓

Worker Pulls Job

↓

Execute Job

↓

Update PostgreSQL

↓

Notify Client

---

# 5. Authentication Flow

Client

↓

Login

↓

JWT Generated

↓

Store JWT

↓

Authenticated Requests

↓

Authorization Check

↓

API Access

---

# 6. Worker Flow

Worker Starts

↓

Registers Itself

↓

Waits for Jobs

↓

Receives Job

↓

Execute

↓

Success?

↓

Yes

↓

Completed

↓

No

↓

Retry

↓

Maximum Retry?

↓

Yes

↓

Dead Letter Queue

---

# 7. Deployment Architecture

                Nginx
                  |
          -----------------
          |               |
      FastAPI         React
          |
      PostgreSQL
          |
        Redis
          |
     Worker Pool

Everything runs inside Docker containers.

---

# 8. Why This Architecture?

## Separate API & Workers

Benefits

- API remains responsive
- Horizontal scaling
- Independent deployment

---

## Redis

Benefits

- Fast Queue
- Lightweight
- Reliable
- Excellent Celery Support

---

## PostgreSQL

Benefits

- Reliable
- ACID
- Strong SQL Support

---

## Docker

Benefits

- Same Environment Everywhere
- Easy Deployment
- Easy Development

---

## WebSockets

Benefits

- Real-time Dashboard
- Better User Experience

---

# 9. Scalability

The system scales by

- Adding more workers
- Increasing Redis resources
- Scaling FastAPI horizontally
- Read replicas for PostgreSQL (future)

No application code changes are required.

---

# 10. Fault Tolerance

If

Worker crashes

↓

Job remains in Redis

↓

Another Worker Processes It

If

API crashes

↓

Workers continue

If

Worker repeatedly fails

↓

Dead Letter Queue

---

# 11. Future Improvements

- RabbitMQ Support
- Kafka Support
- Kubernetes
- Multi-region Deployment
- Distributed Scheduler
- Multi-tenancy
- Metrics Dashboard
- Auto Scaling
