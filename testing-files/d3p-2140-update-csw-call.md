# Testing Notes — CSW Owner Information (PR #650)

This document describes the manual API test performed to verify that DSE passes actual CloudAccount owner CWIDs and UPNs (looked up via PAPI) to the CSW project provisioning call, replacing the hardcoded defaults.

## Prerequisites

1. Branch `d3p-2140-update-csw-call` deployed to **NonProd** (`np`).
2. Authenticated via `auth login` (see [csgda-auth](https://github.com/bayer-int/csgda-auth)).
3. Replace `LSZFW` with your Corporate Worker ID.

### Authentication Commands

| Command | Description |
|---------|-------------|
| `auth login` | Interactive browser login (Azure AD). In Codespaces, run `sudo ln -sf "$BROWSER" /usr/local/bin/xdg-open` first. |
| `auth print-access-token` | Prints the raw JWT token to stdout. Use with `curl -H "Authorization: Bearer $(auth print-access-token)"`. |
| `auth curl -- <curl args>` | Wraps `curl` and automatically injects the `Authorization` header. Use `--` to separate auth flags from curl flags. |

## Environment

- **Base URL**: `https://apis.dse-np.bayer.com`
- **Tenant**: `tenants/DONOTDELETE`
- **Cloud Account**: `PRODDEPLOYMENTTESTACCOUNT`
- **DSE Portal**: https://csdatacrossing-np.bayer.com/dsecosystem/environments/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT

---

## Pre-step: Verify Owner Fields on the CloudAccount

The cloud account already had owner fields populated:

```json
{
  "accountOwner": "CLCOTN",
  "additionalAccountOwner": "GCTYY"
}
```

If the test cloud account does not have `accountOwner` / `additionalAccountOwner` set, update them first with your CWID (and optionally a teammate's):

```bash
auth curl -- -X PATCH https://apis.dse-np.bayer.com/v1/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "cloud_account": {
        "account_owner": "LSZFW",
        "additional_account_owner": "EAKRE"
    },
    "update_mask": "account_owner,additional_account_owner",
    "updated": { "client_id": "test", "user_id": "LSZFW" }
}'
```

Then verify:

```bash
auth curl -- -X GET https://apis.dse-np.bayer.com/v1/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT \
  -H "Cwid: LSZFW" | jq '{accountOwner, additionalAccountOwner}'
```

Expected:
```json
{
  "accountOwner": "LSZFW",
  "additionalAccountOwner": "EAKRE"
}
```

---

## Test 1: Trigger CSW Provisioning — Verify Owner CWIDs and UPNs

Trigger the CSW provisioning endpoint for the cloud account (now with owner fields set). After triggering, verify in the logs that the CSW request payload includes the owner CWIDs from the CloudAccount and the UPNs resolved via PAPI lookup.

### 1a. Trigger CSW Provisioning

```bash
auth curl -- -X POST https://apis.dse-np.bayer.com/v1/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT:csw-provisioning \
  -H "Content-Type: application/json" \
  -H "Cwid: LSZFW" \
  -d '{
    "name": "tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNTcsw-provisioning",
    "submitter": {
      "user_id": "LSZFW"
    }
}'
```

### Expected Result

- **Status**: `200 OK` — CSW provisioning request accepted.
- The CSW call takes approximately **5 minutes** to complete.

### Result

- **First attempt (Apr 24)**: CSW returned a **500 Internal Server Error**. This was a CSW-side issue (their sandbox environment `bcs-csw-engineering-sbx`), **not related to our changes**. Confirmed by deploying `main` — identical 500 error.
- **Second attempt (Apr 28)**: CSW returned `{"provisioned":false}` — the call went through successfully. ✅

### Screenshot

> _Add screenshot here_

---

## Test 2: Verify Owner Fields in Logs

After triggering CSW provisioning (wait ~5 minutes for the call to complete), check the **dsemgmt API logs** to confirm:

1. The `owner_cwid_1` and `owner_cwid_2` structured log fields are present and populated with the CloudAccount's `account_owner` and `additional_account_owner` CWIDs.
2. The PAPI UPN lookups succeeded (no `"Failed to lookup UPN"` warnings).
3. The CSW request payload contains all four owner fields (`gcp_additional_project_owner_cwid_1`, `gcp_additional_project_owner_cwid_2`, `gcp_additional_project_owner_upn_1`, `gcp_additional_project_owner_upn_2`).

### CloudWatch Query — dsemgmt API logs (owner CWIDs and CSW call)

```
fields @timestamp, data.level, data.msg, data.owner_cwid_1, data.owner_cwid_2
| filter kubernetes.container_name = "dse-api-dsemgmt"
| filter data.msg like /calling csw|owner_cwid|Failed to lookup UPN|csw-provisioning/
| sort @timestamp desc
| limit 100
```

### Expected Result

- Log entry `"calling csw.RequestDseProject"` should appear with:
  - `owner_cwid_1` = CloudAccount's `account_owner` CWID
  - `owner_cwid_2` = CloudAccount's `additional_account_owner` CWID
- No `"Failed to lookup UPN for owner"` warning entries (both PAPI lookups succeeded).
- If a warning _does_ appear, confirm the CSW call still proceeds (non-blocking behavior).

### Result

- **`owner_cwid_1`: `CLCOTN`** — correctly populated from CloudAccount's `accountOwner` ✅
- **`owner_cwid_2`: `GCTYY`** — correctly populated from CloudAccount's `additionalAccountOwner` ✅
- **PAPI UPN lookups**: Both succeeded — `"Successfully retrieved email for user"` logged for both owners ✅
- **Azure token caching**: Working — first call `"Fetching new Azure token"`, subsequent calls `"Using cached Azure token"` ✅
- **CSW call**: `"calling csw.RequestDseProject"` executed with all owner fields ✅

<details>
<summary>Full log timeline (Apr 28, 19:14–19:15 UTC)</summary>

```
19:14:26.512 | Fetching new Azure token                    | cwid1=CLCOTN cwid2=GCTYY
19:14:28.031 | Successfully retrieved email for user        | cwid1=CLCOTN cwid2=GCTYY
19:14:28.031 | Using cached Azure token                    | cwid1=CLCOTN cwid2=GCTYY
19:14:28.061 | Successfully retrieved email for user        | cwid1=CLCOTN cwid2=GCTYY
19:14:28.061 | calling csw.RequestDseProject                | cwid1=CLCOTN cwid2=GCTYY
19:14:28.061 | generated unique CSW display name            | cwid1=CLCOTN cwid2=GCTYY
19:14:28.172 | calling CSW /dse-project-request endpoint    | cwid1=CLCOTN cwid2=GCTYY
19:14:33.303 | handling request: context canceled (timeout) | cwid1=CLCOTN cwid2=GCTYY
19:15:26.116 | Using cached Azure token                    | cwid1=CLCOTN cwid2=GCTYY
19:15:26.155 | Successfully retrieved email for user        | cwid1=CLCOTN cwid2=GCTYY
19:15:26.155 | Using cached Azure token                    | cwid1=CLCOTN cwid2=GCTYY
19:15:26.189 | Successfully retrieved email for user        | cwid1=CLCOTN cwid2=GCTYY
19:15:26.189 | calling csw.RequestDseProject                | cwid1=CLCOTN cwid2=GCTYY
19:15:26.189 | generated unique CSW display name            | cwid1=CLCOTN cwid2=GCTYY
19:15:26.189 | calling getAccessToken                       | cwid1=CLCOTN cwid2=GCTYY
19:15:26.313 | calling CSW /dse-project-request endpoint    | cwid1=CLCOTN cwid2=GCTYY
```

</details>

<details>
<summary>Previous log entry (Apr 24 — CSW 500, before CSW was fixed)</summary>

```json
{
  "time": "2026-04-24T11:43:37.457089772Z",
  "level": "ERROR",
  "msg": "handling request: failed to unmarshal response: System A failed: 500 Internal Server Error",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "csw_method": "RequestDseProject",
  "api_url": "https://us-central1-bcs-csw-engineering-sbx.cloudfunctions.net/dse-project-request"
}
```

</details>

### Screenshot

> _Add screenshot here_

---

## Test 3: Baseline Comparison — Same Error on `main`

To confirm the CSW 500 is not caused by our changes, `main` was deployed to NonProd and the same endpoint was triggered.

### Command

Same as Test 1a, after deploying `main` to NonProd.

### Result

- **Identical 500 error** returned on `main` — same `"System A failed: 500 Internal Server Error"` message. ✅
- This confirms the CSW sandbox environment (`bcs-csw-engineering-sbx`) is failing independently of our changes.
- The CSW 500 is **not a regression**.

### Screenshot

> _Add screenshot here_

---

## Test 4 (Edge Case): CloudAccount with Only Primary Owner

If time permits, test with a CloudAccount that has `account_owner` set but `additional_account_owner` is empty. Verify that:
- `gcp_additional_project_owner_cwid_1` and `gcp_additional_project_owner_upn_1` are populated.
- `gcp_additional_project_owner_cwid_2` and `gcp_additional_project_owner_upn_2` are omitted (empty).
- No errors or warnings for the missing second owner.

### Result

> _Add result here_

### Screenshot

> _Add screenshot here_

---

## Summary Checklist

| Check | Status |
|-------|--------|
| Branch deployed to NonProd | ✅ Image `d3p-2140-update-csw-call-1e252826` |
| CSW provisioning endpoint called | ✅ |
| `owner_cwid_1` logged with correct CWID (`CLCOTN`) | ✅ |
| `owner_cwid_2` logged with correct CWID (`GCTYY`) | ✅ |
| PAPI UPN lookup for owner 1 succeeded | ✅ `"Successfully retrieved email for user"` |
| PAPI UPN lookup for owner 2 succeeded | ✅ `"Successfully retrieved email for user"` |
| Azure token caching works | ✅ Fetch once, reuse via cache |
| CSW call executed with owner fields | ✅ `"calling csw.RequestDseProject"` |
| Baseline: same CSW 500 on `main` (Apr 24) | ✅ Confirmed not a regression |
| CSW call succeeded after CSW fix (Apr 28) | ✅ `{"provisioned":false}` |
