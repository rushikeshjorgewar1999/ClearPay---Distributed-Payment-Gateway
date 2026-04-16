# ClearPay — Distributed Payment Gateway

A **PCI-compliant**, high-throughput distributed payment gateway supporting **Card** and **Net Banking** transactions, built for financial consistency and security at scale.

---

## Architecture Overview

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Browser JS │────▶│  API Gateway │────▶│  Payment Service│
│     SDK     │     └──────────────┘     └────────┬────────┘
└─────────────┘                                   │
                                         ┌────────▼────────┐
                                         │   Kafka Topics  │
                                         └────────┬────────┘
                      ┌──────────────────┬────────┴──────────────────┐
                      ▼                  ▼                            ▼
             ┌────────────────┐  ┌───────────────┐       ┌──────────────────┐
             │  Card Vault    │  │  SAGA/Outbox  │       │ Webhook Delivery │
             │  (AES-256)     │  │  Orchestrator │       │    Engine        │
             └────────────────┘  └───────────────┘       └──────────────────┘
                                         │
                              ┌──────────▼──────────┐
                              │  Settlement Engine  │
                              │  (Nightly Batch)    │
                              └─────────────────────┘
```

---

## Key Features

### High-Throughput Processing
- Sustains **10,000 transactions/sec** using non-blocking I/O via **Spring WebFlux**
- Kafka-backed event streaming for decoupled, fault-tolerant microservice communication
- Supports **Card** and **Net Banking** payment methods

### PCI-Compliant Card Vault
- Raw PANs (Primary Account Numbers) exist **in-memory for <50ms** — never written to logs, databases, or Kafka events
- **AES-256 encryption** for all stored card data
- Browser-side **JS SDK** tokenizes card data before it ever hits the server, minimizing PCI scope

### Financial Consistency via SAGA + Outbox
- **SAGA pattern** with choreography across **6 microservices** ensures no partial transaction states
- **Transactional Outbox** guarantees at-least-once event publishing — no lost events even on crash
- **Resilience4j circuit breakers** prevent cascading failures under load
- Eliminates **double-charges** under partial failures or network partitions

### Webhook Delivery Engine
- **HMAC-SHA256** signed payloads for tamper-proof webhook delivery
- **7-attempt exponential backoff** for reliable merchant notification
- Dead-letter handling for undeliverable webhooks

### Nightly Settlement Engine
- Batch job computes **per-merchant net payouts** nightly
- Handles refunds, disputes, and fee deductions before settlement
- Produces auditable settlement reports per merchant

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot, Spring WebFlux |
| Messaging | Apache Kafka |
| Security | AES-256, HMAC-SHA256 |
| Resilience | Resilience4j (Circuit Breaker, Retry) |
| Pattern | SAGA, Transactional Outbox |
| Frontend SDK | Vanilla JS (Browser-side tokenization) |
| Build | Gradle |

---

## Microservices

| Service | Responsibility |
|---|---|
| **API Gateway** | Rate limiting, routing, auth |
| **Payment Service** | Transaction orchestration |
| **Card Vault Service** | PAN tokenization, AES-256 storage |
| **SAGA Orchestrator** | Distributed transaction coordination |
| **Webhook Service** | Signed delivery with retry logic |
| **Settlement Service** | Nightly batch payout computation |

---

## Security Highlights

```
PAN Lifecycle:
Browser → JS SDK (tokenize) → API (token only) → Vault (<50ms decrypt) → Charge → Discard
                                                        ↑
                                               Never hits DB / Logs / Kafka
```

- PCI DSS scope minimized via browser-side tokenization
- All inter-service events carry tokens, never raw card data
- HMAC-signed webhooks prevent payload tampering by third parties

---

## Performance

| Metric | Value |
|---|---|
| Throughput | 10,000 txn/sec |
| PAN in-memory window | < 50ms |
| Webhook retry attempts | 7 (exponential backoff) |
| Microservices | 6 |

---

## Getting Started

### Prerequisites
- Java 17+
- Apache Kafka
- Docker (optional, for local infra)

### Run Locally

```bash
# Clone the repository
git clone https://github.com/your-username/payshield.git
cd payshield

# Start Kafka (via Docker)
docker-compose up -d

# Build and run
./gradlew bootRun
```

### Environment Variables

```env
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
VAULT_AES_SECRET=<32-byte-secret>
WEBHOOK_HMAC_SECRET=<your-hmac-secret>
DB_URL=jdbc:postgresql://localhost:5432/payshield
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.
