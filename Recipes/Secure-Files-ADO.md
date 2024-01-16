---
summary: Secure Files usage examples with Azure DevOps Pipelines
authors:
  - Dariusz Porowski
date: 2022-11-29
tags:
  - Secure Files
  - Secrets Store
  - Azure Pipelines
  - Security
---

# Azure DevOps Secure Files

Secure files allow you to store files you can share across pipelines. Use the Secure Files library to store files like:

- signing certificates
- Apple Provisioning Profiles
- Android Keystore files
- SSH keys

These files can be stored on the server without committing them to your repository.

## Use Secure Files

Follow the documentation with an example of using Secure Files in your pipeline: [Use secure files](https://learn.microsoft.com/azure/devops/pipelines/library/secure-files)
