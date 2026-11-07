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

1. Add or edit a JSON file under `changes/` (see `changes/example-ios-compliance-update.json`).
2. Merge/push to `main`.
3. The workflow dispatches event `compliance-policy-update` to the private executor repo.

### Change request shape

```json
{
  "action": "updateCompliancePolicy",
  "platform": "iOS",
  "policyId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "patch": {
    "description": "Updated from public intent repo"
  }
}
```

- `platform`: `iOS` or `Android` (hint for operators / future routing)
- `policyId`: Intune `deviceCompliancePolicy` GUID
- `patch`: JSON body fields sent to Graph `PATCH`

## One-time setup (dispatch token)

The public workflow needs a secret that can trigger workflows in the **private** repo:

1. Create a fine-grained PAT (or GitHub App installation token flow later) with access to `intune-cicd-poc` only:
   - **Actions: Read and write**
2. In this public repo → Settings → Secrets and variables → Actions:
   - `EXECUTOR_DISPATCH_TOKEN` = that token
   - Optional var `EXECUTOR_REPO` (default: `SureshVijayaraghavan/intune-cicd-poc`)

Only grant write access on this public repo to people you trust — they can trigger dispatches once the secret exists.
