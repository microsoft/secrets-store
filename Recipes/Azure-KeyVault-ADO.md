# Key Vault and Azure DevOps

Two common scenarios exist for Key Vault usage in Azure Pipelines. Directly in the pipeline or with [Variable Groups](https://learn.microsoft.com/azure/devops/pipelines/library/variable-groups) as a proxy to Kay Vault. The most significant difference between them is reusability. It's a factor that determines your choice.

- [Azure Key Vault Task](#azure-key-vault-task) is suitable for a one-time job. For this case, Variable Group configuration is not needed.
- [Variable Group linked to Key Vault](#variable-group-linked-to-key-vault) brings more value if any need exists to reuse data (or it's a plan to reuse it in the future) from the Kay Vault across multiple pipelines or jobs.

## Azure Key Vault Task

Follow the documentation with an example of how to use Azure Key Vault Task in your pipeline: [Use Azure Key Vault secrets in Azure Pipelines](https://learn.microsoft.com/azure/devops/pipelines/release/azure-key-vault?view=azure-devops&tabs=yaml)

## Variable Group linked to Key Vault

Follow the documentation with an example of how to link Azure Key Vault with Variable Group and use it in your pipeline: [Link secrets from an Azure Key Vault to the Variable Groups](https://learn.microsoft.com/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml#link-secrets-from-an-azure-key-vault)
