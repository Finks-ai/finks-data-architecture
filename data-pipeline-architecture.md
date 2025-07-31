# Financial Data Pipeline Architecture with Prefect & AWS

## Technology Stack Overview

### Core Technologies
- **Pulumi**: Infrastructure as Code platform using real programming languages to define and deploy all AWS resources
- **Prefect**: Workflow orchestration platform that schedules, monitors, and manages all data pipeline tasks
- **AWS ECS (Fargate)**: Serverless container platform hosting Prefect components, data connectors, and dbt processes
- **dbt Core**: SQL-based transformation framework running on ECS for turning raw data into analytics-ready datasets
- **AWS S3**: Object storage serving as the data lake with Bronze/Silver/Gold zones
- **AWS Glue**: Managed ETL service for Bronze-to-Silver data cleansing and schema validation
- **AWS Athena**: Serverless SQL query engine for analyzing data directly in S3
- **Kinesis Data Streams**: Real-time data ingestion for streaming sources
- **Kinesis Firehose**: Managed service for reliably loading streaming data into S3
- **AWS Lambda**: Serverless functions for event processing and lightweight API endpoints
- **PostgreSQL RDS**: Managed database storing Prefect metadata and orchestration state
- **DynamoDB**: NoSQL database for tracking data lineage and pipeline metadata
- **AWS Secrets Manager**: Secure storage for API keys, database credentials, and sensitive configuration
- **API Gateway**: Managed API service for exposing data lake contents to consumers
- **CloudWatch**: Monitoring service for logs, metrics, and operational insights
- **Docker**: Container technology for packaging all services and ensuring consistency across environments
- **GitHub Actions**: CI/CD platform for automated testing and deployment of infrastructure and code changes

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Core Components](#core-components)
3. [Infrastructure as Code with Pulumi](#infrastructure-as-code-with-pulumi)
4. [Data Lake Architecture](#data-lake-architecture)
5. [Detailed Flow](#detailed-flow)
6. [Service Architecture](#service-architecture)
7. [Transformation Layer with dbt](#transformation-layer-with-dbt)
8. [Technology Decisions](#technology-decisions)
9. [Security Architecture](#security-architecture)
10. [Cost Optimization Strategy](#cost-optimization-strategy)
11. [Monitoring & Observability](#monitoring--observability)

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

## Data Lake Architecture

### Medallion Architecture Pattern
The Bronze → Silver → Gold pattern provides clear data quality tiers:

#### Bronze Zone (Raw Data)
- **Purpose**: Preserve raw data exactly as received
- **Format**: Original format (JSON, CSV, API responses)
- **Partitioning**: By source, ingestion time
- **Retention**: Long-term (archive to Glacier after 90 days)
- **Schema**: No schema enforcement
```
s3://bronze-zone/
└── source={source_name}/
    └── year={YYYY}/
        └── month={MM}/
            └── day={DD}/
                └── hour={HH}/
                    └── run_id={prefect_run_id}/
                        └── {timestamp}_{batch_id}.json
```

#### Silver Zone (Cleansed Data)
- **Purpose**: Standardized, deduplicated, validated data
- **Format**: Parquet with compression
- **Partitioning**: By business entity and time
- **Retention**: Medium-term (1-2 years hot storage)
- **Schema**: Enforced via Glue Schema Registry
```
s3://silver-zone/
└── entity={entity_name}/
    └── year={YYYY}/
        └── month={MM}/
            └── day={DD}/
                └── {entity}_{version}.parquet
```

#### Gold Zone (Business Data)
- **Purpose**: Aggregated, business-ready datasets
- **Format**: Parquet optimized for analytics
- **Partitioning**: By use case and access patterns
- **Retention**: Based on business needs
- **Schema**: Documented business glossary
```
s3://gold-zone/
└── domain={business_domain}/
    └── dataset={dataset_name}/
        └── version={schema_version}/
            └── year={YYYY}/
                └── month={MM}/
                    └── {dataset}_{date}.parquet
```

## Detailed Flow

### Data Ingestion Flow
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Prefect   │────▶│  ECS Task   │────▶│   Bronze    │
│   Trigger   │     │  (Extract)  │     │     S3      │
└─────────────┘     └─────────────┘     └─────────────┘
       │                                         │
       │                                         ▼
       │                                 ┌─────────────┐
       └────────────────────────────────▶│  Metadata   │
                                         │  DynamoDB   │
                                         └─────────────┘
```

1. **Scheduled Trigger**: Prefect Server initiates flow based on schedule or event
2. **Ephemeral Extraction**: Prefect spins up ECS task for specific data source
3. **Raw Data Landing**: Extracted data written to Bronze zone with metadata
4. **Lineage Tracking**: Metadata recorded in DynamoDB for audit trail

#### Handling Historical Backfills
The architecture is designed to support large-scale historical backfills. Prefect flows are parameterized (e.g., by `start_date` and `end_date`). A backfill can be triggered by running a flow with a date range, and Prefect's concurrency capabilities will schedule and manage the parallel execution of hundreds or thousands of historical tasks.

```python
@flow(name="backfill-ingestion")
def backfill_historical_data(
    source: str,
    start_date: datetime,
    end_date: datetime,
    parallel_days: int = 30
):
    """Backfill historical data with parallel execution"""
    
    # Generate date ranges for parallel processing
    date_ranges = generate_date_chunks(start_date, end_date, parallel_days)
    
    # Submit parallel ingestion tasks
    futures = []
    for date_range in date_ranges:
        future = ingest_data.submit(
            source=source,
            start_date=date_range[0],
            end_date=date_range[1]
        )
        futures.append(future)
    
    # Wait for all tasks to complete
    results = [future.result() for future in futures]
    
    # Trigger downstream processing
    trigger_silver_processing(source, start_date, end_date)
```

### Data Processing Flow
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Bronze    │────▶│  Glue Job   │────▶│   Silver    │
│     S3      │     │ (Cleanse)   │     │     S3      │
└─────────────┘     └─────────────┘     └─────────────┘
                            │
                            ▼
                    ┌─────────────┐
                    │   Quality   │
                    │    Gate     │
                    └─────────────┘
```

1. **Bronze to Silver**: AWS Glue job performs schema validation and cleansing
2. **Quality Gates**: Data quality checks must pass before promotion
3. **Schema Registry**: Validated schemas registered for downstream use

### Transformation Flow
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Silver    │────▶│  dbt Core   │────▶│    Gold     │
│     S3      │     │   on ECS    │     │     S3      │
└─────────────┘     └─────────────┘     └─────────────┘
                            │
                            ▼
                    ┌─────────────┐
                    │  dbt Tests  │
                    │   & Docs    │
                    └─────────────┘
```

1. **Business Logic**: dbt models transform Silver data to Gold
2. **Testing**: dbt tests ensure transformation quality
3. **Documentation**: Auto-generated documentation for lineage

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

## Transformation Layer with dbt

### dbt on ECS Architecture
Running dbt Core on ECS provides:
- **Isolation**: Each dbt run in clean environment
- **Scalability**: Multiple concurrent transformations
- **Version Control**: Docker images tagged with dbt version
- **Resource Management**: CPU/memory limits per project

### dbt Project Structure
```
dbt_project/
├── models/
│   ├── staging/           # Silver zone queries
│   │   ├── stg_market_data.sql
│   │   └── stg_transactions.sql
│   ├── intermediate/      # Complex joins
│   │   └── int_enriched_trades.sql
│   └── marts/            # Gold zone outputs
│       ├── finance/
│       │   └── daily_portfolio_summary.sql
│       └── risk/
│           └── var_calculations.sql
├── tests/                # Data quality tests
├── macros/              # Reusable SQL functions
└── docs/                # Business documentation
```

### dbt Execution Pattern with Prefect Integration
```python
# Using Prefect's native ECS integration for robust task execution
from prefect import task, flow
from prefect_aws.ecs import ECSTask

@task
def run_dbt_models(model_selector: str, target: str):
    # Load pre-configured ECS task block
    ecs_task_runner = ECSTask.load("dbt-ecs-task")
    
    # Run dbt as ephemeral ECS task with proper monitoring
    return ecs_task_runner.run(
        task_definition_arn="arn:aws:ecs:region:account:task-definition/dbt-core-task",
        overrides={
            "containerOverrides": [{
                "name": "dbt",
                "command": ["dbt", "run", "--select", model_selector],
                "environment": [
                    {"name": "DBT_TARGET", "value": target},
                    {"name": "DBT_PROFILES_DIR", "value": "/secrets"}
                ]
            }]
        },
        wait_for_completion=True,
        poll_interval=10
    )

@flow
def dbt_transformation_flow(models: list[str], target: str = "prod"):
    # Run models in parallel using Prefect's native concurrency
    for model in models:
        run_dbt_models.submit(model, target)
```

### CI/CD Workflow for dbt (Slim CI Pattern)
```yaml
# .github/workflows/dbt-pr.yml
name: dbt Pull Request CI

on:
  pull_request:
    paths:
      - 'dbt_project/**'

jobs:
  slim-ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Download production manifest
        run: |
          aws s3 cp s3://dbt-artifacts/prod/manifest.json ./prod_manifest.json
      
      - name: Run dbt Slim CI
        run: |
          # Only build and test modified models and their downstream dependencies
          dbt build --select state:modified+ --defer --state ./prod_manifest
        env:
          DBT_TARGET: ci
          DBT_PROFILES_DIR: ./ci_profiles
      
      - name: Upload CI artifacts
        if: always()
        run: |
          aws s3 sync ./target s3://dbt-artifacts/ci/${{ github.sha }}/
```

#### Artifact Promotion Strategy
The CI/CD pipeline follows a "build once, deploy many" strategy. A Docker image is built and tested in the staging environment. To promote to production, the exact same image is re-tagged (e.g., from `myapp:staging-v1.2` to `myapp:prod-v1.2`) and deployed. No new code is introduced during promotion, guaranteeing that what was tested is what is released.

```bash
# Example promotion script
docker pull myregistry/dbt-core:staging-v1.2
docker tag myregistry/dbt-core:staging-v1.2 myregistry/dbt-core:prod-v1.2
docker push myregistry/dbt-core:prod-v1.2
# Update ECS task definition to use prod-v1.2
```

### Data Quality Integration
The architecture uses a unified approach to data quality:
- **dbt-expectations**: Primary data quality framework integrated directly into dbt
- **dbt tests**: Execute as part of transformation flows
- **Quality gates**: Failed tests block data promotion to Gold zone

Example dbt-expectations test:
```yaml
# models/silver/schema.yml
models:
  - name: silver_market_data
    tests:
      - dbt_expectations.expect_table_row_count_to_be_between:
          min_value: 1000
          max_value: 1000000
    columns:
      - name: price
        tests:
          - not_null
          - dbt_expectations.expect_column_values_to_be_between:
              min_value: 0
              max_value: 1000000
```

## Technology Decisions

### Why Pulumi over Terraform?
- **Type Safety**: Catch errors before deployment
- **Programming Constructs**: Loops, conditionals, functions
- **IDE Support**: Full IntelliSense and refactoring
- **Testing**: Unit test infrastructure code
- **Abstraction**: Create high-level components

### Why dbt Core on ECS over alternatives?
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
version: '3.8'
services:
  prefect-server:
    image: prefecthq/prefect:2-latest
    command: prefect server start
    environment:
      PREFECT_SERVER_API_HOST: 0.0.0.0
    ports:
      - "4200:4200"
  
  postgres:
    image: postgres:14
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
    build: ./dbt_project
    volumes:
      - ./dbt_project:/dbt
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

### Network Security
```
┌─────────────────────────────────────────┐
│              VPC (10.0.0.0/16)          │
├─────────────────┬───────────────────────┤
│  Public Subnet  │   Private Subnet      │
│  ┌───────────┐  │  ┌─────────────────┐ │
│  │    ALB    │  │  │   ECS Tasks     │ │
│  │           │  │  │   RDS           │ │
│  └───────────┘  │  │   ElastiCache   │ │
│                 │  └─────────────────┘ │
│  ┌───────────┐  │  ┌─────────────────┐ │
│  │    NAT    │  │  │ VPC Endpoints   │ │
│  │  Gateway  │  │  │ (S3, DynamoDB)  │ │
│  └───────────┘  │  └─────────────────┘ │
└─────────────────┴───────────────────────┘
```

### Identity & Access Management
- **Service Accounts**: Each service has minimal IAM role
- **Cross-Account Roles**: For multi-account deployments
- **Temporary Credentials**: No long-lived access keys
- **MFA Enforcement**: For human users

### Data Security
- **Encryption at Rest**: S3 SSE-S3, RDS encryption
- **Encryption in Transit**: TLS 1.2+ everywhere
- **Key Management**: AWS KMS with key rotation
- **Data Masking**: PII masked in Silver/Gold zones

### Secrets Management
```
AWS Secrets Manager
├── /prefect/database/connection
├── /mongodb/connection-string
├── /fmp-api/api-key
├── /dbt/profiles/production
└── /monitoring/datadog-api-key
```

## Cost Optimization Strategy

### Compute Optimization
- **Graviton Processors**: 40% better price-performance
- **Spot Instances**: For non-critical batch jobs
- **Fargate Spot**: For Prefect agents
- **Right-sizing**: Regular analysis of resource usage

### Storage Optimization
- **S3 Intelligent-Tiering**: Automatic tier management
- **Lifecycle Policies**: 
  - Bronze: Standard → Standard-IA (30 days) → Glacier (90 days)
  - Silver: Standard → Standard-IA (60 days)
  - Gold: Standard (always hot)
- **Compression**: Parquet with Snappy compression
- **Partitioning**: Reduce data scanned by queries
- **Small File Compaction**: Scheduled Glue jobs merge small files into optimal sizes

### Small File Compaction Strategy
```python
# Prefect flow for periodic file compaction
@flow(name="small-file-compaction")
def compact_silver_zone_files(date: datetime):
    """Merge small Parquet files into larger, optimized files"""
    
    glue_job = GlueJob.load("silver-compaction-job")
    
    # Run compaction for previous day's data
    glue_job.run(
        arguments={
            "--partition_date": date.strftime("%Y-%m-%d"),
            "--target_file_size": "512MB",
            "--source_path": f"s3://silver-zone/entity=trades/year={date.year}/month={date.month:02d}/day={date.day:02d}/",
            "--temp_path": "s3://temp-zone/compaction/"
        }
    )
    
    # Verify compaction results
    validate_compaction_results(date)

# Schedule to run nightly
compact_silver_zone_files_deployment = Deployment.build_from_flow(
    flow=compact_silver_zone_files,
    name="nightly-compaction",
    schedule=CronSchedule(cron="0 2 * * *"),  # 2 AM daily
    parameters={"date": "{{ yesterday_ds }}"}
)
```

### Data Transfer Optimization
- **VPC Endpoints**: Avoid NAT Gateway charges
- **S3 Transfer Acceleration**: For large uploads
- **CloudFront**: For frequently accessed Gold data

### Reserved Capacity
- **RDS Reserved Instances**: For Prefect database
- **Compute Savings Plans**: For predictable ECS usage

## Monitoring & Observability

### Metrics Collection
```
Application Metrics → CloudWatch → Grafana
     │                    │           │
     └── StatsD ─────────┘           │
                                     │
Infrastructure Metrics → CloudWatch ──┘
```

### Key Metrics

#### Pipeline Health
- **Flow Success Rate**: Percentage of successful runs
- **Data Freshness**: Time since last successful update
- **Processing Time**: End-to-end latency per source
- **Error Rate**: Failures by type and stage

#### Data Quality
- **Schema Drift**: Changes detected in sources
- **Quality Score**: Percentage passing dbt tests
- **Completeness**: Records processed vs expected
- **Anomaly Detection**: Statistical outliers

#### Cost Metrics
- **Cost per GB**: Processing cost by data source
- **Resource Utilization**: CPU/memory efficiency
- **Storage Growth**: Month-over-month trends
- **Query Costs**: Athena scan costs by user

### Alerting Strategy
```
Priority Levels:
├── P1: Pipeline Failure (PagerDuty)
├── P2: Quality Degradation (Slack)
├── P3: Cost Anomaly (Email)
└── P4: Performance Warning (Dashboard)
```

### Observability Stack
- **Logs**: CloudWatch Logs → ElasticSearch
- **Traces**: AWS X-Ray for distributed tracing
- **Metrics**: CloudWatch + Prometheus
- **Dashboards**: Grafana for unified view

#### Correlated Logs and Traces
To enable distributed tracing, the Prefect Flow Run ID is injected as an environment variable into every ECS task (Ingestion, Glue, dbt). All logging within the containers is configured to include this ID. This allows for filtering logs in CloudWatch or ElasticSearch to see the complete, end-to-end trace of a single data pipeline execution across all its distributed components.

```python
# Example: Injecting flow run ID into ECS tasks
@task
def run_ecs_ingestion(source: str):
    flow_run_id = prefect.context.get("flow_run_id")
    
    ecs_task = ECSTask.load("ingestion-task")
    return ecs_task.run(
        overrides={
            "containerOverrides": [{
                "environment": [
                    {"name": "PREFECT_FLOW_RUN_ID", "value": flow_run_id},
                    {"name": "LOG_CORRELATION_ID", "value": flow_run_id},
                    {"name": "SOURCE", "value": source}
                ]
            }]
        }
    )

# Container logging configuration
import logging
import os

correlation_id = os.getenv("LOG_CORRELATION_ID", "unknown")
logging.basicConfig(
    format=f'%(asctime)s - %(name)s - %(levelname)s - [flow_run_id={correlation_id}] - %(message)s'
)
```

## Conclusion

This architecture provides a robust, scalable, and cost-effective foundation for financial data processing. The combination of Pulumi for infrastructure management, Prefect for orchestration, and dbt for transformations creates a modern data stack that balances power with maintainability. The medallion architecture ensures data quality improves at each stage while maintaining the flexibility to adapt to changing business requirements.

## Complete Architecture Flow

### End-to-End Data Flow Diagram
```
┌─────────────────────────────────────────────────────────────────────────┐
│                        INFRASTRUCTURE LAYER (Pulumi)                     │
│         Defines VPC, IAM Roles, S3 Buckets, ECS Clusters, RDS          │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
┌────────────────────────────────┴────────────────────────────────────────┐
│                        ORCHESTRATION LAYER                               │
│  ┌─────────────────┐    ┌─────────────────┐    ┌──────────────────┐   │
│  │  Prefect Server │────│  Prefect Agents │────│ PostgreSQL RDS   │   │
│  │   (ECS Fargate) │    │ (ECS Fargate)   │    │   (Metadata)     │   │
│  └─────────────────┘    └─────────────────┘    └──────────────────┘   │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
        ┌───────────▼──────────┐   ┌─────────▼──────────┐
        │   BATCH INGESTION    │   │ STREAM INGESTION  │
        │                      │   │                    │
        │ ┌─────────────────┐ │   │ ┌───────────────┐ │
        │ │ MongoDB ECS Task│ │   │ │News Stream    │ │
        │ └────────┬────────┘ │   │ │ECS Service    │ │
        │ ┌────────▼────────┐ │   │ └───────┬───────┘ │
        │ │FMP API ECS Task │ │   │         │         │
        │ └────────┬────────┘ │   │ ┌───────▼───────┐ │
        │          │          │   │ │Kinesis Streams│ │
        └──────────┼──────────┘   │ └───────┬───────┘ │
                   │              │ ┌────────▼────────┐ │
                   │              │ │Kinesis Firehose│ │
                   │              │ └────────┬────────┘ │
                   │              └──────────┼──────────┘
                   │                         │
        ┌──────────▼─────────────────────────▼──────────┐
        │              BRONZE ZONE (S3)                 │
        │         Raw, Immutable Data Storage           │
        │    Partitioned by source/year/month/day       │
        └─────────────────────┬──────────────────────────┘
                              │
                              ▼
        ┌────────────────────────────────────────────────┐
        │            S3 EVENT NOTIFICATION               │
        │                     │                          │
        │                     ▼                          │
        │            ┌───────────────┐                  │
        │            │ Lambda Function│                  │
        │            │  (Trigger)    │                  │
        │            └───────┬───────┘                  │
        │                    │                          │
        │            ┌───────▼───────┐                  │
        │            │   DynamoDB    │                  │
        │            │  (Lineage)    │                  │
        │            └───────────────┘                  │
        └────────────────────┬───────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Prefect Flow   │
                    │   Triggered     │
                    └────────┬────────┘
                             │
        ┌────────────────────▼───────────────────────────┐
        │           DATA CLEANSING LAYER                 │
        │                                                │
        │         ┌─────────────────────┐               │
        │         │   AWS Glue Jobs    │               │
        │         │ - Schema Validation │               │
        │         │ - Type Casting     │               │
        │         │ - Deduplication    │               │
        │         └──────────┬──────────┘               │
        │                    │                          │
        │         ┌──────────▼──────────┐               │
        │         │ Data Quality Gates  │               │
        │         │ (Great Expectations)│               │
        │         └──────────┬──────────┘               │
        └────────────────────┼───────────────────────────┘
                             │
        ┌────────────────────▼───────────────────────────┐
        │              SILVER ZONE (S3)                  │
        │      Cleansed, Validated, Standardized         │
        │         Parquet Format, Partitioned            │
        └────────────────────┬───────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Prefect Flow   │
                    │  (dbt trigger)  │
                    └────────┬────────┘
                             │
        ┌────────────────────▼───────────────────────────┐
        │         TRANSFORMATION LAYER                   │
        │                                                │
        │         ┌─────────────────────┐               │
        │         │  dbt Core on ECS   │               │
        │         │ - Business Logic   │               │
        │         │ - Aggregations     │               │
        │         │ - Calculations     │               │
        │         └──────────┬──────────┘               │
        │                    │                          │
        │         ┌──────────▼──────────┐               │
        │         │    dbt Tests       │               │
        │         │ (Quality Checks)   │               │
        │         └──────────┬──────────┘               │
        └────────────────────┼───────────────────────────┘
                             │
        ┌────────────────────▼───────────────────────────┐
        │               GOLD ZONE (S3)                   │
        │        Business-Ready Analytics Data           │
        │         Optimized Parquet Format               │
        └────────────────────┬───────────────────────────┘
                             │
                    ┌────────┴─────────┬─────────────────┐
                    │                  │                 │
        ┌───────────▼────────┐ ┌───────▼──────┐ ┌───────▼───────┐
        │   AWS Glue Catalog │ │ Lake Formation│ │Glue Crawler   │
        │   (Metadata)       │ │ (Permissions) │ │(Auto-discovery)│
        └────────────────────┘ └───────────────┘ └───────────────┘
                    │                  │                 │
                    └────────┬─────────┴─────────────────┘
                             │
        ┌────────────────────┴───────────────────────────┐
        │              SERVING LAYER                     │
        │                                                │
        │  ┌────────────┐  ┌──────────┐  ┌───────────┐ │
        │  │   Athena   │  │   API    │  │QuickSight │ │
        │  │(SQL Query) │  │ Gateway  │  │(BI Tools) │ │
        │  └────────────┘  └─────┬────┘  └───────────┘ │
        │                        │                      │
        │                 ┌──────▼──────┐               │
        │                 │   Lambda    │               │
        │                 │ (REST API)  │               │
        │                 └─────────────┘               │
        └────────────────────────────────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      END USERS          │
                    │ - Data Scientists       │
                    │ - Business Analysts     │
                    │ - External APIs         │
                    │ - Dashboard Consumers   │
                    └─────────────────────────┘
```

### Data Flow Summary

1. **Infrastructure Provisioning**: Pulumi defines and manages all AWS resources
2. **Orchestration Setup**: Prefect Server coordinates all pipeline activities
3. **Data Ingestion**: 
   - Batch: Prefect triggers ephemeral ECS tasks for MongoDB/API extraction
   - Stream: Continuous ECS service publishes to Kinesis → Firehose → S3
4. **Bronze Landing**: All raw data lands in Bronze S3 zone
5. **Event Processing**: S3 events trigger Lambda → DynamoDB lineage → Prefect flow
6. **Data Cleansing**: Glue jobs validate and cleanse Bronze → Silver
7. **Quality Gates**: Data must pass quality checks before promotion
8. **Business Transformation**: dbt Core on ECS transforms Silver → Gold
9. **Cataloging**: Glue Crawler discovers and catalogs new data
10. **Data Serving**: Multiple access patterns serve different user needs
11. **Monitoring**: CloudWatch tracks all metrics throughout the flow