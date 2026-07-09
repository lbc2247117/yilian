# Driving Lesson Booking Platform — Backend

An O2O platform for the driving school industry, connecting students, instructors, and operations staff. Handles thousands of daily orders on a Kratos microservices architecture with full observability and service governance.

## Tech Stack

Go · Kratos · gRPC · etcd · Jaeger · Redis · MySQL · Kafka · EFK · Prometheus · Docker · Kubernetes

## Architecture

### Three-Layer Microservices

```
┌──────────────────────────────────────────────┐
│           BFF Layer (3 services)              │
│  Student App · Instructor App · Admin Panel   │
├──────────────────────────────────────────────┤
│        Domain Services (5 services)           │
│  User · Order · Config · Messaging · 3rd-Party│
├──────────────────────────────────────────────┤
│           Task Scheduling Layer               │
│     Cron Jobs · Message Queue Workers         │
└──────────────────────────────────────────────┘
```

- **BFF Layer**: Tailored APIs for each client (student mini-program, instructor mini-program, admin web panel), handling data aggregation and format adaptation.
- **Domain Services**: Encapsulate core business logic. Each service owns its database and exposes capabilities via gRPC.
- **Task Scheduling**: Handles delayed messages, scheduled settlements, data statistics, and other async workloads.

### Communication

- Internal: **gRPC** with Protocol Buffers for synchronous service-to-service calls
- External: Unified **HTTP** endpoints, protocol conversion at the BFF layer
- Async: **Kafka** for event-driven scenarios — order status changes, push notifications

### Service Governance

- **Service Discovery**: etcd cluster with health checks and automatic node removal
- **Load Balancing**: Client-side LB with round-robin and weighted random
- **Rate Limiting**: Token bucket algorithm, per-client + per-endpoint at BFF, per-caller at domain services
- **Circuit Breaking**: Sliding window error rate detection, auto-open on threshold, half-open probing for recovery
- **Retry**: Exponential backoff with max retries, enabled only for idempotent endpoints

## Observability

### Distributed Tracing

Full **Jaeger** integration. gRPC calls automatically propagate Trace Context. Key business checkpoints (order placement, payment, settlement) are manually instrumented, enabling end-to-end trace from BFF to domain services to database/cache.

### Metrics

Every service exposes two standard endpoints:

- `/debug/pprof`: CPU flame graphs, heap profiles, goroutine analysis
- `/metrics`: Prometheus format — QPS, latency percentiles (P50/P90/P99), error rate, gRPC status code distribution, circuit breaker state, rate limit rejections

Prometheus → Grafana dashboards → AlertManager notifications.

### Logging

- JSON-formatted logs written to stdout with traceID, spanID, service name, and log level
- **Fluentd** collects container logs → **Elasticsearch** → **Kibana** for search and visualization
- TraceID allows one-click log correlation across all services in a single request

## Database & Caching

- **MySQL**: Core business data (orders, users, vehicles) with read-write splitting
- **Redis**: Hot data caching (instructor schedules, city/vehicle configs), distributed locking (settlement task mutual exclusion), session storage
- **ETag**: HTTP cache negotiation based on resource versioning to reduce redundant transfers

## Key Features

### Student App
Identity verification · Package purchase · Instructor booking · Online payment · Driving records · IM messaging

### Instructor App
Onboarding review · Order management & scheduling · Revenue settlement · Withdrawal · Notifications

### Admin Panel
User management · Instructor verification · Vehicle dispatch · Order management · Financial reconciliation · Operations dashboard

## My Role

- Designed the 3-layer microservices architecture and service communication protocols
- Independently developed all backend APIs (Go / Kratos / gRPC)
- Built the observability stack: Jaeger tracing, Prometheus metrics, EFK logging platform
- Independently developed the admin panel web frontend
- Set up CI/CD pipelines and Kubernetes deployments
