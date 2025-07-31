## Service Architecture

### Prefect Server Deployment

- **Infrastructure**: ECS Fargate with auto-scaling
- **Database**: RDS PostgreSQL Multi-AZ
- **Load Balancer**: ALB with SSL termination
- **Storage**: EFS for shared artifacts

### Prefect Agent Architecture

```
Agent Pools:
├── ingestion-pool      # High memory for data extraction
├── transformation-pool # High CPU for dbt runs
├── lightweight-pool    # Burstable for notifications
└── gpu-pool           # ML workloads (future)
```

### ECS Task Patterns

#### Ingestion Tasks (Ephemeral)

- **Lifecycle**: Created by Prefect, terminated after completion
- **Resources**: Configured per data source requirements
- **Networking**: Private subnet with NAT Gateway
- **IAM Role**: Scoped to specific S3 paths and secrets

#### dbt Core on ECS (Ephemeral Tasks)

- **Deployment**: Ephemeral ECS tasks triggered by Prefect (not long-running service)
- **Architecture**:
  ```
  ECS Task Definition
  ├── dbt Container
  │   ├── dbt project pulled from Git at runtime
  │   ├── Profiles from Secrets Manager
  │   └── Connection to Athena/Redshift
  └── Sidecar Container (Optional)
      └── Metrics collector
  ```
- **Execution Model**: Each dbt run is a fresh ECS task
- **Scaling**: Unlimited parallel tasks based on workload
- **Storage**: S3 for dbt artifacts and logs
