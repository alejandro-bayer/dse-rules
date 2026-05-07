# Common Jobs Issues & Troubleshooting

Known issues encountered while working with the DSE Jobs API and related infrastructure. For authorization architecture details, see [CREST Authorization Guide](../guides/crest_authorization.md).

---

## 1. CREST Entitlement Identifier Mismatch (403 on Update/Approve)

**Symptom**: Job deployment approval (`UpdateJobVersionDeployment`) returns **403 Forbidden**, even when the user has the correct PAPI entitlements.

**Root Cause**: The entitlement identifier string in CREST did not match what the codebase policies expected. For example:
- CREST had: `dse.api.data.jobs.update`
- Codebase expected: `dse.api.jobs.update`

The identifier is arbitrary text, but it **must match exactly** between CREST and the policy definitions in the deploy templates (`deploy/<service>/templates/`).

**How the auth flow works**:
1. Both the DSE portal (frontend) and the API (backend) use the **same app client** per environment.
2. CREST checks at the **gateway level**: "can this client access this endpoint?" — this uses the entitlement identifiers.
3. The API then does **additional PAPI checks**: "does this specific user belong to the PAPI group with the custom write entitlement for this job?"
4. If the CREST identifier doesn't match the policy, the gateway rejects the request before PAPI checks even happen.

**Portal and curl are affected equally**: Since the portal and curl use the same app client, a CREST mismatch blocks **both**. If the portal works but curl doesn't, the issue is likely PAPI (Layer 3), not CREST (Layer 1).

**Resolution**: Fix the entitlement identifier in CREST (via the Velocity UI at `velocity-np.ag/profile/applications/DSE-APIS/configuration/entitlements`) to match the codebase policy definitions. Must be done per environment (dev, nonprod, prod).

**Where to check**:
- CREST entitlements: `https://velocity-np.ag/profile/applications/DSE-APIS/configuration/entitlements`
- Codebase policies: `deploy/dse-api-jobs/templates/` (look for entitlement identifier strings)

---

## 2. Job Names Must Not Contain Underscores

**Symptom**: Job is created successfully, but the **imgbuild worker** fails with `"not enough regex capture groups available"`.

**Root Cause**: The `validID` regex in `internal/resourcename/resource_name.go` is `[a-zA-Z0-9.\-]+`. Underscores are not included. The Jobs API accepts them (no validation at that layer), but downstream workers fail.

**Resolution**: Always use hyphens in job names (e.g., `my-training-job` instead of `MY_TRAINING_JOB`).

---

## 3. `skip_papi` Only Affects Job Creation, Not Approval

**Symptom**: Job created with `skip_papi_add_application_entitlement_request: true` cannot be approved — returns 403.

**Root Cause**: The `skip_papi` flag only skips PAPI application/entitlement creation during `CreateJob`. It creates fake entitlement IDs (`SKIP-ADD-ENTITLEMENT-WAS-TRUE`). However, `UpdateJobVersionDeployment` (approve) **always** validates against PAPI server-side, checking if the user has the write entitlement. Since the entitlement is fake, PAPI rejects it.

**Server-side override**: The env var `DSEMGMT_SKIP_PAPI_ENTITLEMENT_CHECK=true` on the server skips PAPI validation on approve, but this is not set in dev/nonprod by default.

**Resolution**: For E2E testing, always create jobs with `skip_papi: false` so real PAPI entitlements are created, then add yourself to the write entitlement.

---

## 4. Orphaned PAPI Applications

**Symptom**: Creating a job fails with `"application with the provided id already exists"`.

**Root Cause**: If job creation fails **after** the PAPI application is created (e.g., during the entitlement setup step), the PAPI application remains registered. Subsequent attempts with the same `display_name` collide because PAPI derives the application ID from the display name.

**Resolution**: Manually clean up the orphaned application in PAPI, or use a different `display_name` for the new job.

---

## 5. CloudWatch Log Groups for Jobs Pipeline

When debugging the jobs deployment pipeline, logs are spread across different log groups:

| Component | Log Group (dev example) | Container Filter |
|-----------|------------------------|------------------|
| Jobs API | `/platform-dev-use1-eks/jobs` | `dse-api-jobs` |
| dsemgmt API | `/platform-dev-use1-eks/dsemgmt` | `dse-api-dsemgmt` |
| AssetStack Worker | `/platform-dev-use1-eks/workers` | `dse-assetstack-worker` |
| Terraform Worker | `/platform-dev-use1-eks/workers` | `dse-terraform-worker` |

