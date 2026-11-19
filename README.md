# Mohamed Hirey

**Azure Cloud Engineer** · [linkedin.com/in/mohamedhirey](https://linkedin.com/in/mohamedhirey)

---

## Proof of work

Both projects below were deployed to a real Azure subscription, checked against live Azure and not just against Terraform's own report, then torn down in the same session. Each repository keeps the captured output of that run in `docs/evidence/`. Account identifiers are redacted. Nothing else is edited.

| | [Hub-and-Spoke Landing Zone](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone) | [Secure Web App](https://github.com/mohamedhirey/azure-secure-web-app) |
|---|---|---|
| **What it is** | Network topology, default-deny NSGs, keyless remote state, a secretless OIDC deployment pipeline | App Service with a private-only, keyless data tier, a managed identity and an alert that has been seen firing |
| **Deployed** | 19 Nov 2026 (UTC) · 29 resources · by GitHub Actions | 19 Nov 2026 (UTC) · 32 resources · by Terraform |
| **Checks** | [![ci](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/workflows/ci.yml/badge.svg)](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/workflows/ci.yml) | [![ci](https://github.com/mohamedhirey/azure-secure-web-app/actions/workflows/ci.yml/badge.svg)](https://github.com/mohamedhirey/azure-secure-web-app/actions/workflows/ci.yml) |
| **Evidence** | [docs/evidence](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/tree/main/docs/evidence) | [docs/evidence](https://github.com/mohamedhirey/azure-secure-web-app/tree/main/docs/evidence) |
| **Left running** | Nothing | Nothing |

---

### [Azure Hub-and-Spoke Landing Zone](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone)

A hub and two spokes in Terraform, deployed by a GitHub Actions pipeline that holds no secret. Plan, apply and destroy each stop at an approval gate, sign in to Azure with a short-lived OIDC token, and are public: [plan](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44445970210) · [apply](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44446223300) · [destroy](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44447025676).

[![The apply run: approved at the production gate, job green](https://raw.githubusercontent.com/mohamedhirey/azure-hub-spoke-landing-zone/main/docs/evidence/pipeline-apply-run.png)](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44446223300)

| Claim | Proof |
|---|---|
| The pipeline signs in with no secret | One federated credential, pinned to the repository's immutable ID and one environment. 0 client secrets. [`02-identity-oidc.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/02-identity-oidc.txt) |
| The pipeline can touch one resource group and one storage account | Two role assignments, both scoped below the subscription. [`02-identity-oidc.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/02-identity-oidc.txt) |
| State cannot be read with an account key | Refused, `KeyBasedAuthenticationNotPermitted`. [`03-keyless-remote-state.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/03-keyless-remote-state.txt) |
| Every NSG fails closed, and dev cannot reach prod | `DenyAllInbound` at 4096 on all four. Four peerings `Connected`, none between the spokes. [`06-live-azure-verification.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/06-live-azure-verification.txt) |
| The tests can fail | Break the rule, watch 2 of 4 fail, restore. [`01-unit-tests.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/01-unit-tests.txt) |
| Nothing was left behind | 0 resources, roles, app registrations or secrets. [`07-teardown.txt`](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/blob/main/docs/evidence/07-teardown.txt) |

Deploying it for real also found three things offline validation could not, including a change in the shape of GitHub's OIDC subject that would have had Entra ID reject the pipeline. [What it found](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/tree/main/docs/evidence#what-deploying-it-for-real-found).

---

### [Azure Secure Web App](https://github.com/mohamedhirey/azure-secure-web-app)

A Python web app on App Service whose Cosmos DB and Key Vault have no public network path and no keys. It reaches both through private endpoints, as a managed identity.

The strongest single proof is the vault's own audit log. The app's identity is refused when its request arrives from a public address, and allowed when it arrives over the private endpoint. The subscription owner is then refused twice: first by RBAC and, once granted the role, by the network.

```text
Time (UTC)  Operation  Result     Caller address           Caller               Reason
02:48:56    SecretGet  Forbidden  <app-service-public-ip>  app identity         Public network access is disabled
02:49:28    SecretGet  OK         10.60.1.254              app identity
02:59:08    SecretGet  Forbidden  <public address>         subscription owner   Caller is not authorized
03:00:24    SecretGet  Forbidden  <public address>         owner, role granted  Public network access is disabled
```

| Claim | Proof |
|---|---|
| The app works with no credential anywhere | Counter rises and `secret_from_key_vault` is `true`. [`03-app-works.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/03-app-works.txt) |
| The app's identity can do two things and nothing else | Read secrets in one vault. Read and write items in one database. [`04-identity-and-least-privilege.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/04-identity-and-least-privilege.txt) |
| The data tier has no public path | Key Vault answers `ForbiddenByConnection`. Cosmos DB, given a valid Entra ID token from the internet, answers `403`. [`05-data-tier-private.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/05-data-tier-private.txt) |
| Every request and every vault access is logged | [`06-logs-and-traces.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/06-logs-and-traces.txt) |
| The alert has been seen firing | 12 errors at 02:58, fired at 03:01, resolved at 03:08. [`07-alert-fired.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/07-alert-fired.txt) |
| Nothing was left behind | The subscription ended with 0 resources and nothing soft-deleted. [`08-teardown.txt`](https://github.com/mohamedhirey/azure-secure-web-app/blob/main/docs/evidence/08-teardown.txt) |

Deploying it for real also found a defect that offline validation and four passing unit tests could not: `terraform destroy` did not work. Azure Resource Manager has no DELETE for a Key Vault secret, and the teardown stopped with the vault still standing. The cause was in the Terraform. It was fixed, and the fix was proven on Azure the same day. [What it found](https://github.com/mohamedhirey/azure-secure-web-app/tree/main/docs/evidence#what-deploying-it-for-real-found).
