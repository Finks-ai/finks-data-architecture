## Infrastructure as Code with Pulumi

### Why Pulumi?

- **Real Programming Languages**: Use TypeScript, Python, Go, or C# instead of DSLs
- **Type Safety**: Catch infrastructure errors at compile time
- **Reusable Components**: Create custom component resources
- **State Management**: Built-in state management with encryption
- **Multi-Cloud Ready**: Same codebase can deploy to AWS, Azure, or GCP

### Pulumi Architecture Pattern

```
pulumi-infrastructure/
├── __main__.py                 # Main entry point
├── components/
│   ├── networking.py          # VPC, subnets, security groups
│   ├── data_lake.py           # S3 buckets, lifecycle policies
│   ├── orchestration.py       # Prefect infrastructure
│   ├── ingestion.py           # ECS task definitions
│   └── transformation.py      # dbt ECS service, Glue jobs
├── config/
│   ├── dev.yaml              # Development environment config
│   ├── staging.yaml          # Staging environment config
│   └── prod.yaml             # Production environment config
└── policies/                  # Pulumi policy packs for compliance
```

### Component Resource Pattern

Pulumi components encapsulate related infrastructure:

```python
class DataLake(pulumi.ComponentResource):
    def __init__(self, name, args, opts=None):
        super().__init__('custom:storage:DataLake', name, {}, opts)

        # Bronze bucket with versioning and lifecycle
        self.bronze_bucket = s3.Bucket(...)

        # Silver bucket with different retention
        self.silver_bucket = s3.Bucket(...)

        # Gold bucket optimized for queries
        self.gold_bucket = s3.Bucket(...)

        # Glue database for all zones
        self.glue_database = glue.Database(...)
```
