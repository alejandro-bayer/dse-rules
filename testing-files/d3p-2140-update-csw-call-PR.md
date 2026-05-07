## Manual Tests — CSW Owner Information (PR #650)

### Prerequisites

1. Branch `d3p-2140-update-csw-call` deployed to **NonProd**.
2. Authenticated via `auth login`.
3. Replace `<YOUR_CWID>` with your Corporate Worker ID.

**Environment**: `https://apis.dse-np.bayer.com` | Tenant: `tenants/DONOTDELETE` | Cloud Account: `PRODDEPLOYMENTTESTACCOUNT`

---

## Pre-step: Verify Owner Fields on CloudAccount

```bash
auth curl -- -s -X GET "https://apis.dse-np.bayer.com/v1/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT" \
  -H "Cwid: <YOUR_CWID>" | jq '{accountOwner, additionalAccountOwner}'
```

**Expected**: Both `accountOwner` and `additionalAccountOwner` are populated with valid CWIDs.

**Result**: ✅ Pass

<details><summary>Response</summary>

```json
{
  "accountOwner": "CLCOTN",
  "additionalAccountOwner": "GCTYY"
}
```

</details>

<details><summary>Full CloudAccount response</summary>

```json
{
  "name": "tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT",
  "displayName": "DSE Functional Prod Deploy",
  "parent": "tenants/DONOTDELETE",
  "beatId": "BEAT04037530",
  "environment": "PROD",
  "requesterCwid": "EAKRE",
  "controlledData": false,
  "accountOwner": "CLCOTN",
  "additionalAccountOwner": "GCTYY",
  "highestInformationClassification": "INTERNAL",
  "provisioningStatus": "READY",
  "aws": {
    "awsAccountId": "038462751656"
  },
  "csw": {
    "gcpProjectId": "bcs-dse-func-dse-prod-taccount",
    "status": true
  },
  "ownerPapiGroup": "DSE-DEV-TEAM",
  "financialData": true,
  "created": {
    "clientId": "0d38b242-efbf-47d3-b312-77bef4b82c4d",
    "userId": "EAIBU",
    "time": "2025-02-04T19:48:43.264895479Z"
  },
  "updated": {
    "userId": "<YOUR_CWID>",
    "time": "2026-04-30T20:10:01.805677360Z"
  }
}
```

</details>

---

## Test 1: Trigger CSW Provisioning

Triggers the CSW provisioning endpoint. The branch code should:
1. Read `accountOwner` and `additionalAccountOwner` CWIDs from the CloudAccount.
2. Look up each owner's UPN via PAPI (Profile API).
3. Pass all four fields (`gcp_additional_project_owner_cwid_1/2`, `gcp_additional_project_owner_upn_1/2`) to the CSW payload.

```bash
auth curl -- -s -X POST "https://apis.dse-np.bayer.com/v1/tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNT:csw-provisioning" \
  -H "Content-Type: application/json" -H "Cwid: <YOUR_CWID>" \
  -d '{
    "name": "tenants/DONOTDELETE/cloud-accounts/PRODDEPLOYMENTTESTACCOUNTcsw-provisioning",
    "submitter": { "user_id": "<YOUR_CWID>" }
  }'
```

**Expected**: `200 OK` — CSW provisioning request accepted and processed.

**Result**: ✅ Pass

<details><summary>Response</summary>

```json
{
  "provisioned": false
}
```

</details>

> `provisioned: false` is expected — CSW sandbox does not actually provision, but the request was accepted and our code executed the full owner lookup flow.

---

## Test 2: Verify Owner Fields in CloudWatch Logs

After triggering CSW provisioning, verify in the **dsemgmt** CloudWatch logs that:
1. `owner_cwid_1` and `owner_cwid_2` are logged with the correct CWIDs.
2. PAPI UPN lookups succeeded for both owners.
3. Azure token caching works (fetch once, then reuse).
4. The CSW `/dse-project-request` endpoint was called.

### CloudWatch Query

```
filter @message like /owner_cwid/ or @message like /UPN/
  or @message like /populateOwnerUPNs/
  or @message like /csw/ and @message like /PRODDEPLOYMENTTESTACCOUNT/
| sort @timestamp desc
| limit 30
```

Log group: `/platform-nonprod-use1-eks/dsemgmt`

**Expected**: Full pipeline — token fetch → UPN lookups → CSW call, with both owner CWIDs present.

**Result**: ✅ Pass — All log entries show `owner_cwid_1: CLCOTN`, `owner_cwid_2: GCTYY`

<details><summary>Log timeline (Apr 30, 20:09 UTC)</summary>

