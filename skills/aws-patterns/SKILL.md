---
name: aws-patterns
description: AWS cloud patterns for data platforms, serverless, CDK/IaC, IAM, and cost optimization on AWS.
origin: dham-claude-code
---

# AWS Patterns

Use this skill for designing, building, and reviewing AWS infrastructure for data and application workloads.

## Core Data Platform Stack

- **Ingestion**: Kinesis, Glue, DMS, AppFlow
- **Storage**: S3 (data lake), RDS/Aurora, Redshift, DynamoDB
- **Processing**: EMR, Glue ETL, Lambda, Step Functions
- **Serving**: Athena, QuickSight, API Gateway

## S3 Data Lake Layout

```
s3://your-data-lake/
├── bronze/          # raw, partitioned by source and date
│   └── source=salesforce/year=2025/month=03/
├── silver/          # cleaned, partitioned by domain
│   └── domain=sales/year=2025/month=03/
└── gold/            # aggregated, organized by use case
    └── reporting/daily_sales/
```

## CDK Stack Patterns

```python
# Separate stacks per bounded context
class DataIngestionStack(Stack):
    def __init__(self, scope, id, env_name: str, **kwargs):
        super().__init__(scope, id, **kwargs)
        # S3 bucket with lifecycle rules
        bucket = s3.Bucket(self, "RawBucket",
            bucket_name=f"{env_name}-raw-data",
            encryption=s3.BucketEncryption.S3_MANAGED,
            block_public_access=s3.BlockPublicAccess.BLOCK_ALL,
            lifecycle_rules=[
                s3.LifecycleRule(
                    transitions=[s3.Transition(
                        storage_class=s3.StorageClass.INTELLIGENT_TIERING,
                        transition_after=Duration.days(90)
                    )]
                )
            ]
        )

# Cross-stack references via SSM
ssm.StringParameter(self, "BucketArn",
    parameter_name=f"/{env_name}/data/raw-bucket-arn",
    string_value=bucket.bucket_arn
)
```

## IAM Least-Privilege Pattern

```python
# Glue job role — scoped to specific bucket + KMS key
glue_role = iam.Role(self, "GlueRole",
    assumed_by=iam.ServicePrincipal("glue.amazonaws.com")
)
glue_role.add_to_policy(iam.PolicyStatement(
    actions=["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
    resources=[f"{bucket.bucket_arn}/bronze/*"]
))
```

## Step Functions for Pipeline Orchestration

```json
{
  "Comment": "Data pipeline with error handling",
  "StartAt": "ExtractData",
  "States": {
    "ExtractData": {
      "Type": "Task",
      "Resource": "arn:aws:states:::glue:startJobRun.sync",
      "Parameters": { "JobName": "extract-salesforce" },
      "Retry": [{ "ErrorEquals": ["States.ALL"], "MaxAttempts": 3, "IntervalSeconds": 60 }],
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "NotifyFailure" }],
      "Next": "TransformData"
    }
  }
}
```

## Key Decision Matrix

| Workload | Recommended Service |
|----------|-------------------|
| Serverless ETL <15 min | AWS Glue |
| Large Spark jobs | EMR Serverless |
| Event-driven micro-transforms | Lambda |
| Long-running pipelines | Step Functions |
| Ad-hoc SQL on S3 | Athena |
| Structured warehouse | Redshift Serverless |
| Real-time streaming | Kinesis + Lambda or MSK |
