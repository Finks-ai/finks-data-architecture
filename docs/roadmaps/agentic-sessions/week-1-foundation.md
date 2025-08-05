# Week 1: Foundation (4 Parallel Sessions)

This week focuses on establishing the core development environment and infrastructure foundation. Sessions 1 and 2 can be run in parallel, followed by Session 3, with Session 4 possible at any time after Session 1.

## Session Overview

| Session | Duration | Dependencies | Repository | Focus |
|---------|----------|--------------|------------|-------|
| 1 | 3-4 hours | Session 0 | finks-infrastructure | Local development environment |
| 2 | 4-5 hours | Session 0 | finks-infrastructure | Pulumi project structure |
| 3 | 5-6 hours | Session 2 | finks-infrastructure | Core AWS components |
| 4 | 2-3 hours | Session 1 | All repos | Repository scaffolding |

## 🟡 Session 1: Local Development Environment

**Repository**: `finks-infrastructure`  
**Estimated Duration**: 3-4 hours (includes testing)  
**Dependencies**: Session 0 complete  
**Difficulty**: ⭐⭐

### Context Prime for Claude

```
I need you to create a complete local development environment for the Finks data pipeline.
We're using Docker Compose to simulate AWS services locally with LocalStack.
Focus on developer experience - it should be easy to start/stop/reset the environment.
Include comprehensive error handling and helpful error messages.
```

### Required Resources

- Docker Desktop running
- 8GB RAM available
- Ports 4200 (Prefect), 4566 (LocalStack) free
- uv installed for Python environment management

### Specific Tasks

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
   - config/ directory with .env.dev template

3. Environment Configuration (15 min)
   - Set up config/ directory structure
   - Create .env.dev, .env.staging templates
   - Add environment loader (see [Environment Management](../../environment-management/index.md))

4. Testing Harness (30 min)
   - Test script that validates all services
   - Sample data upload to LocalStack S3
   - Prefect connection test

5. Documentation (30 min)
   - README with troubleshooting guide
   - Architecture diagram
   - Common issues section

6. Integration Test (30 min)
   - Full stack startup
   - Create test Prefect flow
   - Upload to S3 and verify
```

### Success Criteria

- [ ] `make start` brings up all services in <2 minutes
- [ ] `make test` passes all checks
- [ ] Can create and run a Prefect flow locally
- [ ] LocalStack S3 operations work

### Common Pitfalls

- Docker resource limits too low
- Port conflicts with existing services
- LocalStack Pro features attempted (use free tier only)

### Recovery Strategy
If session fails, commit working Docker Compose and continue in next session

### Template to Accelerate

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

### Handoff Documentation

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

## 🟡 Session 2: Pulumi Project Structure

**Repository**: `finks-infrastructure`  
**Estimated Duration**: 4-5 hours (includes testing)  
**Dependencies**: Session 0 complete  
**Difficulty**: ⭐⭐⭐

### Context Prime for Claude

```
I need you to create a well-structured Pulumi project for AWS infrastructure.
Use Python with strong typing and component-based architecture.
Each component should be testable in isolation.
Follow Pulumi best practices for resource naming and organization.
The goal is a maintainable, extensible infrastructure codebase.
```

### Required Resources

- Pulumi CLI installed and logged in
- uv installed with Python 3.12+
- AWS credentials configured

### Specific Tasks

```yaml
1. Project Initialization (30 min)
   - Pulumi new aws-python
   - Configure pyproject.toml with uv
   - Add dependencies (pulumi, pulumi-aws, mypy)
   - Set up type hints and linting

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

### Success Criteria

- [ ] `pulumi preview` runs without errors
- [ ] All components have unit tests
- [ ] Type checking passes with mypy
- [ ] CI workflow triggers on PR

### Common Pitfalls

- Circular dependencies between components
- Missing resource provider registrations
- Incorrect typing for Pulumi outputs

### Template to Accelerate

```toml
# pyproject.toml for finks-infrastructure
[project]
name = "finks-infrastructure"
version = "0.1.0"
description = "Pulumi IaC for Finks data pipeline"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "pulumi>=3.100.0",
    "pulumi-aws>=6.0.0",
    "pulumi-docker>=4.0.0",
]

[project.optional-dependencies]
dev = [
    "mypy>=1.0.0",
    "pytest>=7.0.0",
    "pytest-cov>=4.0.0",
    "ruff>=0.1.0",
    "black>=23.0.0",
]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.mypy]
python_version = "3.12"
strict = true
```

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

