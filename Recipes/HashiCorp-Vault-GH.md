---
summary: HashiCorp Vault usage examples with GitHub Workflows
authors:
  - Dariusz Porowski
date: 2022-11-30
tags:
  - HashiCorp
  - Vault
  - Secrets Store
  - GitHub Workflows
  - Security
---

# HashiCorp Vault and GitHub

Official GitHub Action for getting secrets from HashiCorp Vault exists[^1], and this example shows you how to retrieve and use secrets in your GitHub Workflow.

[^1]: [HashiCorp Vault Action](https://github.com/marketplace/actions/vault-secrets)

## Get secrets from HashiCorp Vault using HashiCorp Vault Action with OpenID Connect (OIDC)

Official GitHub documentation contains an example configuration[^2], but in some places may need clarification. Therefore, the below code extends it for your convenience.

[^2]: [Configuring OpenID Connect in HashiCorp Vault
](https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-hashicorp-vault)

```yaml
name: Example HashiCorp Vault flow

on:
  push:
    branches:
      - main
  workflow_dispatch:

variables:
  - VAULT_ADDR: https://myvault.mycompany.com:8200

jobs:
  retrieve-secret:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      # checkout the repo
      - uses: actions/checkout@v3

      # Retrieve secret from Vault using OIDC
      - uses: hashicorp/vault-action@v2
        id: vaultSecrets
        with:
          exportToken: true
          method: jwt
          jwtGithubAudience: sigstore
          url: ${{ env.VAULT_ADDR }}
          role: github-role
          path: github
          secrets: |
            myProject/data/acr containerUsername ;
            myProject/data/acr containerPassword ;

      # login into Azure Container Registry
      - uses: azure/docker-login@v1
        with:
          login-server: myregistry.azurecr.io
          username: ${{ steps.vaultSecrets.outputs.containerUsername }}
          password: ${{ steps.vaultSecrets.outputs.containerPassword }}

      - name: Revoke Vault token
        # This step always runs at the end regardless of the previous steps result
        if: always()
        run: |
          curl -X POST -sv -H "X-Vault-Token: ${{ env.VAULT_TOKEN }}" \
            ${{ env.VAULT_ADDR }}/v1/auth/token/revoke-self

      # build and push Docker image
      - run: |
          docker build . -t myregistry.azurecr.io/myapp:${{ github.sha }}
          docker push myregistry.azurecr.io/myapp:${{ github.sha }}

      # deploy to Azure Web App
      - uses: azure/webapps-deploy@v2
        with:
          app-name: myapp
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          images: "myregistry.azurecr.io/myapp:${{ github.sha }}"
```

The script below will bootstrap your Vault configuration with a basic setup (OIDC GitHub auth for HashiCorp Vault Action) to use with GitHub Workflow above.

```bash
#!/bin/bash

# Basic Vault config
export VAULT_ADDR="https://myvault.mycompany.com:8200"
export VAULT_TOKEN="MyToken"

# Variables
GH_ORG="myCompany"               # GitHub User name or Organization name
GH_REPO="myRepo"                 # GitHub Repository name
SECRETS_ENGINE_NAME="myProject"  # Vault secrets engine name
ACR_STORE_NAME="acr"             # ACR Store name
ACR_USERNAME="myAcrUsername"     # ACR username
ACR_PASSWORD="myAcrPassword"     # ACR password

# Enable Vault Secrets Engine
vault secrets enable -version=2 -path="${SECRETS_ENGINE_NAME}" kv

# Put ACR creds into secrets store
vault kv put "${SECRETS_ENGINE_NAME}"/"${ACR_STORE_NAME}" containerUsername="${ACR_USERNAME}" containerPassword="${ACR_PASSWORD}"

# Enable JWT/OIDC auth method for GitHub
vault auth enable -path=github jwt

# Configure JWT/OIDC auth method for GitHub
vault write auth/github/config bound_issuer="https://token.actions.githubusercontent.com" oidc_discovery_url="https://token.actions.githubusercontent.com"

# Create read policy for GitHub with read-only access to secrets engine
vault policy write github-policy - <<EOF
# Read-only permission on '${SECRETS_ENGINE_NAME}/data/*' path

path "${SECRETS_ENGINE_NAME}/data/*" {
  capabilities = [ "read" ]
}
EOF

# Create role for GitHub JWT/OIDC auth method
vault write auth/github/role/github-role - <<EOF
{
  "role_type": "jwt",
  "user_claim": "actor",
  "bound_claims": {
    "repository": "${GH_ORG}/${GH_REPO}"
  },
  "policies": ["github-policy"],
  "ttl": "15m"
}
EOF
```

> **NOTE**
>
> Before proceeding, you must plan your security strategy to ensure that access tokens are only allocated predictably. To control how your cloud provider issues access tokens, you must define at least one condition so that untrusted repositories can't request access tokens for your cloud resources. For more information, see [Configuring the OIDC trust with the cloud](https://docs.github.com/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect#configuring-the-oidc-trust-with-the-cloud).

## Additional Resources

- [Using OIDC With HashiCorp Vault and GitHub Actions](https://youtu.be/lsWOx9bzAwY)
