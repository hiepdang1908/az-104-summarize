# AZ-104 Objective Map

This map uses the official **AZ-104: Microsoft Azure Administrator** skills measured as of **April 17, 2026**.

## Status Rules

- **COMPLETE**: the linked guide explains the definition, purpose, core behavior, material constraints, and relevant comparison or trap.
- **PARTIAL**: the topic is present, but a narrowly scoped knowledge gap remains.
- **MISSING**: no meaningful knowledge coverage exists.

This is a knowledge-first map. A status does not depend on a lab, practice question, or scenario exercise.

## Manage Azure Identities and Governance (15)

| Objective | Guide section | Status |
|---|---|---|
| Create users and groups | [Identity: Users and Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#users-and-groups) | COMPLETE |
| Manage user and group properties | Identity: User Properties and Groups | COMPLETE |
| Manage licenses in Microsoft Entra ID | Identity: License Management | COMPLETE |
| Manage external users | Identity: External Users (Microsoft Entra B2B) | COMPLETE |
| Configure self-service password reset | Identity: Self-Service Password Reset | COMPLETE |
| Manage built-in Azure roles | Identity: Azure RBAC / Role Types | COMPLETE |
| Assign roles at different scopes | Identity: Azure RBAC / Scope Hierarchy | COMPLETE |
| Interpret access assignments | Identity: Azure RBAC / Control Plane vs. Data Plane | COMPLETE |
| Implement and manage Azure Policy | Identity: Azure Policy | COMPLETE |
| Configure resource locks | Identity: Resource Locks | COMPLETE |
| Apply and manage tags | Identity: Tags | COMPLETE |
| Manage resource groups | Identity: Resource Groups | COMPLETE |
| Manage subscriptions | Identity: Azure Subscriptions | COMPLETE |
| Manage costs with alerts, budgets, and Azure Advisor | Identity: Cost Management | COMPLETE |
| Configure management groups | Identity: Management Groups | COMPLETE |

## Implement and Manage Storage (17)

| Objective | Guide section | Status |
|---|---|---|
| Configure Azure Storage firewalls and virtual networks | [Storage: Storage Networking Security](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-networking-security) | COMPLETE |
| Create and use SAS tokens | Storage: Shared Access Signature | COMPLETE |
| Configure stored access policies | Storage: Shared Access Signature | COMPLETE |
| Manage access keys | Storage: Account Key | COMPLETE |
| Configure identity-based access for Azure Files | Storage: Azure Files / Identity-Based Authentication | COMPLETE |
| Create and configure storage accounts | Storage: Azure Storage Account | COMPLETE |
| Configure Azure Storage redundancy | Storage: Storage Redundancy | COMPLETE |
| Configure object replication | Storage: Blob Replication | COMPLETE |
| Configure storage account encryption | Storage: Encryption | COMPLETE |
| Manage data using Storage Explorer and AzCopy | Storage: Managing Data with AzCopy and Storage Explorer | COMPLETE |
| Create and configure an Azure file share | Storage: Azure Files | COMPLETE |
| Create and configure a Blob Storage container | Storage: Blob Containers and Structure | COMPLETE |
| Configure storage tiers | Storage: Access Tiers | COMPLETE |
| Configure soft delete for blobs and containers | Storage: Soft Delete | COMPLETE |
| Configure snapshots and soft delete for Azure Files | Storage: File Share Snapshots and File Share Soft Delete | COMPLETE |
| Configure blob lifecycle management | Storage: Blob Lifecycle Management | COMPLETE |
| Configure blob versioning | Storage: Blob Versioning | COMPLETE |

## Deploy and Manage Azure Compute Resources (24)

| Objective | Guide section | Status |
|---|---|---|
| Interpret an ARM template | [Compute: ARM Templates](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-templates) | COMPLETE |
| Modify an ARM template | Compute: Modifying ARM Templates | COMPLETE |
| Modify a Bicep file | Compute: Bicep | COMPLETE |
| Deploy ARM templates or Bicep files | Compute: Deployment | COMPLETE |
| Export a deployment or convert ARM to Bicep | Compute: ARM Template Export | COMPLETE |
| Create a virtual machine | Compute: Virtual Machines | COMPLETE |
| Configure encryption at host | Compute: Encryption at Host | COMPLETE |
| Move a virtual machine | Compute: Moving VMs | COMPLETE |
| Manage virtual machine sizes | Compute: VM Sizing | COMPLETE |
| Manage virtual machine disks | Compute: VM Disks and Managed Disks | COMPLETE |
| Deploy to availability zones and availability sets | Compute: Virtual Machine Availability | COMPLETE |
| Deploy and configure Virtual Machine Scale Sets | Compute: VM Scale Set | COMPLETE |
| Create and manage Azure Container Registry | Compute: Azure Container Registry | COMPLETE |
| Provision a container with Azure Container Instances | Compute: Azure Container Instances | COMPLETE |
| Provision a container with Azure Container Apps | Compute: Azure Container Apps | COMPLETE |
| Manage container sizing and scaling | Compute: Container Sizing and Resource Limits | COMPLETE |
| Provision an App Service plan | Compute: App Service Plan | COMPLETE |
| Configure App Service plan scaling | Compute: App Service Plan | COMPLETE |
| Create an App Service | Compute: App Service | COMPLETE |
| Configure certificates and TLS for App Service | Compute: TLS/SSL Certificates | COMPLETE |
| Map an existing custom DNS name to App Service | Compute: Custom DNS Names and Domain Mapping | COMPLETE |
| Configure App Service backup | Compute: App Service Backup | COMPLETE |
| Configure App Service networking | Compute: App Service Networking | COMPLETE |
| Configure App Service deployment slots | Compute: Deployment Slots | COMPLETE |

## Implement and Manage Virtual Networking (13)

| Objective | Guide section | Status |
|---|---|---|
| Create and configure virtual networks and subnets | [Networking: VNet and Subnets](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-and-subnets) | COMPLETE |
| Create and configure virtual network peering | Networking: VNet Peering | COMPLETE |
| Configure public IP addresses | Networking: Public IP Addresses | COMPLETE |
| Configure user-defined routes | Networking: Route Tables and User-Defined Routes | COMPLETE |
| Troubleshoot network connectivity | Networking: Network Troubleshooting Flow | COMPLETE |
| Create and configure NSGs and ASGs | Networking: NSG and Application Security Groups | COMPLETE |
| Evaluate effective security rules | Networking: Effective Security Rules | COMPLETE |
| Implement Azure Bastion | Networking: Azure Bastion | COMPLETE |
| Configure service endpoints for PaaS | Networking: Service Endpoints | COMPLETE |
| Configure private endpoints for PaaS | Networking: Private Endpoints | COMPLETE |
| Configure Azure DNS | Networking: Azure DNS and Private DNS Zones | COMPLETE |
| Configure an internal or public Load Balancer | Networking: Load Balancer | COMPLETE |
| Troubleshoot load balancing | Networking: Load Balancer and Network Troubleshooting Flow | COMPLETE |

## Monitor and Maintain Azure Resources (13)

| Objective | Guide section | Status |
|---|---|---|
| Interpret metrics in Azure Monitor | [Monitoring: Metrics](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#metrics) | COMPLETE |
| Configure log settings in Azure Monitor | Monitoring: Logs and Log Analytics Workspace | COMPLETE |
| Query and analyze logs in Azure Monitor | Monitoring: Logs / KQL | COMPLETE |
| Set up alert rules, action groups, and alert processing rules | Monitoring: Alerts and Action Groups | COMPLETE |
| Configure and interpret VM, storage, and network monitoring using Insights | Monitoring: Azure Monitor Insights and Storage and Network Monitoring | COMPLETE |
| Use Network Watcher and Connection Monitor | Monitoring: Network Watcher | COMPLETE |
| Create a Recovery Services vault | Recovery: Azure Backup / Vault comparison | COMPLETE |
| Create an Azure Backup vault | Recovery: Recovery Services Vault vs. Backup Vault | COMPLETE |
| Create and configure a backup policy | Recovery: Backup Policy | COMPLETE |
| Perform Azure Backup backup and restore operations | Recovery: Backup and Restore Flow | COMPLETE |
| Configure Azure Site Recovery | Recovery: Azure Site Recovery | COMPLETE |
| Perform Site Recovery failover | Recovery: Failover choices | COMPLETE |
| Configure and interpret backup reports and alerts | Recovery: Backup Monitoring, Reports, and Alerts | COMPLETE |

## Summary

| Metric | Count |
|---|---:|
| Official objective bullets mapped | 82 |
| COMPLETE | 82 |
| PARTIAL | 0 |
| MISSING | 0 |

All current blueprint objectives are mapped to knowledge coverage in this repository. This does not guarantee exam success or imply that every possible exam question is represented.

## Source of Truth

- [Official AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Official source index](sources/official-sources.md)
