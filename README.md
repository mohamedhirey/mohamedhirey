# Mohamed Hirey

**Azure Cloud Engineer** · [linkedin.com/in/mohamedhirey](https://linkedin.com/in/mohamedhirey)

Two Azure builds in Terraform. Each one was deployed to a real subscription, checked against live Azure and torn down in the same session. The captured output is in each repository's `docs/evidence/` folder.

---

## [Azure Hub-and-Spoke Landing Zone](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone)

[![ci](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/workflows/ci.yml/badge.svg)](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/workflows/ci.yml)

A hub and two spoke networks, deployed by a GitHub Actions pipeline that holds no secret.

- **Secretless pipeline.** Each run stops at an approval gate, then signs in to Azure with a short-lived OIDC token. The runs are public: [plan](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44460878382) · [apply](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44461006767) · [destroy](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/actions/runs/44461612790).
- **Fails closed.** Every network security group ends in a deny-all inbound rule. Input validation and unit tests enforce it, and the pipeline reads it back from Azure after the apply.
- **Keyless state.** The storage account that holds the Terraform state refuses its own account keys.
- **Dev cannot reach prod.** There is no peering between the spokes, confirmed from Azure.
- **Deployed twice.** The first run found four problems that offline validation could not. [What it found](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/tree/main/docs/evidence#what-the-first-deployment-found).

29 resources · no standing cost · [evidence](https://github.com/mohamedhirey/azure-hub-spoke-landing-zone/tree/main/docs/evidence)

---

## [Azure Secure Web App](https://github.com/mohamedhirey/azure-secure-web-app)

[![ci](https://github.com/mohamedhirey/azure-secure-web-app/actions/workflows/ci.yml/badge.svg)](https://github.com/mohamedhirey/azure-secure-web-app/actions/workflows/ci.yml)

A Python web app on App Service whose Cosmos DB and Key Vault have no public network path and accept no keys.

- **No keys to the data.** The app reaches Cosmos DB and Key Vault as a managed identity, through private endpoints. Cosmos DB account keys are disabled.
- **Private, and proven so.** The vault's own audit log shows the subscription owner refused from the internet, even after being granted the role.
- **Least privilege.** The app's identity can read secrets in one vault, and read and write items in one database.
- **Tested alerting.** The HTTP 5xx alert was made to fire, and its fired and resolved times are recorded.
- **A real defect, found and fixed.** The first `terraform destroy` failed. The cause, the fix and the proof are recorded. [What it found](https://github.com/mohamedhirey/azure-secure-web-app/tree/main/docs/evidence#what-deploying-it-for-real-found).

32 resources · torn down the same day · [evidence](https://github.com/mohamedhirey/azure-secure-web-app/tree/main/docs/evidence)
