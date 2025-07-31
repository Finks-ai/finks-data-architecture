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
      - "dbt_project/**"

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
