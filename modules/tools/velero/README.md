# Azure Resource Group
[![Changelog](https://img.shields.io/badge/changelog-release-green.svg)](CHANGELOG.md) [![Notice](https://img.shields.io/badge/notice-copyright-yellow.svg)](NOTICE) [![Apache V2 License](https://img.shields.io/badge/license-Apache%20V2-orange.svg)](LICENSE) [![TF Registry](https://img.shields.io/badge/terraform-registry-blue.svg)](https://registry.terraform.io/modules/claranet/aks/azurerm)

This module deploy Velero on an existing K8S cluster via Helm 3 and the associated Block blob storage account.

## Version compatibility

| Module version    | Terraform version | Helm version | Kubernetes version |
|-------------------|-------------------|--------------|--------------------|
| >= 3.x.x          | 0.12.x            | = 1.1.1      | ~> 1.11.1          |
| >= 2.x.x, < 3.x.x | 0.12.x            | N/A          | N/A                |
| <  2.x.x          | 0.11.x            | N/A          | N/A                |

## Usage

```hcl
module "rg" {
  source  = "claranet/rg/azurerm"
  version = "3.0.0"

  location    = module.azure-region.location
  client_name = var.client_name
  environment = var.environment
  stack       = var.stack

  custom_rg_name = local.support_bastion_rg_name
}

module "azure-region" {
  source  = "claranet/regions/azurerm"
  version = "3.0.0"

  azure_region = var.azure_region
}

module "velero" {
  source = "claranet/aks/azurerm//modules/tools/velero"

  enable_velero = var.enable_velero

  client_name    = var.client_name
  stack          = var.stack
  environment    = var.environment
  location       = module.azure-region.location
  location_short = module.azure-region.location_short
  name_prefix    = var.name_prefix

  resource_group_name           = module.rg.resource_group_name
  aks_nodes_resource_group_name = var.aks_nodes_resource_group_name
  service_principal             = var.service_principal
  nodes_subnet_id               = var.nodes_subnet_id

  velero_namespace        = var.velero_namespace
  velero_chart_version    = var.velero_chart_version
  velero_values           = var.velero_values
  velero_storage_settings = var.velero_storage_settings
}

```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| aks\_nodes\_resource\_group\_name | Name of AKS nodes resource group | `string` | n/a | yes |
| client\_name | Client name/account used in naming | `string` | n/a | yes |
| enable\_velero | Enable velero on AKS cluster | `bool` | `true` | no |
| environment | Project environment | `string` | n/a | yes |
| location | Azure region to use | `string` | n/a | yes |
| location\_short | Short name of Azure regions to use | `string` | n/a | yes |
| name\_prefix | prefix used in naming | `string` | `""` | no |
| nodes\_subnet\_id | Id of the subnet used for nodes | `string` | n/a | yes |
| resource\_group\_name | Name of the resource group for Velero's Storage Account | `string` | n/a | yes |
| service\_principal | Service principal used by AKS to interract with Azure API | <pre>object({<br>    client_id     = string,<br>    client_secret = string,<br>    object_id     = string<br>  })</pre> | n/a | yes |
| stack | Project stack name | `string` | n/a | yes |
| velero\_chart\_version | Velero helm chart version to use | `string` | `"2.7.3"` | no |
| velero\_namespace | Kubernetes namespace in which to deploy Velero | `string` | `"system-velero"` | no |
| velero\_storage\_settings | Settings for Storage account and blob container for Velero <br /><br><pre><br>map(object({ <br /><br>  name                     = string <br /><br>  resource\_group\_name      = string <br /><br>  location                 = string <br /><br>  account\_tier             = string <br /><br>  account\_replication\_type = string <br /><br>  tags                     = map(any) <br /><br>  allowed\_cirds            = list(string) <br /><br>  container\_name           = string <br /><br>}))<br /><br></pre> | `map(any)` | `{}` | no |
| velero\_values | Settings for Velero helm chart<br><br><pre><br>map(object({ <br /><br>  configuration.backupStorageLocation.bucket                = string <br /><br>  configuration.backupStorageLocation.config.resourceGroup  = string <br /><br>  configuration.backupStorageLocation.config.storageAccount = string <br /><br>  configuration.backupStorageLocation.name                  = string <br /><br>  configuration.provider                                    = string <br /><br>  configuration.volumeSnapshotLocation.config.resourceGroup = string <br /><br>  configuration.volumeSnapshotLocation.name                 = string <br /><br>  credential.exstingSecret                                  = string <br /><br>  credentials.useSecret                                     = string <br /><br>  deployRestic                                              = string <br /><br>  env.AZURE\_CREDENTIALS\_FILE                                = string <br /><br>  metrics.enabled                                           = string <br /><br>  rbac.create                                               = string <br /><br>  schedules.daily.schedule                                  = string <br /><br>  schedules.daily.template.includedNamespaces               = string <br /><br>  schedules.daily.template.snapshotVolumes                  = string <br /><br>  schedules.daily.template.ttl                              = string <br /><br>  serviceAccount.server.create                              = string <br /><br>  snapshotsEnabled                                          = string <br /><br>  initContainers[0].name                                    = string <br /><br>  initContainers[0].image                                   = string <br /><br>  initContainers[0].volumeMounts[0].mountPath               = string <br /><br>  initContainers[0].volumeMounts[0].name                    = string <br /><br>  image.repository                                          = string <br /><br>  image.tag                                                 = string <br /><br>  image.pullPolicy                                          = string <br /><br><br>}))<br /><br></pre> | `map(string)` | `{}` | no |

## Related documentation

- Velero documentation : [velero.io/docs/v1.3.2/](https://velero.io/docs/v1.3.2/)
- Velero Azure plugin documentation : [github.com/vmware-tanzu/velero-plugin-for-microsoft-azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure)
- BlockBlob storage account documentation : [docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-create-account-block-blob?tabs=azure-portal](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-create-account-block-blob?tabs=azure-portal)