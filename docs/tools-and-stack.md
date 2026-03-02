```md id="2z9k2h"
# Tools and Stack Report (DevSprint 2026)

This project uses a microservices + DevOps-oriented stack chosen for **reliability, observability, and clean service boundaries**.

---

## Backend

### Node.js + TypeScript

- Enables rapid development with strong typing, maintainability, and clear contracts between services.
- Works well for microservices and shared types.

### Express

- Lightweight HTTP server framework suitable for small independent services.
- Simple routing and middleware model helps keep each service focused.

### MongoDB

- Persistent storage for users, inventory, and orders.
- Flexible document model supports fast iteration during hackathon development.

### Redis

Used for multiple performance and reliability features:

- **BullMQ** queue backend storage
- **Pub/Sub** for order status events (`orders:status`)
- **Cache** for fast stock checks (`stock:<itemId>`)
- **Rate limiting counters** (login throttling)

### BullMQ

- Provides durable asynchronous job processing for the kitchen pipeline.
- Separates “order accepted” from “order prepared”, improving responsiveness and resilience.

### Socket.IO

- Enables real-time order status updates without polling.
- Notification service emits updates to per-order rooms for targeted delivery.

### Zod

- Validates request payloads with consistent error handling.
- Prevents invalid data from entering service logic or databases.

### Helmet + CORS

- Basic security hardening for HTTP services.
- Restricts and manages cross-origin access during development and demo.

### Prometheus Metrics (`prom-client`)

- Standard `/metrics` endpoint in Prometheus text format.
- Enables live visibility into throughput and latency per service.

---

## Frontend

### React + TypeScript + Vite

- Fast developer workflow with a type-safe UI layer.
- Supports clear separation of student and admin experiences.

### Utility-first styling (custom UI components)

- Consistent UI patterns and quick iteration for the admin dashboard and student SPA.

### Socket.IO Client

- Live updates for order tracking in the student app.
- Matches the event-driven architecture and avoids polling traffic.

---

## DevOps / Delivery

### Docker + Docker Compose

- Reproducible local deployment with a single command:
  - `docker compose up -d --build`
- Runs each microservice in its own container to demonstrate microservice separation.

### GitHub Actions (CI)

- Automated test execution and validation on pushes to `main`.
- Helps demonstrate CI/CD practices and prevents regressions during iteration.

---

## Why This Stack Fits DevSprint 2026

- Clear microservices boundaries (identity, gateway, stock, kitchen, notification)
- Queue-based asynchronous processing (BullMQ)
- Observability (`/health`, `/metrics`)
- Reproducible deployment (Docker Compose)
- CI automation (GitHub Actions)
```
