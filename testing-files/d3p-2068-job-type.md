# Testing Notes — `job_type` Field (PR #634)

This document describes the manual API tests performed to verify the `job_type` field addition to the Jobs API. Each test creates a job using a different `job_type` configuration and validates the response.

## Prerequisites

1. Code deployed to the target environment (e.g., `dev`).
2. Authenticated via `auth login` (see [csgda-auth](https://github.com/bayer-int/csgda-auth)).
3. Replace `LSZFW` with your Corporate Worker ID.

### Authentication Commands

| Command | Description |
|---------|-------------|
| `auth login` | Interactive browser login (Azure AD). In Codespaces, run `sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open` first. |
| `auth print-access-token` | Prints the raw JWT token to stdout. Use with `curl -H "Authorization: Bearer $(auth print-access-token)"`. |
| `auth curl -- <curl args>` | Wraps `curl` and automatically injects the `Authorization` header. Use `--` to separate auth flags from curl flags. |
| `auth proxy` | Runs a local proxy for Postman. |

## Environment

- **Base URL**: `https://apis.dse-dev.bayer.com`
- **Tenant**: `tenants/DONOTDELETE`
- **PAPI Group**: `DSE-DEV-TEAM`

---

## Test 1: Create Job — Default (no `job_type` provided)

When `job_type` is not included in the request, the API should default it to `STANDARD`.

### 1a. Create

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job": {
        "name": "jobs/TEST_DEFAULT_JOBTYPE",
        "display_name": "Test Default JobType",
        "description": "Verifies that omitting job_type defaults to STANDARD",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

### 1b. Verify (GET)

```bash
auth curl -- -X GET https://apis.dse-dev.bayer.com/v1/jobs/TEST_DEFAULT_JOBTYPE \
  -H "Cwid: LSZFW"
```

### Expected Result

- **Status**: `200 OK` on both Create and GET.
- **Response** should contain `"jobType": "STANDARD"` (defaulted by the server).
- Fields `modelName` and `versionBump` should be empty/absent.

### Result

- **Create**: `200 OK` — response contains `"jobType": "STANDARD"` (defaulted). `modelName` and `versionBump` absent. ✅
- **GET**: `200 OK` — confirmed persisted value matches. ✅

### Screenshot

> _Add screenshot here_

---

## Test 2: Create Job — Explicit `STANDARD`

Explicitly setting `job_type` to `STANDARD` should be accepted and preserved.

### 2a. Create

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job": {
        "name": "jobs/TEST_STANDARD_JOBTYPE",
        "display_name": "Test Standard JobType",
        "description": "Verifies explicit STANDARD job type",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "STANDARD"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

### 2b. Verify (GET)

```bash
auth curl -- -X GET https://apis.dse-dev.bayer.com/v1/jobs/TEST_STANDARD_JOBTYPE \
  -H "Cwid: LSZFW"
```

### Expected Result

- **Status**: `200 OK` on both Create and GET.
- **Response** should contain `"jobType": "STANDARD"`.
- Fields `modelName` and `versionBump` should be empty/absent.

### Result

- **Create**: `200 OK` — response contains `"jobType": "STANDARD"`. `modelName` and `versionBump` absent. ✅
- **GET**: `200 OK` — confirmed persisted value matches. ✅

### Screenshot

> _Add screenshot here_

---

## Test 3: Create Job — `TRAINING`

Setting `job_type` to `TRAINING` requires `model_name` and `version_bump`. All three fields should be stored and returned in the response.

### Pre-step: Find a valid model name

The `model_name` field must reference an existing model in the target environment. List available models first:

```bash
auth curl -- -X GET https://apis.dse-dev.bayer.com/v1/models \
  -H "Cwid: LSZFW"
```

Pick a model `name` from the response (e.g., `models/2NGyxJT9k8L9nMzC`) and use it in the request below.

### 3a. Create

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job": {
        "name": "jobs/TEST_TRAINING_JOBTYPE",
        "display_name": "Test Training JobType",
        "description": "Verifies TRAINING job type with model and version bump",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "TRAINING",
        "model_name": "models/5a4fjqijvhj568sj48aj4f9qja84fiqa",
        "version_bump": "MINOR"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

### 3b. Verify (GET)

```bash
auth curl -- -X GET https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_JOBTYPE \
  -H "Cwid: LSZFW"
```

### Expected Result

- **Status**: `200 OK` on both Create and GET.
- **Response** should contain:
  - `"jobType": "TRAINING"`
  - `"modelName": "models/..."` (the value provided)
  - `"versionBump": "MINOR"`

### Result

- **Create**: `200 OK` — response contains `"jobType": "TRAINING"`, `"modelName": "models/5a4fjqijvhj568sj48aj4f9qja84fiqa"`, `"versionBump": "MINOR"`. ✅
- **GET**: `200 OK` — confirmed all three fields persisted correctly. ✅

### Screenshot

> _Add screenshot here_

---

## Test 4: E2E — Verify `job_type` reaches Terraform Worker

This test validates the full pipeline: Job → JobVersion → JobVersionDeployment → Approve → AssetStack SQS → dsemgmt API → Terraform SQS → Terraform Worker. The goal is to confirm that `job_type`, `model_name`, and `version_bump` are included in the terraform variables payload.

### Prerequisites

- Your CWID must have **PAPI write entitlement** for the job's application. Either:
  - Add yourself to the PAPI entitlement via the PAPI UI, or
  - Deploy the branch to NP and use the DSE UI to approve.
- A valid S3 job artifact URI (reuse one from an existing job version).
- A non-sandbox cloud account (e.g., `do-not-delete-jobs`).

### 4a. Create TRAINING Job (without skip_papi)

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job": {
        "name": "jobs/TEST_TRAINING_E2E",
        "display_name": "Test Training E2E",
        "description": "E2E test: verifies job_type reaches terraform worker",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "TRAINING",
        "model_name": "models/5a4fjqijvhj568sj48aj4f9qja84fiqa",
        "version_bump": "MINOR"
    },
    "skip_papi_add_application_entitlement_request": false,
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

> **Important**: `skip_papi_add_application_entitlement_request` must be `false` so real PAPI entitlements are created. Otherwise the approval step will return 403.

Save the `papi.writeEntitlementId` from the response — you may need to add yourself to this entitlement in the PAPI UI.

### 4b. Create Job Version

Use an existing S3 artifact from another job version. To find one:

```bash
auth curl -- -X GET "https://apis.dse-dev.bayer.com/v1/jobs/5a4fjqijvhj568sj48aj4f9qja84fiqa/job-versions?page_size=1" \
  -H "Cwid: LSZFW"
```

Then create the version:

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E/job-versions \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job_version": {
        "name": "jobs/TEST_TRAINING_E2E/job-versions/v0.0.1",
        "parent": "jobs/TEST_TRAINING_E2E",
        "version_id": "v0.0.1",
        "git_commit_sha": "dd01bbb138272fae585e1f62474aa3cac1c300fb",
        "job_artifact_uri": "s3://dse-jobs-registry-dev/jobs/TREYTESTDEPLOYMENT/job-versions/v0.0.1/job.tar.gz"
    },
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

