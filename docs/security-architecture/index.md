## Security Architecture

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
