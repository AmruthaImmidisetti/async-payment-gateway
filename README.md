# Async Payment Gateway System (Deliverable 2)

A **production-ready payment gateway** designed with asynchronous processing, background job queues, secure webhook delivery, embeddable checkout SDK, and refund handling.  
Built to demonstrate **scalable fintech system design** with reliability and fault tolerance.

---

##  Features Overview

###  Asynchronous Payment Processing
- Background execution using **Bull + Redis**
- Separate queues for:
  - Payment execution
  - Webhook delivery
  - Refund processing
- Independent worker services for each queue
- Payment state lifecycle:
```

pending → success / failed

```

---

###  Webhook Delivery System
- **HMAC-SHA256 signature** sent via `X-Webhook-Signature`
- Automatic retry using **exponential backoff**
- Production: 1min → 5min → 30min → 2hr
- Test mode: 5s → 10s → 15s → 20s
- Maximum of **5 retry attempts**
- Supported events:
- `payment.success`
- `payment.failed`
- `refund.processed`
- Webhook URL and secret configurable via dashboard

---

###  Embeddable Checkout SDK
- Modal / iframe-based checkout
- No external redirection
- Secure cross-window communication using `postMessage`
- Mobile and desktop responsive
- Lifecycle callbacks:
- `onSuccess`
- `onFailure`
- `onClose`

---

###  Refund Handling
- Supports **full and partial refunds**
- Refunds processed asynchronously
- Webhook notification on completion
- Refund amount validation enforced
- Refund states:
```

pending → processed

```

---

###  Idempotency Support
- Requests protected using `Idempotency-Key`
- Duplicate requests return cached responses
- Keys expire after **24 hours**
- Scoped per merchant
- Prevents duplicate charges

---

##  Architecture

```

Payment Gateway System
│
├── Backend (Express.js | Port 8000)
│   ├── REST APIs
│   ├── Bull Queues
│   │   ├── paymentWorker.js
│   │   ├── webhookWorker.js
│   │   └── refundWorker.js
│   └── PostgreSQL
│       ├── merchants
│       ├── orders
│       ├── payments
│       ├── refunds
│       ├── webhook_logs
│       └── idempotency_keys
│
├── Redis (Port 6379)
│   └── Job queue storage
│
├── Frontend
│   ├── Dashboard (React + Vite | Port 3000)
│   │   ├── /dashboard
│   │   ├── /dashboard/transactions
│   │   ├── /dashboard/webhooks
│   │   └── /dashboard/docs
│   │
│   ├── Checkout App (React + Vite | Port 3001)
│   │   ├── /checkout
│   │   └── /checkout?embedded=true&order_id=X
│   │
│   └── Checkout SDK (checkout.js)
│
└── Test Utilities
└── Webhook Receiver (Port 3002)

````

---

##  Getting Started

### Prerequisites
- Docker Desktop
- Git

---

### Setup & Run

1. **Clone repository**
```bash
git clone https://github.com/AmruthaImmidisetti/async-payment-gateway.git
cd async-payment-gateway
````

2. **Build and start services**

```bash
docker-compose up -d --build
```

3. **Check running containers**

```bash
docker-compose ps
```

---

### Service URLs

* API: [http://localhost:8000](http://localhost:8000)
* Dashboard: [http://localhost:3000](http://localhost:3000)
* Checkout: [http://localhost:3001](http://localhost:3001)

---

### Health Check

```bash
curl http://localhost:8000/health
```

---

##  Test Merchant Details

Auto-seeded during startup:

| Field      | Value                                       |
| ---------- | ------------------------------------------- |
| Email      | [test@example.com](mailto:test@example.com) |
| API Key    | key_test_abc123                             |
| API Secret | secret_test_xyz789                          |

---

## 📡 API Usage

### Base URL

```
http://localhost:8000
```

### Authentication Headers

```http
X-Api-Key: key_test_abc123
X-Api-Secret: secret_test_xyz789
```

---

### Create Order

**POST** `/api/v1/orders`

```json
{
  "amount": 50000,
  "currency": "INR",
  "receipt": "receipt_123",
  "notes": {
    "customer_name": "John Doe"
  }
}
```

---

### Create Payment

**POST** `/api/v1/payments`

UPI:

```json
{
  "order_id": "order_id",
  "method": "upi",
  "vpa": "user@paytm"
}
```

Card:

```json
{
  "order_id": "order_id",
  "method": "card",
  "card": {
    "number": "4111111111111111",
    "expiry_month": "12",
    "expiry_year": "2025",
    "cvv": "123",
    "holder_name": "John Doe"
  }
}
```

Payments are returned immediately and finalized asynchronously.

---

##  Frontend Applications

### Dashboard (Port 3000)

* Merchant login
* API credential visibility
* Payment statistics
* Transactions list
* Webhook configuration
* API documentation

### Checkout (Port 3001)

* Order summary
* Payment method selection
* Processing indicator
* Success / failure screens

---

##  Database Design

Tables used:

* merchants
* orders
* payments
* refunds
* webhook_logs
* idempotency_keys

(Refer to schema files for details.)

---

##  Test Mode

```bash
TEST_MODE=true
TEST_PAYMENT_SUCCESS=true
TEST_PROCESSING_DELAY=1000
```

Provides deterministic behavior for evaluation.

---

##  Async Processing Flow

```
Client submits payment
→ API stores payment (pending)
→ Job queued
→ Worker processes payment
→ Status updated
→ Webhook triggered
→ Merchant notified
```

---

##  Project Structure

```
async-payment-gateway/
├── backend/
├── frontend/dashboard/
├── checkout/
├── docker-compose.yml
├── README.md
└── .env.example
```

---

##  Deployment Notes

* Secure secret management
* TLS for all services
* Independent worker scaling
* Observability for webhook delivery
* Retry and backoff tuning for production

---

##  Deliverable 2 Coverage

* Async job queues
* Redis-backed workers
* Secure webhook delivery with retries
* Checkout SDK
* Refund handling
* Idempotency enforcement
* Docker-based deployment
* Merchant dashboard
* Test utilities

---

##  Summary

This project demonstrates a **real-world payment gateway architecture**, emphasizing asynchronous workflows, system reliability, and production readiness for fintech platforms.
