## Manual Tests — `job_type` Field (PR #634)

### Prerequisites

1. Code deployed to target environment.
2. Authenticated via `auth login`.
3. Replace `<YOUR_CWID>` with your Corporate Worker ID.

**Environment**: `https://apis.dse-dev.bayer.com` | Tenant: `tenants/DONOTDELETE`

---

## Test 1: Default `job_type` (omitted)

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/TEST-DEFAULT-JOBTYPE",
        "display_name": "Test Default JobType",
        "description": "Verifies that omitting job_type defaults to STANDARD",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

**Expected**: `200 OK`, `"jobType": "STANDARD"` (defaulted), `modelName` and `versionBump` empty.

**Result**: ✅ Pass

<details><summary>Response</summary>

{"name":"jobs/TEST_DEFAULT_JOBTYPE", "displayName":"Test Default JobType", "description":"Verifies that omitting job_type defaults to STANDARD", "tenantName":"tenants/DONOTDELETE", "status":"ACTIVE", "papi":{"applicationId":"TEST-PAPI-APPLICATION", "readEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE", "writeEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE"}, "ownerPapiGroup":"DSE-DEV-TEAM", "gitSourceUri":"https://github.com/bayer-int/dse-apis", "jobType":"STANDARD", "modelName":"", "versionBump":"VERSION_BUMP_UNSPECIFIED"}

</details>

<img width="1764" height="191" alt="image" src="https://github.com/user-attachments/assets/ac6987a8-7d6f-49b0-97d3-5a2ecd8c3404" />
<img width="1776" height="94" alt="image" src="https://github.com/user-attachments/assets/1597e20a-79c6-416c-a26d-645d6e887797" />

---

## Test 2: Explicit `STANDARD`

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/TEST-STANDARD-JOBTYPE",
        "display_name": "Test Standard JobType",
        "description": "Verifies explicit STANDARD job type",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "STANDARD"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

**Expected**: `200 OK`, `"jobType": "STANDARD"`, `modelName` and `versionBump` empty.

**Result**: ✅ Pass

<details><summary>Response</summary>

{"name":"jobs/TEST_STANDARD_JOBTYPE", "displayName":"Test Standard JobType", "description":"Verifies explicit STANDARD job type", "tenantName":"tenants/DONOTDELETE", "status":"ACTIVE", "papi":{"applicationId":"TEST-PAPI-APPLICATION", "readEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE", "writeEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE"}, "ownerPapiGroup":"DSE-DEV-TEAM", "gitSourceUri":"https://github.com/bayer-int/dse-apis", "jobType":"STANDARD", "modelName":"", "versionBump":"VERSION_BUMP_UNSPECIFIED", "created":{"clientId":"test", "userId":"<YOUR_CWID>", "time":"2026-04-08T13:28:37.623860670Z"}, "updated":null, "deleted":null}

</details>

<img width="1763" height="316" alt="image" src="https://github.com/user-attachments/assets/1a9b6c5a-7cb0-4a61-ac33-ff8f3c88aa63" />
<img width="1793" height="95" alt="image" src="https://github.com/user-attachments/assets/b4bd7421-c44d-4b31-8de1-63a280cd288d" />

---

## Test 3: `TRAINING` with `model_name` and `version_bump`

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/TEST-TRAINING-JOBTYPE",
        "display_name": "Test Training JobType",
        "description": "Verifies TRAINING job type with model and version bump",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "TRAINING",
        "model_name": "models/<MODEL_ID>",
        "version_bump": "MINOR"
    },
    "skip_papi_add_application_entitlement_request": true,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

**Expected**: `200 OK`, `"jobType": "TRAINING"`, `"modelName": "models/..."`, `"versionBump": "MINOR"`.

**Result**: ✅ Pass

<details><summary>Response</summary>

{"name":"jobs/TEST_TRAINING_JOBTYPE", "displayName":"Test Training JobType", "description":"Verifies TRAINING job type with model and version bump", "tenantName":"tenants/DONOTDELETE", "status":"ACTIVE", "papi":{"applicationId":"TEST-PAPI-APPLICATION", "readEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE", "writeEntitlementId":"SKIP-ADD-ENTITLEMENT-WAS-TRUE"}, "ownerPapiGroup":"DSE-DEV-TEAM", "gitSourceUri":"https://github.com/bayer-int/dse-apis", "jobType":"TRAINING", "modelName":"models/5a4fjqijvhj568sj48aj4f9qja84fiqa", "versionBump":"MINOR", "created":{"clientId":"test", "userId":"<YOUR_CWID>", "time":"2026-04-08T13:34:47.351996885Z"}, "updated":null, "deleted":null}

