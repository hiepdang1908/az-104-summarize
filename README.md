# AZ-104 Knowledge-First Study Guide

A structured, knowledge-first study repository for **AZ-104: Microsoft Azure Administrator**. It follows the current Microsoft Learn skills measured and focuses on clear definitions, service comparisons, configuration boundaries, common misconceptions, and administrator decision-making.

This is an independent learning project. Microsoft Learn and current Azure product documentation are the technical source of truth.

## What This Repository Focuses On

- Clear conceptual understanding of AZ-104 services and features
- Exam-relevant comparisons between easily confused options
- Configuration scope, prerequisites, constraints, and dependencies
- Common misconceptions and administrator-oriented mental models
- Fast review tables after the detailed domain material

## What It Intentionally Avoids

- Exam dumps, copied exam questions, or leaked certification material
- Lab walkthroughs, deployment exercises, and sandbox activities
- Excessive command memorization or infrastructure-as-code tutorials
- Broad Azure architecture content outside AZ-104 administration scope

## Learning Path

1. Read [AZ-104_MASTER_MENTAL_MAP.md](AZ-104_MASTER_MENTAL_MAP.md) for the cross-domain model.
2. Study the domain guides:
   - [Identities and governance](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md)
   - [Storage](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md)
   - [Compute](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md)
   - [Networking, monitoring, and recovery](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md)
3. Review [AZ-104_HIGH_YIELD_RECALL.md](AZ-104_HIGH_YIELD_RECALL.md).
4. Track the official objectives in [AZ-104_OBJECTIVE_MAP.md](AZ-104_OBJECTIVE_MAP.md).

## Current Blueprint Structure

| Domain | Official weight | Repository location |
|---|---:|---|
| Manage Azure identities and governance | 20-25% | Identity and governance guide |
| Implement and manage storage | 15-20% | Storage guide |
| Deploy and manage Azure compute resources | 20-25% | Compute guide |
| Implement and manage virtual networking | 15-20% | Networking section of the combined guide |
| Monitor and maintain Azure resources | 10-15% | Monitoring and recovery sections of the combined guide |

The repository maps all **82** objective bullets in the April 17, 2026 study guide. Mapping is not a guarantee of exam coverage or exam success. See the objective map for the current knowledge-quality status of each objective.

## Documentation Principles

- Microsoft Learn wins when this repository conflicts with a source.
- Version-, SKU-, region-, and workload-dependent behavior is stated as conditional.
- Fixed prices, quotas, and SLA percentages are avoided unless a current official source makes them essential.
- Changes should be small, source-backed, AZ-104 relevant, and recorded in [CHANGELOG.md](CHANGELOG.md).

## Core Governance Distinction

| Feature | Controls | Does not replace |
|---|---|---|
| **Azure RBAC** | Who can perform which Azure action at which scope | Azure Policy or data-plane authorization |
| **Azure Policy** | Whether resource configurations are allowed, audited, or remediated | RBAC permissions or a lock |
| **Resource lock** | Azure Resource Manager deletion or modification of the locked resource | Data-plane protection, RBAC, or Policy |

A `CanNotDelete` lock prevents Azure Resource Manager deletion; a `ReadOnly` lock also prevents Azure Resource Manager modifications. A principal with sufficient lock permissions can remove a lock. Locks do not prevent storage data operations such as deleting a blob with a valid data-plane credential.

## Repository Structure

```text
az-104-summarize/
├── README.md
├── AZ-104_MASTER_MENTAL_MAP.md
├── AZ-104_HIGH_YIELD_RECALL.md
├── AZ-104_OBJECTIVE_MAP.md
├── CHANGELOG.md
├── Manage_Azure_identities_and_governance/
├── Implement_and_manage_storage/
├── Deploy_and_manage_Azure_compute_resources/
├── Networking_monitoring_and_recovery/
└── sources/
```

## Limitations

Azure changes continuously. Before implementing a production configuration, validate current Microsoft documentation for region support, pricing, quotas, preview status, API versions, and feature combinations.

## Certification Disclaimer

This is an independent educational resource. It is not affiliated with, sponsored by, or approved by Microsoft. It does not contain actual, reconstructed, remembered, confidential, or protected certification exam content.

## Sources and Contributions

The maintained official source list is [sources/official-sources.md](sources/official-sources.md). Contributions should preserve the knowledge-first scope, cite Microsoft documentation, avoid copied certification materials, and update the objective map when coverage changes.
