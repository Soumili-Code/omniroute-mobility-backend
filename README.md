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

---

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
