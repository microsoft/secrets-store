---
summary: HashiCorp Vault usage examples with Azure DevOps
authors:
  - Dariusz Porowski
date: 2022-12-29
tags:
  - HashiCorp
  - Vault
  - Secrets Store
  - Azure DevOps
  - Security
---

# HashiCorp Vault and Azure DevOps

In the absence of the official Azure DevOps Tasks Extension, the preferred integration method is Azure Auth Method - it allows authentication against Vault using [Azure Managed Service Identity (MSI)](https://learn.microsoft.com/en-us/azure/active-directory/managed-service-identity/overview). This method supports authentication for System-assigned and User-assigned managed identities. The limitation of this method is it's only available to private Azure DevOps agents running on Azure. Configuration for this scenario is pretty well documented on HashiCorp sites. Please follow this guide: [Azure Auth Method](https://developer.hashicorp.com/vault/docs/auth/azure).

If in your solution, you would like to use hosted agents or private agents not run on Azure (e.g., on-premises) then the AppRole auth method is your answer. This guide shows you how to use AppRole[^1] auth method to extract secrets from HashiCorp Vault.

[^1]: [HashiCorp Vault AppRole Auth Method](https://developer.hashicorp.com/vault/docs/auth/approle)

## Get secrets from HashiCorp Vault using AppRole auth method

The example below uses Azure DevOps [Generic Service Connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#generic-service-connection) to store AppRole access details to the Vault, and community extension ([Authenticated Scripts](https://marketplace.visualstudio.com/items?itemName=cloudpup.authenticated-scripts)) to utilize Generic Service Connection data inside a Bash script.

{% raw %}

```yaml
trigger:
  - main

name: Example HashiCorp Vault flow

pool:
  vmImage: ubuntu-latest

steps:
  - script: |
      wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
      echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
      sudo apt update && sudo apt install vault
      vault --version
    displayName: Setup Vault CLI

  - task: AuthenticatedBash@1
    displayName: Get secrets from Vault
    inputs:
      serviceConnection: "Vault Service Connection"
      targetType: "inline"
      script: |
        # Set Vault URL
        export VAULT_ADDR="${AS_SC_URL}"

        # Get AppRole token and set Vault token
        export VAULT_TOKEN==$(vault write auth/ado-approle/login role_id="${AS_SC_USERNAME}" secret_id="${AS_SC_PASSWORD}" -format="json" | jq -rc '.auth.client_token')

        # Get secrets
        containerUsername=$(vault kv get -field=containerUsername myProject/acr)
        containerPassword=$(vault kv get -field=containerPassword myProject/acr)

        # Set secrets for next steps
        echo "##vso[task.setvariable variable=containerUsername;issecret=true]${containerUsername}"
        echo "##vso[task.setvariable variable=containerPassword;issecret=true]${containerPassword}"

  - script: |
      docker login myregistry.azurecr.io --username $(containerUsername) --password $(containerPassword)
    displayName: Login to Docker Registry
```

{% endraw %}

The script below will bootstrap your environment with an example configuration that is required for the above Azure Pipelines.

- Vault configuration with a AppRole auth
- Vault KVv2 store with examples secrets
- Azure DevOps extensions setup
- Azure DevOps Generic Service Connection setup

{% raw %}

```bash
#!/bin/bash

# Variables
AZURE_DEVOPS_ORGANIZATION_URL="https://dev.azure.com/MyAdoOrg"
AZURE_DEVOPS_PROJECT_NAME="MyAdoProject"
AZURE_DEVOPS_SERVICE_ENDPOINT_NAME="Vault Service Connection" # Azure DevOps Service Connection Name

# Basic Vault config
export VAULT_ADDR="https://myvault.mycompany.com:8200" # Vault address
export VAULT_TOKEN="MyToken"                           # Vault token

# Secrets config
VAULT_SECRETS_ENGINE_NAME="myProject" # Vault secrets engine name
ACR_STORE_NAME="acr"                  # ACR Store name
ACR_USERNAME="myAcrUsername"          # ACR username
ACR_PASSWORD="myAcrPassword"          # ACR password

# Helper for Azure CLI DevOps Extension check/setup
function _setup_az_cli_ado_extension() {
    if ! az extension show --name azure-devops >/dev/null; then
        az extension add --name azure-devops
    else
        az extension update --name azure-devops
    fi
}

# Helper for Azure DevOps Extensions
function _setup_ado_extensions() {
    # Install Authenticated Scripts extension (https://marketplace.visualstudio.com/items?itemName=cloudpup.authenticated-scripts)
    az devops extension install --publisher-id cloudpup --extension-id authenticated-scripts --organization "${AZURE_DEVOPS_ORGANIZATION_URL}" --project "${AZURE_DEVOPS_PROJECT_NAME}"
}

# Enable Vault Secrets Engine and put ACR creds into secrets store
function setup_vault_secrets_engine() {
    # Enable Vault Secrets Engine
    vault secrets enable -version=2 -path="${VAULT_SECRETS_ENGINE_NAME}" kv

    # Put ACR creds into secrets store
    vault kv put "${VAULT_SECRETS_ENGINE_NAME}"/"${ACR_STORE_NAME}" containerUsername="${ACR_USERNAME}" containerPassword="${ACR_PASSWORD}"
}

# Enable AppRole auth method and create role for Azure DevOps
function setup_vault_approle() {
    # Enable AppRole auth method for # Vault address
    vault auth enable -path=ado-approle approle

    # Create read policy for Azure DevOps with read-only access to secrets engine
    vault policy write azure-policy - <<EOF
# Read-only permission on '${VAULT_SECRETS_ENGINE_NAME}/data/*' path

path "${VAULT_SECRETS_ENGINE_NAME}/data/*" {
  capabilities = [ "read" ]
}
EOF

    # Create role for Azure DevOps AppRole auth method
    vault write auth/ado-approle/role/azure-role token_policies="azure-policy" token_ttl=30m token_max_ttl=1h

    # Get AppRole RoleID and SecretID
    role_id=$(vault read auth/ado-approle/role/azure-role/role-id -format="json" | jq -rc '.data.role_id')
    secret_id=$(vault write -f auth/ado-approle/role/azure-role/secret-id -format="json" | jq -rc '.data.secret_id')

    # Create Azure DevOps Service Connection temporarily configuration file
    cat <<EOF >.AdoServiceConnectionGeneric.json
{
  "data": {},
  "name": "${AZURE_DEVOPS_SERVICE_ENDPOINT_NAME}",
  "type": "Generic",
  "url": "${VAULT_ADDR}",
  "authorization": {
    "parameters": {
      "username": "${role_id}",
      "password": "${secret_id}"
    },
    "scheme": "UsernamePassword"
  },
  "isShared": false,
  "isReady": true
}
EOF
}

# Create Azure DevOps Generic Service Connection for Vault
function setup_ado_service_connection() {
    _setup_az_cli_ado_extension
    _setup_ado_extensions

    svccon_id=$(az devops service-endpoint create --service-endpoint-configuration ./.AdoServiceConnectionGeneric.json --organization "${AZURE_DEVOPS_ORGANIZATION_URL}" --project "${AZURE_DEVOPS_PROJECT_NAME}" --output tsv --query id)

    # enable service connection for all pipelines (optional)
    az devops service-endpoint update --id "${svccon_id}" --enable-for-all true --organization "${AZURE_DEVOPS_ORGANIZATION_URL}" --project "${AZURE_DEVOPS_PROJECT_NAME}"

    # Cleanup temporary configuration file
    rm -f .AdoServiceConnectionGeneric.json
}

# Execute
setup_vault_secrets_engine
setup_vault_approle
setup_ado_service_connection
```

{% endraw %}

> **NOTE**
>
> Before proceeding, you must plan your security strategy to ensure that access tokens are only allocated predictably. The attached script does only basic configuration for example purposes. Read more about Vault AppRole security and design before you run in production.
>
> - [AppRole Usage Best Practices](https://developer.hashicorp.com/vault/tutorials/auth-methods/approle-best-practices)
> - [How (and Why) to Use AppRole Correctly in HashiCorp Vault](https://www.hashicorp.com/blog/how-and-why-to-use-approle-correctly-in-hashicorp-vault)

## Additional Resources

- [AppRole Pull Authentication](https://developer.hashicorp.com/vault/tutorials/auth-methods/approle)
- [Authenticating Applications with HashiCorp Vault AppRole](https://www.hashicorp.com/blog/authenticating-applications-with-vault-approle)
