# Step 1 — Create landing storage in Azure Portal

Use subscription **sriramunnambasic** (or your active subscription). No secrets go in this repo.

## 1. Resource group

1. Portal → **Resource groups** → **Create**
2. **Subscription:** your subscription  
3. **Resource group:** e.g. `rg-lakehouse-dev`  
4. **Region:** pick one close to you (e.g. East US, Central India) — use the **same region** for storage and Databricks later  
5. **Review + create**

## 2. Storage account (ADLS Gen2)

1. Portal → **Storage accounts** → **Create**
2. **Basics**
   - Resource group: `rg-lakehouse-dev`
   - **Storage account name:** globally unique, e.g. `stlakehouse<yourinitials>dev` (lowercase, no spaces)
   - Region: same as resource group
   - Performance: **Standard**
   - Redundancy: **LRS** (cheapest for learning)
3. **Advanced**
   - **Enable hierarchical namespace:** **On** (required for ADLS Gen2 / good Delta layout)
4. **Data protection** — defaults OK for dev
5. **Networking** — **Enable public access** for now (simplest for CLI upload from laptop). Tighten later for production story.
6. **Review + create** → wait until deployed

Write down:

- Storage account name: `________________`
- Resource group: `rg-lakehouse-dev`

## 3. Blob container

1. Open the storage account → **Containers** (under Data storage)
2. **+ Container**
   - Name: `lake` (or match `AZURE_STORAGE_CONTAINER` in `.env.example`)
   - Public access: **Private**

Landing path convention:

```text
lake/landing/<yyyymmdd_hhmmss>.csv
lake/landing/<yyyymmdd_hhmmss>.json
```

## 4. IAM — allow your user to upload (data plane)

1. Storage account → **Access control (IAM)** → **Add** → **Add role assignment**
2. Role: **Storage Blob Data Contributor**
3. Members: **User, group, or service principal** → select your account (`sri9unnam@outlook.com`)
4. **Review + assign**

Wait 1–2 minutes for propagation.

## 5. Verify in Portal

After upload (next doc), you should see files under **Containers** → `lake` → `landing/`.

## Cost note

Storage + a few thousand small files is usually **cents/month**. Most spend will be **Databricks compute** when you run jobs — keep clusters off when idle.
