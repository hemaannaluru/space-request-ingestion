# Request Ingestion & Minute Metrics Service

This is a simple Spring Boot service that accepts incoming requests, tracks unique IDs per minute, and generates per-minute metrics.

The main goal is correctness under high load and safe behaviour when multiple instances are running.

---

## What the service does

- Accepts a GET request with an `id`
- Tracks unique IDs per calendar minute
- Ignores duplicate IDs within the same minute
- Works correctly with multiple application instances
- Every minute:
  - counts unique IDs
  - logs the result
  - stores the data in the database

---

## Tech stack

- Java 8+
- Spring Boot
- Maven
- MySQL
- JdbcTemplate

---

## How to run

### Database tables

```sql
CREATE TABLE minute_seen (
  minute_start DATETIME NOT NULL,
  request_id VARCHAR(64) NOT NULL,
  PRIMARY KEY (minute_start, request_id)
);

CREATE TABLE minute_stats (
  minute_start DATETIME PRIMARY KEY,
  unique_id_count BIGINT NOT NULL
);

Start the application
mvn spring-boot:run


Application runs on http://localhost:8080.
---
API
Endpoint
GET /api/space/accept

Parameters

id (required)

Example
curl "http://localhost:8080/api/space/accept?id=100"
curl "http://localhost:8080/api/space/accept?id=101"
curl "http://localhost:8080/api/space/accept?id=100"

Response

ok – request processed

failed – invalid input or unexpected error

How deduplication works

Requests are grouped by calendar minute

Each (minute, id) is stored only once

MySQL enforces uniqueness using a composite primary key:

(minute_start, request_id)


Duplicate inserts are ignored by the database

Multi-instance support (Extension 2)

The application works correctly when multiple instances are running behind a load balancer.

All instances write to the same database.
The database guarantees global deduplication, so the same ID is counted only once per minute, even if two servers receive it at the same time.

Per-minute aggregation

A scheduled job runs once per minute

It processes the previous minute

It:

counts unique IDs

writes a log entry like:

2026-02-06T10:15 -> 3 unique ids


stores the result in minute_stats

Error handling

Missing or empty id → failed

Duplicate IDs → safely ignored

Unexpected errors → failed

Extensions implemented

Extension 2 – Multi-instance / load-balancer safe deduplication
