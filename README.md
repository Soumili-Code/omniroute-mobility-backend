# OmniRoute: Distributed Corporate Transit & Mobility Platform

[![Java 21](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/projects/jdk/21/)
[![Spring Boot 3.3+](https://img.shields.io/badge/Spring%20Boot-3.3+-green.svg)](https://spring.io/projects/spring-boot)
[![Apache Kafka](https://img.shields.io/badge/Apache%20Kafka-3.7-black.svg)](https://kafka.apache.org/)
[![PostgreSQL PostGIS](https://img.shields.io/badge/PostgreSQL-PostGIS-blue.svg)](https://postgis.net/)
[![PostgreSQL Vector](https://img.shields.io/badge/PostgreSQL-pgvector-blue.svg)](https://github.com/pgvector/pgvector)
[![Redis Stack](https://img.shields.io/badge/Redis-GEO%20%26%20Stack-red.svg)](https://redis.io/)
[![Spring AI](https://img.shields.io/badge/Spring%20AI-1.0-green.svg)](https://spring.io/projects/spring-ai)

**OmniRoute** is an enterprise-grade, high-concurrency corporate transit, cab-pooling, and real-time mobility backend engine modeled after platforms like Routmatic, Rapido, and Uber. Built with **Java 21 Virtual Threads (Project Loom)**, **Spring Boot 3**, **Apache Kafka**, **Redis GEO**, **PostgreSQL (PostGIS & pgvector)**, and **Spring AI**.

The platform solves complex distributed mobility problems: high-frequency telemetry ingestion, Uber H3 hexagonal spatial surge pricing, graph-based shuttle pooling algorithms, transactional outbox Kafka streaming, event-driven Saga billing, and autonomous GenAI safety guardians.
Think of OmniRoute as the complete corporate transit and ride-hailing backend engine (similar to Routmatic, Rapido, or Uber). It handles corporate shift bookings, pairs employees into shared cabs, matches nearby drivers in real-time, tracks vehicles live on a map, and processes fare settlements automatically—backed by a smart AI safety assistant.

---

## 📖 Core Platform Functionality

* **🔐 User Auth & Access:** Secure login issuing stateless JWT passes with Employee, Driver, and Transport Admin permissions[cite: 1].
* **🚌 Smart Corporate Cab-Pooling:** Groups employees living along overlapping routes into shared cabs using spatial graph clustering algorithms[cite: 1].
* **⬢ Uber H3 Dynamic Surge Pricing:** Partitions city maps into hexagonal cells to calculate dynamic demand/supply surge multipliers ($1.2x - 2.0x$) per cell[cite: 1].
* **📍 Sub-10ms Driver Search:** Finds nearby available drivers instantly using Redis GEO memory indexing[cite: 1].
* **🛰️ Live GPS Map Streaming:** Pushes real-time driver coordinates to client app maps over STOMP WebSockets[cite: 1].
* **🛡️ Telemetry Spoof Prevention:** Calculates driver speed between GPS pings to block fraudulent GPS location spoofing[cite: 1].
* **🔄 Reliable Billing (Kafka Saga & Outbox):** Handles multi-step ride payouts via Kafka event sagas, auto-refunding corporate allowances if trips cancel[cite: 1].
* **👁️ Multimodal AI Driver Onboarding:** Uses AI Vision models to automatically read and verify driver license photos[cite: 1].
* **🤖 Autonomous AI Safety Guardian:** AI agent that monitors stationary vehicle pings during late-night trips and auto-triggers safety checks[cite: 1].

## 🛠️ System Architecture & Core Modules

```text
                                [ Employee App / Cab Driver App / Postman ]
                                                     │
                                                     ▼
                                         [ Spring Cloud Gateway ]
                                    (Auth Validation & Resilience4j)
                                                     │
            ┌────────────────────────────────────────┼────────────────────────────────────────┐
            ▼                                        ▼                                        ▼
   [ Auth & Profile Service ]               [ Route & AI Service ]                  [ Matching & Dispatch ]
   (Spring Security / JWT)                  (Spring AI + PostGIS)                   (State Machine & Redis GEO)
            │                                        │                                        │
            │ (Driver/Rider Profiles)                │ (Route Embeddings)                     │ (High-Frequency Writes)
            ▼                                        ▼                                        ▼
   ┌─────────────────┐                      ┌─────────────────┐                      ┌─────────────────┐
   │ PostgreSQL DB   │                      │  pgvector Engine│                      │  Redis GEO      │
   └─────────────────┘                      └─────────────────┘                      └─────────────────┘
                                                     │                                        │
                                                     │ (Tool-Calling Executions)              │ (Trip Events)
                                                     ▼                                        ▼
                                         [ Spring AI ChatClient ]                 [ Apache Kafka Broker ]
                                          (Function-Calling Engine)                           │
                                                                                              ├───────────────────────┐
                                                                                              ▼                       ▼
                                                                                    [ Billing & Settlement ] [ Location Stream ]
                                                                                   (Saga Compensation)       (WebSockets/STOMP)
                                                                                              │                       │
                                                                                              ▼                       ▼
                                                                                    ┌─────────────────┐     ┌─────────────────┐
                                                                                    │ PostgreSQL DB   │     │ Driver Lat/Long │
                                                                                    └─────────────────┘     └─────────────────┘
