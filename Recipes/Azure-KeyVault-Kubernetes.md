# Key Vault and Kubernetes on the Edge

Managing secrets for Edge/Kubernetes clusters in DDIL environments (Denied, Disrupted, Intermittent, and Limited) can present unique challenges, particularly due to the potential loss of connectivity to these clusters. Utilizing standard patterns such as Pod Identities or accessing secrets via Key Vault SDKs may not function reliably if the Kubernetes cluster experiences disconnection from Azure.

Other methods of managing secrets, such as storing encrypted secrets in code repositories through use of tools such as [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets), are also not ideal, as they lack the key features needed to maintain the integrity of secrets, such as:

- Enforcement of secret rotation and expiry
- Fine-Grained access control

This document covers an implementation for injecting secrets from a Key Vault into an edge/on-prem cluster which may only have occasional connection to Azure Services, while still maintaining an Azure Key Vault as a central repository for all secrets management.

Kubernetes encompasses a [Secret Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/), which permits the integration of secrets from external repositories directly into pods as a volume. Upon attaching the volume to the pod, the information can be mounted into the container's filesystem and accessed accordingly. An [Azure Key Vault Provider for the Secret Store CSI Driver](https://learn.microsoft.com/azure/aks/csi-secrets-store-driver) is available, which enables the injection of secrets from Azure Key Vault into pods using the CSI Driver.

There are 2 main scenarios in which one would access secrets in a Kubernetes Context:

- [Key Vault and Kubernetes on the Edge](#key-vault-and-kubernetes-on-the-edge)
  - [Referencing Secrets from an application running inside a pod](#referencing-secrets-from-an-application-running-inside-a-pod)
  - [Referencing secret values in config files](#referencing-secret-values-in-config-files)

**_Please note that the following guidance makes use of Cluster Extensions for [Arc Enabled Kubernetes](https://learn.microsoft.com/azure/azure-arc/kubernetes/conceptual-extensions), this requires your Kubernetes Cluster to be onboarded with [Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/overview)_**

## Referencing Secrets from an application running inside a pod

In this scenario you would have a pod that's been deployed in your cluster, and needs secrets to be mounted into the pods volume. The [existing documentation](https://learn.microsoft.com/azure/azure-arc/kubernetes/tutorial-akv-secrets-provider) describing how to deploy the Key Vault Secrets Provider into an Arc-enabled Kubernetes Cluster can be followed. There is however, one recommended exception, and that is to enable Secret Rotation when deploying the Kubernetes Extension:

```bash
#!/bin/bash
az k8s-extension create --cluster-name $CLUSTER_NAME \
--resource-group $RESOURCE_GROUP \
--cluster-type connectedClusters \
--extension-type Microsoft.AzureKeyVaultSecretsProvider \
--name $EXTENSION_NAME \
--configuration-settings secrets-store-csi-driver.enableSecretRotation=true
--configuration-settings secrets-store-csi-driver.rotationPollInterval=2m
```

## Referencing secret values in config files

In this scenario, you would have a config file that needs to reference a Kubernetes Secret. You cannot directly use the secrets provider as done in the previous scenario, as a config file does not have the capability to access a mounted volume. As part of the Secrets Store CSI Driver, there is a feature to create [Kubernetes Secret Objects from secrets ingested by the provider](https://secrets-store-csi-driver.sigs.k8s.io/topics/sync-as-kubernetes-secret.html).

In order to achieve this, it is first recommended to the follow the [existing documentation](https://learn.microsoft.com/azure/azure-arc/kubernetes/tutorial-akv-secrets-provider), but with 2 minor modifications:

1. When installing the Key Vault Secrets Provider Extension, enable Key Rotation, and Sync Secrets:

    ```bash
    #!/bin/bash
    az k8s-extension create --cluster-name $CLUSTER_NAME \
    --resource-group $RESOURCE_GROUP \
    --cluster-type connectedClusters \
    --extension-type Microsoft.AzureKeyVaultSecretsProvider \
    --name akvsecretsprovider \
    --configuration-settings secrets-store-csi-driver.enableSecretRotation=true \
    --configuration-settings secrets-store-csi-driver.rotationPollInterval=2m \
    --configuration-settings secrets-store-csi-driver.syncSecret.enabled=true
    ```

1. After following the existing documentation, create a Kubernetes deployment that references this secrets provider and mounts the required secrets into its volume. This will trigger the creation of Kubernetes Secrets for everything ingested by the secrets provider, allowing it to then be referenced in various configuration files:

      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: secrets-sync-agent
        labels:
          app: secrets-sync-agent
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: secrets-sync-agent
        template:
          metadata:
            labels:
              app: secrets-sync-agent
          spec:
            volumes:
              - name: secrets-store-inline
                csi:
                  driver: secrets-store.csi.k8s.io
                  readOnly: true
                  volumeAttributes:
                    secretProviderClass: $SECRET_PROVIDER_CLASS_NAME
                  nodePublishSecretRef:
                    name: secrets-store-creds
            containers:
              - name: alpine
                image: alpine:latest
                imagePullPolicy: IfNotPresent
                command:
                  - "/bin/sleep"
                  - "10000"
                volumeMounts:
                  - name: secrets-store-inline
                    mountPath: "/mnt/secrets-store"
                    readOnly: true
      ```