**Useful CloudWatch queries**:

> **Important**: In nonprod, logs are JSON-wrapped inside a `log` field. Fields like `data.msg` and `kubernetes.container_name` are NOT top-level queryable. Use `@message like /keyword/` instead.

```
# dsemgmt: terraform message construction
fields @timestamp, @message
| filter @message like /constructed message for Terraform Worker/
| sort @timestamp desc
| limit 50

# Terraform Worker: received payload
fields @timestamp, @message
| filter @message like /payload/ or @message like /message subscribed/
| sort @timestamp desc
| limit 50
```

> **Note**: CloudWatch Insights does not support alternation (`|`) inside `like /regex/`. Use `or` to combine multiple filter conditions.

---

## 6. Cross-Environment Artifact Mismatch (imgbuild failure)

**Symptom**: imgbuild worker fails after deployment approval. Logs show errors about missing base images in ECR (e.g., `dse-sdk-python-cpu:v0.7.1 not found`).

**Root Cause**: The `job_artifact_uri` points to an artifact from another job or another environment (e.g., using a dev artifact URI in nonprod). The Dockerfile inside the artifact references base images that may not exist in the target environment's ECR.

**Resolution**: Do NOT reuse artifact URIs from other jobs or environments. Use the **DSE UI** (`https://csdatacrossing-np.bayer.com/dsecosystem/`) which handles artifact upload correctly. For creating job code, use the [DS Starter Kit template](https://github.com/bayer-int/dse-sdk/pull/172).

---

## 7. CloudWatch Log Format (nonprod)

**Symptom**: CloudWatch Insights queries using `data.msg = "..."` or `kubernetes.container_name = "..."` return no results.

**Root Cause**: In nonprod, log entries are JSON-wrapped — the actual log content is inside a `log` field as a stringified JSON object. Nested fields like `data.msg` are not queryable as top-level CloudWatch fields.

**Resolution**: Use `filter @message like /keyword/` instead of `filter data.msg = "keyword"`. The `@message` field contains the full raw log line and supports keyword matching.

---

## 8. dsemgmt Rejects New Terraform Inputs ("not in the asset stack inputs map")

**Symptom**: Deployment approval succeeds, but the assetstack worker → dsemgmt step fails with `"input <field_name> is not in the asset stack inputs map"`.

**Root Cause**: The `dse-assetstacks` repo (`job_task/assetstack.yaml`) does not list the new input variable in its `inputs:` section. The dsemgmt API validates all terraform inputs against the assetstack's declared input list (`assetStack.GetInputs()` in `internal/dsemgmt/assetstackdeployments.go`) and rejects any that aren't defined.

This commonly happens when a new terraform variable is added to `dse-terraform-modules/job_task/aws/variables.tf` and the mapping code in `dse-apis/internal/jobs/job_version_deployments.go`, but the corresponding input is not added to `dse-assetstacks/job_task/assetstack.yaml`.

**Resolution**:
1. Add the new input to `job_task/assetstack.yaml` in `dse-assetstacks` and bump the assetstack version.
2. Update `internal/svcconfig/svcconfig.go` in `dse-apis` to reference the new assetstack version (all config structs: `DsemgmtConfig`, `ModelConfig`, `JobAPIConfig`, `AppAPIConfig`).
3. Update `deploy/values/{dev,nonprod,prod}/values.yaml` to match the new version.

See [Jobs E2E Testing Guide — Cross-Repository Dependencies](../guides/jobs-e2e-testing.md#cross-repository-dependencies) for the full coordination checklist.

---

## 9. AssetStack Version Mismatch (svcconfig vs assetstack.yaml)

**Symptom**: Deployment uses the wrong assetstack version, or dsemgmt fetches an old version of the assetstack that doesn't include newly added inputs.

**Root Cause**: The `AssetStackJobTaskVersion` in `internal/svcconfig/svcconfig.go` does not match the version in `dse-assetstacks/job_task/assetstack.yaml`. The svcconfig version is what dsemgmt uses to fetch the assetstack metadata.

**Resolution**: After bumping the version in `dse-assetstacks`, update ALL of:
- `internal/svcconfig/svcconfig.go` — the `AssetStackJobTaskVersion` default value in all config structs
- `deploy/values/dev/values.yaml` — `asset_stack_job_task_version`
- `deploy/values/nonprod/values.yaml` — `asset_stack_job_task_version`
- `deploy/values/prod/values.yaml` — `asset_stack_job_task_version`
