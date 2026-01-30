# PayFlowX – Serverless Multi-Provider Payment Orchestrator

**PayFlowX** is a hands-on backend project that demonstrates how modern fintech systems like Razorpay, Cashfree, Stripe, and PayU orchestrate payments using **event-driven**, **cloud-native** architecture.

The system models real-world payment workflows with **asynchronous processing**, **idempotent APIs**, **provider abstraction**, and **fault-tolerant retries**.

---

## 🚀 Why PayFlowX?

This project is designed to practice **production-grade backend patterns** commonly used in payment systems:

- Event-driven workflows using queues  
- Idempotency for safe retries and duplicate request handling  
- Asynchronous processing for scalability and resilience  
- Clean provider abstraction for multi-gateway support  
- Durable state management using DynamoDB  

PayFlowX can be evolved into a full-fledged payment orchestration layer for real payment gateways.

---

## ⚙️ Key Features

### Payment Orchestrator API (Node.js + Express)

- `POST /payments`  
  Creates a payment request and returns immediately with `PENDING` status.

- `GET /payments/:id`  
  Fetches the current status of a payment (`PENDING`, `SUCCESS`, `FAILED`).

---

### Idempotent Payment Creation

- Uses a dedicated **Idempotency** table in DynamoDB.
- Ensures the same `idempotencyKey` always maps to the same `transactionId`.
- Prevents duplicate charges when clients retry requests.

---

### Asynchronous Processing (SQS + Worker)

- API enqueues payment jobs into an SQS queue.
- A background **Provider Worker** consumes jobs and processes payments asynchronously.
- Improves throughput, reliability, and fault isolation.

---

### Pluggable Payment Providers

- `MockProvider` – base mock provider  
- `RazorpayMockProvider` – simulates Razorpay-style behavior  
- `CashfreeMockProvider` – simulates Cashfree-style behavior  

Providers follow a common interface, making it easy to integrate real gateways (Razorpay, Cashfree, Stripe) without modifying core orchestration logic.

---

### Provider Routing Logic

Simple routing rules determine which provider handles a transaction:

- INR payments → `RazorpayMock`
- High-value transactions → `CashfreeMock`
- Default fallback → `MockProvider`

Routing logic is implemented in `src/providers/index.js`.

---

### Retries & Failure Handling

- Worker tracks processing attempts in DynamoDB.
- Failed transactions are retried up to `MAX_RETRIES`.
- Transactions are marked `FAILED` after exceeding retry limits.
- Supports safe recovery from transient provider failures.

---

### End-to-End Audit Trail

The DynamoDB **Transactions** table stores complete transaction state:

- Status transitions: `PENDING → SUCCESS / FAILED`
- Provider metadata (`providerRef`)
- Retry count and last error details
- Creation and update timestamps

This enables traceability, debugging, and reconciliation.

---

## 🧱 Architecture Overview

### High-Level Flow

1. Client sends `POST /payments`
2. API:
   - Validates request payload
   - Performs idempotency check
   - Creates or updates transaction in DynamoDB
   - Publishes a payment job to SQS
3. Provider Worker:
   - Polls SQS for jobs
   - Loads transaction state from DynamoDB
   - Selects the appropriate provider
   - Calls `provider.charge(transaction)`
   - Updates transaction status (`SUCCESS` / `FAILED`) with retry handling
4. Client (or downstream service) queries `GET /payments/:id` to fetch the latest payment status

---

## 🧠 What This Project Demonstrates

- Real-world payment orchestration patterns  
- Event-driven system design using AWS primitives  
- Safe handling of retries and failures  
- Clean separation between API, worker, and providers  
- Fintech-grade backend engineering practices  

---

## 📌 Future Enhancements

- Integrate real payment gateways (Razorpay / Cashfree / Stripe)
- Add webhook handling for provider callbacks
- Introduce DLQ monitoring and replay tooling
- Add observability (metrics, tracing, alerts)
- Infrastructure as Code (Terraform / CDK)

---

## 📄 License

MIT License
