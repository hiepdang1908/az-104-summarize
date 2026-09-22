# AZ-104 Objective Map

This map links the Microsoft AZ-104 skills measured to the corresponding sections in this repository. It helps readers identify where each exam objective is covered without duplicating the study guides.

Blueprint verified against the official Microsoft study guide for **skills measured as of April 17, 2026**. `Covered partially` means the repository explains the administrator decision or configuration model but intentionally omits a step-by-step lab.

## 1. Manage Azure identities and governance (20–25%)

| Microsoft objective | Repository coverage | Status |
|---|---|---|
| Create users and groups | [Users and Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#users-and-groups) | Covered |
| Manage user and group properties | [User Properties](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#user-properties) and [Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#groups) | Covered |
| Manage licenses in Microsoft Entra ID | [License Management](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#license-management) | Covered |
| Manage external users | [Microsoft Entra B2B](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#external-users-microsoft-entra-b2b) | Covered |
| Configure self-service password reset (SSPR) | [SSPR](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#self-service-password-reset-sspr) | Covered |
| Manage built-in Azure roles | [Azure RBAC role types](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#role-types) | Covered |
| Assign roles at different scopes | [RBAC scope hierarchy](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#scope-hierarchy) | Covered |
| Interpret access assignments | [Azure RBAC](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#azure-rbac-role-based-access-control) | Covered |
| Implement and manage Azure Policy | [Azure Policy](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#azure-policy) | Covered |
| Configure resource locks | [Resource Locks](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#resource-locks) | Covered |
| Apply and manage tags on resources | [Tags](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#tags) | Covered |
| Manage resource groups | [Resource Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#resource-groups) | Covered |
| Manage subscriptions | [Azure Subscriptions](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#azure-subscriptions) | Covered |
| Manage costs by using alerts, budgets, and Azure Advisor recommendations | [Cost Management](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#cost-management) | Covered |
| Configure management groups | [Management Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#management-groups) | Covered |

## 2. Implement and manage storage (15–20%)

| Microsoft objective | Repository coverage | Status |
|---|---|---|
| Configure Azure Storage firewalls and virtual networks | [Storage Networking Security](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-networking-security) | Covered |
| Create and use SAS tokens | [Shared Access Signature](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#shared-access-signature-sas) | Covered |
| Configure stored access policies | [Shared Access Signature](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#shared-access-signature-sas) | Covered |
| Manage access keys | [Account Key](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#account-key-primarysecondary) | Covered |
| Configure identity-based access for Azure Files | [Azure Files identity-based authentication](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#identity-based-authentication) | Covered |
| Create and configure storage accounts | [Azure Storage Account](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#azure-storage-account) | Covered |
| Configure Azure Storage redundancy | [Storage Redundancy](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-redundancy) | Covered |
| Configure object replication | [Object Replication](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#object-replication) | Covered |
| Configure storage account encryption | [Encryption](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#encryption) | Covered |
| Manage data by using Azure Storage Explorer and AzCopy | [Storage Explorer and AzCopy](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#managing-data-with-azcopy-and-storage-explorer) | Covered |
| Create and configure an Azure file share | [Azure Files](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#azure-files) | Covered |
| Create and configure a Blob Storage container | [Blob containers](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-containers-and-structure) | Covered |
| Configure storage tiers | [Access Tiers](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#access-tiers) | Covered |
| Configure soft delete for blobs and containers | [Blob Soft Delete](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#soft-delete) | Covered |
| Configure snapshots and soft delete for Azure Files | [File Share Snapshots](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#file-share-snapshots) and [File Share Soft Delete](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#file-share-soft-delete) | Covered |
| Configure blob lifecycle management | [Blob Lifecycle Management](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-lifecycle-management) | Covered |
| Configure blob versioning | [Blob Versioning](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-versioning) | Covered |

## 3. Deploy and manage Azure compute resources (20–25%)

| Microsoft objective | Repository coverage | Status |
|---|---|---|
| Interpret an ARM template or Bicep file | [ARM Templates and Bicep](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#infrastructure-as-code-arm-templates-and-bicep) | Covered |
| Modify an existing ARM template | [Modifying ARM Templates](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#modifying-arm-templates) | Covered |
| Modify an existing Bicep file | [Bicep](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#bicep) | Covered |
| Deploy resources by using an ARM template or Bicep file | [Deployment](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#deployment) | Covered |
| Export a deployment as ARM or convert ARM to Bicep | [ARM Template Export](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-template-export) and [Bicep](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#bicep) | Covered |
| Create a virtual machine | [Virtual Machines](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#virtual-machines) | Covered partially |
| Configure encryption at host | [Encryption at Host](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#encryption-at-host) | Covered |
| Move a VM to another resource group, subscription, or region | [Moving VMs](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#moving-vms) | Covered |
| Manage VM sizes | [VM Sizing](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-sizing) | Covered |
| Manage VM disks | [VM Disks](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-disks) | Covered |
| Deploy VMs to availability zones and availability sets | [Virtual Machine Availability](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#virtual-machine-availability) | Covered |
| Deploy and configure Virtual Machine Scale Sets | [VM Scale Set](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-scale-set) | Covered |
| Create and manage Azure Container Registry | [Azure Container Registry](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-registry-acr) | Covered |
| Provision a container by using Azure Container Instances | [Azure Container Instances](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-instances-aci) | Covered |
| Provision a container by using Azure Container Apps | [Azure Container Apps](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-apps) | Covered |
| Manage container sizing and scaling | [Container Sizing and Resource Limits](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#container-sizing-and-resource-limits) | Covered |
| Provision an App Service plan | [App Service Plan](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-plan) | Covered |
| Configure scaling for an App Service plan | [App Service Plan](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-plan) | Covered |
| Create an App Service | [App Service](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-1) | Covered partially |
| Configure certificates and TLS for App Service | [TLS/SSL Certificates](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#tlsssl-certificates) | Covered |
| Map a custom DNS name to App Service | [Custom DNS Names and Domain Mapping](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#custom-dns-names-and-domain-mapping) | Covered |
| Configure App Service backup | [App Service Backup](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-backup) | Covered |
| Configure App Service networking | [App Service Networking](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-networking) | Covered |
| Configure App Service deployment slots | [Deployment Slots](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#deployment-slots) | Covered |

## 4. Implement and manage virtual networking (15–20%)

| Microsoft objective | Repository coverage | Status |
|---|---|---|
| Create and configure virtual networks and subnets | [VNet and Subnets](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-and-subnets) | Covered |
| Create and configure VNet peering | [VNet Peering](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-peering) | Covered |
| Configure public IP addresses | [Public IP Addresses](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#public-ip-addresses) | Covered |
| Configure user-defined routes | [Route Tables and UDRs](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#route-tables-and-user-defined-routes) | Covered |
| Troubleshoot network connectivity | [Network Troubleshooting Flow](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-troubleshooting-flow) | Covered |
| Create and configure NSGs and ASGs | [NSG](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#nsg-network-security-group) and [ASG](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#application-security-groups-asg) | Covered |
| Evaluate effective NSG security rules | [Effective Security Rules](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#effective-security-rules) | Covered |
| Implement Azure Bastion | [Azure Bastion](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-bastion) | Covered |
| Configure service endpoints for Azure PaaS | [Service Endpoints](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#service-endpoints) | Covered |
| Configure private endpoints for Azure PaaS | [Private Endpoints](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#private-endpoints) | Covered |
| Configure Azure DNS | [Azure DNS](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-dns) and [Private DNS Zones](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#private-dns-zones) | Covered |
| Configure an internal or public load balancer | [Azure Load Balancer](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#load-balancer) | Covered |
| Troubleshoot load balancing | [Health Probes](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#health-probes) and [Network Troubleshooting Flow](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-troubleshooting-flow) | Covered |

## 5. Monitor and maintain Azure resources (10–15%)

| Microsoft objective | Repository coverage | Status |
|---|---|---|
| Interpret metrics in Azure Monitor | [Metrics](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#metrics) | Covered |
| Configure log settings in Azure Monitor | [Logs](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#logs) and [Log Analytics Workspace](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#log-analytics-workspace) | Covered |
| Query and analyze Azure Monitor logs | [KQL examples](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#logs) | Covered |
| Set up alert rules, action groups, and alert processing rules | [Alerts](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#alerts) and [Action Group](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#action-group) | Covered |
| Configure and interpret Insights for VMs, storage, and networks | [Azure Monitor Insights](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-monitor-insights) | Covered |
| Use Network Watcher and Connection Monitor | [Network Watcher](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-watcher) and [Connection Monitor](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#connection-monitor) | Covered |
| Create a Recovery Services vault | [Recovery Services Vault vs. Backup Vault](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#recovery-services-vault-vs-backup-vault) | Covered partially |
| Create an Azure Backup vault | [Recovery Services Vault vs. Backup Vault](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#recovery-services-vault-vs-backup-vault) | Covered partially |
| Create and configure a backup policy | [Backup Policy](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#backup-policy) | Covered |
| Perform Azure Backup backup and restore operations | [Backup and Restore Flow](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#backup-and-restore-flow) | Covered |
| Configure Site Recovery for Azure resources | [Azure Site Recovery](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-site-recovery) | Covered |
| Perform a failover to a secondary region by using Site Recovery | [Site Recovery failover choices](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#key-concepts) | Covered |
| Configure and interpret backup reports and alerts | [Azure Backup Monitoring, Alerts, and Reports](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-backup-monitoring-alerts-and-reports) | Covered |

## Coverage Notes

- All five current AZ-104 domains are represented.
- Partial entries reflect the repository's knowledge-first scope: it explains selection, dependencies, and behavior but does not duplicate portal labs.
- Microsoft can revise the blueprint. Recheck the [official AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104) when planning an exam attempt.
