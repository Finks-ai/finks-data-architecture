# Data Platform Implementation Roadmap

This document outlines the three-month engineering plan to materialize the financial data platform. The project is divided into three one-month phases, designed to deliver incremental value and build upon a solid foundation.

## Guiding Principles

- **Foundation First**: Build core infrastructure, security, and automation before application logic.
- **Deliver Incrementally**: Complete one end-to-end data pipeline before scaling to others.
- **Automate Everything**: All infrastructure, configuration, and deployments should be managed through code.
- **Document as We Go**: Onboarding and maintenance should be straightforward.

---

## Phase 1 (Month 1): The Foundation

**Goal**: Establish the secure, automated, and scalable cloud foundation. By the end of this phase, the core infrastructure will be deployed via CI/CD, and the orchestration platform will be live and ready to manage workflows.

### Epic 1: Infrastructure as Code & Project Setup

- **Task**: `[ ]` Initialize Pulumi project in Python with a clear directory structure (`components/`, `config/`).
- **Task**: `[ ]` Configure Pulumi for three environments: `dev`, `staging`, `prod` using separate stack configuration files.
- **Task**: `[ ]` Provision a versioned S3 bucket to store Pulumi's state remotely.
- **Task**: `[ ]` Set up local development quality tools: `black` for formatting, `ruff` for linting, and `pre-commit` hooks.

### Epic 2: Core Cloud Infrastructure (Pulumi)

- **Task**: `[ ]` **Networking**: Implement a `VPC` component in Pulumi to define the VPC, public/private subnets, and NAT Gateways.
- **Task**: `[ ]` **Security**: Implement a `Security` component to manage Security Groups and baseline IAM Roles (e.g., `ecs-task-execution-role`).
- **Task**: `[ ]` **Secrets**: Deploy AWS Secrets Manager and establish a clear naming convention (e.g., `/prod/prefect/database_url`). Store initial secrets.
- **Task**: `[ ]` **Data Lake Storage**: Implement a `DataLake` component to create and configure the `bronze`, `silver`, and `gold` S3 buckets with versioning, access logging, and lifecycle policies.

### Epic 3: CI/CD Automation (GitHub Actions)

- **Task**: `[ ]` Create a GitHub Actions workflow that runs `pulumi preview` on every pull request to validate infrastructure changes.
- **Task**: `[ ]` Configure the workflow to run `pulumi up --yes` on merges to `main`, targeting the `staging` environment.
- **Task**: `[ ]` Add a manual approval gate (`environment: production`) in the workflow for all deployments to the `production` environment.
- **Task**: `[ ]` Build and publish a common base Docker image to AWS ECR for our Python services to ensure consistent dependencies.

### Epic 4: Orchestration Platform Deployment (Prefect)

- **Task**: `[ ]` Implement a Pulumi component to deploy Prefect Server on ECS Fargate, including task definitions and service configuration.
- **Task**: `[ ]` Provision a Multi-AZ PostgreSQL RDS instance for the Prefect backend.
- **Task**: `[ ]` Configure an Application Load Balancer (ALB) with an ACM certificate to provide secure HTTPS access to the Prefect UI.
- **Task**: `[ ]` Implement a Python script to manage Prefect Blocks as Code, defining S3 storage and AWS credentials blocks, and run it via CI/CD.

---

## Phase 2 (Month 2): The First End-to-End Pipeline

**Goal**: Deliver the first tangible value by building a complete, production-grade data pipeline for a single batch data source (Financial Modeling Prep API). This creates a reusable pattern for all future pipelines.

### Epic 1: Batch Ingestion Connector (FMP API)

- **Task**: `[ ]` Define the data schema for the target API endpoint.
- **Task**: `[ ]` Develop a robust Python extraction script with error handling, retries, and parameterization (e.g., for dates).
- **Task**: `[ ]` Containerize the script and create a Prefect flow (`fmp-api-ingestion`) that runs the extraction task and lands the raw JSON data in the `bronze-zone` with proper partitioning (`source=fmp/year=...`).
- **Task**: `[ ]` Define the `ECSTask` resource in Pulumi for this connector, specifying its memory/CPU and IAM permissions.

### Epic 2: Bronze-to-Silver Cleansing & Validation

