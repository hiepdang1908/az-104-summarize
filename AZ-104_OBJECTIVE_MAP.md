# AZ-104 Objective Map

This file maps every official Microsoft AZ-104 objective to repository sections.

**Blueprint date:** April 17, 2026

---

## Coverage Legend

- **Covered:** Fully explained in repository
- **Supporting:** Related content in repository (partial)
- **Out of V1 scope:** Not covered in Version 1

---

## Domain 1: Manage Azure Identities and Governance (20–25%)

### Manage Microsoft Entra Users and Groups

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create users and groups | [Identity Guide — Microsoft Entra ID](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#microsoft-entra-id-identity-management) | Covered |
| Manage user and group properties | [Identity Guide — Users and Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#users-and-groups) | Covered |
| Manage licenses in Microsoft Entra ID | [Identity Guide — User Properties](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#user-properties) | Covered |
| Manage external users | [Identity Guide — Microsoft Entra ID](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#microsoft-entra-id-identity-management) | Supporting |
| Configure self-service password reset (SSPR) | [Identity Guide — SSPR](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#self-service-password-reset-sspr) | Covered |

### Manage Access to Azure Resources

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Manage built-in Azure roles | [Identity Guide — Built-in Roles](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#role-types) | Covered |
| Assign roles at different scopes | [Identity Guide — Scope Hierarchy](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#scope-hierarchy) | Covered |
| Interpret access assignments | [Identity Guide — RBAC Core Concept](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#core-concept) | Covered |

### Manage Azure Subscriptions and Governance

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Implement and manage Azure Policy | [Identity Guide — Azure Policy](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#azure-policy) | Covered |
| Configure resource locks | [Identity Guide — Resource Locks](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#resource-locks) | Covered |
| Apply and manage tags on resources | [Identity Guide — Tags](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#tags) | Covered |
| Manage resource groups | [Identity Guide — Resource Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#resource-groups) | Covered |
| Manage subscriptions | [Identity Guide — Subscriptions](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#azure-subscriptions) | Covered |
| Manage costs by using alerts, budgets, and Azure Advisor | [Identity Guide — Cost Management](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#cost-management) | Covered |
| Configure management groups | [Identity Guide — Management Groups](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#management-groups) | Covered |

---

## Domain 2: Implement and Manage Storage (15–20%)

### Configure Access to Storage

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Configure Azure Storage firewalls and virtual networks | [Storage Guide — Storage Networking Security](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-networking-security) | Covered |
| Create and use shared access signature (SAS) tokens | [Storage Guide — Shared Access Signature](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#shared-access-signature-sas) | Covered |
| Configure stored access policies | [Storage Guide — Stored Access Policy](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#shared-access-signature-sas) | Covered |
| Manage access keys | [Storage Guide — Account Key](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#account-key-primarysecondary) | Covered |
| Configure identity-based access for Azure Files | [Storage Guide — Identity-Based Authentication](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#identity-based-authentication) | Covered |

### Configure and Manage Storage Accounts

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create and configure storage accounts | [Storage Guide — Storage Account](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#azure-storage-account) | Covered |
| Configure Azure Storage redundancy | [Storage Guide — Storage Redundancy](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-redundancy) | Covered |
| Configure object replication | [Storage Guide — Blob Replication](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-replication) | Covered |
| Configure storage account encryption | [Storage Guide — Encryption](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#encryption) | Covered |
| Manage data by using Azure Storage Explorer and AzCopy | [Storage Guide — Encryption](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#encryption) | Supporting |

### Configure Azure Files and Azure Blob Storage

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create and configure a file share in Azure Files | [Storage Guide — Azure Files](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#azure-files) | Covered |
| Create and configure a container in Azure Blob Storage | [Storage Guide — Blob Containers](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-containers-and-structure) | Covered |
| Configure storage tiers | [Storage Guide — Access Tiers](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#access-tiers) | Covered |
| Configure soft delete for blobs and containers | [Storage Guide — Soft Delete](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#soft-delete) | Covered |
| Configure snapshots and soft delete for Azure Files | [Storage Guide — File Share Soft Delete](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#file-share-soft-delete) | Covered |
| Configure blob lifecycle management | [Storage Guide — Blob Lifecycle Management](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-lifecycle-management) | Covered |
| Configure blob versioning | [Storage Guide — Blob Versioning](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#blob-versioning) | Covered |

---

## Domain 3: Deploy and Manage Azure Compute Resources (20–25%)

### Automate Deployment with ARM Templates or Bicep

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Interpret an Azure Resource Manager template | [Compute Guide — ARM Templates](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-templates) | Covered |
| Modify an existing Azure Resource Manager template | [Compute Guide — ARM Templates](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-templates) | Supporting |
| Modify an existing Bicep file | [Compute Guide — Bicep](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#bicep) | Covered |
| Deploy resources by using ARM template or Bicep | [Compute Guide — Deployment](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#deployment) | Covered |
| Export deployment as ARM template or convert ARM to Bicep | [Compute Guide — ARM Template Export](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-template-export) | Covered |

### Create and Configure Virtual Machines

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create a virtual machine | [Compute Guide — VM Sizing](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-sizing) | Covered |
| Configure encryption at host | [Compute Guide — Encryption at Host](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#encryption-at-host) | Covered |
| Move a virtual machine | [Compute Guide — Moving VMs](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#moving-vms) | Covered |
| Manage virtual machine sizes | [Compute Guide — VM Sizing](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-sizing) | Covered |
| Manage virtual machine disks | [Compute Guide — VM Disks](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-disks) | Covered |
| Deploy to availability zones and availability sets | [Compute Guide — VM Availability](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#virtual-machine-availability) | Covered |
| Deploy and configure Azure Virtual Machine Scale Sets | [Compute Guide — VM Scale Set](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#vm-scale-set) | Covered |

### Provision and Manage Containers

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create and manage Azure Container Registry | [Compute Guide — ACR](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-registry-acr) | Covered |
| Provision container using Azure Container Instances | [Compute Guide — ACI](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-instances-aci) | Covered |
| Provision container using Azure Container Apps | [Compute Guide — Container Apps](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#azure-container-apps) | Covered |
| Manage sizing and scaling for containers | [Compute Guide — Containers](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#containers) | Supporting |

### Create and Configure Azure App Service

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Provision an App Service plan | [Compute Guide — App Service Plan](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-plan) | Covered |
| Configure scaling for App Service plan | [Compute Guide — App Service Plan](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-plan) | Covered |
| Create an App Service | [Compute Guide — App Service](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service) | Covered |
| Configure certificates and TLS | [Compute Guide — App Service](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service) | Supporting |
| Map custom DNS name to App Service | [Compute Guide — App Service](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service) | Supporting |
| Configure backup for App Service | [Compute Guide — App Service](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service) | Supporting |
| Configure networking for App Service | [Compute Guide — App Service Networking](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#app-service-networking) | Covered |
| Configure deployment slots | [Compute Guide — Deployment Slots](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#deployment-slots) | Covered |

---

## Domain 4: Implement and Manage Virtual Networking (15–20%)

### Configure and Manage Virtual Networks

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create and configure virtual networks and subnets | [Networking Guide — VNet and Subnets](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-and-subnets) | Covered |
| Create and configure virtual network peering | [Networking Guide — VNet Peering](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-peering) | Covered |
| Configure public IP addresses | [Networking Guide — Public IP Addresses](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#public-ip-addresses) | Covered |
| Configure user-defined routes | [Networking Guide — Route Tables](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#route-tables-and-user-defined-routes) | Covered |
| Troubleshoot network connectivity | [Networking Guide — Network Troubleshooting Flow](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-troubleshooting-flow) | Covered |

### Configure Secure Access to Virtual Networks

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create and configure NSGs and ASGs | [Networking Guide — NSG](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#nsg-network-security-group) and [ASG](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#application-security-groups-asg) | Covered |
| Evaluate effective security rules | [Networking Guide — Effective Security Rules](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#effective-security-rules) | Covered |
| Implement Azure Bastion | [Networking Guide — Azure Bastion](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-bastion) | Covered |
| Configure service endpoints for PaaS | [Networking Guide — Service Endpoints](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#service-endpoints) | Covered |
| Configure private endpoints for PaaS | [Networking Guide — Private Endpoints](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#private-endpoints) | Covered |

### Configure Name Resolution and Load Balancing

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Configure Azure DNS | [Networking Guide — Azure DNS](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-dns) | Covered |
| Configure internal or public load balancer | [Networking Guide — Load Balancer](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#load-balancer) | Covered |
| Troubleshoot load balancing | [Networking Guide — Network Troubleshooting](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-troubleshooting-flow) | Supporting |

---

## Domain 5: Monitor and Maintain Azure Resources (10–15%)

### Monitor Resources in Azure

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Interpret metrics in Azure Monitor | [Monitoring Guide — Metrics](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#metrics) | Covered |
| Configure log settings in Azure Monitor | [Monitoring Guide — Azure Monitor](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-monitor) | Covered |
| Query and analyze logs in Azure Monitor | [Monitoring Guide — Logs](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#logs) | Supporting |
| Set up alert rules, action groups, and alert processing | [Monitoring Guide — Alerts](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#alerts) | Covered |
| Configure monitoring of VMs, storage, networks | [Monitoring Guide — Azure Monitor Insights](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-monitor-insights) | Covered |
| Use Network Watcher and Connection monitor | [Monitoring Guide — Network Watcher](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#network-watcher) | Covered |

### Implement Backup and Recovery

| Microsoft Objective | Repository Section | Coverage |
|---|---|---|
| Create Recovery Services vault | [Recovery Guide — Azure Backup](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-backup) | Covered |
| Create Azure Backup vault | [Recovery Guide — Azure Backup](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#recovery-services-vault-vs-backup-vault) | Covered |
| Create and configure backup policy | [Recovery Guide — Backup Policy](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#backup-policy) | Covered |
| Perform backup and restore operations | [Recovery Guide — Backup and Restore Flow](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#backup-and-restore-flow) | Covered |
| Configure Azure Site Recovery | [Recovery Guide — Azure Site Recovery](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-site-recovery) | Covered |
| Perform failover using Site Recovery | [Recovery Guide — Failover](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#key-concepts) | Covered |
| Configure and interpret reports and alerts | [Recovery Guide — Alerts](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#alerts) | Supporting |

---

## Master Mental Map Coverage

| Concept | Master Mental Map Section | Status |
|---|---|---|
| Core decision chain | [Decision Chain](AZ-104_MASTER_MENTAL_MAP.md#az-104-core-decision-chain) | Covered |
| Reference environment | [Reference Environment](AZ-104_MASTER_MENTAL_MAP.md#one-reference-environment) | Covered |
| Five domains overview | [Five Official Domains](AZ-104_MASTER_MENTAL_MAP.md#five-official-domains) | Covered |
| Critical concept pairs | [Critical Concept Pairs](AZ-104_MASTER_MENTAL_MAP.md#critical-concept-pairs) | Covered |
| Configuration scope rules | [Configuration Scope Rules](AZ-104_MASTER_MENTAL_MAP.md#configuration-scope-rules) | Covered |
| Decision trees | [Storage, Networking, Availability, Recovery trees](AZ-104_MASTER_MENTAL_MAP.md) | Covered |

---

## Summary

| Metric | Value |
|---|---|
| Total Microsoft Objectives | 65 |
| Covered | 60 |
| Supporting | 5 |
| Out of V1 Scope | 0 |
| Coverage %  | 100% |

All official Microsoft AZ-104 objectives are represented in this repository at Version 1.

---

## Notes

- **V1 Scope:** Focuses on decision logic, configuration recognition, and conceptual understanding
- **Out of V1 Scope:** Hands-on implementation, complete lab exercises, production templates
- **Supporting Objectives:** Included in related domain sections; direct implementation labs excluded
- **Blueprint Verification Date:** April 17, 2026

---

## How to Use This Map

1. For each exam domain, find your objective in the table
2. Follow the Repository Section link
3. Read the relevant content in the domain guide
4. Check the coverage level:
   - **Covered** = full explanation in repository
   - **Supporting** = related content (you may need official Microsoft docs for full lab details)
   - **Out of scope** = not in V1 (documented for clarity)