### 4c. Create Job Version Deployment

Target a **non-sandbox** cloud account. Available NONPROD+READY accounts in dev:

| Display Name | Cloud Account ID |
|---|---|
| `do-not-delete-jobs` | `2x0OcFGjCJ9hXb4JTCZDzLjNTu1` |
| `DEPLOYMENT` | `DEPLOYMENT` |
| `testing-inputs-outputs` | `2x93UNPGbgC4qd6Xu92fieWfHXG` |

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E/job-versions/v0.0.1/job-version-deployments \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "job_version_deployment": {
        "parent": "jobs/TEST_TRAINING_E2E/job-versions/v0.0.1",
        "target_cloud_account_name": "tenants/DONOTDELETE/cloud-accounts/2x0OcFGjCJ9hXb4JTCZDzLjNTu1",
        "approval_details": {
            "allowed_respondents": ["LSZFW"],
            "submitter": "LSZFW"
        },
        "display_name": "E2E test deployment",
        "job_config": {
            "workload_size": 1
        }
    },
    "created": { "client_id": "test", "user_id": "LSZFW" }
}'
```

Save the returned deployment `name` — you need the KSUID (last segment) for the next step.

### 4d. Approve the Deployment

Replace `DEPLOYMENT_KSUID` with the ID from step 4c:

```bash
auth curl -- -X PUT "https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E/job-versions/v0.0.1/job-version-deployments/DEPLOYMENT_KSUID" \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "name": "jobs/TEST_TRAINING_E2E/job-versions/v0.0.1/job-version-deployments/DEPLOYMENT_KSUID",
    "approval_details": {
        "allowed_respondents": ["LSZFW"],
        "actual_respondent": "LSZFW",
        "approval_status": 2,
        "submitter": "LSZFW"
    },
    "skip_create_asset_stack_deployment": false,
    "updated": { "client_id": "test", "user_id": "LSZFW" }
}'
```

> If you get a **403**, add yourself to the PAPI write entitlement for the job's application (see `papi.writeEntitlementId` from step 4a) or ask someone with access in dev chat.

### 4e. Verify in CloudWatch

After approval, the flow is:
1. Jobs API publishes to **AssetStack SQS FIFO queue**
2. AssetStack Worker calls **dsemgmt API** `CreateAssetStackDeployment`
3. dsemgmt API builds terraform message and publishes to **Terraform SQS FIFO queue**
4. Terraform Worker receives the message

**CloudWatch query for dsemgmt API** (look for the terraform message construction):

```
fields @timestamp, data.level, data.msg, data.terraform_queue_message
| filter kubernetes.container_name = "dse-api-dsemgmt"
| filter data.msg = "constructed message for Terraform Worker"
| sort @timestamp desc
| limit 100
```

**CloudWatch query for Terraform Worker** (look for the received payload):

```
fields @timestamp, data.level, data.msg, log
| filter kubernetes.container_name = "dse-terraform-worker"
| filter data.msg like /payload is|message subscribed/
| sort @timestamp desc
| limit 100
```

### Expected Result

- The dsemgmt log `"constructed message for Terraform Worker"` should include terraform values with:
  - `job_type` = `"TRAINING"`
  - `model_name` = `"models/5a4fjqijvhj568sj48aj4f9qja84fiqa"`
  - `version_bump` = `"MINOR"`
- The terraform worker log should show the same values in the received payload.

### Result

> _Pending — blocked on PAPI write entitlement_

### Screenshot

> _Add screenshot here_

---

## Cleanup

After testing, delete the test resources:

```bash
# Delete E2E deployment (replace DEPLOYMENT_KSUID)
auth curl -- -X DELETE "https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E/job-versions/v0.0.1/job-version-deployments/DEPLOYMENT_KSUID" \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'

# Delete E2E job version
auth curl -- -X DELETE "https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E/job-versions/v0.0.1" \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'

# Delete E2E job
auth curl -- -X DELETE "https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_E2E" \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'

# Delete Test 1-3 jobs
auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST_DEFAULT_JOBTYPE \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'

auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST_STANDARD_JOBTYPE \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'

auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST_TRAINING_JOBTYPE \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{ "deleted": { "client_id": "test", "user_id": "LSZFW" } }'
```
