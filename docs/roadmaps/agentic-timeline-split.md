# Agentic Programming Timeline Split for Claude Code

This document provides a battle-tested approach to splitting the 6-week timeline across multiple Claude Code sessions, with realistic time estimates, dependency management, and recovery strategies.

## Core Principles

1. **Context-Focused Sessions**: Each session has a single, clear objective
2. **Dependency-Aware**: Explicit dependencies and parallel opportunities
3. **Test-Driven**: Every session includes testing in its time estimate
4. **Recovery-Ready**: Each session can be re-run or recovered
5. **Progress Tracking**: Clear validation gates and success metrics

## Session Dependency Graph

```mermaid
graph TD
    S0[Session 0: Preparation]
    S1[S1: Local Dev Environment]
    S2[S2: Pulumi Structure]
    S3[S3: Core AWS]
    S4[S4: Repository Setup]
    
    S0 --> S1
    S0 --> S2
    S2 --> S3
    S1 --> S4
    
    S5[S5: ECS Infrastructure]
    S6[S6: RDS & Networking]
    S7[S7: CI/CD Pipelines]
    
    S3 --> S5
    S3 --> S6
    S4 --> S7
    
    S8[S8: Prefect Server]
    S9[S9: Prefect Agent]
    S10[S10: Prefect Blocks]
    
    S5 --> S8
    S6 --> S8
    S8 --> S9
    S9 --> S10
    
    S11[S11: Monitoring]
    S12[S12: dbt on ECS]
    S13[S13: Glue Setup]
    S14[S14: Data Models]
    
    S8 --> S11
    S5 --> S12
    S3 --> S13
    S12 --> S14
    
    S15[S15: FMP Connector]
    S16[S16: Orchestration]
    S17[S17: Data Quality]
    S18[S18: Backfill]
    
    S10 --> S16
    S14 --> S16
    S15 --> S16
    S16 --> S17
    S16 --> S18
    
    S19[S19: API Layer]
    S20[S20: Advanced Monitoring]
    S21[S21: Security]
    S22[S22: Documentation]
    
    S16 --> S19
    S11 --> S20
    S19 --> S21
    S21 --> S22
    
    style S0 fill:#f9f,stroke:#333,stroke-width:4px
    style S16 fill:#9f9,stroke:#333,stroke-width:4px
```

## Progress Tracking System

### Session Status Indicators
- 🔴 **Blocked**: Missing dependencies
- 🟡 **Ready**: Dependencies met, can start
- 🟢 **Complete**: Finished with tests passing
- 🔵 **In Progress**: Currently active
- ⚪ **Optional**: Can be deferred

### Validation Gates
Each week ends with a validation gate before proceeding:
- **Week 1 Gate**: Local dev works, core AWS deployed
- **Week 2 Gate**: Full infrastructure deployed
- **Week 3 Gate**: Prefect operational
- **Week 4 Gate**: Transformation infrastructure ready
- **Week 5 Gate**: End-to-end pipeline working
- **Week 6 Gate**: Production ready

---

## Session 0: Preparation (Do This First!)

### 🔴 Session 0: Environment and Credentials Setup
**Duration**: 1-2 hours (human time, not Claude time)
**When**: Before any other session

**Human Tasks**:
```bash
# 1. Set up AWS credentials
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_DEFAULT_REGION="ca-central-1"

# 2. Install required tools
brew install pulumi
brew install aws-cli
brew install docker
brew install python@3.12

# 3. Create GitHub repositories (empty)
- finks-infrastructure
- finks-pipelines  
- finks-dbt
- finks-ingestion

# 4. Set up Pulumi account and backend
pulumi login s3://your-pulumi-state-bucket

# 5. Prepare FMP API key
export FMP_API_KEY="your-fmp-key"
```

**Validation**: Can run `aws s3 ls` and `pulumi version` successfully

---

## Week 1: Foundation (4 Parallel Sessions)

### 🟡 Session 1: Local Development Environment
**Repository**: `finks-infrastructure`
**Estimated Duration**: 3-4 hours (includes testing)
**Dependencies**: Session 0 complete

**Context Prime for Claude**:
```
I need you to create a complete local development environment for the Finks data pipeline.
We're using Docker Compose to simulate AWS services locally with LocalStack.
Focus on developer experience - it should be easy to start/stop/reset the environment.
Include comprehensive error handling and helpful error messages.
```

