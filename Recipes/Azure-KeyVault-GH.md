---
summary: Azure Key Vault usage examples with GitHub Workflows
authors:
  - Dariusz Porowski
date: 2022-11-29
tags:
  - Azure Key Vault
  - Key Vault
  - Secrets Store
  - GitHub Workflows
  - Security
---

# Key Vault and GitHub

Official GitHub Action for getting secrets form Key Vault exists[^1] but is deprecated. The recommended alternative is to use the [Azure CLI Action](https://github.com/marketplace/actions/azure-cli-action) and pass a custom script to access Azure Key Vault.

[^1]: [Azure Key Vault Action](https://github.com/Azure/get-keyvault-secrets)

## Get secrets from Key Vault using Azure CLI Action

Follow the documentation with an example of using Azure Key Vault in your GitHub Workflow: [Use Key Vault secrets in GitHub Actions workflows](https://learn.microsoft.com/azure/developer/github/github-key-vault) with one exception. Documentation shows an example workflow with the deprecated Azure Key Vault Action mentioned above. Instead of providing the YAML code from docs, use the below with Azure CLI Action.

```yaml
name: Example Key Vault flow

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # checkout the repo
      - uses: actions/checkout@v3

      # login to Azure with OIDC
      - uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      # masking sensitive values prevents a string or variable from being printed in the log
      - uses: azure/CLI@v1
        with:
          inlineScript: |
            echo "::add-mask::$(az keyvault secret show --output tsv --vault-name containervault --name containerUsername --query value)"
            echo "::add-mask::$(az keyvault secret show --output tsv --vault-name containervault --name containerPassword --query value)"

      # get secrets from Key Vault
      - uses: azure/CLI@v1
        id: myGetSecretAction
        with:
          inlineScript: |
            echo "containerUsername=$(az keyvault secret show --output tsv --vault-name containervault --name containerUsername --query value)" >> $GITHUB_OUTPUT
            echo "containerPassword=$(az keyvault secret show --output tsv --vault-name containervault --name containerPassword --query value)" >> $GITHUB_OUTPUT

      # login into Azure Container Registry
      - uses: azure/docker-login@v1
        with:
          login-server: myregistry.azurecr.io
          username: ${{ steps.myGetSecretAction.outputs.containerUsername }}
          password: ${{ steps.myGetSecretAction.outputs.containerPassword }}

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

> **NOTE**
>
> ***OIDC***
>
> OpenID Connect (OIDC) is used in this example. It's a preferred authentication method and allows your GitHub Actions workflows to access resources in Azure without needing to store the Azure credentials as long-lived GitHub secrets. Read more about [Configuring OpenID Connect in Azure](https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure).
>
> ***Masking***
>
> The example above uses the GitHub masking feature that prevents a string or variable from being printed in the log. Each masked word separated by whitespace is replaced with the `*` character. Read more about [Masking a value in log](https://docs.github.com/actions/using-workflows/workflow-commands-for-github-actions#masking-a-value-in-log).
