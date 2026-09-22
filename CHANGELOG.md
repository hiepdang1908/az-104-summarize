# Changelog

All notable changes to this knowledge-only AZ-104 repository are recorded here.

## [2.2] - 2026-09-22

### Corrected and strengthened

- Added Azure Policy definitions, initiatives, assignments, exclusions, exemptions, current effects, and remediation identity boundaries.
- Added Azure Files SMB identity-source selection, share-level authorization, file/directory authorization, managed-identity distinction, and network dependency guidance.
- Added current Blob object-replication prerequisites, supported data types, topology constraints, and redundancy comparison.
- Corrected App Service backup support to include Basic, distinguished automatic and custom backups, and documented restore, certificate, and deployment-slot constraints.
- Clarified Network Security Group statefulness and combined NIC/subnet evaluation; added VNet peering gateway transit, private DNS linking, VPN Gateway connection types, and current virtual network flow-log guidance.
- Replaced schema-dependent flow-log KQL with valid Azure Activity KQL and clarified how virtual network flow-log data is represented.
- Added workload-aware Azure Backup recovery, monitoring, reporting, alerting, and Azure Site Recovery prerequisite/failover guidance.
- Updated the objective matrix to use knowledge-only completion criteria. It currently records 23 COMPLETE and 59 PARTIAL objectives; all 82 current blueprint objectives remain mapped.

## [2.1] - 2026-09-22

### Corrected

- Replaced the unsupported `65/65` objective claim with an official-blueprint matrix containing **82** objectives: 15 identity/governance, 17 storage, 24 compute, 13 networking, and 13 monitoring/recovery.
- Changed keyword-based “Covered” labels to `COMPLETE`, `PARTIAL`, and `MISSING` criteria that include accuracy, depth, scenario readiness, and a required action.
- Corrected Blob access-tier guidance, Azure RBAC inheritance, resource-lock scope, Storage data-plane authorization, VM moves, ARM incremental deployment behavior, NSG priority processing, reserved subnet addresses, private-endpoint DNS, and vault/workload distinctions.
- Removed fixed prices, SLA values, quotas, timing, and SKU claims where behavior varies by region, tier, configuration, or current documentation.
- Replaced legacy Log Analytics agent guidance with Azure Monitor Agent guidance and made virtual network flow logs the forward-looking flow-log choice.

### Removed

- Removed superseded review and final-audit artifacts that reported an incorrect objective count or unsupported “100% verified/complete” results.
- Removed the stale source-conflicts document; the official source index and current Microsoft documentation are the maintained references.

### Scope

- The repository remains knowledge-only. No labs, deployment projects, portal walkthroughs, or sandbox exercises were added.

## [2.0] - 2026-09-20

Historical release metadata was superseded by the 2026-09-22 audit because its objective count and completion assertions could not be verified against the current official study guide.