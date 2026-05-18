# Step 2 — Manual upload from local `landing_data/azure`

Source files from the sandbox generator (parent project `empty-window/mock_landing`):

```text
...\empty-window\mock_landing\landing_data\azure\
  20260516_082903.csv
  20260516_082903.json
```

## PowerShell

```powershell
az account show --output table   # confirm subscription

$STORAGE_ACCOUNT = "YOUR_STORAGE_ACCOUNT_NAME"   # not the full URL
$CONTAINER       = "lake"
$PREFIX          = "landing"
$LOCAL           = "C:\Users\sri9u\.cursor\projects\empty-window\mock_landing\landing_data\azure"

az storage blob upload-batch `
  --account-name $STORAGE_ACCOUNT `
  --destination $CONTAINER `
  --destination-path $PREFIX `
  --source $LOCAL `
  --pattern "*.*" `
  --auth-mode login `
  --overwrite

az storage blob list `
  --account-name $STORAGE_ACCOUNT `
  --container-name $CONTAINER `
  --prefix "$PREFIX/" `
  --auth-mode login `
  --output table
```

## Git Bash

```bash
export AZURE_STORAGE_ACCOUNT_NAME="YOUR_STORAGE_ACCOUNT_NAME"
export AZURE_STORAGE_CONTAINER="lake"
export AZURE_BLOB_PREFIX="landing"
LOCAL="/c/Users/sri9u/.cursor/projects/empty-window/mock_landing/landing_data/azure"

az storage blob upload-batch \
  --account-name "$AZURE_STORAGE_ACCOUNT_NAME" \
  --destination "$AZURE_STORAGE_CONTAINER" \
  --destination-path "${AZURE_BLOB_PREFIX%/}" \
  --source "$LOCAL" \
  --pattern "*.*" \
  --auth-mode login \
  --overwrite
```

## Success criteria

- Portal shows blobs under `lake/landing/`
- `az storage blob list` returns your `.csv` and `.json` files
- No storage account keys used (`--auth-mode login` only)