</details>

<img width="1760" height="296" alt="image" src="https://github.com/user-attachments/assets/8cbf35d4-c02c-49d3-992d-277dea678fd9" />
<img width="1758" height="100" alt="image" src="https://github.com/user-attachments/assets/8f33314c-431e-44e8-91f9-aec1dc40c4b1" />

---

## Test 4: E2E — `job_type` reaches Terraform Worker

Full pipeline: Job → Version → Deployment → Approve → AssetStack SQS → dsemgmt → Terraform SQS → Terraform Worker.

### 4a. Create TRAINING Job (with real PAPI)

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job": {
        "name": "jobs/jt-e2e-test",
        "display_name": "JT E2E Test",
        "description": "E2E test: verifies job_type reaches terraform worker",
        "tenant_name": "tenants/DONOTDELETE",
        "git_source_uri": "https://github.com/bayer-int/dse-apis",
        "owner_papi_group": "DSE-DEV-TEAM",
        "job_type": "TRAINING",
        "model_name": "models/<MODEL_ID>",
        "version_bump": "MINOR"
    },
    "skip_papi_add_application_entitlement_request": false,
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

Save the `papi.writeEntitlementId` — you may need to add yourself via the PAPI UI.

<details><summary>Response (nonprod)</summary>

```json
{
    "name": "jobs/np-e2e-training-job",
    "displayName": "NP E2E Training Job",
    "description": "E2E test: verifies job_type fields reach terraform worker",
    "tenantName": "tenants/DONOTDELETE",
    "status": "ACTIVE",
    "papi": {
        "applicationId": "NP-E2E-TRAINING-JOB",
        "readEntitlementId": "02E261D0-4402-11F1-A1F5-1108E1B9CAA7",
        "writeEntitlementId": "06665870-4402-11F1-8415-EF93C2106494"
    },
    "ownerPapiGroup": "DSE-DEV-TEAM",
    "gitSourceUri": "https://github.com/bayer-int/dse-apis",
    "jobType": "TRAINING",
    "modelName": "models/id02010230201",
    "versionBump": "MINOR"
}
```

</details>

### 4b. Create Job Version

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs/jt-e2e-test/job-versions \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job_version": {
        "name": "jobs/jt-e2e-test/job-versions/v0.0.1",
        "parent": "jobs/jt-e2e-test",
        "version_id": "v0.0.1",
        "git_commit_sha": "dd01bbb138272fae585e1f62474aa3cac1c300fb",
        "job_artifact_uri": "<S3_ARTIFACT_URI>"
    },
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

<details><summary>Response (nonprod)</summary>

```json
{
    "name": "jobs/np-e2e-training-job/job-versions/v0.5.0",
    "parent": "jobs/np-e2e-training-job",
    "versionId": "v0.5.0",
    "gitCommitSha": "dd01bbb138272fae585e1f62474aa3cac1c300fb",
    "jobArtifactUri": "s3://dse-jobs-registry-nonprod/jobs/np-e2e-training-job/job-versions/v0.2.0/job.tar.gz",
    "created": {
        "clientId": "test",
        "userId": "LSZFW",
        "time": "2026-04-30T13:34:01.080467249Z"
    }
}
```

</details>

### 4c. Create Deployment

Target a non-sandbox cloud account (e.g., `do-not-delete-jobs` = `2x0OcFGjCJ9hXb4JTCZDzLjNTu1`):

```bash
auth curl -- -X POST https://apis.dse-dev.bayer.com/v1/jobs/jt-e2e-test/job-versions/v0.0.1/job-version-deployments \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "job_version_deployment": {
        "parent": "jobs/jt-e2e-test/job-versions/v0.0.1",
        "target_cloud_account_name": "tenants/DONOTDELETE/cloud-accounts/2x0OcFGjCJ9hXb4JTCZDzLjNTu1",
        "approval_details": { "allowed_respondents": ["<YOUR_CWID>"], "submitter": "<YOUR_CWID>" },
        "display_name": "E2E test deployment",
        "job_config": { "workload_size": 1 }
    },
    "created": { "client_id": "test", "user_id": "<YOUR_CWID>" }
}'
```

Wait for `containerImageStatus: BUILD_SUCCEEDED` before approving.

