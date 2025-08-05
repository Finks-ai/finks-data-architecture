# Week 0: Environment Preparation

This preparation phase must be completed before starting any Claude Code sessions. It involves setting up your local environment, AWS credentials, and creating the necessary repositories.

## 🔴 Session 0: Environment and Credentials Setup

**Duration**: 1-2 hours (human time, not Claude time)  
**When**: Before any other session  
**Type**: Manual setup (no Claude needed)

### Prerequisites Checklist

Before starting, ensure you have:

- [ ] AWS account with appropriate permissions
- [ ] GitHub account with ability to create repositories
- [ ] macOS or Linux development machine
- [ ] At least 16GB RAM for local development
- [ ] Stable internet connection

### Step 1: Install Required Tools

```bash
# For macOS users
brew install pulumi
brew install awscli
brew install docker
brew install git
brew install jq
brew install make

# Install uv for Python environment management
curl -LsSf https://astral.sh/uv/install.sh | sh

# For Linux users
# Follow official installation guides for each tool
```

### Step 2: Configure AWS Credentials

```bash
# Option 1: Using AWS CLI
aws configure
# Enter your:
# - AWS Access Key ID
# - AWS Secret Access Key  
# - Default region: ca-central-1
# - Default output format: json

# Option 2: Using environment variables
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_DEFAULT_REGION="ca-central-1"

# Verify AWS access
aws sts get-caller-identity
```

### Step 3: Set Up Pulumi

```bash
# Create S3 bucket for Pulumi state (if not using Pulumi Cloud)
aws s3 mb s3://finks-pulumi-state-$(aws sts get-caller-identity --query Account --output text)

# Configure Pulumi to use S3 backend
pulumi login s3://finks-pulumi-state-$(aws sts get-caller-identity --query Account --output text)

# Verify Pulumi installation
pulumi version
```

### Step 4: Create GitHub Repositories

Create these empty repositories in your GitHub account:

#### 1. finks-infrastructure
- **Description**: "Pulumi IaC for Finks data pipeline AWS resources"
- **Visibility**: Private repository
- **Initialize with README**: No
- **Add .gitignore**: No
- **Choose license**: No

#### 2. finks-pipelines
- **Description**: "Prefect orchestration flows for Finks data pipeline"
- **Visibility**: Private repository
- **Initialize with README**: No

#### 3. finks-dbt
- **Description**: "dbt transformation models for Finks data pipeline"
- **Visibility**: Private repository
- **Initialize with README**: No

#### 4. finks-ingestion
- **Description**: "Data source connectors for Finks data pipeline"
- **Visibility**: Private repository
- **Initialize with README**: No

### Step 5: Clone Repositories Locally

```bash
# Create workspace directory
mkdir -p ~/workspace/finks
cd ~/workspace/finks

# Clone all repositories
git clone git@github.com:YOUR_USERNAME/finks-infrastructure.git
git clone git@github.com:YOUR_USERNAME/finks-pipelines.git
git clone git@github.com:YOUR_USERNAME/finks-dbt.git
git clone git@github.com:YOUR_USERNAME/finks-ingestion.git
```

### Step 6: Prepare API Keys and Secrets

```bash
# Financial Modeling Prep API key
export FMP_API_KEY="your-fmp-api-key"

# Create a secure file for secrets (don't commit this!)
# This is a temporary global env file - each repo will have its own later
cat > ~/workspace/finks/.env << EOF
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_DEFAULT_REGION=ca-central-1
FMP_API_KEY=your-fmp-api-key
PULUMI_BACKEND_URL=s3://finks-pulumi-state-YOUR_ACCOUNT_ID
ENVIRONMENT=dev
EOF

# Secure the file
chmod 600 ~/workspace/finks/.env
```

### Step 7: Configure Docker

```bash
# Start Docker Desktop (macOS)
open -a Docker

# Verify Docker is running
docker ps

# Configure Docker resources (via Docker Desktop preferences)
# - CPUs: 4+
# - Memory: 8GB+
# - Disk: 50GB+
```

### Step 8: Set Up Python Environment

```bash
# Verify uv is installed
uv --version

# Create a new Python project in each repository
cd ~/workspace/finks/finks-infrastructure

# Initialize Python project with uv
uv init --python 3.12

# This creates a pyproject.toml file with Python 3.12
# The pyproject.toml will be customized for each repository

# Verify Python version
uv run python --version  # Should show 3.12.x
```

### Step 9: Prepare Claude Code Context

Create a context file for Claude Code sessions:

```bash
cat > ~/workspace/finks/CONTEXT.md << 'EOF'
# Finks Data Pipeline Context

## Project Overview
Building a financial data pipeline with:
- Prefect for orchestration
- AWS for infrastructure
- dbt for transformations
- Docker for containerization

## Repository Structure
- finks-infrastructure: Pulumi IaC
- finks-pipelines: Prefect flows
- finks-dbt: dbt models
- finks-ingestion: Data connectors

## AWS Region
ca-central-1

## Key Technologies
- Python 3.12 (with uv package manager)
- Pulumi (Python)
- Prefect 2.x
- dbt Core
- AWS ECS Fargate
- AWS S3, Glue, Athena

## Python Environment
Using uv for fast, reliable Python package management.
All projects use pyproject.toml for dependency management.
EOF
```

### Step 10: Validation Checklist

Run these commands to verify everything is set up correctly:

```bash
# Check AWS access
aws s3 ls  # Should list your S3 buckets

# Check Pulumi
pulumi whoami  # Should show your identity

# Check Docker
docker run hello-world  # Should print success message

# Check Python with uv
uv run python -c "import sys; print(sys.version)"  # Should show 3.12.x

# Check Git
git config --global user.name  # Should show your name
git config --global user.email  # Should show your email

# Check repositories
cd ~/workspace/finks
ls -la  # Should show all 4 repositories
```

## ✅ Success Criteria

You're ready to proceed to Week 1 when:

- [ ] All tools are installed and working
- [ ] AWS credentials are configured and tested
- [ ] Pulumi backend is configured
- [ ] All 4 GitHub repositories are created and cloned
- [ ] Docker is running with adequate resources
- [ ] uv is installed and Python 3.12 environment is set up
- [ ] All validation commands pass

## 🚨 Common Issues and Solutions

### Issue: AWS credentials not working
```bash
# Check credentials
aws configure list

# Test with a simple command
aws sts get-caller-identity

# If using SSO
aws sso login
```

### Issue: Docker not starting
```bash
# macOS: Reset Docker Desktop
killall Docker && open -a Docker

# Check Docker daemon
docker version
```

### Issue: Pulumi state bucket access denied
```bash
# Check bucket exists
aws s3 ls s3://finks-pulumi-state-YOUR_ACCOUNT_ID

# Check bucket policy
aws s3api get-bucket-policy --bucket finks-pulumi-state-YOUR_ACCOUNT_ID
```

## 📋 Next Steps

Once everything is validated:
1. Review the [Week 1 Foundation sessions](./week-1-foundation.md)
2. Prepare for Session 1 (Local Development Environment)
3. Ensure Docker is running before starting Session 1
4. Have this checklist handy for reference

## 💾 Backup Your Setup

Before proceeding, backup your configuration:

```bash
# Create backup of configurations
cd ~/workspace/finks
tar -czf ../finks-setup-backup.tar.gz .env CONTEXT.md

# Store securely (not in git!)
```

---

**Ready to start?** Proceed to [Week 1: Foundation Sessions](./week-1-foundation.md)