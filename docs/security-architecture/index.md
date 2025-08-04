## Security Architecture

### Network Security

```mermaid
graph TD;
    subgraph "VPC (10.0.0.0/16)";
        subgraph "Public Subnet";
            ALB;
            NAT_Gateway["NAT Gateway"];
        end;
        subgraph "Private Subnet";
            ECS_Tasks["ECS Tasks, RDS, ElastiCache"];
            VPC_Endpoints["VPC Endpoints (S3, DynamoDB)"];
        end;
    end;

    ALB --> ECS_Tasks;
    ECS_Tasks -- "Egress" --> NAT_Gateway;
    ECS_Tasks -- "AWS Services" --> VPC_Endpoints;
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

```mermaid
graph TD
    subgraph AWS Secrets Manager
        A["/prefect/database/connection"];
        B["/mongodb/connection-string"];
        C["/fmp-api/api-key"];
        D["/dbt/profiles/production"];
        E["/monitoring/datadog-api-key"];
    end
```
