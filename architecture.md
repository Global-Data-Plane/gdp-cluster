# Global Data Plane Cluster – Reference Architecture

**Version:** 0.1 (Draft)  
**Date:** May 13, 2026  
 
## 1. Purpose

The **Global Data Plane Cluster** is a simple, reusable pattern for running the Global Data Plane as a distributed system.

It provides:
- A single **front-end** SDTP server (the "web server")
- A network of independent **ContainerTables** (the "back-end services / modules")
- A clean, idiomatic way to add new computational tables that feels as natural as importing Python modules

**Goal:** Make deploying and extending a GDP cluster as easy as running a modern web application.

## 2. Core Analogy

| Traditional Web Application       | Global Data Plane Cluster              |
|-----------------------------------|----------------------------------------|
| Web server / API layer            | SDTP Server (FastAPI)                  |
| Databases + microservices         | ContainerTables                        |
| Backend workers / scripts         | ContainerTables                        |
| `import revenue_calculator`       | `monthly-revenue` service              |

## 3. High-Level Architecture

```mermaid
graph TD
    A[Users / Clients] --> B[SDTP Server<br/>(FastAPI)]
    B <--> C[Docker Internal Network]
    C <--> D[ContainerTable: monthly-revenue]
    C <--> E[ContainerTable: customer-clv]
    C <--> F[ContainerTable: ...]
```
## 4. Design Principles

- Simplicity First: Start with docker-compose as the reference pattern
- Service Discovery by Name: No hardcoded IPs or complex config
- ContainerTable = Module: Adding a new table should feel like adding a Python module
- Loose Coupling: Each ContainerTable owns its logic, dependencies, and versioning
- Progressive Enhancement: Add complexity (scaling, observability, etc.) only when needed

## 5. Components

| Component | Technology | Role | Status |
|-----------|------------|------|--------|
| SDTP Server | FastAPI | Front-end, routing,  auth,  service resolution | Flask → FastAPI |
| ContainerTable | Docker | Implements SDTP protocol + business logic | Ready |
| Service Resolver | Python | service_name → http://service-name:port | Completed |
| Networking | Docker | Automatic DNS discovery | Built-in |

## 6. Recommended Development Pattern (docker-compose)
This is the reference implementation — the one we want others to copy.
See: docker-compose.yml (to be created)
Typical structure:
```
textgdp-cluster/
├── docker-compose.yml
├── tables/
│   ├── monthly-revenue/
│   ├── customer-clv/
│   └── ...
├── config/
└── README.md
```
## 7. Future Evolution Path

- Phase 1 (Now): docker-compose reference pattern
- Phase 2: Helm chart for easy Kubernetes deployment
- Phase 3: Lightweight Kubernetes Operator (when we have many tables + complex lifecycle needs)
- Phase 4: Full autoscaling, canary deployments, observability, etc.

## 8. Next Actions
 - Rewrite SDTP Server in FastAPI
 - Create clean docker-compose.yml reference template
 - Define standard project layout and conventions
 - Add healthcheck / readiness standards for ContainerTables
 - Create simple CLI helper (gdp up, gdp add-table, etc.)
