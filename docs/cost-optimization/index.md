## Cost Optimization Strategy

### Compute Optimization

- **Graviton Processors**: 40% better price-performance
- **Spot Instances**: For non-critical batch jobs
- **Fargate Spot**: For Prefect agents
- **Right-sizing**: Regular analysis of resource usage

### Storage Optimization

- **S3 Intelligent-Tiering**: Automatic tier management
- **Lifecycle Policies**:
  - Bronze: Standard → Standard-IA (30 days) → Glacier (90 days)
  - Silver: Standard → Standard-IA (60 days)
  - Gold: Standard (always hot)
- **Compression**: Parquet with Snappy compression
- **Partitioning**: Reduce data scanned by queries
- **Small File Compaction**: Scheduled Glue jobs merge small files into optimal sizes

### Small File Compaction Strategy

```python
# Prefect flow for periodic file compaction
@flow(name="small-file-compaction")
def compact_silver_zone_files(date: datetime):
    """Merge small Parquet files into larger, optimized files"""

    glue_job = GlueJob.load("silver-compaction-job")

    # Run compaction for previous day's data
    glue_job.run(
        arguments={
            "--partition_date": date.strftime("%Y-%m-%d"),
            "--target_file_size": "512MB",
            "--source_path": f"s3://silver-zone/entity=trades/year={date.year}/month={date.month:02d}/day={date.day:02d}/",
            "--temp_path": "s3://temp-zone/compaction/"
        }
    )

    # Verify compaction results
    validate_compaction_results(date)

# Schedule to run nightly
compact_silver_zone_files_deployment = Deployment.build_from_flow(
    flow=compact_silver_zone_files,
    name="nightly-compaction",
    schedule=CronSchedule(cron="0 2 * * *"),  # 2 AM daily
    parameters={"date": "{{ yesterday_ds }}"}
)
```

### Data Transfer Optimization

- **VPC Endpoints**: Avoid NAT Gateway charges
- **S3 Transfer Acceleration**: For large uploads
- **CloudFront**: For frequently accessed Gold data

### Reserved Capacity

- **RDS Reserved Instances**: For Prefect database
- **Compute Savings Plans**: For predictable ECS usage