**Required Resources**:
- Docker Desktop running
- 8GB RAM available
- Port 4200 (Prefect), 4566 (LocalStack) free

**Specific Tasks**:
```yaml
1. Docker Compose Setup (45 min)
   - Prefect Server with PostgreSQL
   - LocalStack with S3, Glue, Athena services
   - Proper health checks
   - Volume persistence

2. Developer Tools (30 min)
   - Makefile with targets:
     - make start
     - make stop  
     - make reset
     - make test
   - .env.example file

3. Testing Harness (45 min)
   - Test script that validates all services
   - Sample data upload to LocalStack S3
   - Prefect connection test

4. Documentation (30 min)
   - README with troubleshooting guide
   - Architecture diagram
   - Common issues section

5. Integration Test (30 min)
   - Full stack startup
   - Create test Prefect flow
   - Upload to S3 and verify
```

**Success Criteria**:
- [ ] `make start` brings up all services in <2 minutes
- [ ] `make test` passes all checks
- [ ] Can create and run a Prefect flow locally
- [ ] LocalStack S3 operations work

**Common Pitfalls**:
- Docker resource limits too low
- Port conflicts with existing services
- LocalStack Pro features attempted (use free tier only)

**Recovery Strategy**:
If session fails, commit working Docker Compose and continue in next session

**Template to Accelerate**:
```yaml
# docker-compose.yml starter
version: '3.8'
services:
  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: prefect
      POSTGRES_PASSWORD: prefect
      POSTGRES_DB: prefect
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U prefect"]
      interval: 10s
      timeout: 5s
      retries: 5

  prefect-server:
    image: prefecthq/prefect:2-python3.12
    command: prefect server start
    environment:
      PREFECT_SERVER_API_HOST: 0.0.0.0
      PREFECT_API_DATABASE_CONNECTION_URL: postgresql+asyncpg://prefect:prefect@postgres:5432/prefect
    ports:
      - "4200:4200"
    depends_on:
      postgres:
        condition: service_healthy

  localstack:
    image: localstack/localstack:latest
    environment:
      SERVICES: s3,glue,athena,iam,sts
      DEFAULT_REGION: ca-central-1
      DATA_DIR: /tmp/localstack/data
    ports:
      - "4566:4566"
    volumes:
      - "./localstack:/tmp/localstack"
```

**Handoff Documentation**:
```markdown
## Session 1 Handoff
- ✅ Docker Compose working with all services
- ✅ Makefile with developer commands  
- ✅ LocalStack S3 tested and working
- ✅ Prefect Server accessible at http://localhost:4200
- 📁 Files created:
  - docker-compose.yml
  - Makefile
  - .env.example
  - tests/validate_stack.py
  - README.md
- 🔗 Next: Can proceed with Session 2 or 4
```

---

### 🟡 Session 2: Pulumi Project Structure
**Repository**: `finks-infrastructure`
**Estimated Duration**: 4-5 hours (includes testing)
**Dependencies**: Session 0 complete

**Context Prime for Claude**:
```
I need you to create a well-structured Pulumi project for AWS infrastructure.
Use Python with strong typing and component-based architecture.
Each component should be testable in isolation.
Follow Pulumi best practices for resource naming and organization.
The goal is a maintainable, extensible infrastructure codebase.
```

**Required Resources**:
- Pulumi CLI installed and logged in
- Python 3.12+ environment
- AWS credentials configured

**Specific Tasks**:
```yaml
1. Project Initialization (30 min)
   - Pulumi new aws-python
   - Set up virtual environment
   - Configure pyproject.toml
   - Add type hints dependencies

2. Component Architecture (90 min)
   - BaseComponent abstract class
   - NetworkingComponent (VPC, subnets)
   - StorageComponent (S3 buckets)
   - ComputeComponent (ECS, Lambda)
   - MonitoringComponent (CloudWatch)

3. Configuration System (45 min)
   - Environment configs (dev, staging, prod)
   - Config validation
   - Secret handling patterns

4. Testing Framework (60 min)
   - Unit test setup with mocks
   - Component integration tests
   - Pulumi testing utilities

5. CI/CD Setup (45 min)
   - GitHub Actions workflow
   - Pulumi preview on PR
   - ESC integration for secrets
```

