````md
# Requirement Analysis & System Architecture (DevSprint 2026)

## Problem Statement

University cafeterias often suffer from long queues, unclear stock availability, and poor order visibility.  
This project provides a **microservices-based cafeteria ordering system** that supports:

- secure authentication
- accurate inventory handling
- wallet-based payments
- asynchronous kitchen processing
- real-time order status updates without polling
- operational visibility via metrics/health checks

---

## Requirements

### Functional Requirements

1. **Authentication**
   - Students/admins log in and receive JWT tokens.
   - Services validate JWT where appropriate (student-facing routes).

2. **Menu & Inventory**
   - Students can view available items, prices, and details.
   - Stock decrements are atomic and safe under concurrency.

3. **Wallet**
   - Students have a wallet balance.
   - Admins can recharge wallets.
   - Wallet debit occurs during order creation.

4. **Ordering Workflow**
   - Student places order via a single gateway endpoint.
   - The order is persisted and processed asynchronously.

5. **Real-Time Tracking**
   - Students track order status via Socket.IO (no polling).
   - Status updates flow through Redis Pub/Sub and a Notification service.

6. **Admin Operations**
   - Health checks for each service.
   - Metrics scraping via Prometheus format.
   - Chaos testing endpoint to simulate service failures.
   - Admin tables: students, inventory, orders, wallet transactions.

### Non-Functional Requirements

1. **Idempotency**
   - Wallet recharge and debit must be idempotent.
   - Stock decrement must be idempotent.
   - Order creation uses an Idempotency-Key to prevent duplicates.

2. **Resilience**
   - Asynchronous kitchen pipeline via queue.
   - Failure paths should avoid double charging or double decrementing.

3. **Observability**
   - Each service exposes `/health` and `/metrics`.

4. **Reproducible Local Deployment**
   - Entire system runs using Docker Compose.

---

## Architecture Overview

### Services

- **Identity Service**
  - Auth/login, `/me`, wallet operations, admin student management
- **Gateway Service**
  - Entry point for orders, integrates wallet + stock, persists orders, enqueues kitchen jobs
- **Stock Service**
  - Inventory list, atomic decrement with idempotency, upsert/restock
- **Kitchen Worker**
  - BullMQ consumer to simulate cooking and publish status
- **Notification Service**
  - Redis Pub/Sub subscriber and Socket.IO hub
- **MongoDB**
  - Persistent data: users, inventory, orders
- **Redis**
  - BullMQ backend (queue), Pub/Sub events, cache, rate limit counters

---

## Data Flow (Event Driven)

### Text Diagram

```text
[Student SPA]  <--JWT-->  [Identity Service]
     |
     |  POST /orders (JWT + Idempotency-Key)
     v
[Gateway Service] --(cache check)-> Redis stock:<itemId>
     |
     |  (wallet debit) -> Identity /wallet/debit (admin-secret + Idempotency-Key)
     |  (stock decrement) -> Stock /stock/decrement (Idempotency-Key)
     |
     |  enqueue job (Queue="orders")
     v
   [Redis]  BullMQ queue: bull:orders:wait
     |
     v
[Kitchen Worker]  (3–7s processing)
     |
     |  PUBLISH orders:status {orderId,status,message}
     v
   [Redis Pub/Sub]  channel: orders:status
     |
     v
[Notification Service]  (Redis subscriber + Socket.IO hub)
     |
     | emit to room: order:<orderId>
     v
[Student SPA LiveStatusPage]  real-time updates (no polling)
```
````

---

## Key Design Choices

### 1) Gateway Orchestration

The Gateway orchestrates:

- wallet debit
- stock decrement
- order persistence
- enqueueing kitchen jobs

This centralizes the business workflow while keeping services single-responsibility.

### 2) Idempotency Everywhere

Idempotency is enforced on:

- wallet recharge (admin)
- wallet debit (gateway)
- stock decrement (gateway)
- order creation using Idempotency-Key

This prevents double charges and double stock reductions during retries.

### 3) Real-Time Updates Without Polling

Kitchen publishes status events to Redis Pub/Sub. Notification forwards them via Socket.IO to the student UI.
This satisfies real-time requirements efficiently and avoids polling traffic.

### 4) Observability & Ops

Each service exposes:

- `/health` for readiness checks
- `/metrics` in Prometheus format

Admin dashboard consumes both to show live status and performance.

---

## Deployment Model (Local)

Docker Compose starts:

- mongo, redis
- identity, gateway, stock, kitchen, notification
- frontend

One-command run:

- `docker compose up -d --build`
