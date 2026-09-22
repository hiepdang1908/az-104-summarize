# AZ-104 Objective Matrix

---

**[Home](README.md)** | **[High-Yield Recall](AZ-104_HIGH_YIELD_RECALL.md)**

---

This matrix follows the official **AZ-104: Microsoft Azure Administrator** skills measured as of **April 17, 2026**.

## Assessment Standard

An objective is **COMPLETE** only when its material teaches what it is, why it exists, how it works, when to use it, its nearest confusion, a material constraint or trap, and a scenario or decision application. A keyword is not coverage.

- **COMPLETE**: meets every requirement above.
- **PARTIAL**: represented, but has a material gap or inaccuracy.
- **MISSING**: no meaningful treatment exists.

This is the conservative audit baseline. It replaces the legacy, unsupported claim that all objectives were fully covered.

| Domain | Objective | File / section | Coverage | Accuracy | Depth | Scenario readiness | Action |
|---|---|---|---|---|---|---|---|
| Identity | Create users and groups | [Identity guide](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#users-and-groups) | PARTIAL | Needs correction | Moderate | Weak | Separate Entra groups from ASGs; add a selection scenario. |
| Identity | Manage user and group properties | [Identity guide](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md#user-properties) | PARTIAL | Verified | Moderate | Weak | Add property and membership boundaries. |
| Identity | Manage licenses in Entra ID | Identity guide / User Properties | PARTIAL | Verified | Weak | Weak | Add direct versus group-based licensing scenario. |
| Identity | Manage external users | Identity guide / Entra ID | PARTIAL | Verified | Weak | Weak | Add B2B guest lifecycle and access scenario. |
| Identity | Configure SSPR | Identity guide / SSPR | PARTIAL | Needs correction | Moderate | Weak | Remove overbroad license claim; add scope trap. |
| Identity | Manage built-in Azure roles | Identity guide / Role Types | PARTIAL | Verified | Strong | Moderate | Add least-privilege role selection. |
| Identity | Assign roles at different scopes | Identity guide / Scope Hierarchy | PARTIAL | Needs correction | Strong | Moderate | Remove false lower-scope override rule. |
| Identity | Interpret access assignments | Identity guide / RBAC | PARTIAL | Needs correction | Moderate | Weak | Clarify inherited and deny assignments. |
| Identity | Implement and manage Azure Policy | Identity guide / Azure Policy | PARTIAL | Needs correction | Moderate | Weak | Add initiative, exemption, remediation choices. |
| Identity | Configure resource locks | Identity guide / Resource Locks | PARTIAL | Needs correction | Moderate | Moderate | Clarify control-plane scope and data-plane limitation. |
| Identity | Apply and manage tags | Identity guide / Tags | PARTIAL | Verified | Moderate | Moderate | Add inheritance and policy scenario. |
| Identity | Manage resource groups | Identity guide / Resource Groups | PARTIAL | Verified | Strong | Moderate | Add move and lifecycle constraints. |
| Identity | Manage subscriptions | Identity guide / Azure Subscriptions | PARTIAL | Needs correction | Moderate | Weak | Remove fixed quota claim; add scope decisions. |
| Identity | Manage costs using alerts, budgets, and Advisor | Identity guide / Cost Management | PARTIAL | Verified | Moderate | Moderate | Add budget versus policy scenario. |
| Identity | Configure management groups | Identity guide / Management Groups | PARTIAL | Verified | Moderate | Moderate | Add hierarchy and inheritance scenario. |
| Storage | Configure Storage firewalls and VNets | [Storage guide](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md#storage-networking-security) | PARTIAL | Needs correction | Strong | Moderate | Remove regional price; add access-method scenario. |
| Storage | Create and use SAS tokens | Storage guide / SAS | PARTIAL | Needs correction | Moderate | Weak | Clarify user-delegation SAS and security trap. |
| Storage | Configure stored access policies | Storage guide / SAS | PARTIAL | Verified | Moderate | Weak | Add supported-SAS and revocation scenario. |
| Storage | Manage access keys | Storage guide / Account Key | PARTIAL | Needs correction | Moderate | Moderate | Correct rotation sequence and broad-access risk. |
| Storage | Configure identity-based access for Azure Files | Storage guide / Identity-Based Authentication | PARTIAL | Needs correction | Moderate | Weak | Distinguish identity sources and access layers. |
| Storage | Create and configure storage accounts | Storage guide / Storage Account | PARTIAL | Needs correction | Moderate | Weak | Correct default-tier and SKU guidance. |
| Storage | Configure Storage redundancy | Storage guide / Storage Redundancy | PARTIAL | Needs correction | Strong | Moderate | Correct GRS and zone-resilience language. |
| Storage | Configure object replication | Storage guide / Object Replication | PARTIAL | Needs correction | Moderate | Weak | Add prerequisites and replication comparison. |
| Storage | Configure storage account encryption | Storage guide / Encryption | PARTIAL | Needs correction | Moderate | Weak | Add CMK and encryption-in-transit decision. |
| Storage | Use Storage Explorer and AzCopy | Storage guide / AzCopy and Storage Explorer | PARTIAL | Verified | Moderate | Moderate | Add tool-selection scenario. |
| Storage | Create and configure file shares | Storage guide / Azure Files | PARTIAL | Needs correction | Moderate | Weak | Add protocol and tier constraints. |
| Storage | Create and configure blob containers | Storage guide / Blob Containers | PARTIAL | Verified | Moderate | Weak | Add public-access and naming trap. |
| Storage | Configure storage tiers | Storage guide / Access Tiers | PARTIAL | Needs correction | Moderate | Weak | Add Cold, archive limits, and cost trade-offs. |
| Storage | Configure blob/container soft delete | Storage guide / Soft Delete | PARTIAL | Verified | Moderate | Weak | Distinguish recovery layers. |
| Storage | Configure Azure Files snapshots and soft delete | Storage guide / File Share Snapshots | PARTIAL | Needs correction | Moderate | Weak | Add share deletion versus file recovery. |
| Storage | Configure blob lifecycle management | Storage guide / Blob Lifecycle Management | PARTIAL | Needs correction | Moderate | Weak | Correct rule conditions and archive limitation. |
| Storage | Configure blob versioning | Storage guide / Blob Versioning | PARTIAL | Verified | Moderate | Weak | Add recovery-tool choice. |
| Compute | Interpret an ARM template | [Compute guide](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md#arm-templates) | PARTIAL | Needs correction | Strong | Moderate | Add API-version and dependency traps. |
| Compute | Modify an ARM template | Compute guide / Modifying ARM Templates | PARTIAL | Needs correction | Strong | Moderate | Correct incremental-mode delete misconception. |
| Compute | Modify an existing Bicep file | Compute guide / Bicep | PARTIAL | Verified | Moderate | Weak | Add parameter/module decision. |
| Compute | Deploy ARM or Bicep | Compute guide / Deployment | PARTIAL | Verified | Moderate | Weak | Add deployment scope and what-if. |
| Compute | Export or convert a deployment | Compute guide / ARM Template Export | PARTIAL | Needs correction | Moderate | Weak | Explain exports are a starting point. |
| Compute | Create a virtual machine | Compute guide / Virtual Machines | PARTIAL | Needs correction | Moderate | Weak | Add dependency scenario. |
| Compute | Configure encryption at host | Compute guide / Encryption at Host | PARTIAL | Needs correction | Moderate | Weak | Remove incorrect Disk Encryption memory alternative. |
| Compute | Move a virtual machine | Compute guide / Moving VMs | PARTIAL | Needs correction | Weak | Weak | Add supported-resource validation and region move method. |
| Compute | Manage VM sizes | Compute guide / VM Sizing | PARTIAL | Needs correction | Moderate | Weak | Avoid fixed SKU limits; add resize constraints. |
| Compute | Manage VM disks | Compute guide / VM Disks | PARTIAL | Needs correction | Moderate | Weak | Add disk SKU and shared-disk constraints. |
| Compute | Deploy to zones and availability sets | Compute guide / VM Availability | PARTIAL | Needs correction | Strong | Moderate | Remove universal SLA and same-size claims. |
| Compute | Deploy and configure VM Scale Sets | Compute guide / VM Scale Set | PARTIAL | Needs correction | Moderate | Weak | Add orchestration and autoscale scenario. |
| Compute | Create and manage ACR | Compute guide / ACR | PARTIAL | Verified | Weak | Weak | Add registry/repository/image scenario. |
| Compute | Provision containers with ACI | Compute guide / ACI | PARTIAL | Needs correction | Moderate | Weak | Remove “instant scaling”; add one-off choice. |
| Compute | Provision containers with Container Apps | Compute guide / Container Apps | PARTIAL | Needs correction | Moderate | Weak | Add environment, revision, and ingress scenario. |
| Compute | Manage container sizing and scaling | Compute guide / Container Sizing and Resource Limits | PARTIAL | Needs correction | Moderate | Weak | Remove fixed prices/limits and Kubernetes-only terms. |
| Compute | Provision an App Service plan | Compute guide / App Service Plan | PARTIAL | Needs correction | Moderate | Weak | Correct SKU feature claims. |
| Compute | Configure scaling for an App Service plan | Compute guide / App Service Plan | PARTIAL | Verified | Moderate | Weak | Add manual versus autoscale choice. |
| Compute | Create an App Service | Compute guide / App Service | PARTIAL | Verified | Moderate | Weak | Add runtime/plan dependency scenario. |
| Compute | Configure certificates and TLS | Compute guide / TLS/SSL Certificates | PARTIAL | Needs correction | Moderate | Weak | Correct managed-certificate eligibility. |
| Compute | Map custom DNS name | Compute guide / Custom DNS Names | PARTIAL | Needs correction | Moderate | Weak | Correct provider and verification details. |
| Compute | Configure App Service backup | Compute guide / App Service Backup | PARTIAL | Needs correction | Moderate | Weak | Remove fixed retention/content claims. |
| Compute | Configure App Service networking | Compute guide / App Service Networking | PARTIAL | Verified | Moderate | Moderate | Add inbound/outbound selection scenario. |
| Compute | Configure deployment slots | Compute guide / Deployment Slots | PARTIAL | Needs correction | Moderate | Weak | Add swap and sticky-settings trap. |
| Networking | Create and configure VNets and subnets | [Networking guide](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#vnet-and-subnets) | PARTIAL | Needs correction | Moderate | Weak | Correct overlap and reserved-IP rules. |
| Networking | Create and configure VNet peering | Networking guide / VNet Peering | PARTIAL | Needs correction | Moderate | Weak | Remove “free regional peering”; add gateway transit. |
| Networking | Configure public IP addresses | Networking guide / Public IP Addresses | PARTIAL | Needs correction | Weak | Weak | Remove universal pricing; add SKU/allocation choice. |
| Networking | Configure UDRs | Networking guide / Route Tables and UDRs | PARTIAL | Needs correction | Moderate | Weak | Correct unmatched-route and next-hop behavior. |
| Networking | Troubleshoot network connectivity | Networking guide / Network Troubleshooting Flow | PARTIAL | Verified | Strong | Moderate | Add effective route/security interpretation. |
| Networking | Configure NSGs and ASGs | Networking guide / NSG | PARTIAL | Needs correction | Moderate | Weak | Correct first-match priority and statefulness. |
| Networking | Evaluate effective security rules | Networking guide / Effective Security Rules | PARTIAL | Needs correction | Moderate | Weak | Add NIC/subnet evaluation scenario. |
| Networking | Implement Azure Bastion | Networking guide / Azure Bastion | PARTIAL | Needs correction | Moderate | Weak | Add dedicated subnet and SKU caveats. |
| Networking | Configure service endpoints | Networking guide / Service Endpoints | PARTIAL | Verified | Strong | Moderate | Add endpoint-selection scenario. |
| Networking | Configure private endpoints | Networking guide / Private Endpoints | PARTIAL | Needs correction | Strong | Moderate | Correct private DNS zone and price. |
| Networking | Configure Azure DNS | Networking guide / Azure DNS | PARTIAL | Verified | Moderate | Weak | Add delegation and record selection. |
| Networking | Configure internal/public Load Balancer | Networking guide / Load Balancer | PARTIAL | Needs correction | Moderate | Weak | Remove fixed probe values. |
| Networking | Troubleshoot load balancing | Networking guide / Load Balancer | PARTIAL | Needs correction | Moderate | Weak | Add probe, rule, and NSG scenario. |
| Monitoring | Interpret Azure Monitor metrics | [Monitoring guide](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#metrics) | PARTIAL | Needs correction | Moderate | Weak | Distinguish platform and guest metrics. |
| Monitoring | Configure Azure Monitor log settings | Monitoring guide / Logs | PARTIAL | Needs correction | Moderate | Weak | Add diagnostic settings and data collection rules. |
| Monitoring | Query and analyze Azure Monitor logs | Monitoring guide / Logs | PARTIAL | Needs correction | Moderate | Weak | Replace invalid KQL and schema assumptions. |
| Monitoring | Configure alerts, action groups, processing rules | Monitoring guide / Alerts | PARTIAL | Verified | Moderate | Moderate | Add suppression and routing scenario. |
| Monitoring | Monitor VMs, storage, networks using Insights | Monitoring guide / Azure Monitor Insights | PARTIAL | Needs correction | Moderate | Weak | Add storage/network insight coverage. |
| Monitoring | Use Network Watcher and Connection Monitor | Monitoring guide / Network Watcher | PARTIAL | Needs correction | Moderate | Weak | Mark retiring NSG flow logs; use VNet flow logs. |
| Recovery | Create a Recovery Services vault | [Recovery guide](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md#azure-backup) | PARTIAL | Needs correction | Moderate | Weak | Add vault choice and protection scenario. |
| Recovery | Create an Azure Backup vault | Recovery guide / Vault comparison | PARTIAL | Needs correction | Moderate | Weak | Correct protected-workload comparison. |
| Recovery | Create and configure a backup policy | Recovery guide / Backup Policy | PARTIAL | Verified | Moderate | Weak | Add schedule/retention scenario. |
| Recovery | Perform Azure Backup backup and restore | Recovery guide / Backup and Restore Flow | PARTIAL | Needs correction | Moderate | Weak | Explain restore choices without universal backup behavior. |
| Recovery | Configure Azure Site Recovery | Recovery guide / Azure Site Recovery | PARTIAL | Needs correction | Moderate | Weak | Remove fixed RPO claim; add supported scenario. |
| Recovery | Perform Site Recovery failover | Recovery guide / Key Concepts | PARTIAL | Needs correction | Moderate | Weak | Add test/planned/unplanned decision table. |
| Recovery | Configure and interpret backup reports and alerts | Recovery guide / Alerts | PARTIAL | Needs correction | Weak | Weak | Add Backup center, job state, reporting, and alert scenario. |

## Baseline Summary

| Domain | Objectives | COMPLETE | PARTIAL | MISSING | Representation |
|---|---:|---:|---:|---:|---:|
| Manage Azure identities and governance | 15 | 0 | 15 | 0 | 100% |
| Implement and manage storage | 17 | 0 | 17 | 0 | 100% |
| Deploy and manage Azure compute resources | 24 | 0 | 24 | 0 | 100% |
| Implement and manage virtual networking | 13 | 0 | 13 | 0 | 100% |
| Monitor and maintain Azure resources | 13 | 0 | 13 | 0 | 100% |
| **Total** | **82** | **0** | **82** | **0** | **100%** |

`100% representation` means every official objective has a mapped location. It is **not** complete learning coverage. At this baseline, complete coverage is $0 / 82 = 0\%$ because the legacy material does not consistently meet the stated COMPLETE standard.

## Source of Truth

- [Official AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Official source index](sources/official-sources.md)

## Scope

This repository remains knowledge-only. Command and template fragments explain configuration concepts; they are not deployment exercises or labs.