# Secrets Store

A secrets store is a secure location for storing your project's secrets and making them available as required to various parts of a project. This may include usage in apps, services, APIs, or deployment processes.

## Problem Statement

Almost every application is part of a larger ecosystem that connects with other services like databases, APIs, etc. Before the "Secure Secret Store" era, secrets used for connection to other services were stored in code, configuration files, initialization parameters, version control repositories, external files, databases, etc. That approach experiences "Secrets Sprawl", which refers the state in an organization where secrets are stored in many different places without access control, audit, logging, and many cases without encryption.

## Solution

Secrets should not to be include in source code, nor checked into git. All sensitive pieces of configuration should be kept in a secure, managed secrets store. Benefits include:

- Centralized location for storing secrets, keys, and certificates.
- Long-term scalability of storage location.
- Automatic encryption when stored and decription when retrieved.
- Enabling automated logging when secrets are stored and used, and by whom.
- Secure access from DevOps workflows and automation as required.
- Removes dependency on direct access to secrets during development and in production.

## Tool Selection Matrix

The table below has populated a selection of tools covering similar functionality.

It is important to note that other factors may influence your choices, such as the availability of an official Azure DevOps Extension / GitHub Action that can be used in a Workflow or its ability to be used from a shell, among other things.

| Tools                                                                                                                  | Use Case                                                                                                   | Command Line                                                                                                                                                                  | GitHub Action                                                                                                             | Azure DevOps Extension / Task                                                                                           | Service                                                                                                                                                     |
|------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Azure Key Vault](https://learn.microsoft.com/azure/key-vault/)                                                        | Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets. | [Azure CLI](https://learn.microsoft.com/azure/key-vault/secrets/quick-create-cli) / [PowerShell](https://learn.microsoft.com/azure/key-vault/secrets/quick-create-powershell) | No / Yes (via [Azure CLI Action](https://github.com/marketplace/actions/azure-cli-action))                                | [Azure Key Vault Task](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/azure-key-vault-v2)           | Cloud-managed ([Azure](https://azure.microsoft.com/))                                                                                                       |
| [HashiCorp Vault](https://www.vaultproject.io)                                                                         | Securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets. | [Vault CLI](https://developer.hashicorp.com/vault/docs/commands)                                                                                                              | [Vault Secrets Action](https://github.com/marketplace/actions/vault-secrets)                                              | No / Yes (via [Vault CLI](https://developer.hashicorp.com/vault/docs/commands))                                         | Self-managed / Cloud-managed ([HashiCorp Cloud Platform (HCP)](https://cloud.hashicorp.com/))                                                               |
| [GitHub Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)                       | Securely store tokens, passwords, API keys, and other secrets for GitHub Workflows.                        | [GH CLI](https://cli.github.com/manual/gh_secret)                                                                                                                             | [Native Support](https://docs.github.com/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow) | No / Yes (via [GH CLI](https://cli.github.com/manual/gh_secret))                                                        | Cloud-managed ([GitHub](https://github.com/))                                                                                                               |
| [Azure DevOps Secure Files](https://learn.microsoft.com/azure/devops/pipelines/library/secure-files?view=azure-devops) | Securely store certificates, provisioning profiles, keystore files, SSH keys for Azure Pipelines.          | N/A                                                                                                                                                                           | N/A                                                                                                                       | [Download Secure File Task](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/download-secure-file-v1) | Self-managed ([Azure DevOps Server](https://azure.microsoft.com/products/devops/server/)) / Cloud-managed ([Azure DevOps Services](https://dev.azure.com/)) |

## Implementation examples

The links below will direct you to implementation examples that showcase common scenarios.

- [Azure Key Vault usage examples with GitHub Workflows](Recipes/Azure-KeyVault-GH.md)
- [Azure Key Vault usage examples with Azure DevOps Pipelines](Recipes/Azure-KeyVault-ADO.md)
- [HashiCorp Vault usage examples with GitHub Workflows](Recipes/HashiCorp-Vault-GH.md)
- [HashiCorp Vault usage examples with Azure DevOps Pipelines](Recipes/HashiCorp-Vault-ADO.md)
- [GitHub Encrypted Secrets usage examples with GitHub Workflows](Recipes/GitHub-Encrypted-Secrets.md)
- [Secure Files usage examples with Azure DevOps Pipelines](Recipes/Secure-Files-ADO.md)
- [Azure Key Vault usage examples with Edge Kubernetes Clusters](Recipes/Azure-KeyVault-Kubernetes.md)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit <https://cla.opensource.microsoft.com>.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow [Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