### Handoff Documentation

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

## 🟡 Session 3: Core AWS Components

**Repository**: `finks-infrastructure`  
**Estimated Duration**: 5-6 hours (includes testing and deployment)  
**Dependencies**: Session 2 complete  
**Difficulty**: ⭐⭐⭐

### Context Prime for Claude

```
I need you to implement the core AWS infrastructure using the Pulumi components from Session 2.
Focus on VPC with proper networking, S3 buckets with lifecycle policies, and IAM roles.
Deploy to the dev environment and validate everything works.
This forms the foundation for all other services.
```

### Required Resources

- Pulumi components from Session 2
- AWS account with appropriate permissions
- Dev environment configuration

### Specific Tasks

```yaml
1. VPC Implementation (90 min)
   - VPC with 10.0.0.0/16 CIDR
   - 2 public subnets (different AZs)
   - 2 private subnets (different AZs)
   - Internet Gateway
   - NAT Gateway (1 for dev, 2 for prod)
   - Route tables and associations

2. S3 Buckets (60 min)
   - Bronze bucket with raw data settings
   - Silver bucket with lifecycle policies
   - Gold bucket optimized for queries
   - Logging bucket for audit trails
   - Proper bucket policies and CORS

3. Secrets Manager (30 min)
   - Create secret for FMP API key
   - Create secret for database credentials
   - Set up rotation policies
   - IAM policies for access

4. IAM Roles (60 min)
   - ECS task execution role
   - ECS task role (for apps)
   - Lambda execution role
   - Glue job role
   - Cross-service trust policies

5. Deployment & Validation (60 min)
   - Deploy to dev environment
   - Validate all resources created
   - Test S3 access
   - Document outputs
```

### Success Criteria

- [ ] VPC with working public/private subnets
- [ ] All S3 buckets created with correct policies
- [ ] Secrets Manager storing credentials
- [ ] IAM roles with least privilege
- [ ] Successfully deployed to AWS dev

### Common Pitfalls

- NAT Gateway in wrong subnet (must be public)
- S3 bucket names not globally unique
- IAM policies too permissive
- Missing resource tags

### Validation Commands

```bash
# Verify VPC
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=finks-dev-vpc"

# Check subnets
aws ec2 describe-subnets --filters "Name=vpc-id,Values=<vpc-id>"

# List S3 buckets
aws s3 ls | grep finks

# Verify secrets
aws secretsmanager list-secrets | grep finks
```

### Handoff Documentation

```markdown
## Session 3 Handoff
- ✅ VPC deployed with proper networking
- ✅ S3 buckets created (bronze, silver, gold, logs)
- ✅ Secrets Manager configured
- ✅ IAM roles created
- 📊 Key Outputs:
  - VPC ID: vpc-xxxxxxxxx
  - Bronze Bucket: finks-dev-bronze-xxxxx
  - Silver Bucket: finks-dev-silver-xxxxx
  - Gold Bucket: finks-dev-gold-xxxxx
- 🔗 Next: Session 5 (ECS) and 6 (RDS) can proceed
```

---

## 🟡 Session 4: Repository Scaffolding

**Repository**: All repositories  
**Estimated Duration**: 2-3 hours  
**Dependencies**: Session 1 complete  
**Difficulty**: ⭐⭐

### Context Prime for Claude

```
I need you to set up the initial structure for our three application repositories:
- finks-pipelines (Prefect flows)
- finks-dbt (transformation models)
- finks-ingestion (data connectors)
Each should have proper Python project structure, testing setup, and Docker configuration.
```

### Specific Tasks

```yaml
1. finks-pipelines Setup (45 min)
   - Prefect project structure
   - flows/ directory organization
   - tests/ with pytest setup
   - Docker configuration
   - pyproject.toml with uv
   - Basic flow template

2. finks-dbt Setup (45 min)
   - dbt project init
   - Medallion architecture folders
   - Docker setup for local dev
   - profiles.yml template
   - Basic model examples
   - Testing structure

3. finks-ingestion Setup (45 min)
   - Connector architecture
   - Shared utilities module
   - Docker template for connectors
   - Testing framework
   - Error handling patterns
   - S3 upload utilities

4. CI/CD Templates (30 min)
   - GitHub Actions for each repo
   - Testing workflows
   - Docker build workflows
   - Linting and formatting
```

