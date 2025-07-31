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
