## Technology Decisions

### Why Pulumi over Terraform?

- **Type Safety**: Catch errors before deployment
- **Programming Constructs**: Loops, conditionals, functions
- **IDE Support**: Full IntelliSense and refactoring
- **Testing**: Unit test infrastructure code
- **Abstraction**: Create high-level components

### Why dbt-core on ECS over alternatives?

- **Control**: Full control over dbt version and plugins
- **Cost**: No per-seat licensing like dbt Cloud
- **Integration**: Direct integration with Prefect
- **Customization**: Add custom packages and macros
- **Security**: Runs in your VPC with your IAM roles

### Why Medallion Architecture?

- **Clear Quality Tiers**: Each zone has defined quality
- **Reprocessing**: Can rebuild Silver/Gold from Bronze
- **Schema Evolution**: Raw data preserved for new requirements
- **Audit Trail**: Complete lineage from source to insight

### Why Ephemeral ECS Tasks?

- **Cost Optimization**: Pay only for actual runtime
- **Resource Efficiency**: No idle containers
- **Isolation**: Each run starts fresh
- **Scalability**: Unlimited parallel tasks

### Developer Experience & Local Development

#### Local Development Stack

```yaml
# docker-compose.yml for local development
version: "3.8"
services:
  prefect-server:
    image: prefecthq/prefect:latest
    command: prefect server start
    environment:
      PREFECT_SERVER_API_HOST: 0.0.0.0
    ports:
      - "4200:4200"

  postgres:
    image: postgres:latest
    environment:
      POSTGRES_DB: prefect
      POSTGRES_USER: prefect
      POSTGRES_PASSWORD: prefect

  localstack:
    image: localstack/localstack:latest
    environment:
      SERVICES: s3,glue,athena,secretsmanager
      DEFAULT_REGION: ca-central-1
    ports:
      - "4566:4566"

  dbt:
    build: ./finks-dbt
    volumes:
      - ./finks-dbt:/dbt
    environment:
      DBT_PROFILES_DIR: /dbt/profiles
```

#### Development Workflow

1. **Infrastructure Development**:

   - Use Pulumi with local state for experimentation
   - Test components against LocalStack
   - Validate IAM policies with AWS IAM Policy Simulator

2. **Pipeline Development**:

   - Develop Prefect flows locally against LocalStack S3
   - Test dbt models with subset of production data
   - Use dbt's `--defer` flag to reference production models

3. **Testing Strategy**:
   - Unit tests for Pulumi components
   - Integration tests for Prefect flows
   - dbt data tests for transformations
   - End-to-end tests in staging environment