```
20:09:44.685 | received request                                        | dsemgmt.(*Server).CswProvisioning
20:09:44.700 | received request                                        | dsemgmt.(*Server).GetCloudAccount
20:09:44.772 | calling makeCswRequest                                   | dsemgmt.(*Server).handleCswRequest
20:09:44.773 | Fetching new Azure token             [CLCOTN,GCTYY]     | clients.PapiClient.getOrFetchAzureToken
20:09:45.392 | Successfully retrieved email for user cwid=CLCOTN       | profileapi.GetUserEmailById
20:09:45.392 | Using cached Azure token             [CLCOTN,GCTYY]     | clients.PapiClient.getOrFetchAzureToken
20:09:45.418 | Successfully retrieved email for user cwid=GCTYY        | profileapi.GetUserEmailById
20:09:45.418 | calling csw.RequestDseProject         [CLCOTN,GCTYY]     | dsemgmt.(*Server).makeCswRequest
20:09:45.418 | generated unique CSW display name                        | csw.RequestDseProject
20:09:45.418 | calling getAccessToken                                   | csw.RequestDseProject
20:09:45.567 | calling CSW /dse-project-request endpoint [CLCOTN,GCTYY] | csw.RequestDseProject
```

</details>

<details><summary>Raw log entries (JSON)</summary>

```json
{
  "time": "2026-04-30T20:09:45.392114635Z",
  "level": "INFO",
  "msg": "Successfully retrieved email for user",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "cwid": "CLCOTN",
  "method_name": "LookupUpnFromCwid",
  "function_name": "internal/profileapi.GetUserEmailById"
}
```

```json
{
  "time": "2026-04-30T20:09:45.418339543Z",
  "level": "INFO",
  "msg": "Successfully retrieved email for user",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "cwid": "GCTYY",
  "method_name": "LookupUpnFromCwid",
  "function_name": "internal/profileapi.GetUserEmailById"
}
```

```json
{
  "time": "2026-04-30T20:09:45.418317841Z",
  "level": "INFO",
  "msg": "calling csw.RequestDseProject",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "function_name": "internal/dsemgmt.(*Server).makeCswRequest"
}
```

```json
{
  "time": "2026-04-30T20:09:45.567507856Z",
  "level": "INFO",
  "msg": "calling CSW /dse-project-request endpoint",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "csw_method": "RequestDseProject",
  "function_name": "internal/csw.RequestDseProject"
}
```

```json
{
  "time": "2026-04-30T20:09:44.773Z",
  "level": "INFO",
  "msg": "Fetching new Azure token",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "function_name": "internal/clients.PapiClient.getOrFetchAzureToken"
}
```

```json
{
  "time": "2026-04-30T20:09:45.39227164Z",
  "level": "INFO",
  "msg": "Using cached Azure token",
  "owner_cwid_1": "CLCOTN",
  "owner_cwid_2": "GCTYY",
  "function_name": "internal/clients.PapiClient.getOrFetchAzureToken"
}
```

</details>

### Key observations

| Check | Evidence |
|-------|----------|
| `owner_cwid_1` = `CLCOTN` (from `accountOwner`) | ✅ All log entries |
| `owner_cwid_2` = `GCTYY` (from `additionalAccountOwner`) | ✅ All log entries |
| UPN lookup for owner 1 (`CLCOTN`) | ✅ `"Successfully retrieved email for user"` |
| UPN lookup for owner 2 (`GCTYY`) | ✅ `"Successfully retrieved email for user"` |
| Azure token fetched once, then cached | ✅ `"Fetching new Azure token"` → `"Using cached Azure token"` |
| CSW endpoint called with owner fields | ✅ `"calling CSW /dse-project-request endpoint"` |
| No errors or warnings | ✅ All entries at `INFO` level |

---

## Summary Checklist

| Check | Status |
|-------|--------|
| Branch deployed to NonProd | ✅ |
| CloudAccount has `accountOwner` (`CLCOTN`) and `additionalAccountOwner` (`GCTYY`) | ✅ |
| CSW provisioning endpoint called successfully | ✅ `200 OK` |
| `owner_cwid_1` logged with correct CWID (`CLCOTN`) | ✅ |
| `owner_cwid_2` logged with correct CWID (`GCTYY`) | ✅ |
| PAPI UPN lookup for owner 1 succeeded | ✅ `"Successfully retrieved email for user"` |
| PAPI UPN lookup for owner 2 succeeded | ✅ `"Successfully retrieved email for user"` |
| Azure token caching works | ✅ Fetch once, reuse via cache |
| CSW call executed with all owner fields | ✅ `"calling CSW /dse-project-request endpoint"` |
| No errors or regressions | ✅ |
