# Application Requirements Specification: OmniRoute Mobility Engine

## 1. Architectural Scope & Domain Boundary
**OmniRoute** (`omniroute-mobility-backend`) is an enterprise-grade corporate transit, ride-pooling, and real-time mobility backend platform (similar to Routmatic / Rapido / Uber). Built as a modular microservices architecture utilizing **Java 21 Virtual Threads (Project Loom)** and **Spring Boot 3**, the system processes high-frequency spatial GPS telemetry, automates shift-based employee cab pooling, orchestrates distributed ride-billing transactions via Kafka Sagas, and leverages Spring AI for RAG-based route planning, document vision verification, and autonomous safety monitoring.

---

## 2. Functional Requirements (FRs)

### Module A: Identity & Access Management (`/auth`)
* **FR-1.1 (User Registration & Authentication):** Support registration for corporate employees, transit drivers, and transport administrators with encrypted password storage (BCrypt).
* **FR-1.2 (Stateless JWT Emission):** Issue cryptographically signed, stateless JSON Web Tokens (JWT) containing user claims, expiration timestamps, and assigned roles (`ROLE_EMPLOYEE`, `ROLE_DRIVER`, `ROLE_TRANSPORT_ADMIN`).
* **FR-1.3 (Role-Based Access Control):** Enforce strict endpoint-level security via Spring Security 6 to authorize distinct operational capabilities across roles.

### Module B: Shift Pooling & Spatial Dispatch (`/dispatch`)
* **FR-2.1 (Shift Booking & Capacity Limits):** Allow corporate employees to book shift rides with guaranteed seating allocations enforced by optimistic concurrency control (`@Version`).
* **FR-2.2 (Graph-Based Shuttle Pooling):** Cluster overlapping employee pickup locations along transit corridors into shared cab routes using a spatial graph-clustering algorithm (JGraphT / PostGIS network topologies).
* **FR-2.3 (Uber H3 Spatial Hexagonal Surge Pricing):** Index real-time rider demand and driver supply into Uber H3 spatial hexagonal cells stored in Redis Stack to calculate dynamic surge pricing multipliers ($1.2\times–2.0\times$) per cell.

### Module C: Trip State Machine & Distributed Saga Settlement (`/trip`, `/billing`)
* **FR-3.1 (Deterministic Trip Lifecycle):** Enforce strict status lifecycle transitions (`SCHEDULED` → `DRIVER_ASSIGNED` → `EN_ROUTE` → `COMPLETED` / `CANCELLED`) via Spring StateMachine.
* **FR-3.2 (Kafka Saga Orchestration):** Coordinate distributed billing transactions across Trip, Billing, and Settlement services over Apache Kafka. Execute automatic compensating transactions (e.g., releasing driver allocation and refunding transit passes) if downstream billing fails.
* **FR-3.3 (Transactional Outbox Pattern):** Commit relational database updates and outgoing Kafka event payloads atomically within a `database_outbox` table to guarantee zero-loss event streaming without dual-write inconsistencies.
* **FR-3.4 (Settlement Idempotency):** Validate unique transaction keys in Redis (`SETNX`) prior to settlement execution to prevent duplicate billing during network retries.

### Module D: High-Frequency GPS Tracking & STOMP Streaming (`/tracking`)
* **FR-4.1 (Redis GEO Telemetry Ingestion):** Ingest 3-second driver GPS pings into Redis Stack via `GEOADD` and query nearest available drivers within single-digit milliseconds using `GEOSEARCH`.
* **FR-4.2 (Live Coordinate STOMP WebSockets):** Establish full-duplex STOMP over WebSocket connections to stream driver coordinates directly to client map interfaces.
* **FR-4.3 (Geospatial Telemetry Anti-Spoofing):** Compute velocity ($\Delta\text{distance} / \Delta\text{time}$) between pings and apply Resilience4j circuit breakers to flag driver accounts when calculated speeds exceed $180\text{ km/h}$.

### Module E: Spring AI & Autonomous Safety Guardian (`/ai`)
* **FR-5.1 (Multimodal Vision Driver Onboarding):** Process driver license uploads via Spring AI Multimodal Vision (GPT-4o / Ollama LLaVA) to extract identity numbers, expiry dates, and photo metadata into typed Java Records via `BeanOutputConverter`.
* **FR-5.2 (RAG Semantic Route Search):** Store spatial route embeddings in PostgreSQL (`pgvector`) using Spring AI `VectorStore` to serve natural-language route recommendations.
* **FR-5.3 (Voice-to-Trip Natural Language Dispatcher):** Map backend Java service beans (`bookShiftTransit`, `findNextShuttle`) as Spring AI executable tools, enabling natural language text or voice transcriptions to book transit.
* **FR-5.4 (Autonomous Safety Guardian Agent):** Trigger an asynchronous Kafka consumer when vehicles remain stationary outside traffic nodes for $>5\text{ mins}$. The AI agent uses Tool Calling to query traffic APIs and initiate passenger safety checks.
* **FR-5.5 (Async Driver Review Sentiment Pipeline):** Process post-ride feedback off Kafka, parse sentiment scores via Spring AI structured output converters, and save safety flags to PostgreSQL.

---

## 3. Non-Functional Requirements (NFRs)

* **NFR-1 (High Concurrency):** Enable Java 21 **Virtual Threads** (`spring.threads.virtual.enabled=true`) to serve thousands of concurrent STOMP WebSocket streams and REST pings without OS thread starvation.
* **NFR-2 (Sub-10ms Read/Write Latency):** Maintain sub-10ms latency for driver GPS location pings and radial spatial queries using Redis Stack.
* **NFR-3 (System Resilience):** Apply Resilience4j Token-Bucket Rate Limiters and Circuit Breakers to dynamic pricing endpoints and third-party mapping APIs.
* **NFR-4 (Distributed Tracing & Observability):** Instrument end-to-end flows with Micrometer Tracing and OpenTelemetry headers (`traceId`/`spanId`) across HTTP, WebSocket, and Kafka event boundaries.
* **NFR-5 (Zero-Mock Integration Testing):** Validate integration flows using **Testcontainers** to launch containerized PostgreSQL (PostGIS + pgvector) and Apache Kafka instances during automated build execution.
