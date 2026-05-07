# CREST Authorization Guide

## What is CREST?

CREST is Bayer's **API gateway-level authorization system**. It acts as the first layer of security for all API requests, validating whether the calling **app client** is authorized to access a specific endpoint before the request ever reaches the application code.

CREST is part of the broader Bayer API management platform and works alongside other auth systems (like PAPI) to provide multi-layered access control.

---

## How CREST Works

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Application** | A registered API service in CREST (e.g., `DSE-APIS`). Each application defines its own set of entitlements. |
| **App Client** | A client identity that makes API calls (e.g., `dse-platform-non-prod`). Clients are granted access to specific entitlements. |
| **Entitlement** | A permission associated with an application. Has a text identifier (e.g., `dse.api.jobs.create`) that maps to specific API operations. |
| **Policy** | Defined in the codebase (Helm templates), policies connect entitlement identifiers to specific API routes/methods. |

### Entitlement Identifiers

Each entitlement has a **text identifier** that is completely arbitrary — you can name it whatever you want. However, the identifier **must match exactly** between:

1. The CREST configuration (managed via Velocity UI)
2. The policy definitions in the codebase (`deploy/<service>/templates/`)

If these don't match, the gateway will reject requests with a `403 Forbidden`.

**Example entitlement identifiers for Jobs API:**
- `dse.api.jobs.create` — allows creating jobs
- `dse.api.jobs.update` — allows updating/approving job deployments
- `dse.api.jobs.delete` — allows deleting jobs
- `dse.api.jobs.read` — allows reading job data

### Shared App Client

Both the DSE **portal** (frontend) and the DSE **API** (backend) use the **same app client** per environment. The exact app client name can be verified in the Velocity UI under `Applications → DSE-APIS → Configuration → Entitlements`.

This means:
- If CREST blocks an operation, it blocks **both** portal and curl equally
- If the portal can do something but curl cannot, the issue is likely **PAPI** (Layer 3), not CREST (Layer 1)
- A CREST 403 (`"requestor not authorized"`) affects all callers using that app client

---

## Managing CREST Entitlements

### Velocity UI

Entitlements are managed through the Velocity portal:

| Environment | URL |
|-------------|-----|
| Dev | `https://velocity-dev.ag/profile/applications/DSE-APIS/configuration/entitlements` |
| Non-Prod | `https://velocity-np.ag/profile/applications/DSE-APIS/configuration/entitlements` |
| Prod | `https://velocity.ag/profile/applications/DSE-APIS/configuration/entitlements` |

From this UI you can:
- View all registered entitlements for the application
- Create new entitlements
- Edit entitlement identifiers
- Assign app clients to entitlements

### Codebase Policies

The entitlement identifiers must match what's defined in the deploy templates. For the Jobs API:

```
deploy/dse-api-jobs/templates/
```

Look for files that define authorization policies — they contain the entitlement identifier strings that CREST validates against.

---

## CREST in the DSE Auth Flow

```
Request → CREST Gateway → Application Code → PAPI Check → Action
           (Layer 1)         (Layer 2)        (Layer 3)
```

### Layer 1: CREST (Gateway)
- **Question**: "Is this app client allowed to call this endpoint?"
- **Scope**: Per app client, per API operation
- **Failure**: `403 Forbidden` with `"requestor not authorized"`
- **Fix**: Update CREST entitlement in Velocity UI or fix policy in deploy templates

### Layer 2: Application Code
- Business logic, input validation, database operations
- Custom authorization checks (e.g., tenant membership)

### Layer 3: PAPI (Application-Level)
- **Question**: "Does this specific user have the write entitlement for this specific resource?"
- **Scope**: Per user, per resource (e.g., per job)
- **Failure**: `403 Forbidden` with messages about PAPI/write access
- **Fix**: Add user to the PAPI entitlement for that resource

### How to Distinguish CREST vs PAPI 403s

| Error Message | Source | Fix |
|---------------|--------|-----|
| `"requestor not authorized"` | CREST (gateway) | Fix entitlement identifier in Velocity UI or deploy templates |
| `"user does not have PAPI write access"` or similar | PAPI (application) | Add user to PAPI write entitlement for the resource |
| `"code": 403` with no detailed message | Likely CREST | Check Velocity UI entitlements |

---

## Common CREST Issues

### 1. Entitlement Identifier Mismatch

**Symptom**: 403 on a specific API operation (e.g., update/approve).

**Cause**: The entitlement identifier in CREST doesn't match the policy definition in the codebase. For example:
- CREST has: `dse.api.data.jobs.update`
- Codebase expects: `dse.api.jobs.update`

**Important**: Since portal and curl use the same app client, a CREST mismatch blocks **both** equally. If only curl fails, the issue is more likely PAPI, not CREST.

**Fix**: Correct the identifier in the Velocity UI to match the codebase, or vice versa. Must be done **per environment**.

### 2. Missing Entitlement for New Endpoint

**Symptom**: New API endpoint returns 403 for all callers.

**Cause**: When adding a new API endpoint, the corresponding CREST entitlement needs to be:
1. Defined in the deploy templates (policy)
2. Created in the Velocity UI
3. Assigned to the app client

**Fix**: Add the entitlement in all three places.

### 3. Environment-Specific Issues

**Symptom**: API works in dev but not in nonprod (or vice versa).

**Cause**: CREST entitlements are configured **per environment**. A fix in nonprod doesn't automatically apply to dev or prod.

**Fix**: Verify and fix entitlements in each environment independently via the corresponding Velocity URL.

---

## Quick Reference

### Check Current Entitlements
Go to the Velocity UI for your environment and navigate to:
`Applications → DSE-APIS → Configuration → Entitlements`

### Verify Policy Definitions
```bash
# Find entitlement identifier strings in deploy templates
grep -r "entitlement" deploy/dse-api-jobs/templates/ --include="*.yaml"
```

### Debug a 403
1. Check the error message — is it CREST or PAPI?
2. If CREST: verify the entitlement identifier matches between Velocity UI and deploy templates
3. If PAPI: verify the user is assigned to the resource's write entitlement
4. Check that you're hitting the right environment
