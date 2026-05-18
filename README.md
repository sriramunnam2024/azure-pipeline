# Azure Lakehouse Pipeline (portfolio)

Medallion-style data platform on Azure: **landing (ADLS)** → **Databricks (Delta)** with **ephemeral compute** deployed via Terraform or Databricks Asset Bundles (DAB).

## Design principles

- **No secrets in git.** Auth via `az login` / Databricks CLI profiles locally; Key Vault + managed identity in Azure for runtime.
- **Permanent:** storage, Unity Catalog objects, job/pipeline definitions, Delta tables (data at rest).
- **Ephemeral:** clusters, SQL warehouses, bundle-deployed compute — tear down when not demoing.
- **GitHub:** public showcase; **Databricks Repos** can mirror this repo for job execution.

## Repo layout

| Path | Purpose |
|------|---------|
| `docs/` | Setup guides (Portal, upload, architecture) |
| `infra/terraform/` | Long-lived resources + optional compute modules |
| `bundles/` | Databricks Asset Bundle (`databricks.yml`) |
| `src/notebooks`, `src/sql`, `src/pipelines` | Job/pipeline source |
| `scripts/` | Local helpers (upload, validate) |

## Prerequisites

1. Azure subscription + `az login`
2. Storage account + container (see `docs/01-azure-storage-portal.md`)
3. Databricks workspace on Azure (later step)
4. [Databricks CLI](https://docs.databricks.com/dev-tools/cli/) configured locally

## Quick start (local, after storage exists)

```bash
# From Git Bash — upload landing files (see docs/02-manual-upload.md)
az storage blob upload-batch \
  --account-name "$AZURE_STORAGE_ACCOUNT_NAME" \
  --destination "$AZURE_STORAGE_CONTAINER" \
  --destination-path "$AZURE_BLOB_PREFIX" \
  --source "/path/to/landing_data/azure" \
  --auth-mode login \
  --overwrite
```

## Git

```bash
cd azure-pipeline
git init
git add .
git commit -m "Initial scaffold: docs, infra placeholders, src layout"
```

Push to GitHub when ready; keep `.env` and `terraform.tfvars` local only.