**Success Criteria**:
- [ ] `pulumi preview` runs without errors
- [ ] All components have unit tests
- [ ] Type checking passes with mypy
- [ ] CI workflow triggers on PR

**Common Pitfalls**:
- Circular dependencies between components
- Missing resource provider registrations
- Incorrect typing for Pulumi outputs

**Template to Accelerate**:
```python
# components/base.py
from abc import ABC, abstractmethod
from typing import Any, Dict
import pulumi

class BaseComponent(pulumi.ComponentResource, ABC):
    def __init__(self, name: str, args: Dict[str, Any], opts: pulumi.ResourceOptions = None):
        super().__init__(self._get_type(), name, {}, opts)
        self.name = name
        self.args = args
        self._create_resources()
        self.register_outputs(self._get_outputs())
    
    @abstractmethod
    def _get_type(self) -> str:
        """Return the component type string."""
        pass
    
    @abstractmethod
    def _create_resources(self) -> None:
        """Create all resources for this component."""
        pass
    
    @abstractmethod
    def _get_outputs(self) -> Dict[str, Any]:
        """Return component outputs."""
        pass

# components/networking.py
from typing import Dict, Any
import pulumi_aws as aws
from .base import BaseComponent

class NetworkingComponent(BaseComponent):
    def _get_type(self) -> str:
        return "custom:infrastructure:Networking"
    
    def _create_resources(self) -> None:
        # Create VPC
        self.vpc = aws.ec2.Vpc(
            f"{self.name}-vpc",
            cidr_block="10.0.0.0/16",
            enable_dns_hostnames=True,
            enable_dns_support=True,
            tags={"Name": f"{self.name}-vpc"}
        )
        
        # Create subnets...
```

**Handoff Documentation**:
```markdown
## Session 2 Handoff
- ✅ Pulumi project structure created
- ✅ Component-based architecture implemented
- ✅ All components have unit tests
- ✅ GitHub Actions CI configured
- 📁 Files created:
  - __main__.py
  - components/base.py
  - components/networking.py
  - components/storage.py
  - components/compute.py
  - tests/test_components.py
  - .github/workflows/pulumi.yml
- 🔗 Next: Session 3 can use these components
```

---

### Session 3: Core AWS Components
**Repository**: `finks-infrastructure`
**Duration**: 4-5 hours
**Context**: AWS resource creation

```
Tasks for Claude:
1. Implement VPC with public/private subnets
2. Create S3 buckets (bronze, silver, gold) with:
   - Lifecycle policies
   - Versioning
   - Access logging
3. Set up Secrets Manager
4. Create base IAM roles
5. Deploy and test in dev environment
```

**Handoff**: Core AWS infrastructure deployed and tested

---

### Session 4: Repository Scaffolding
**Duration**: 2 hours
**Context**: Create all repository structures

```
Tasks for Claude:
1. Create finks-pipelines repository:
   - Prefect project structure
   - Basic flow templates
   - Testing framework
2. Create finks-dbt repository:
   - dbt project initialization
   - Medallion folder structure
   - Docker setup for local dev
3. Create finks-ingestion repository:
   - Connector template
   - Shared utilities structure
   - Docker templates
```

**Handoff**: All repositories initialized with proper structure

---

## Week 2: Infrastructure Completion (3 Parallel Sessions)

### Session 5: ECS Infrastructure
**Repository**: `finks-infrastructure`
**Duration**: 4-5 hours
**Context**: Container orchestration setup

```
Tasks for Claude:
1. Create ECS cluster with Fargate
2. Define task definitions for:
   - Prefect Server
   - Prefect Agent
   - Generic worker template
3. Set up ECR repositories
4. Configure CloudWatch log groups
5. Test task execution
```

**Handoff**: ECS infrastructure ready for services

---

### Session 6: RDS and Networking
**Repository**: `finks-infrastructure`
**Duration**: 3-4 hours
**Context**: Database and network security

```
Tasks for Claude:
1. Deploy Multi-AZ RDS PostgreSQL
2. Configure security groups:
   - RDS access
   - ECS tasks
   - ALB rules
3. Set up ALB with SSL
4. Create NAT Gateway
5. Test connectivity between components
```

**Handoff**: Secure networking and database ready

---

### Session 7: CI/CD Pipelines
**Repository**: All
**Duration**: 3-4 hours
**Context**: GitHub Actions setup

