# Global Data Plane Cluster – Reference Architecture

**Version:** 0.2 (Draft)  
**Date:** May 21, 2026  
**Authors:** Rick McGeer & Aiko  

## 1. Purpose

The **Global Data Plane Cluster** is a simple, reusable pattern for running the Global Data Plane as a distributed system.

It provides:
- A single **front-end** SDTP server (the "web server")
- A network of independent **ContainerTables** (the "back-end services / modules")
- A clean, idiomatic way to add new computational tables that feels as natural as importing Python modules

**Core Goal:** Make deploying and extending a GDP cluster as easy and natural as running a modern web application, while learning from the successes and mistakes of PlanetLab, GENI, and early XOS.

## 2. Historical Context & Lessons Learned

PlanetLab proved the power of distributed systems research at planetary scale, but had operational weaknesses (shipping hardware, reliance on local IT). GENI and early XOS attempted broader visions but became overly complex, especially when tightly coupled to heavy stacks like OpenStack + ONOS.

**Key Lesson:** It is easier to generalize from a strong, specific, working system than to boil the ocean. Therefore we are intentionally starting narrow and focused.

## 3. Core Analogy

| Traditional Web Application       | Global Data Plane Cluster                  |
|-----------------------------------|--------------------------------------------|
| Web server / API layer            | SDTP Server (FastAPI)                      |
| Databases + microservices         | ContainerTables                            |
| Backend workers / scripts         | ContainerTables                            |
| `import revenue_calculator`       | `monthly-revenue` service                  |

## 4. High-Level Architecture

```mermaid
graph TD
    A[Users / Clients] --> B[SDTP Server<br/>(FastAPI)]
    B <--> C[Docker Internal Network]
    C <--> D[ContainerTable: monthly-revenue]
    C <--> E[ContainerTable: customer-clv]
    C <--> F[ContainerTable: ...]
```

## 5. Design Principles

- Simplicity First: Start with docker-compose as the canonical reference pattern.
- Service Discovery by Name: Containers are addressed by logical service name (no hardcoded IPs or complex configuration).
- ContainerTable = Module: Adding new functionality should feel as natural as importing a Python module.
- Loose Coupling: Each ContainerTable owns its own logic, dependencies, versioning, data movement, replication strategy, and state management.
- SDQL as Intermediate Form: User-facing query languages (natural language, spreadsheet-style, SQL-like, etc.) will compile down to SDQL.
- Platform Agnosticism: The cluster does not prescribe data movement, replication, caching, or state management — these decisions belong to the authors of individual ContainerTables.
- Progressive Enhancement: Add complexity (scaling, observability, advanced orchestration) only when real usage requires it.

## 6. Components
| Component |  Technology |  Responsibility |  Status | 
|-----------|-------------|---------------------------|
| SDTP Server | FastAPI | "Authentication, routing, discovery, service resolution" | Flask → FastAPI (in progress) | 
| ContainerTable | Docker | "Data storage, computation, and SDTP protocol implementation" | Ready | 
| Service Resolver | Python | Maps service_name → reachable URL | Completed | 
| Networking | Docker | Automatic DNS-based service discovery | Built-in | 
|-----------|-------------|---------------|------------|

## 7. Recommended Development Pattern
**docker-compose** is the primary reference implementation for local development and small-to-medium deployments.
**Recommended Project Layout:**
```
global-data-plane-cluster/
├── docker-compose.yml
├── tables/                    # One subdirectory per ContainerTable
│   ├── monthly-revenue/
│   ├── customer-clv/
│   └── ...
├── config/
├── scripts/
├── docs/
└── README.md
```
This structure makes it very easy for new users to understand the system and add their own ContainerTables.