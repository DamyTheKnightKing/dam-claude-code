---
description: Review AWS infrastructure for security, cost optimization, and architecture best practices. Invokes aws-architect agent.
---

Review the AWS infrastructure in this codebase (CDK stacks, CloudFormation, Terraform, or described architecture).

Use the `aws-architect` agent to:

1. Check IAM policies for least-privilege violations (overly broad ARNs, wildcards)
2. Review S3 bucket configurations (encryption, public access, lifecycle rules)
3. Identify missing VPC endpoints, security groups, or network exposure
4. Evaluate service selections against the workload requirements
5. Flag cost optimization opportunities (Spot, Intelligent-Tiering, right-sizing)
6. Verify secrets management (no hardcoded credentials)

Produce findings categorized as: **Security** (immediate risk), **Cost** (financial impact), **Reliability** (availability/DR gaps), **Performance** (throughput/latency issues).
