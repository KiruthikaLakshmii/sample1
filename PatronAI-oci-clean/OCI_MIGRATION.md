# PatronAI — OCI Migration

This branch adds OCI (Oracle Cloud Infrastructure) deployment support.
All existing AWS scripts are untouched. New OCI scripts mirror the AWS equivalents exactly.

## New Files

| File | Purpose | AWS Equivalent |
|------|---------|----------------|
| `deploy_to_oci.sh` | Deploy codebase from Mac → OCI VM | `deploy_to_ec2.sh` |
| `prereqs_oci.sh` | Create OCI resources + generate .env | `prereqs.sh` |
| `oci-policy.json` | OCI IAM policy for PatronAI | `iam-policy.json` |
| `ghost-ai-scanner/scripts/migrate_data.sh` | Migrate S3 + Grafana volume from AWS → OCI | — (new) |

## Migration Flow

```
STEP 1  bash deploy_to_oci.sh
         Copies codebase from Mac → OCI VM
         Installs Docker if needed
         Sets up MCP server + LLM

STEP 2  bash prereqs_oci.sh
         Creates OCI Object Storage bucket
         Generates Customer Secret Keys
         Creates OCI Notifications topic
         Generates .env with OCI values
         SCPs .env → OCI VM

STEP 3  bash ghost-ai-scanner/scripts/migrate_data.sh
         Syncs S3 → OCI Object Storage via rclone
         Migrates grafana-data Docker volume
         Skips LLM model (auto re-downloads)

STEP 4  SSH into OCI VM
         cd ghost-ai-scanner
         bash scripts/start.sh

STEP 5  Verify healthy
         docker ps -a
         curl -k https://<OCI_IP>/
         curl -k https://<OCI_IP>/grafana/

STEP 6  Switch DNS
         patronai.giggso.com A → <OCI_IP>

STEP 7  Monitor 24-48 hours → terminate AWS EC2
```

## AWS → OCI Service Mapping

| AWS | OCI |
|-----|-----|
| EC2 | OCI Compute VM |
| S3 | OCI Object Storage (S3-compatible) |
| IAM user + keys | Customer Secret Keys |
| SNS | OCI Notifications (ONS) |
| SES | OCI Email Delivery (or use webhook) |
| VPC Flow Logs | VCN Flow Logs |

## Requirements

- OCI CLI installed and configured (`oci setup config`)
- rclone installed (for S3 migration)
- SSH key for OCI VM
- OCI Compute VM running (Oracle Linux 8 or Ubuntu 22.04)
- Ports 22, 80, 443 open in OCI Security List

## Git Flow

```bash
# Create branch
git checkout -b feature/oci-migration

# Add files
git add deploy_to_oci.sh prereqs_oci.sh oci-policy.json
git add ghost-ai-scanner/scripts/migrate_data.sh

# Commit
git commit -m "feat: add OCI migration scripts"
git push origin feature/oci-migration

# After migration verified → merge to main
git checkout main
git merge feature/oci-migration
git push origin main
```

---
Giggso Inc x TrinityOps.ai x AIRTaaS