- **Task**: `[ ]` Write an AWS Glue script (PySpark) that reads the raw FMP data, applies schema validation, casts data types, and writes the output as Parquet to the `silver-zone`.
- **Task**: `[ ]` Define the Glue Job as a Pulumi resource.
- **Task**: `[ ]` Create a Prefect flow (`bronze-to-silver-fmp`) that triggers the Glue Job. Use an S3 event on the Bronze bucket as the trigger for this flow.

### Epic 3: Business Transformation with dbt

- **Task**: `[ ]` Initialize the `dbt_project` repository and set up the `profiles.yml` to connect to AWS Athena.
- **Task**: `[ ]` Create a Docker image for the dbt project and define an ephemeral dbt ECS task runner in Pulumi.
- **Task**: `[ ]` Build a staging model (`stg_fmp_market_data.sql`) that cleans and prepares data from the Silver table.
- **Task**: `[ ]` Build a mart model (`daily_portfolio_summary.sql`) that creates a business-ready, aggregated table in the `gold-zone`.
- **Task**: `[ ]` Add basic dbt tests (`not_null`, `unique`) to the primary keys in the dbt models.
- **Task**: `[ ]` Document the columns and description for the new models in their `schema.yml` file.

### Epic 4: Full End-to-End Orchestration

- **Task**: `[ ]` Create a master Prefect flow (`master-fmp-pipeline`) that chains the ingestion, cleansing, and dbt transformation flows in the correct dependency order.
- **Task**: `[ ]` Implement basic success and failure notifications to a Slack channel from the master flow.
- **Task**: `[ ]` Schedule the master flow to run nightly using a `CronSchedule`.
- **Task**: `[ ]` Implement the parameterized backfill flow pattern to enable reprocessing of historical data.

---

## Phase 3 (Month 3): Scale, Harden & Serve

**Goal**: Expand the platform by adding new data types, implementing robust monitoring and data governance, and exposing data to end-users.

### Epic 1: Add a Streaming Data Source (e.g., News Feed)

- **Task**: `[ ]` Define Kinesis Data Streams and Kinesis Firehose resources in Pulumi to create a ingestion path to the `bronze-zone`.
- **Task**: `[ ]` Develop and containerize the news streaming producer application and deploy it as a long-running ECS service.
- **Task**: `[ ]` Adapt the Bronze-to-Silver process to handle the new streaming data, including a small-file compaction strategy.
- **Task**: `[ ]` Build new dbt models for the news data to demonstrate joining batch and stream sources.

### Epic 2: Advanced Data Quality & Governance

- **Task**: `[ ]` Integrate the `dbt-expectations` package and add more comprehensive data quality tests to all dbt models.
- **Task**: `[ ]` Configure the master Prefect flow to halt and alert if `dbt test` fails, creating a true Quality Gate.
- **Task**: `[ ]` Implement the "Slim CI" pattern for dbt in GitHub Actions to only build and test modified models and their downstream dependencies in PRs.
- **Task**: `[ ]` Use AWS Lake Formation to apply fine-grained, column-level access controls to tables in the Gold zone.

### Epic 3: Observability & Cost Optimization

- **Task**: `[ ]` Create a unified Grafana dashboard to monitor key platform metrics (Prefect success rates, data freshness, ECS utilization, Athena query costs).
- **Task**: `[ ]` Implement correlated logging by injecting the `prefect.flow_run_id` into all ECS and Glue task logs, enabling end-to-end tracing.
- **Task**: `[ ]` Configure Prefect agent pools to run on Fargate Spot to reduce compute costs.
- **Task**: `[ ]` Configure VPC Endpoints for S3 and Glue to reduce NAT Gateway data transfer costs.

### Epic 4: Data Serving Layer & Documentation

- **Task**: `[ ]` Implement an API Gateway with a Lambda authorizer to provide secure access to data.
- **Task**: `[ ]` Create a "serverless API" Lambda function that queries the Gold zone via Athena to serve the `daily_portfolio_summary` dataset.
- **Task**: `[ ]` Create a "Cookbook" page in the documentation site detailing the step-by-step process for onboarding a new data source.
- **Task**: `[ ]` Document the local development stack (`docker-compose.yml`) and workflow for new developers.
