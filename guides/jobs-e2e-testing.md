# Jobs E2E Testing Guide

Step-by-step guide for manually testing the full Jobs deployment pipeline end-to-end. Use this whenever a code change touches the Jobs API, dsemgmt, assetstack worker, or terraform worker and you need to verify the complete flow.

**Pipeline flow**:

```
Jobs API → AssetStack SQS → AssetStack Worker → dsemgmt API → Terraform SQS → Terraform Worker
```

---

## Prerequisites

1. **Auth**: Authenticated via `auth login`. If in Codespaces, run `sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open` first.
2. **AWS SSO**: Logged in via `aws sso login --profile nonprod` (or the target environment profile).
3. **Code deployed**: Your branch must be deployed to the target environment. Merging to `main` auto-deploys to nonprod; use `make deploy env=dev` or the GitHub Actions manual deploy for other environments.
4. **PAPI access**: You must be a member of the `DSE-DEV-TEAM` PAPI group (or the relevant group for the tenant).
5. **Existing test job**: Use an existing job (e.g., `np-e2e-training-job`) or create a new one (Step 1).

### Environment URLs

| Environment | Base URL |
|-------------|----------|
| Dev | `https://apis.dse-dev.bayer.com` |
| NonProd | `https://apis.dse-np.bayer.com` |
| Prod | `https://apis.dse.bayer.com` |

### Test Tenant and Cloud Accounts

| Resource | Value |
|----------|-------|
| Tenant | `tenants/DONOTDELETE` |
| Cloud Account (nonprod deploy) | `tenants/DONOTDELETE/cloud-accounts/NONPRODDEPLOYMENT` |
| Cloud Account KSUID (do-not-delete-jobs) | `2x0OcFGjCJ9hXb4JTCZDzLjNTu1` |

> **Important**: For E2E testing that triggers terraform, you must use a **non-sandbox** cloud account. Sandbox accounts may not have the full assetstack infrastructure.

---

## Step 1: Create Job (with real PAPI)

For E2E testing, always use `skip_papi_add_application_entitlement_request: false` so real PAPI entitlements are created. If you use `skip_papi: true`, the approval step will fail with 403 because PAPI validation always runs on `UpdateJobVersionDeployment`.

```bash
auth curl -- -X POST <BASE_URL>/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/<JOB_NAME>",
        "display_name": "<Display Name>",
        "description": "<Description>",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/<YOUR_REPO>",
        "owner_papi_group": "DSE-DEV-TEAM"
    },
    "skip_papi_add_application_entitlement_request": false,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

**Important naming rules**:
- Job names must use **hyphens**, not underscores (e.g., `my-test-job`, NOT `MY_TEST_JOB`). Underscores pass API validation but cause imgbuild worker failures downstream.
- Save the `papi.writeEntitlementId` from the response — you may need to add yourself to the write entitlement via the PAPI UI.

### Verify

```bash
auth curl -- -X GET <BASE_URL>/v1/jobs/<JOB_NAME> -H "Cwid: <YOUR_CWID>"
```

Confirm the job exists with status `ACTIVE` and all fields you sent are persisted.

---

## Step 2: Create Job Version

You need a valid `job_artifact_uri` pointing to an S3 artifact in the **same environment's** registry. Do NOT reuse artifact URIs from other environments — the Dockerfile references base images that may not exist in the target ECR.

> **Tip**: If you already have a job with previous versions, you can reuse the same `job_artifact_uri` from an earlier version for testing purposes.

```bash
auth curl -- -X POST <BASE_URL>/v1/jobs/<JOB_NAME>/job-versions \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job_version": {
        "name": "jobs/<JOB_NAME>/job-versions/<VERSION>",
        "parent": "jobs/<JOB_NAME>",
        "version_id": "<VERSION>",
        "git_commit_sha": "<COMMIT_SHA>",
        "job_artifact_uri": "<S3_URI>"
    },
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

---

## Step 3: Create Deployment

