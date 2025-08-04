## Data Lake Architecture

### Medallion Architecture Pattern

The Bronze → Silver → Gold pattern provides clear data quality tiers:

#### Bronze Zone (Raw Data)

- **Purpose**: Preserve raw data exactly as received
- **Format**: Original format (JSON, CSV, API responses)
- **Partitioning**: By source, ingestion time
- **Retention**: Long-term (archive to Glacier after 90 days)
- **Schema**: No schema enforcement

```mermaid
graph TD
    subgraph Bronze Zone
        direction LR
        A["s3://bronze-zone/"] --> B["source={source_name}/"];
        B --> C["year={YYYY}/"];
        C --> D["month={MM}/"];
        D --> E["day={DD}/"];
        E --> F["hour={HH}/"];
        F --> G["run_id={prefect_run_id}/"];
        G --> H["{timestamp}_{batch_id}.json"];
    end
```

#### Silver Zone (Cleansed Data)

- **Purpose**: Standardized, deduplicated, validated data
- **Format**: Parquet with compression
- **Partitioning**: By business entity and time
- **Retention**: Medium-term (1-2 years hot storage)
- **Schema**: Enforced via Glue Schema Registry

```mermaid
graph TD
    subgraph Silver Zone
        direction LR
        A["s3://silver-zone/"] --> B["entity={entity_name}/"];
        B --> C["year={YYYY}/"];
        C --> D["month={MM}/"];
        D --> E["day={DD}/"];
        E --> F["{entity}_{version}.parquet"];
    end
```

#### Gold Zone (Business Data)

- **Purpose**: Aggregated, business-ready datasets
- **Format**: Parquet optimized for analytics
- **Partitioning**: By use case and access patterns
- **Retention**: Based on business needs
- **Schema**: Documented business glossary

```mermaid
graph TD
    subgraph Gold Zone
        direction LR
        A["s3://gold-zone/"] --> B["domain={business_domain}/"];
        B --> C["dataset={dataset_name}/"];
        C --> D["version={schema_version}/"];
        D --> E["year={YYYY}/"];
        E --> F["month={MM}/"];
        F --> G["{dataset}_{date}.parquet"];
    end
```
