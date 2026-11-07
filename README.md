# intune-policy-changes

**Public intent repo** for the Intune CI/CD home-lab POC.

This repository only describes *what* should change (compliance policy patch requests).  
It does **not** hold Azure credentials, certificates, or Graph/Intune execution logic.

## Pairing

| Repo | Visibility | Role |
| --- | --- | --- |
| `intune-policy-changes` (this repo) | Public | Change requests / trigger |
| [`intune-cicd-poc`](https://github.com/SureshVijayaraghavan/intune-cicd-poc) | Private | Auth + APIM + Graph execution |

```text
Public: changes/*.json  ──repository_dispatch──►  Private: self-hosted runner
                                                      │ client cert
                                                      ▼
                                                    APIM (Managed Identity)
                                                      ▼
                                                    Microsoft Graph / Intune
```

## How to request a change

1. Add or edit a JSON file under `changes/` (see the examples there).
2. Merge/push to `main`.
3. The workflow dispatches event `compliance-policy-update` to the private executor repo.

### Change request shape (Android fully managed)

Maps Intune UI fields → Graph `androidDeviceOwnerCompliancePolicy` properties.
See `changes/example-android-compliance-update.json`.

```json
{
  "action": "updateCompliancePolicy",
  "platform": "Android",
  "policyId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "patch": {
    "@odata.type": "#microsoft.graph.androidDeviceOwnerCompliancePolicy",
    "osMinimumVersion": "13.0",
    "minAndroidSecurityPatchLevel": "2026-07-05"
  }
}
```

| Intune UI | Graph property |
| --- | --- |
| Minimum OS version | `osMinimumVersion` |
| Minimum security patch level | `minAndroidSecurityPatchLevel` |

- `platform`: `iOS` or `Android` (hint for operators / future routing)
- `policyId`: Intune `deviceCompliancePolicy` GUID
- `patch`: JSON body fields sent to Graph `PATCH` (include `@odata.type` for typed Android/iOS policies)


## One-time setup (dispatch token)

The public workflow needs a secret that can trigger workflows in the **private** repo:

1. Create a fine-grained PAT (or GitHub App installation token flow later) with access to `intune-cicd-poc` only:
   - **Actions: Read and write**
2. In this public repo → Settings → Secrets and variables → Actions:
   - `EXECUTOR_DISPATCH_TOKEN` = that token
   - Optional var `EXECUTOR_REPO` (default: `SureshVijayaraghavan/intune-cicd-poc`)

Only grant write access on this public repo to people you trust — they can trigger dispatches once the secret exists.
