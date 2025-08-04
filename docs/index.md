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
- [Data Pipeline Diagrams](data-pipeline-diagrams.mmd)
- [Detailed Flow](detailed-flow/index.md)
- [Infrastructure as Code](infrastructure-as-code/index.md)
- [Monitoring and Observability](monitoring-observability/index.md)
- [Security Architecture](security-architecture/index.md)
- [Service Architecture](service-architecture/index.md)
- [Technology Decisions](technology-decisions/index.md)
- [Technology Stack](technology-stack/index.md)
- [Transformation Layer](transformation-layer/index.md)
