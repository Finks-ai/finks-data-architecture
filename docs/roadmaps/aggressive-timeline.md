# Aggressive MVP Timeline: End-to-End Value in One Month

This document outlines an accelerated, horizontally-focused engineering plan to deliver a production-grade, end-to-end data pipeline within one month. The goal is to build a thin, functional slice of the platform to deliver value quickly and establish a reusable pattern, while explicitly deferring non-critical features.

**Guiding Principle**: Deliver a complete, queryable data product first, then harden and scale.

---

## One-Month MVP Plan

### **Weeks 1-2: Foundational Infrastructure**

This combines the most critical infrastructure tasks into a two-week sprint. The focus is on getting the non-negotiable, automated foundation in place.

- **Epic: Infrastructure as Code & CI/CD**

  - **Task**: `[ ]` Initialize Pulumi project in Python with a clear directory structure.
  - **Task**: `[ ]` Configure Pulumi for `dev`, `staging`, and `prod` environments.
  - **Task**: `[ ]` Provision a versioned S3 bucket to store Pulumi's state.
  - **Task**: `[ ]` Set up local development tools (`black`, `ruff`, `pre-commit`).
  - **Task**: `[ ]` Create a GitHub Actions workflow that runs `pulumi preview` on PRs and `pulumi up --yes` on merges to `main` (targeting `staging`).

- **Epic: Core Cloud Resources (via Pulumi)**
  - **Task**: `[ ]` **Networking**: Implement a `VPC` with public/private subnets.
  - **Task**: `[ ]` **Security**: Implement baseline Security Groups and IAM Roles.
  - **Task**: `[ ]` **Secrets**: Deploy AWS Secrets Manager and store initial credentials.
  - **Task**: `[ ]` **Data Lake**: Create `bronze`, `silver`, and `gold` S3 buckets with logging and versioning.

---

### **Week 3: Orchestration, Transformation & Serving**

With the core cloud running, the focus shifts to deploying the software that will manage, transform, and serve the data.

- **Epic: Orchestration Platform Deployment**

  - **Task**: `[ ]` Implement a Pulumi component to deploy Prefect Server on ECS Fargate.
  - **Task**: `[ ]` Provision a Multi-AZ PostgreSQL RDS instance for the Prefect backend.
  - **Task**: `[ ]` Configure an ALB with an ACM certificate for secure UI access.
  - **Task**: `[ ]` Implement a Python script to manage Prefect Blocks as Code (for S3 storage and AWS credentials).

- **Epic: Transformation & Serving Setup**
  - **Task**: `[ ]` Initialize the `dbt_project` repository.
  - **Task**: `[ ]` Create a Pulumi component for an ephemeral dbt ECS task runner.
  - **Task**: `[ ]` **Infrastructure**: Define the AWS Glue Job as a Pulumi resource.
  - **Task**: `[ ]` **API**: Implement an API Gateway with a Lambda authorizer to provide secure data access.
  - **Task**: `[ ]` **API**: Create a "serverless API" Lambda function in Pulumi that queries the Gold zone via Athena.

---

### **Week 4: The First End-to-End Pipeline**

This is the final sprint to build and orchestrate the first data flow, delivering a tangible, API-accessible data product.

- **Epic: Build the Pipeline**

  - **Task**: `[ ]` **Ingest**: Develop a containerized Python script to extract FMP API data and land it in the `bronze-zone`.
  - **Task**: `[ ]` **Cleanse**: Write an AWS Glue script (PySpark) that reads raw data, validates the schema, casts data types, and writes the output as Parquet to the `silver-zone`.
  - **Task**: `[ ]` **Transform**: Build a dbt staging model (`stg_fmp_*`) and a final mart model (`daily_portfolio_summary`). Include basic `not_null` tests and column documentation.

- **Epic: Orchestrate and Schedule**
  - **Task**: `[ ]` Create a Prefect flow that runs the ingestion container as an ECS Task.
  - **Task**: `[ ]` Create a second Prefect flow that triggers the AWS Glue job for cleansing.
  - **Task**: `[ ]` Create a master Prefect flow that chains the ingestion, cleansing, and dbt transformation flows in the correct dependency order.
  - **Task**: `[ ]` Schedule the master flow to run nightly.

---

## What is Deferred (Post-MVP)

This aggressive timeline is achieved by explicitly deferring features that are not critical for the first, single pipeline. These items can be prioritized in a "Phase 2" once the MVP is delivered.

### Deferred from Original Plan:

- **Phase 1 (Foundation):**

  - Manual approval gate for production deployments (can be added to the CI/CD workflow later).
  - Publishing a common base Docker image to ECR (services can use public images for now).

- **Phase 2 (First Pipeline):**

  - **Notifications**: Success/failure Slack notifications from the master Prefect flow.
  - **Backfills**: A generalized, parameterized backfill pattern for reprocessing historical data.

- **Phase 3 (Scale & Harden):**
  - **Streaming Data**: Ingestion of a second, streaming data source (e.g., Kinesis).
  - **Advanced Data Quality**: Integration of `dbt-expectations` and data quality gates in CI/CD.
  - **"Slim CI"**: dbt pattern to only build/test modified models in pull requests.
  - **Data Governance**: Fine-grained access control with AWS Lake Formation.
  - **Advanced Observability**: A unified Grafana dashboard and correlated logging.
  - **Cost Optimization**: Use of Fargate Spot or VPC Endpoints.
  - **Documentation**: "Cookbook" for onboarding new data sources and documenting the local developer setup.

This approach delivers a functioning, secure, and automated data product in one month, providing a solid foundation to iterate upon.