```
Tasks for Claude:
1. finks-infrastructure:
   - Pulumi preview on PR
   - Deploy to staging on merge
   - Production deploy on tag
2. Other repositories:
   - Test workflows
   - Build and push Docker images
   - Basic deployment workflows
3. Add status badges to READMEs
```

**Handoff**: All repositories have working CI/CD

---

## Week 3: Prefect Deployment (4 Focused Sessions)

### 🟡 Session 8: Prefect Server Deployment (Critical Path)
**Repository**: `finks-infrastructure`
**Estimated Duration**: 6-7 hours (includes debugging)
**Dependencies**: Sessions 5 (ECS) and 6 (RDS) complete
**Difficulty**: ⭐⭐⭐⭐⭐ (Most complex session)

**Context Prime for Claude**:
```
I need you to deploy Prefect Server on ECS Fargate with ALB.
This is complex - Prefect needs specific environment variables and networking.
The server must connect to our existing RDS PostgreSQL.
Focus on getting a minimal working deployment first, then add features.
Expect networking issues - be patient and systematic in debugging.
```

**Required Resources**:
- ECS cluster from Session 5
- RDS instance from Session 6  
- VPC and subnets configured
- ACM certificate for HTTPS

**Specific Tasks**:
```yaml
1. Prefect Server Image Analysis (30 min)
   - Review Prefect Docker image requirements
   - Identify required environment variables
   - Plan resource allocation (CPU/memory)
   
2. Task Definition Creation (60 min)
   - Server container definition
   - Proper environment variables:
     - PREFECT_API_DATABASE_CONNECTION_URL
     - PREFECT_SERVER_API_HOST=0.0.0.0
     - PREFECT_API_URL for agents
   - CloudWatch logging configuration
   - Health check setup

3. ALB Configuration (90 min)
   - Target group with health checks
   - ALB listener rules
   - SSL certificate attachment
   - Security group rules

4. ECS Service Deployment (90 min)
   - Service with desired count=1 initially
   - Network configuration (subnets)
   - Security group assignment
   - Service discovery setup

5. Debugging & Validation (120 min)
   - Check ECS task logs
   - Verify database connectivity
   - Test ALB health checks
   - Access Prefect UI
   - Common issues:
     - Database connection timeouts
     - Security group misconfigurations  
     - Health check failures
     - Environment variable errors

6. Production Readiness (60 min)
   - Auto-scaling configuration
   - Multi-AZ deployment
   - Backup strategy
   - Monitoring setup
```

**Success Criteria**:
- [ ] Prefect UI accessible via HTTPS
- [ ] Can create and run a test flow
- [ ] Server survives ECS task restart
- [ ] Logs visible in CloudWatch
- [ ] Database migrations completed

