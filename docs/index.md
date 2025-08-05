# Welcome to the Finks Data Pipeline Documentation

This site contains the architecture and design documentation for Finks Data.

## Sections to Clarify

During a review of the architecture diagrams, the following points were identified as needing clarification:

1.  **Metadata Capture in Data Ingestion Flow**: The "Data Ingestion Flow" diagram in the [Detailed Flow](detailed-flow/index.md) section shows the ECS Task writing to the Bronze S3 bucket, and separately shows the Prefect Trigger writing to a Metadata DynamoDB table. It's unclear what specific metadata is being captured and by which process. Does the ECS task write operational metadata (e.g., file size, record count) while the Prefect trigger writes orchestration metadata (e.g., flow run ID, parameters)? Clarifying this would improve the understanding of data lineage.

2.  **Quality Gate in Data Processing Flow**: The "Data Processing Flow" diagram in the [Detailed Flow](detailed-flow/index.md) section includes a "Quality Gate" after the Glue Job. It is unclear what this gate represents. Is it a separate, dedicated service that validates the data, or is it a step within the Glue job itself? Defining this process more explicitly would clarify the data promotion criteria from Bronze to Silver.

## Table of Contents

- [Architecture Overview](architecture-overview/index.md)
- [Core Components](core-components/index.md)
- [Cost Optimization](cost-optimization/index.md)
- [Data Lake Architecture](data-lake-architecture/index.md)
- [Detailed Flow](detailed-flow/index.md)
- [Infrastructure as Code](infrastructure-as-code/index.md)
- [Monitoring and Observability](monitoring-observability/index.md)
- [Repositories](repositories/index.md)
- [Security Architecture](security-architecture/index.md)
- [Service Architecture](service-architecture/index.md)
- [Technology Decisions](technology-decisions/index.md)
- [Technology Stack](technology-stack/index.md)
- [Transformation Layer](transformation-layer/index.md)

## Roadmaps

- [Project Roadmap (3-Month)](roadmaps/project-roadmap.md)
- [Aggressive Timeline (6-Week)](roadmaps/aggressive-timeline.md)
- [Agentic Sessions](roadmaps/agentic-sessions/index.md)
  - [Week 0: Preparation](roadmaps/agentic-sessions/week-0-preparation.md)
  - [Week 1: Foundation](roadmaps/agentic-sessions/week-1-foundation.md)
  - [Week 2: Infrastructure](roadmaps/agentic-sessions/week-2-infrastructure.md)
  - [Week 3: Orchestration](roadmaps/agentic-sessions/week-3-orchestration.md)
  - [Week 4: Transformation](roadmaps/agentic-sessions/week-4-transformation.md)
  - [Week 5: Pipeline](roadmaps/agentic-sessions/week-5-pipeline.md)
  - [Week 6: Production](roadmaps/agentic-sessions/week-6-production.md)
  - [Best Practices](roadmaps/agentic-sessions/best-practices.md)
