# Terraform (permanent infra + optional ephemeral compute)

## Layout (planned)

```text
infra/terraform/
  envs/dev/
    main.tf
    variables.tf
    terraform.tfvars.example   # copy to terraform.tfvars (gitignored)
  modules/
    storage/                   # RG, ADLS, container, landing prefix
    databricks-access/         # RBAC, optional UC external location inputs
    compute/                   # OPTIONAL — job cluster policies; destroy when idle
```

## Local workflow

```bash
cd infra/terraform/envs/dev
cp terraform.tfvars.example terraform.tfvars   # edit locally, never commit
az login
terraform init
terraform plan
terraform apply
```

## Destroy discipline

- `terraform destroy -target=module.compute` — drop ephemeral compute only
- Avoid destroying `module.storage` if Delta data and landing history should remain

Placeholder files will be added when you start Terraform; Portal setup is fine for the first upload test.
