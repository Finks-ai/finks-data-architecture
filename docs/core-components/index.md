## Core Components

### 1. Infrastructure Foundation

- **Pulumi**: Infrastructure as Code in TypeScript/Python
- **AWS VPC**: Private network with public/private subnets
- **Secrets Manager**: Centralized credential management
- **CI/CD Pipeline**: GitHub Actions for automated deployments

### 2. Orchestration Layer

- **Prefect Server**: Hosted on ECS Fargate
- **Prefect Agents**: Multiple agent pools for workload isolation
- **PostgreSQL RDS**: Metadata storage with multi-AZ deployment
- **Prefect Blocks**: Reusable configuration for AWS resources

#### Configuration as Code: Managing Prefect Blocks

All Prefect Blocks (for S3 storage, AWS credentials, etc.) are defined and managed as code, typically within the same Pulumi or a separate Python project. They are applied to the Prefect server via a CI/CD script. This ensures that configuration is version-controlled, auditable, and consistent across all environments, eliminating manual UI setup.

```python
# Example: Defining blocks in a Python script
from prefect_aws import AwsCredentials, S3Bucket, ECSTask

def create_prefect_blocks():
    # AWS credentials block
    aws_creds = AwsCredentials(
        aws_access_key_id="...",
        aws_secret_access_key="..."
    )
    aws_creds.save(name="production-aws-creds", overwrite=True)

    # S3 storage block
    s3_block = S3Bucket(
        bucket_name="data-lake-bronze",
        credentials=aws_creds
    )
    s3_block.save(name="bronze-storage", overwrite=True)

    # ECS task block for dbt
    ecs_task = ECSTask(
        task_definition_arn="arn:aws:ecs:ca-central-1:123456789:task-definition/dbt-core",
        cluster="transformation-cluster",
        vpc_id="vpc-xyz",
        credentials=aws_creds
    )
    ecs_task.save(name="dbt-ecs-task", overwrite=True)

if __name__ == "__main__":
    create_prefect_blocks()
```

### 3. Ingestion Layer

- **ECS Tasks**: Ephemeral containers for data extraction
  - MongoDB Connector
  - Financial Modeling Prep API Connector
  - News Streaming Service
- **Kinesis Data Streams**: Real-time data ingestion
- **Kinesis Firehose**: Automatic batching to S3

### 4. Storage Layer - Medallion Architecture

- **Bronze Zone**: Raw, immutable data
- **Silver Zone**: Cleansed, validated, standardized data
- **Gold Zone**: Business-ready, aggregated data
- **AWS Glue Catalog**: Unified metadata repository
- **Lake Formation**: Fine-grained access control

### 5. Transformation Layer

- **dbt Core on ECS**: Containerized transformation engine
- **AWS Glue Jobs**: Bronze to Silver data cleansing
- **Athena**: SQL interface for Silver zone queries
- **Redshift Serverless**: Optional warehouse for complex analytics

### 6. Serving Layer

- **API Gateway + Lambda**: REST/GraphQL APIs
- **Athena**: Direct SQL access for analysts
- **QuickSight**: Business intelligence dashboards