```bash
auth curl -- -X POST <BASE_URL>/v1/jobs/<JOB_NAME>/job-versions/<VERSION>/job-version-deployments \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job_version_deployment": {
        "parent": "jobs/<JOB_NAME>/job-versions/<VERSION>",
        "target_cloud_account_name": "tenants/DONOTDELETE/cloud-accounts/<CLOUD_ACCOUNT_KSUID>",
        "approval_details": {
            "allowed_respondents": ["<YOUR_CWID>"],
            "submitter": "<YOUR_CWID>"
        },
        "display_name": "<Deployment description>",
        "job_config": { "workload_size": 1 }
    },
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

Save the deployment KSUID from the response `name` field.

---

## Step 4: Wait for Image Build

After creating the deployment, the imgbuild worker picks up the request and builds the container image. You must wait until `containerImageStatus` becomes `BUILD_SUCCEEDED` before approving.

### Poll the deployment status

```bash
auth curl -- -X GET "<BASE_URL>/v1/jobs/<JOB_NAME>/job-versions/<VERSION>/job-version-deployments/<DEPLOYMENT_KSUID>" \
  -H "Cwid: <YOUR_CWID>"
```

Look for `"containerImageStatus": "BUILD_SUCCEEDED"`. This typically takes 30-90 seconds.

### If build fails

Check imgbuild worker logs:

```bash
aws logs start-query --profile nonprod --region us-east-1 \
  --log-group-name "/platform-nonprod-use1-eks/imgbuild" \
  --start-time $(date -d '10 minutes ago' +%s) --end-time $(date +%s) \
  --query-string 'filter @message like /<JOB_NAME>/ | sort @timestamp desc | limit 20'
```

Common failures: missing base image in ECR (cross-env artifact mismatch), underscores in job name.

---

## Step 5: Approve Deployment

Once `BUILD_SUCCEEDED`, approve the deployment to trigger the assetstack + terraform pipeline:

```bash
auth curl -- -X PUT "<BASE_URL>/v1/jobs/<JOB_NAME>/job-versions/<VERSION>/job-version-deployments/<DEPLOYMENT_KSUID>" \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "name": "jobs/<JOB_NAME>/job-versions/<VERSION>/job-version-deployments/<DEPLOYMENT_KSUID>",
    "approval_details": {
        "allowed_respondents": ["<YOUR_CWID>"],
        "actual_respondent": "<YOUR_CWID>",
        "approval_status": 2,
        "submitter": "<YOUR_CWID>"
    },
    "skip_create_asset_stack_deployment": false,
    "updated": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

> `approval_status: 2` = APPROVED. After approval, the Jobs API sends a message to the AssetStack SQS queue.

---

## Step 6: Verify in CloudWatch

After approval, the message flows through the pipeline. Wait 1-2 minutes, then check the logs at each stage.

### 6a. dsemgmt API — Constructed Terraform Message

```bash
aws logs start-query --profile nonprod --region us-east-1 \
  --log-group-name "/platform-nonprod-use1-eks/dsemgmt" \
  --start-time $(date -d '10 minutes ago' +%s) --end-time $(date +%s) \
  --query-string 'filter @message like /constructed message/ and @message like /<JOB_NAME>/ | sort @timestamp desc | limit 5'
```

Look for `"constructed message PAYLOAD for Terraform Worker"` — this shows the full JSON payload sent to the terraform queue, including all job inputs.

### 6b. Terraform Worker — Received Payload

```bash
aws logs start-query --profile nonprod --region us-east-1 \
  --log-group-name "/platform-nonprod-use1-eks/workers" \
  --start-time $(date -d '10 minutes ago' +%s) --end-time $(date +%s) \
  --query-string 'filter @message like /message subscribed/ and @message like /<JOB_NAME>/ | sort @timestamp desc | limit 5'
```

Look for the `"message subscribed"` log entry — this contains the full SQS message body with all terraform inputs.

### Getting query results