**Common Pitfalls**:
- Database URL format issues (use asyncpg://)
- Security group doesn't allow ECS→RDS
- Health check path incorrect (/api/health)
- Insufficient CPU/memory allocation
- Missing PREFECT_API_URL for agents

**Debugging Checklist**:
```bash
# 1. Check ECS task status
aws ecs describe-tasks --cluster prefect-cluster --tasks <task-arn>

# 2. View CloudWatch logs
aws logs tail /ecs/prefect-server --follow

# 3. Test database connectivity from ECS task
# Use ECS Exec to connect to container

# 4. Verify security groups
aws ec2 describe-security-groups --group-ids <sg-id>

# 5. Check ALB target health
aws elbv2 describe-target-health --target-group-arn <tg-arn>
```

**Recovery Strategy**:
If Prefect won't start after 2 hours of debugging:
1. Fall back to EC2 deployment temporarily
2. Or use Prefect Cloud for MVP
3. Return to ECS deployment in Phase 2

**Template to Accelerate**:
```python
# Prefect ECS task definition
prefect_task_definition = aws.ecs.TaskDefinition(
    "prefect-server",
    family="prefect-server",
    cpu="1024",
    memory="2048",
    network_mode="awsvpc",
    requires_compatibilities=["FARGATE"],
    execution_role_arn=ecs_execution_role.arn,
    task_role_arn=prefect_task_role.arn,
    container_definitions=json.dumps([{
        "name": "prefect-server",
        "image": "prefecthq/prefect:2-python3.12",
        "command": ["prefect", "server", "start"],
        "environment": [
            {"name": "PREFECT_SERVER_API_HOST", "value": "0.0.0.0"},
            {"name": "PREFECT_SERVER_API_PORT", "value": "4200"},
            {"name": "PREFECT_API_DATABASE_CONNECTION_URL", 
             "value": f"postgresql+asyncpg://prefect:password@{rds_endpoint}/prefect"},
        ],
        "portMappings": [{
            "containerPort": 4200,
            "protocol": "tcp"
        }],
        "healthCheck": {
            "command": ["CMD-SHELL", "curl -f http://localhost:4200/api/health || exit 1"],
            "interval": 30,
            "timeout": 5,
            "retries": 3
        },
        "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
                "awslogs-group": "/ecs/prefect-server",
                "awslogs-region": "ca-central-1",
                "awslogs-stream-prefix": "prefect"
            }
        }
    }])
)
```

**Handoff Documentation**:
```markdown
## Session 8 Handoff
- ✅ Prefect Server running on ECS Fargate
- ✅ Accessible at https://prefect.example.com
- ✅ Connected to RDS PostgreSQL
- ✅ Auto-scaling configured (1-3 tasks)
- ✅ CloudWatch logs working
- 📁 Resources created:
  - ECS Task Definition: prefect-server
  - ECS Service: prefect-server-service
  - ALB: prefect-alb
  - Target Group: prefect-tg
- ⚠️ Known issues:
  - Cold starts take ~2 minutes
  - Need to set PREFECT_API_URL for agents
- 🔗 Next: Session 9 (Prefect Agent)
```

---

### Session 9: Prefect Agent Setup
**Repository**: `finks-infrastructure` + `finks-pipelines`
**Duration**: 3-4 hours
**Context**: Prefect execution layer

```
Tasks for Claude:
1. Deploy Prefect Agent on ECS
2. Configure agent with proper IAM role
3. Create test flow in finks-pipelines
4. Verify agent picks up and executes flow
5. Set up CloudWatch metrics
```

**Handoff**: Prefect can execute flows on ECS

---

### Session 10: Prefect Blocks Configuration
**Repository**: `finks-pipelines`
**Duration**: 2-3 hours
**Context**: Prefect configuration as code

```
Tasks for Claude:
1. Create blocks configuration script:
   - S3 blocks for each zone
   - AWS credentials block
   - ECS task blocks
2. Add blocks deployment to CI/CD
3. Create utility functions for common operations
4. Test all blocks work correctly
```

**Handoff**: Prefect blocks configured and reusable

---

### Session 11: Monitoring Setup
**Repository**: `finks-infrastructure`
**Duration**: 3-4 hours
**Context**: Observability infrastructure

```
Tasks for Claude:
1. Create CloudWatch dashboard for:
   - ECS metrics
   - RDS metrics
   - S3 metrics
2. Set up basic alarms:
   - ECS task failures
   - RDS connection exhaustion
   - High error rates
3. Configure SNS topic for alerts
4. Document monitoring approach
```

**Handoff**: Basic monitoring and alerting active

---

## Week 4: Transformation Infrastructure (3 Parallel Sessions)

### Session 12: dbt on ECS
**Repository**: `finks-dbt` + `finks-infrastructure`
**Duration**: 4-5 hours
**Context**: dbt containerization

```
Tasks for Claude:
1. Create dbt Dockerfile with:
   - dbt-core
   - dbt-athena adapter
   - Python dependencies
2. Create ECS task definition for dbt
3. Set up Athena connection profile
4. Create sample staging model
5. Test dbt run on ECS
```

**Handoff**: dbt can run on ECS against Athena

---

### Session 13: Glue Infrastructure
**Repository**: `finks-infrastructure` + `finks-pipelines`
**Duration**: 3-4 hours
**Context**: AWS Glue setup

```
Tasks for Claude:
1. Create Glue database and tables
2. Write PySpark script for Bronze→Silver
3. Create Glue job definition in Pulumi
4. Add Glue crawler configuration
5. Test job execution
```

**Handoff**: Glue can process Bronze→Silver

---

### Session 14: Data Models Design
**Repository**: `finks-dbt`
**Duration**: 3-4 hours
**Context**: Business logic modeling

```
Tasks for Claude:
1. Design staging models schema
2. Create staging models for FMP data:
   - stg_fmp_quotes
   - stg_fmp_company_profile
3. Create mart model:
   - daily_portfolio_summary
4. Add dbt tests
5. Generate documentation
```

**Handoff**: Core dbt models ready

---

## Week 5: End-to-End Pipeline (4 Integration Sessions)

### Session 15: FMP Ingestion Connector
**Repository**: `finks-ingestion`
**Duration**: 4-5 hours
**Context**: Data extraction

```
Tasks for Claude:
1. Create FMP API connector:
   - Rate limiting
   - Error handling
   - Schema validation
2. Add S3 upload functionality
3. Create comprehensive tests
4. Dockerize the connector
5. Test full extraction flow
```

**Handoff**: Working FMP connector in container

---

### 🟡 Session 16: Orchestration Flows (Integration Point)
**Repository**: `finks-pipelines`  
**Estimated Duration**: 7-8 hours (includes testing)
**Dependencies**: Sessions 10, 14, 15 complete
**Difficulty**: ⭐⭐⭐⭐ (Complex integration)

**Context Prime for Claude**:
```
I need you to create the main Prefect orchestration flows.
This ties together ingestion (ECS), transformation (Glue), and dbt (ECS).
Focus on reliability - use retries, error handling, and notifications.
Make flows idempotent so they can be re-run safely.
This is the heart of our pipeline - take time to get it right.
```

**Required Resources**:
- Prefect Server running (Session 8)
- Prefect blocks configured (Session 10)
- FMP connector image in ECR (Session 15)
- dbt image in ECR (Session 14)
- Glue job created (Session 13)

**Specific Tasks**:
```yaml
1. Base Flow Structure (60 min)
   - Project setup with prefect.yaml
   - Common utilities (logging, errors)
   - Shared configuration
   - Task retry patterns

2. Ingestion Flow (90 min)
   - ECS task execution for FMP
   - S3 upload validation
   - Failure notifications
   - Parameterization for dates
   - Idempotency checks

3. Transformation Flow (90 min)
   - Glue job trigger and monitoring
   - Wait for completion logic
   - Error handling for Glue failures
   - Data quality checks
   - Bronze→Silver validation

4. dbt Flow (90 min)
   - ECS task for dbt run
   - Model selection parameters
   - Test result handling
   - Artifact storage to S3
   - Failure rollback strategy

5. Master Orchestration (120 min)
   - Flow-of-flows pattern
   - Dependency management
   - Conditional execution
   - Comprehensive error handling
   - State management
   - Notifications (success/failure)

6. End-to-End Testing (90 min)
   - Deploy all flows
   - Run with test data
   - Simulate failures
   - Verify recovery
   - Performance testing
```

**Success Criteria**:
- [ ] Ingestion flow completes successfully
- [ ] Transformation flow processes data
- [ ] dbt flow generates Gold data
- [ ] Master flow orchestrates all steps
- [ ] Failures are handled gracefully
- [ ] Flows are idempotent

**Common Pitfalls**:
- ECS task ARN changes between environments
- IAM permissions for cross-service access
- Timeout settings too aggressive
- Missing error handling for partial failures
- State locking issues with concurrent runs

**Testing Scenarios**:
```python
# Test 1: Happy path
python -m pytest tests/test_flows.py::test_happy_path

# Test 2: Ingestion failure
python -m pytest tests/test_flows.py::test_ingestion_failure

# Test 3: Glue job timeout
python -m pytest tests/test_flows.py::test_glue_timeout

# Test 4: dbt test failures
python -m pytest tests/test_flows.py::test_dbt_quality_gate

# Test 5: Concurrent execution
python -m pytest tests/test_flows.py::test_concurrent_runs
```

**Template to Accelerate**:
```python
# flows/master_flow.py
from prefect import flow, task
from prefect_aws.ecs import ECSTask
from prefect_aws.glue import GlueJob
from prefect.deployments import Deployment
from prefect.server.schemas.schedules import CronSchedule
import boto3
from datetime import datetime, timedelta

@task(retries=3, retry_delay_seconds=60)
def run_ingestion(date: datetime) -> str:
    """Run FMP ingestion for specific date."""
    ecs_task = ECSTask.load("fmp-ingestion-task")
    
    # Check if data already exists (idempotency)
    s3 = boto3.client('s3')
    prefix = f"bronze/fmp/date={date.strftime('%Y-%m-%d')}/"
    existing = s3.list_objects_v2(
        Bucket='finks-bronze',
        Prefix=prefix,
        MaxKeys=1
    )
    
    if existing.get('KeyCount', 0) > 0:
        print(f"Data already exists for {date}, skipping ingestion")
        return "skipped"
    
    # Run ingestion
    result = ecs_task.run(
        task_definition_arn="arn:aws:ecs:region:account:task-definition/fmp-ingestion",
        overrides={
            "containerOverrides": [{
                "name": "fmp-connector",
                "environment": [
                    {"name": "EXTRACTION_DATE", "value": date.strftime('%Y-%m-%d')},
                    {"name": "S3_BUCKET", "value": "finks-bronze"}
                ]
            }]
        },
        wait_for_completion=True,
        poll_interval=10
    )
    return "completed"

@task(retries=2, retry_delay_seconds=120)
def run_glue_transformation(date: datetime) -> str:
    """Run Glue job for Bronze to Silver."""
    glue_job = GlueJob.load("bronze-to-silver-job")
    
    run = glue_job.run(
        job_name="bronze-to-silver",
        arguments={
            "--DATE": date.strftime('%Y-%m-%d'),
            "--SOURCE_BUCKET": "finks-bronze",
            "--TARGET_BUCKET": "finks-silver"
        }
    )
    
    # Wait for completion
    run.wait_for_completion()
    
    if run.status != "SUCCEEDED":
        raise Exception(f"Glue job failed: {run.status}")
    
    return "completed"

@task(retries=2, retry_delay_seconds=60)
def run_dbt_models(date: datetime, models: str = None) -> str:
    """Run dbt models on ECS."""
    ecs_task = ECSTask.load("dbt-task")
    
    command = ["dbt", "run"]
    if models:
        command.extend(["--select", models])
    
    result = ecs_task.run(
        task_definition_arn="arn:aws:ecs:region:account:task-definition/dbt-core",
        overrides={
            "containerOverrides": [{
                "name": "dbt",
                "command": command,
                "environment": [
                    {"name": "DBT_VARS", 
                     "value": f'{{"run_date": "{date.strftime("%Y-%m-%d")}"}}'}
                ]
            }]
        },
        wait_for_completion=True
    )
    
    # Run tests
    test_result = ecs_task.run(
        task_definition_arn="arn:aws:ecs:region:account:task-definition/dbt-core",
        overrides={
            "containerOverrides": [{
                "name": "dbt",
                "command": ["dbt", "test"]
            }]
        },
        wait_for_completion=True
    )
    
    return "completed"

@flow(name="daily-data-pipeline", retries=1)
def daily_pipeline(date: datetime = None):
    """Main orchestration flow."""
    if date is None:
        date = datetime.now() - timedelta(days=1)
    
    # Step 1: Ingestion
    ingestion_result = run_ingestion(date)
    
    # Step 2: Transformation (only if ingestion succeeded)
    if ingestion_result == "completed":
        glue_result = run_glue_transformation(date)
        
        # Step 3: dbt (only if Glue succeeded)
        if glue_result == "completed":
            dbt_result = run_dbt_models(date)
    
    return f"Pipeline completed for {date}"

# Create deployment
if __name__ == "__main__":
    deployment = Deployment.build_from_flow(
        flow=daily_pipeline,
        name="daily-pipeline-deployment",
        schedule=CronSchedule(cron="0 2 * * *", timezone="UTC"),
        tags=["production", "daily"]
    )
    deployment.apply()
```

**Handoff Documentation**:
```markdown
## Session 16 Handoff  
- ✅ All Prefect flows created and tested
- ✅ End-to-end pipeline working
- ✅ Deployed with daily schedule
- ✅ Error handling and retries configured
- ✅ Idempotency implemented
- 📁 Flows created:
  - flows/ingestion.py
  - flows/transformation.py  
  - flows/dbt.py
  - flows/master_flow.py
  - tests/test_flows.py
- ⚠️ Configuration needed:
  - Update ECS task ARNs for production
  - Configure SNS topic for notifications
- 🔗 Next: Session 17 (Data Quality)
```

---

### Session 17: Data Quality & Testing
**Repository**: `finks-dbt` + `finks-pipelines`
**Duration**: 3-4 hours
**Context**: Quality assurance

```
Tasks for Claude:
1. Add dbt data tests:
   - Source freshness
   - Referential integrity
   - Business rule validation
2. Create data quality Prefect tasks
3. Add quality gates to flows
4. Create test data generator
5. Run full pipeline test
```

**Handoff**: Pipeline with quality checks

---

### Session 18: Historical Backfill
**Repository**: `finks-pipelines`
**Duration**: 3-4 hours
**Context**: Batch processing

```
Tasks for Claude:
1. Parameterize flows for date ranges
2. Create backfill orchestration flow
3. Add parallel execution logic
4. Test with 30 days of data
5. Monitor resource usage
```

**Handoff**: Ability to process historical data

---

## Week 6: Production Readiness (4 Finalization Sessions)

### Session 19: API Development
**Repository**: `finks-infrastructure`
**Duration**: 4-5 hours
**Context**: Data access layer

```
Tasks for Claude:
1. Create Lambda function for Athena queries
2. Set up API Gateway:
   - REST endpoints
   - Request validation
   - API key authentication
3. Add caching layer
4. Create OpenAPI documentation
5. Test API thoroughly
```

**Handoff**: Working data access API

---

### Session 20: Advanced Monitoring
**Repository**: `finks-infrastructure` + `finks-pipelines`
**Duration**: 3-4 hours
**Context**: Production observability

```
Tasks for Claude:
1. Add custom CloudWatch metrics:
   - Records processed
   - Pipeline latency
   - Data quality scores
2. Create detailed dashboards
3. Set up PagerDuty integration
4. Add distributed tracing
5. Create runbooks
```

**Handoff**: Comprehensive monitoring

---

### Session 21: Security Hardening
**Repository**: All
**Duration**: 4-5 hours
**Context**: Security review

```
Tasks for Claude:
1. Review and tighten IAM policies
2. Enable VPC endpoints
3. Set up AWS GuardDuty
4. Configure AWS Config rules
5. Run security scan
6. Document security posture
```

**Handoff**: Hardened security configuration

---

### Session 22: Documentation & Handoff
**Repository**: All
**Duration**: 3-4 hours
**Context**: Operational readiness

```
Tasks for Claude:
1. Create operational runbooks:
   - Common issues resolution
   - Disaster recovery steps
   - Scaling procedures
2. Generate architecture diagrams
3. Create onboarding guide
4. Document all APIs
5. Create troubleshooting guide
```

**Handoff**: Complete operational documentation

---

## Best Practices for Claude Code Sessions

### 1. Session Preparation
```
Before starting each session:
1. Clear objective and task list
2. Links to relevant documentation
3. Previous session's handoff notes
4. Required credentials in environment
```

### 2. Context Management
```
Help Claude maintain context:
1. Start with: "Continue from Session X handoff"
2. Provide repository structure overview
3. Reference existing code patterns
4. Keep sessions focused on one domain
```

### 3. Testing Strategy
```
Each session should:
1. Write tests before implementation
2. Run tests frequently
3. Document test results
4. Leave passing test suite
```

### 4. Handoff Documentation
```
End each session with:
1. Summary of completed work
2. List of created/modified files
3. Test results
4. Next steps for continuation
5. Any blocking issues
```

### 5. Parallel Work Optimization
```
Week 1-2: Infrastructure sessions can run in parallel
Week 3-4: Repository-specific work in parallel
Week 5-6: Integration sessions need sequencing
```

## Session Scheduling Example

### Week 1 Schedule
- Monday AM: Session 1 (Local Dev)
- Monday PM: Session 2 (Pulumi Structure)
- Tuesday: Session 3 (AWS Components)
- Wednesday AM: Session 4 (Repository Setup)
- Wednesday PM: Review and integration

### Daily Pattern
- Morning: Complex implementation session (4-5 hours)
- Afternoon: Testing/documentation session (2-3 hours)
- End of day: Integration and handoff prep

## Risk Mitigation

### 1. Session Failures
- Each session produces incremental value
- Git commits after each working feature
- Detailed handoff notes for recovery

### 2. Integration Issues
- Dedicated integration sessions
- Test interfaces between components
- Keep sessions loosely coupled

### 3. Context Loss
- Comprehensive documentation in each session
- Use git commits as checkpoints
- Reference previous work explicitly

This approach maximizes Claude Code's effectiveness while maintaining development velocity and code quality.