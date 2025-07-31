## Architecture Overview

This architecture implements a modern ELT (Extract, Load, Transform) data pipeline for financial data using Prefect for orchestration, AWS native services for compute and storage, Pulumi for infrastructure as code, and dbt Core for transformations.

### Key Principles

- **Foundation First**: Infrastructure as Code, CI/CD, and security before application development
- **ELT over ETL**: Load raw data first, transform in-warehouse for flexibility
- **Centralized Orchestration**: Prefect Server manages all workflows
- **Clear Service Boundaries**: ECS for containerized services, Lambda for event-driven tasks
- **Ephemeral Compute**: On-demand task execution rather than always-on services
- **Developer Experience**: Local development environment mirrors production
- **Idempotent by Design**: Workflows are designed to be safely re-run. A failed flow that is restarted should produce the same result as a flow that succeeds on the first try. This is achieved through partitioned data writes and atomic operations

### High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   Infrastructure Layer                       │
│              (Pulumi - TypeScript/Python)                    │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────┴───────────────────────────────┐
│                   Orchestration Layer                        │
│                 (Prefect Server on ECS)                      │
└─────────────────────────────┬───────────────────────────────┘
                              │
    ┌───────────────────────┼───────────────────────┐
    │                       │                       │
    v                       v                       v
┌─────────────┐      ┌─────────────┐       ┌─────────────┐
│  Ingestion  │      │ Processing  │       │   Serving   │
│   Layer     │      │   Layer     │       │    Layer    │
└─────────────┘      └─────────────┘       └─────────────┘
```