### Repository Structure Templates

#### finks-pipelines

```
finks-pipelines/
├── flows/
│   ├── __init__.py
│   ├── ingestion/
│   │   ├── __init__.py
│   │   └── base_ingestion.py
│   ├── transformation/
│   │   ├── __init__.py
│   │   └── base_transformation.py
│   └── master/
│       ├── __init__.py
│       └── daily_pipeline.py
├── tasks/
│   ├── __init__.py
│   └── common.py
├── blocks/
│   └── configure_blocks.py
├── tests/
│   ├── __init__.py
│   └── test_flows.py
├── Dockerfile
├── pyproject.toml
├── .python-version
└── README.md
```

#### finks-dbt

```
finks-dbt/
├── models/
│   ├── staging/
│   │   ├── fmp/
│   │   └── _sources.yml
│   ├── intermediate/
│   └── marts/
│       ├── finance/
│       └── _models.yml
├── tests/
├── macros/
├── seeds/
├── dbt_project.yml
├── profiles.yml
├── Dockerfile
└── README.md
```

#### finks-ingestion

```
finks-ingestion/
├── connectors/
│   ├── __init__.py
│   ├── base.py
│   └── fmp/
│       ├── __init__.py
│       ├── connector.py
│       └── schemas.py
├── utils/
│   ├── __init__.py
│   ├── s3.py
│   └── validation.py
├── tests/
│   ├── __init__.py
│   └── test_connectors.py
├── Dockerfile.template
├── pyproject.toml
└── README.md
```

### Success Criteria

- [ ] All repos have consistent structure
- [ ] Python projects configured with uv and pyproject.toml
- [ ] Docker files ready for use
- [ ] CI/CD workflows in place
- [ ] READMEs with setup instructions

### pyproject.toml Templates

#### finks-pipelines
```toml
[project]
name = "finks-pipelines"
version = "0.1.0"
description = "Prefect orchestration flows for Finks data pipeline"
requires-python = ">=3.12"
dependencies = [
    "prefect>=2.14.0",
    "prefect-aws>=0.4.0",
    "boto3>=1.28.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.21.0",
    "ruff>=0.1.0",
    "black>=23.0.0",
]
```

#### finks-dbt
```toml
[project]
name = "finks-dbt"
version = "0.1.0"
description = "dbt transformation models for Finks data pipeline"
requires-python = ">=3.12"
dependencies = [
    "dbt-core>=1.7.0",
    "dbt-athena-community>=1.7.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "ruff>=0.1.0",
    "sqlfluff>=3.0.0",
]
```

#### finks-ingestion
```toml
[project]
name = "finks-ingestion"
version = "0.1.0"
description = "Data source connectors for Finks pipeline"
requires-python = ">=3.12"
dependencies = [
    "boto3>=1.28.0",
    "requests>=2.31.0",
    "pydantic>=2.0.0",
    "pandas>=2.0.0",
    "pyarrow>=14.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-mock>=3.0.0",
    "ruff>=0.1.0",
    "black>=23.0.0",
]
```

### Handoff Documentation

```markdown
## Session 4 Handoff
- ✅ finks-pipelines repository structured
- ✅ finks-dbt repository initialized
- ✅ finks-ingestion repository ready
- ✅ All repos have CI/CD workflows
- 📁 Ready for development:
  - Prefect flows can be created
  - dbt models can be added
  - Connectors can be implemented
- 🔗 Next: Any Week 2 session can proceed
```

---

## Week 1 Validation Gate

Before proceeding to Week 2, ensure:

### Infrastructure

- [ ] Local development environment fully functional
- [ ] Pulumi project structured and tested
- [ ] Core AWS resources deployed to dev
- [ ] All repositories initialized

### Testing

- [ ] Can run Prefect flows locally
- [ ] Can deploy infrastructure changes
- [ ] All unit tests passing
- [ ] CI/CD pipelines working

### Documentation

- [ ] All handoffs documented
- [ ] READMEs updated
- [ ] Issues tracked
- [ ] Next steps clear

### Ready for Week 2?
If all items are checked, proceed to [Week 2: Infrastructure Completion](./week-2-infrastructure.md)