# Infrastructure as Code

## Getting Started

**Prerequisites:**

- Terraform > 0.14.7
- Azure cli > 2.19.1

**Deploy Steps:**

The following deploys the Azure base infrastructure for the Temperature Events sample. Ensure you are in `e2e_samples/dataset-versioning/infra/`.

1. `az login`
2. `./setup.sh`
3. Run terraform init with the output from step 2
4. `terraform validate && terraform plan && terraform apply`

### #1 `az login`

```bash
> az login
```

Authenticate with the Azure CLI, to ensure you are able interact with your Azure account.
<https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli>

Before running the setup.sh script, make sure you are using the right subscription.
You can check by using this command to get the list of subsciptions and see which one is the default:
```az account list --output table```
If you are using a different subscription and need to change it, use this command:
```az account set --subscription <your_subscription_id>```


### #2 `./setup.sh` to create Terraform support resources

This script will create a resource group, with 2 resources (Key Vault and Storage) to track the secrets and state of your infrastructure. It will also create a new Service Principal with contributor permissions for the Terraform app to use later.

```bash
# Setup infrastructure for Terrafrom state & secrets.
# NOTE: Change the values to a unique name for your account.
# ./setup.sh <resource group> <storage name> <Key Vault name> <region>
az login
./setup.sh rg-my-terraform stmyterraform kv-myterraform eastus2
```

Keep note of the output from `./setup.sh`. The output will be used in the next command

### #3 Run terraform init with the output from step 2

Use the output from the last command, to initialise the Terraform state files and store them in the Azure storage & Key Vault resources created in step 2.

```bash
# Note: This is an example, use the exact output given to you in step 2.
az login
cd terraform/live/dev

terraform init -backend-config="storage_account_name=stmyterraformdev" -backend-config="container_name=terraform-state" -backend-config="access_key=$(az keyvault secret show --name tfstate-storage-key-dev --vault-name kv-myterraform --query value -o tsv)" -backend-config="key=terraform.tfstate"
```

### #4 Use `terraform apply` to create the sample resources

Run the following commands to deploy the sample infrastructure.  
**Note:** Pick a globally unique name, do not use the value `"name=myapp"`.  e.g. `"name=bob123"`

```bash
# NOTE: enter your own unique value instead of `myapp`
terraform plan -var "name=myapp" -out=/tmp/myapp
terraform apply /tmp/myapp
```

## Before sending PR

1. Run [pre-commit](https://pre-commit.com/#install)
1. Make sure this README.md is updated

## Directory Structure/Provisioned resources

```zsh
$tree -d
.
└── terraform
    ├── live
    │   └── dev
    └── modules
        ├── data_factory
        ├── data_lake
        ├── databricks
        ├── keyvault
        ├── service
        └── sql_server
```

```bash
$ terraform state list
    module.service.data.azurerm_client_config.current
    module.service.data.azurerm_resource_group.rg
    module.service.module.azure_data_factory.azurerm_data_factory.data_factory
    module.service.module.azure_data_factory.azurerm_data_factory_linked_service_key_vault.df_kv_ls
    module.service.module.azure_data_factory.azurerm_key_vault_access_policy.principal_id
    module.service.module.data_lake.azurerm_key_vault_secret.datalake_access_key
    module.service.module.data_lake.azurerm_key_vault_secret.datalake_secret
    module.service.module.data_lake.azurerm_storage_account.storage_account
    module.service.module.data_lake.azurerm_storage_data_lake_gen2_filesystem.data_lake_filesystem
    module.service.module.databricks.azurerm_databricks_workspace.databricks
    module.service.module.keyvault.azurerm_key_vault.keyvault
    module.service.module.keyvault.azurerm_key_vault_access_policy.service_principal
    module.service.module.sql_server.azurerm_key_vault_secret.key_vault_secret
    module.service.module.sql_server.azurerm_mssql_database.mssql_database
    module.service.module.sql_server.azurerm_mssql_elasticpool.mssql_elasticpool
    module.service.module.sql_server.azurerm_sql_firewall_rule.sql_firewall_rule
    module.service.module.sql_server.azurerm_sql_server.sql_server
    module.service.module.sql_server.azurerm_storage_account.storage_account
    module.service.module.sql_server.random_id.storage_account
    module.service.module.sql_server.random_password.password
```

### Live

This is the entry point of terraform script. Terraform scripts under each environment (i.e. dev) calls service module with corresponding configurations.

### Modules

Modules are the building blocks of resources. A module is a wrapper with more than one resources. In this project, module needs to contain more than 1 resources.

## References

* [Cobalt project](https://github.com/microsoft/cobalt)