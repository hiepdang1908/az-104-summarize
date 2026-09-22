# Changelog

All notable changes to this knowledge-only AZ-104 repository are recorded here.

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