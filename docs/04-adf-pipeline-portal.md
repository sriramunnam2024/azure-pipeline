# ADF pipeline - 3 Databricks notebooks (Portal)

Manual landing upload stays outside ADF for now.

## Prerequisites

- Resource group: 
g-lakehouse-dev
- Databricks workspace: dbw-lakehouse-dev
- Notebooks in workspace (import or Repo):
  - ronze_autoloader
  - silver_load
  - gold_load
- Landing files already in lake/landing/ before you trigger the pipeline

## 1. Create Data Factory (if you do not have one in this RG)

1. Portal -> Create a resource -> **Data Factory**
2. Name: df-lakehouse-dev
3. Resource group: **rg-lakehouse-dev**
4. Region: **South India**
5. Version: V2
6. Git: Off for now
7. Create

Open **Launch Studio**.

## 2. Linked service - Azure Databricks

1. **Manage** -> **Linked services** -> **+ New**
2. Search **Azure Databricks** -> Continue
3. Name: ls_databricks_lakehouse
4. Connect via: **Azure subscription**
5. Azure Databricks account / workspace: pick **dbw-lakehouse-dev**
6. Authentication (learning-friendly):
   - **Account** access (recommended if available), or
   - **Managed identity** (grant ADF access to workspace - see step 2b)
7. Test connection -> Save

### 2b. If using Managed identity

1. ADF factory -> **Identity** -> copy **Object (principal) ID**
2. Databricks workspace dbw-lakehouse-dev -> **Access control (IAM)** ->
   **Contributor** for that managed identity
3. Retry Test connection

## 3. Import notebooks to Databricks (if not already)

Workspace -> your user folder -> **Import** the three .py notebooks.

Note each **Workspace path**, e.g.:

/Users/sri9unnam@outlook.com/lakehouse/bronze_autoloader

## 4. Create pipeline pl_medallion_manual_landing

1. **Author** -> **+** -> **Pipeline**
2. Name: pl_medallion_manual_landing

### Activity 1 - Bronze

1. Activities -> **Databricks** -> **Notebook** -> drag to canvas
2. Name: 
b_bronze
3. **Azure Databricks** tab:
   - Linked service: ls_databricks_lakehouse
   - Notebook path: browse to ronze_autoloader
4. **Cluster** tab:
   - **New job cluster** (do not use existing personal cluster)
   - Databricks runtime: **14.3 LTS**
   - Node type: smallest that fits quota (**D2** / **D4**)
   - Workers: **0** (single-node job) if allowed
   - Auto termination: **10** minutes
5. **Base parameters** (match notebook widgets):

| Name | Value |
|------|--------|
| catalog_name | dbw_lakehouse_dev |
| bronze_schema | cloud_practice_bronze |
| (others per bronze notebook widgets) | |

### Activity 2 - Silver

1. Add second **Databricks Notebook**
2. Name: 
b_silver
3. Notebook: silver_load
4. Same linked service + **same new job cluster settings** (or reuse job cluster policy)
5. **Depends on**: 
b_bronze (green arrow)
6. Base parameters:

| Name | Value |
|------|--------|
| catalog_name | dbw_lakehouse_dev |
| bronze_schema | cloud_practice_bronze |
| silver_schema | cloud_practice_silver |

### Activity 3 - Gold

1. Add third **Databricks Notebook**
2. Name: 
b_gold
3. Notebook: gold_load
4. **Depends on**: 
b_silver
5. Base parameters:

| Name | Value |
|------|--------|
| catalog_name | dbw_lakehouse_dev |
| silver_schema | cloud_practice_silver |
| gold_schema | cloud_practice_gold |

6. **Publish all** (Publish branch)

## 5. Run

1. Upload new landing files manually (z upload-batch or script)
2. ADF -> pipeline -> **Add trigger** -> **Trigger now**
3. **Monitor** -> Pipeline runs -> open run -> each activity output/logs

## 6. Cost / quota tips

- One **job cluster** per activity = 3 short cluster lifetimes per run (or use one cluster spec - still separate jobs)
- Keep **Workers = 0**, small node, auto-terminate 10 min
- Do not start SQL warehouse during ADF run
- vCPU quota: ensure enough cores in South India for job cluster size

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Linked service test fails | MI Contributor on Databricks workspace or use account connector |
| Notebook not found | Path must match Workspace import path exactly |
| Cluster quota exceeded | Smaller node or request vCPU quota increase |
| Bronze empty | Upload files to landing/ before trigger |
