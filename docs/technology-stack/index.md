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