<details><summary>Response (nonprod)</summary>

```json
{
    "name": "jobs/np-e2e-training-job/job-versions/v0.5.0/job-version-deployments/3D4zIibscvZPmjNXLiYRlT74E3N",
    "parent": "jobs/np-e2e-training-job/job-versions/v0.5.0",
    "targetCloudAccountName": "tenants/DONOTDELETE/cloud-accounts/NONPRODDEPLOYMENT",
    "status": "CREATED",
    "approvalDetails": {
        "allowedRespondents": ["LSZFW"],
        "approvalStatus": "PENDING",
        "submitter": "LSZFW"
    },
    "containerImageStatus": "IMAGE_BUILD_STATUS_UNSPECIFIED",
    "displayName": "E2E retest v0.5.0",
    "jobConfig": { "workloadSize": "XSMALL" }
}
```

</details>

### 4d. Approve

```bash
auth curl -- -X PUT "https://apis.dse-dev.bayer.com/v1/jobs/jt-e2e-test/job-versions/v0.0.1/job-version-deployments/<DEPLOYMENT_KSUID>" \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "name": "jobs/jt-e2e-test/job-versions/v0.0.1/job-version-deployments/<DEPLOYMENT_KSUID>",
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

<details><summary>Response (nonprod)</summary>

```json
{
    "name": "jobs/np-e2e-training-job/job-versions/v0.5.0/job-version-deployments/3D4zIibscvZPmjNXLiYRlT74E3N",
    "status": "CREATED",
    "approvalDetails": {
        "allowedRespondents": ["LSZFW"],
        "actualRespondent": "LSZFW",
        "approvalStatus": "APPROVED",
        "submitter": "LSZFW"
    },
    "containerImageName": "container-images/3D4zIehbBgRRCZYi4e1UoHym8XA",
    "containerImageStatus": "BUILD_SUCCEEDED",
    "displayName": "E2E retest v0.5.0"
}
```

</details>

### 4e. Verify in CloudWatch

**dsemgmt** — look for the terraform message:
```
filter @message like /constructed message for Terraform Worker/ | sort @timestamp desc | limit 50
```

**Terraform Worker** — look for the received payload:
```
filter @message like /payload/ or @message like /message subscribed/ | sort @timestamp desc | limit 50
```

**Expected**: Terraform variables include `job_type: "TRAINING"`, `model_name: "models/..."`, `version_bump: "MINOR"`.

**Result**: ✅ Pass

<details><summary>Terraform Worker payload (nonprod CloudWatch)</summary>

```json
{
    "Type": 1,
    "Message": {
        "AssetStackId": "tenants/DONOTDELETE/cloud-accounts/NONPRODDEPLOYMENT/asset-stack-deployments/3D4zStxFpa94AwIubM675LEKBTt",
        "FunctionalAccountId": "557991640959",
        "FunctionalRegion": "us-east-1",
        "FunctionalEnvironment": "NONPROD",
        "AssetStack": {
            "name": "job_task_v0.0.9",
            "source": "artifactory.bayer.com/dse-terraformmodule-dev-modules__dse-terraform-modules/job_task/aws",
            "inputs": {
                "is_scheduled": "false",
                "job_type": "TRAINING",
                "model_name": "models/id02010230201",
                "project": "DSE Functional Nonprod Deploy",
                "task_cpu": "512",
                "task_image_uri": "557991640959.dkr.ecr.us-east-1.amazonaws.com/np-e2e-training-job:v0.5.0",
                "task_memory": "2048",
                "task_name": "np-e2e-training-job-4496",
                "version_bump": "MINOR"
            },
            "outputs": {
                "job_execution_role": "string",
                "task_definition_arn": "string"
            },
            "version": "v0.0.9",
            "stack": "job_task"
        }
    }
}
```

</details>

---

## Cleanup

```bash
auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST-DEFAULT-JOBTYPE \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ "deleted": { "client_id": "test", "user_id": "<YOUR_CWID>" } }'

auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST-STANDARD-JOBTYPE \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ "deleted": { "client_id": "test", "user_id": "<YOUR_CWID>" } }'

auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/TEST-TRAINING-JOBTYPE \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ "deleted": { "client_id": "test", "user_id": "<YOUR_CWID>" } }'

auth curl -- -X DELETE https://apis.dse-dev.bayer.com/v1/jobs/jt-e2e-test \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{ "deleted": { "client_id": "test", "user_id": "<YOUR_CWID>" } }'
```
