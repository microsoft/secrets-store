---
tags:
  - Secrets Store
  - Secrets Storage
  - Keys Store
  - Keys Storage
  - Security
---

# Secrets Store

## Overview

A secrets store is a secure location for storing your project's secrets and making them available as required to various parts of a project. This may include usage in apps, services, APIs, or deployment processes.

## Problem Statement

Almost every application is part of a larger ecosystem that connects with other services like databases, APIs, etc. Before the "Secure Secret Store" era, secrets used for connection to other services were stored in code, configuration files, initialization parameters, version control repositories, external files, databases, etc. That approach experiences "Secrets Sprawl", and refers to when an organization stores secrets in many different places without access control, audit, logging, and many cases without encryption.

## Solution

The best practice is not to include secrets in source code and to make sure not to store secrets in source control. All sensitive pieces of configuration should be kept in a secure place in a managed way. The Secret Store solves the problem and additionally brings many benefits such as:

- encryption/decryption when used
- audit/logging - who accessed the secret and when
- a central place storing secrets, keys, or certificates to support long-term scalability
- easy access to automation like DevOps workflows to eliminate certain users' dependencies

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
