# GitHub Encrypted Secrets and GitHub

Encrypted Secrets allow you to store sensitive information in your organization, repository, or repository environments. Secrets are represented as environment variables.

## Actions secrets

Actions secrets are environment variables that are encrypted. Anyone with collaborator access to this repository can use these secrets for Actions.

### Organization secrets

For secrets stored at the organization level, you can use access policies to control which repositories can use organization secrets. In addition, organization-level secrets let you share secrets between multiple repositories, reducing the need to create duplicate secrets. Updating an organization secret in one location also ensures that the change takes effect in all repository workflows that use that secret.

### Repository secrets

Repository secrets are specific to a single repository and all environments used there.

### Environment secrets

For secrets stored at the environment level, you can enable required reviewers to control access to the secrets. A workflow job cannot access environment secrets until required approvers approve. You can use environment secrets if you have secrets that are specific to an environment.

## Using encrypted secrets in a workflow

Follow the documentation with an example of using Encrypted Secrets in your GitHub Workflow: [Using encrypted secrets in a workflow](https://docs.github.com/actions/security-guides/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

## Additional Resources

- [Encrypted Secrets](https://docs.github.com/actions/security-guides/encrypted-secrets)