```bash
# Wait 3-5 seconds after start-query, then:
aws logs get-query-results --profile nonprod --region us-east-1 \
  --query-id "<QUERY_ID>"
```

### What to verify

In the terraform worker payload, confirm:
- **AssetStack name**: `job_task_v<VERSION>` matches your expected svcconfig version.
- **Inputs**: All expected terraform variables are present (e.g., `task_name`, `task_image_uri`, and any custom fields you're testing).
- **Source**: Points to the correct Artifactory module path.

> **Note**: Logs in nonprod are JSON-wrapped inside a `log` field. Do NOT use `data.msg` or `kubernetes.container_name` in CloudWatch queries — use `@message like /keyword/` instead. See [common-jobs-issues.md](../troubleshooting/common-jobs-issues.md#7-cloudwatch-log-format-nonprod) for details.

---

## Step 7: Cleanup

Delete the test job when done:

```bash
auth curl -- -X DELETE <BASE_URL>/v1/jobs/<JOB_NAME> \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ "deleted": { "client_id": "test", "user_id": "<YOUR_CWID>" } }'
```

> Deleting a job soft-deletes it (sets audit `Deleted` field). Deployments and versions are retained in the database.

---

## Quick Tests (API-only, no pipeline)

For changes that only affect the Jobs API layer (new fields, validation, defaults), you can use `skip_papi: true` to quickly create/verify jobs without triggering the full pipeline:

```bash
auth curl -- -X POST <BASE_URL>/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/<TEST_NAME>",
        "display_name": "<Display Name>",
        "description": "<Description>",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

This is faster because it skips PAPI registration, but the job **cannot** be approved for deployment (the PAPI entitlements are fake).

---

## Cross-Repository Dependencies

The Jobs deployment pipeline depends on configuration in **multiple repositories**. When adding new terraform variables to jobs, **all three** must be coordinated:

### 1. `dse-apis` — Job code and svcconfig

- **Terraform input mapping**: `internal/jobs/job_version_deployments.go` — the code that maps job fields to terraform `inputs` map.
- **AssetStack version reference**: `internal/svcconfig/svcconfig.go` — the `AssetStackJobTaskVersion` field controls which assetstack version (e.g., `v0.0.9`) the deployment uses.
- **Helm values**: `deploy/values/{dev,nonprod,prod}/values.yaml` — the `asset_stack_job_task_version` field must match.

### 2. `dse-assetstacks` — AssetStack metadata

- **Input definitions**: `job_task/assetstack.yaml` — lists all accepted input variable names. If a new terraform variable is not listed here, dsemgmt will **reject** it with `"input <name> is not in the asset stack inputs map"`.
- The assetstack version in `assetstack.yaml` must match what `svcconfig.go` references.

### 3. `dse-terraform-modules` — Terraform module

- **Variable definitions**: `job_task/aws/variables.tf` — declares the actual terraform variables. If the variable exists in `assetstack.yaml` but not in `variables.tf`, terraform apply will fail.

### Coordination checklist for new terraform variables

1. Add the variable to `variables.tf` in `dse-terraform-modules`.
2. Add the input to `assetstack.yaml` in `dse-assetstacks` and bump the version.
3. Update `svcconfig.go` and Helm values in `dse-apis` to reference the new assetstack version.
4. Add the mapping in `job_version_deployments.go` to pass the field value as a terraform input.

> **Lesson learned**: If you add the variable to `variables.tf` and `dse-apis` but forget `assetstack.yaml`, the deployment will pass imgbuild but fail at dsemgmt with a validation error. The dsemgmt API validates all inputs against the assetstack's declared input list before forwarding to the terraform queue.

---

## Troubleshooting

See [common-jobs-issues.md](../troubleshooting/common-jobs-issues.md) for known issues including:
- CREST entitlement mismatches (403 on approve)
- Job name underscore restrictions
- `skip_papi` limitations
- Orphaned PAPI applications
- CloudWatch log format (nonprod)
- Cross-environment artifact mismatches
