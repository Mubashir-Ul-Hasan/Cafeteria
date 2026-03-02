This is a solid stack for a high-concurrency microservices system. I've integrated all the dependencies from your `package.json` files into a new **Dependencies & Tech Stack** section. I also included a high-level data flow diagram tag to help the judges visualize that event-driven architecture you described.

---

# Cafeteria Microservices System (DevSprint 2026)

A production-style **microservices cafeteria ordering system** built for DevSprint 2026 (DevOps & Microservices).

Students can browse menu items, place orders, and track progress in real-time (no polling). Admins can monitor service health/metrics, run chaos testing, recharge wallets, and manage inventory and students.

---

## Quick Links (Submission)

- **GitHub Repository:** https://github.com/Mubashir-Ul-Hasan/Cafeteria
- **Demo Video (≤ 3 min):** https://drive.google.com/file/d/1KqkgZD0tmcHOo466Rnc4MuKQFkqUxoeb/view?usp=drive_link

---

## Deliverables (Rulebook Checklist)

✅ **1) Requirement Analysis & System Architecture:** `docs/architecture.md`

✅ **2) Tools and Stack Report:** `docs/tools-and-stack.md`

✅ **3) GitHub Link:** add above

✅ **4) Demonstration Video (≤ 3 minutes):** add above

✅ **5) Deployment Link (optional):** add above

---

## Features

### Student App (SPA)

- JWT login
- Menu browsing + item details
- Wallet balance display
- Order placement with **idempotency**
- Live order tracking via **Socket.IO** (no polling)
- Printable order token

### Admin Dashboard

- Service health grid (`/health`)
- Live Prometheus metrics view (`/metrics`)
- Chaos testing: kill services via `/admin/kill`
- Wallet recharge (idempotent)
- Stock upsert/restock + delete items
- Student management (create/list/delete)
- Orders history + wallet transactions history

---

## System Architecture (Summary)

This system is **event-driven**:

1. Student creates an order via Gateway (`POST /orders`)
2. Gateway performs:

- cache-first stock check (Redis)
- wallet debit (Identity service)
- stock decrement (Stock service)
- stores order in Mongo
- enqueues kitchen job (BullMQ)

3. Kitchen worker processes asynchronously (3–7s simulation)
4. Kitchen publishes status events to Redis Pub/Sub
5. Notification service forwards events to students over Socket.IO rooms
6. Student UI updates in real-time

Full details: `docs/architecture.md`

---

## Dependencies & Tech Stack

### Backend Services (Node.js/TypeScript)

- **Framework:** Express.js with TypeScript
- **Security:** `helmet`, `bcryptjs`, `jsonwebtoken` (JWT)
- **Database:** `mongodb` (Primary), `ioredis` (Caching/Pub-Sub)
- **Async Processing:** `bullmq` (Job queues)
- **Observability:** `prom-client` (Prometheus metrics), `morgan` (Logging)
- **Real-time:** `socket.io`
- **Validation:** `zod`
- **Testing:** `jest`, `supertest`

### Frontend (React/Vite)

- **Framework:** React 19 (TypeScript)
- **Routing:** `react-router-dom`
- **Styling:** `tailwindcss`, `autoprefixer`, `postcss`
- **Icons:** `lucide-react`
- **Real-time:** `socket.io-client`
- **Utilities:** `uuid`

---

## Local Setup (Judge-Friendly)

### Prerequisites

- Docker + Docker Compose installed

### 1) Configure environment variables

This project uses Docker Compose variables: `ADMIN_SECRET`, `JWT_SECRET`.

Create `.env` from the example:

```bash
cp .env.example .env

```

### 2) Run everything

```bash
docker compose up -d --build

```

### 3) Open the app

- Frontend: [http://localhost:5173](https://www.google.com/search?q=http://localhost:5173)

### 4) Verify services are up

```bash
docker compose ps

```

Health endpoints:

- Identity: [http://localhost:7001/health](https://www.google.com/search?q=http://localhost:7001/health)
- Gateway: [http://localhost:7002/health](https://www.google.com/search?q=http://localhost:7002/health)
- Stock: [http://localhost:7003/health](https://www.google.com/search?q=http://localhost:7003/health)
- Kitchen: [http://localhost:7004/health](https://www.google.com/search?q=http://localhost:7004/health)
- Notification: [http://localhost:7005/health](https://www.google.com/search?q=http://localhost:7005/health)

Metrics endpoints:

- Identity: [http://localhost:7001/metrics](https://www.google.com/search?q=http://localhost:7001/metrics)
- Gateway: [http://localhost:7002/metrics](https://www.google.com/search?q=http://localhost:7002/metrics)
- Stock: [http://localhost:7003/metrics](https://www.google.com/search?q=http://localhost:7003/metrics)
- Kitchen: [http://localhost:7004/metrics](https://www.google.com/search?q=http://localhost:7004/metrics)
- Notification: [http://localhost:7005/metrics](https://www.google.com/search?q=http://localhost:7005/metrics)

---

## Demo Flow (Suggested, ≤ 3 minutes)

### Admin demo

1. Open Admin Dashboard (login as admin in the UI)
2. Show health + metrics updating
3. Trigger chaos test (kill a service) and watch health turn red
4. Recharge a student wallet (repeat with same Idempotency-Key to show idempotency)
5. Upsert/restock items and refresh inventory
6. View orders history & wallet transactions

### Student demo

1. Login as a student
2. Place an order
3. Watch live status updates without polling
4. Print token

> Note: Demo credentials depend on your seeded users. If you modified seeds, update credentials here so judges can log in instantly.

---

## Important Ports

- Frontend: 5173
- Identity: 7001
- Gateway: 7002
- Stock: 7003
- Kitchen: 7004
- Notification: 7005
- MongoDB: 27017
- Redis: 6379

---

## Resetting the Database (if needed)

This removes Mongo volume data (orders/users/items) and starts fresh:

```bash
docker compose down -v
docker compose up -d --build

```

---

## Repository Structure (High Level)

```text
prototype2/
├─ docker-compose.yml
├─ .env.example
├─ services/
│  ├─ identity/
│  ├─ gateway/
│  ├─ stock/
│  ├─ kitchen/
│  └─ notification/
└─ frontend/

```

---

## AI Tools Used (Required Disclosure)

- **ChatGPT (OpenAI):** used for architecture brainstorming, API contract review, edge-case analysis (idempotency, retries), and documentation drafting.
- **Gemini (Google):** used for README refinement and dependency management documentation.

---

## Team

- Md. Mubashir-Ul-Hasan
- Muhaiminul Haque
- Mugdho Ranin Rahman Mahee

---

## License

No license (hackathon submission).

---
