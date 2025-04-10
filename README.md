# 🔐 API Gateway & Logger Service

A secure, centralized API Gateway built to manage authentication, payload encryption, request validation, and request-response logging across multiple microservices.

This service acts as the first line of defense and insight into all incoming/outgoing traffic — enhancing security, observability, and developer experience.

---

## 📌 Problem Statement

With multiple internal and external APIs, the system lacked:
- A consistent way to apply security (auth + encryption)
- Unified logging for debugging / analytics
- Request validation and error reporting
- Traceable metadata for partner APIs

We needed a modular, middleware-driven gateway to handle:
- Encryption / decryption of all payloads
- JWT validation
- Logging at request + response level
- Schema validation per endpoint

---

## ⚙️ Tech Stack

- **Backend:** Django  
- **Middlewares:** Custom Django middlewares for security & logging  
- **Databases:**  
  - **MySQL** – for structured request logs, timestamps, status codes  
  - **MongoDB** – for storing raw payloads and encrypted content (flexible logging schema)  
- **Encryption:** AES-based symmetric encryption module  
- **Auth:** JWT (internal service-level tokens)  
- **Infra:** Docker, Nginx  
- **Queue (Optional):** Celery (for async log processing, not mandatory)

---

## 🚀 Core Features

### 🔐 Payload Encryption / Decryption
- Incoming encrypted requests are **decrypted in middleware** before reaching views  
- Outgoing responses are **encrypted before returning to the client**  
- AES symmetric key encryption (key exchange handled via IAM or env configs)

### 🛡️ Request Validation & JWT Auth
- Middleware checks for valid JWT in headers  
- Token contains service-level access info  
- Schema validation per endpoint using a rule map

### 📑 Centralized Logging Engine
- **MongoDB** stores flexible payload dumps + request/response metadata  
- **MySQL** logs structured metadata:  
  - Service name  
  - API path  
  - Timestamp  
  - Status code  
  - Time taken (ms)  
- Optional: logged errors + tracebacks for debugging

### 📡 Inter-service Request Tracing
- Each request tagged with a unique `X-Request-ID`  
- Logs correlate across services  
- Middleware injects trace ID into logs, headers, and responses

### 🛠️ Admin Debug Console (Optional)
- Internal dashboard to view logs by:
  - Service
  - Status code
  - Time range
  - Trace ID

---

## 📈 Impact

| Metric | Result |
|--------|--------|
| 🔐 Payloads Secured | 100% API traffic (internal/external) |
| 🧾 Logs Captured | 1M+ requests/month |
| 🧠 Decryption/Encryption Overhead | <50ms avg per request |
| 🔁 Traceability | 100% of requests mapped with unique IDs |
| 🧩 Services Integrated | 6+ microservices behind this gateway |

---

## 📂 Architecture Overview
        ┌──────────────────────────────┐
        │  Client / Partner API Caller │
        └────────────┬─────────────────┘
                     ↓
      ┌─────────────────────────────────────┐
      │            API Gateway              │
      │      (Django + DRF + Middleware)    │
      └─────────────────────────────────────┘
                     ↓
      ┌──────────────────────────────────────────┐
      │ Middleware: JWT Token Authentication     │
      │ - Validates service/user identity        │
      │ - Checks token expiry and scope          │
      └──────────────────────────────────────────┘
                     ↓
     ┌──────────────────────────────────────────┐
     │ Middleware: Payload Decryption           │
     │ - Decrypts request using symmetric key   │
     │ - Converts raw bytes into JSON           │
     └──────────────────────────────────────────┘
                     ↓
     ┌──────────────────────────────────────────┐
     │ DRF Serializer Validation Layer          │
     │ - Validates decrypted payload fields     │
     │ - Applies field types, required, choices │
     │ - Returns structured errors if invalid   │
     └──────────────────────────────────────────┘
                     ↓
     ┌─────────────────────────────────────┐
     │ Django View Logic (Business Layer)  │
     │ - Executes core functionality       │
     └─────────────────────────────────────┘
                     ↓
     ┌──────────────────────────────────────────┐
     │ Middleware: Response Encryption          │
     │ - Encrypts response payload              │
     │ - Adds headers (X-Request-ID, status)    │
     └──────────────────────────────────────────┘
                     ↓
     ┌──────────────────────────────────────────┐
     │ Middleware: Request/Response Logger      │
     │ - MySQL: logs service, time, status, API │
     │ - MongoDB: logs raw payloads & metadata  │
     └──────────────────────────────────────────┘
                     ↓
        🔒 Encrypted Response Returned
---

## 🧠 Takeaways

- Learned to build middleware-first architecture that doesn’t bloat view logic  
- Understood how to separate **structured vs. unstructured logs** using MySQL + Mongo  
- Built performant, traceable request handling under 50ms overhead  
- Delivered a system devs could debug, trace, and audit from a single point  
- Solidified understanding of encryption workflows in real APIs

---

## 📌 Note

This case study is based on a live system running in a production fintech environment. Codebase is private, but structure and architecture reflect real-world impact.
