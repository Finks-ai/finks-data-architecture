## Data Lake Architecture

### Medallion Architecture Pattern

The Bronze → Silver → Gold pattern provides clear data quality tiers:

#### Bronze Zone (Raw Data)

- **Purpose**: Preserve raw data exactly as received
- **Format**: Original format (JSON, CSV, API responses)
- **Partitioning**: By source, ingestion time
- **Retention**: Long-term (archive to Glacier after 90 days)
- **Schema**: No schema enforcement

```
s3://bronze-zone/
└── source={source_name}/
    └── year={YYYY}/
        └── month={MM}/
            └── day={DD}/
                └── hour={HH}/
                    └── run_id={prefect_run_id}/
                        └── {timestamp}_{batch_id}.json
```

#### Silver Zone (Cleansed Data)

- **Purpose**: Standardized, deduplicated, validated data
- **Format**: Parquet with compression
- **Partitioning**: By business entity and time
- **Retention**: Medium-term (1-2 years hot storage)
- **Schema**: Enforced via Glue Schema Registry

```
s3://silver-zone/
└── entity={entity_name}/
    └── year={YYYY}/
        └── month={MM}/
            └── day={DD}/
                └── {entity}_{version}.parquet
```

#### Gold Zone (Business Data)

- **Purpose**: Aggregated, business-ready datasets
- **Format**: Parquet optimized for analytics
- **Partitioning**: By use case and access patterns
- **Retention**: Based on business needs
- **Schema**: Documented business glossary

```
s3://gold-zone/
└── domain={business_domain}/
    └── dataset={dataset_name}/
        └── version={schema_version}/
            └── year={YYYY}/
                └── month={MM}/
                    └── {dataset}_{date}.parquet
```
