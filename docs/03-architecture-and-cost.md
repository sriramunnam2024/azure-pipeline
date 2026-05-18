# Architecture, workspace vs GitHub, cost (all three clouds)

## Your planned split (recommended)

| Layer | Lifetime | Deployed by |
|-------|----------|-------------|
| Object storage (landing + Delta paths) | Permanent | Terraform (once) or Portal for first test |
| Unity Catalog metastore, external locations, grants | Permanent | Terraform / Portal |
| Delta tables (data) | Permanent | Pipelines (data stays in storage) |
| Job / pipeline / notebook definitions | Permanent in workspace | DAB deploy from CI or `databricks bundle deploy` |
| Clusters, SQL warehouses, ephemeral workers | **Temporary** | Terraform or DAB `resources` — destroy after demo |
| GitHub repo | Public showcase | Source mirror; not the runtime secret store |

**GitHub = catalog of how you build.**  
**Databricks Repo = where jobs run from** (sync tag/branch from GitHub).

Use **Repos** in workspace linked to GitHub; jobs reference repo paths (`/Repos/<user>/azure-pipeline/src/...`).

## Secrets

| Environment | Pattern |
|-------------|---------|
| Laptop | `az login`, Databricks CLI profile, `.env` gitignored |
| Databricks jobs | Secret scopes → Key Vault–backed secrets |
| GitHub Actions (optional later) | OIDC to Azure, no long-lived PAT in repo |

## Cost & efficiency — Azure (primary)

- **Storage LRS + lifecycle:** cheap; archive old landing files if demos pile up.
- **Databricks:** use **job clusters** or **serverless** for short runs; **auto-termination 10–15 min**; no always-on all-purpose clusters.
- **ADF:** start with **manual triggers**; avoid 5-minute schedules on toy pipelines.
- **Terraform:** `terraform destroy` only on **compute** modules; never destroy storage if Delta data should remain.

## AWS (when you add `aws-pipeline` repo)

- **S3** landing + Delta on S3: pennies at demo scale.
- **Databricks on AWS** or **Glue + Athena** for cheaper static demos — pick one story per repo.
- Ephemeral: EMR/Databricks job clusters; permanent: S3 buckets, IAM roles (no keys in git).

## GCP (when you add `gcp-pipeline` repo)

- **GCS** + **Databricks on GCP** or BigQuery for gold-only demos.
- Ephemeral: Dataproc/Databricks clusters; permanent: GCS buckets, service accounts (keys never in git; use ADC locally).

## Three-cloud portfolio without triple cost

1. **Build depth on Azure first** (storage → bronze → silver → gold → DLT).
2. **AWS/GCP repos** reuse the **same medallion naming** and generator; swap storage URIs and auth.
3. Run **scheduled jobs only before demos**; delete/destroy compute stacks after.
4. One **tiny dataset** everywhere — you're selling patterns, not petabytes.

## DAB vs Terraform

- **Terraform:** RG, storage, Key Vault, UC external locations, RBAC, optional cluster policies.
- **DAB (`databricks.yml`):** notebooks, jobs, pipelines, cluster job definitions — `bundle deploy` / `bundle destroy` for compute-linked assets.

Do not put PATs or storage keys in either file; use `vars` from CLI or CI secrets.
