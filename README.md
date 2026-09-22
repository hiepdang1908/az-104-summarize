# Hiep's Study Map for Microsoft AZ-104

---

**[Home](README.md)** | **Next:** [Master Mental Map →](AZ-104_MASTER_MENTAL_MAP.md)

---

## Purpose

An independent, concept-first, decision-focused knowledge repository for **Exam AZ-104: Microsoft Azure Administrator**.

This repository focuses on:

- **Azure concepts** — what services exist and why
- **Administrator decisions** — when to use each service
- **Configuration recognition** — what parameters matter
- **Service relationships** — how services connect
- **Real-world administration** — practical Azure operations
- **Certification reasoning** — how to think through exam scenarios

Short examples in PowerShell, Azure CLI, ARM templates, or Bicep appear only when they help you recognize an important command, parameter, property, scope, or role. Microsoft Learn and official Microsoft product documentation are the factual authority.

This is **not** a hands-on lab repository. Version 1 focuses on knowledge and decision logic, not complete production projects.

## Learning Path

> **New learner:** Master Mental Map → domain guide → connected scenario → 30-second recall
>
> **Exam revision:** High-Yield Recall

### Step 1: Master Mental Map

Start with [AZ-104_MASTER_MENTAL_MAP.md](AZ-104_MASTER_MENTAL_MAP.md).

It connects the five domains into one reference architecture, shows the core administrator decision chain, and introduces the most important concept pairs that the exam constantly confuses.

Do **not** skip this. The mental map is the starting point.

### Step 2: Domain Guides

Each domain begins with its **Big Picture** (why does this area exist?), **Mental Model** (how do the concepts connect?), then teaches the owned keywords.

All four guides reuse the same reference environment, so concepts stay connected.

- [Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md)
- [Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md)
- [Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md)
- [Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md)

### Step 3: High-Yield Recall

After domain guides make sense, use [AZ-104_HIGH_YIELD_RECALL.md](AZ-104_HIGH_YIELD_RECALL.md) for final revision.

**Do not** use High-Yield Recall for first learning. It is compressed trigger→think→don't confuse tables, meant only for fast review.

### Step 4: Objective Coverage

[AZ-104_OBJECTIVE_MAP.md](AZ-104_OBJECTIVE_MAP.md) maps every Microsoft blueprint objective to its repository section.

Use it to verify you have covered all official exam domains.

## Release Scope

### Version 1 — This Repository

**Included:**

- Knowledge and core concepts
- Decision logic and reasoning
- Architecture recognition
- Configuration recognition
- Minimal command examples (when they improve understanding)
- Exam reasoning patterns

**Intentionally excluded from V1:**

- Complete production projects
- Huge ARM templates or Terraform configurations
- Full CI/CD pipelines and DevOps workflows
- Large-scale PowerShell automation
- End-to-end hands-on labs
- Application source code

Version 2 (planned separately) will address hands-on implementation and complete Azure examples.

## Version and Verification

**V1 scope:** Concept-first, decision-focused AZ-104 knowledge and configuration recognition.

**AZ-104 blueprint:** Skills measured as of **April 17, 2026**.

**Current audit baseline:** September 22, 2026.

**Blueprint representation:** 82/82 official objectives are mapped.

**Learning-completeness status:** In progress. Representation is not treated as complete teaching coverage; see [AZ-104_OBJECTIVE_MAP.md](AZ-104_OBJECTIVE_MAP.md).

**Accuracy status:** Claims are being corrected against current Microsoft Learn documentation. Do not rely on the legacy audit-completion reports as technical authority.

### Audit Discipline

- The official study guide controls objective counts and scope.
- Current Microsoft product documentation controls technical behavior.
- A topic is marked COMPLETE only when it satisfies the criteria in the objective matrix.
- Region, SKU, workload, and feature availability conditions are stated rather than hidden behind universal rules.

### Important Warnings

Fast-moving Azure features can change. Always verify:

- **Preview features** — check if still in preview or GA
- **Region availability** — confirm your target region supports the feature
- **SKU-specific features** — check if your tier supports the feature
- **API versions** — use current Microsoft API references
- **Deprecated services** — verify current Microsoft terminology

Microsoft documentation is the authoritative source. Practice exams do not define Azure behavior.

## Five Official Domains

The exam measures skills across these five domains with the following weights:

| Domain | Weight | Repository Guide |
|---|---:|---|
| 1. Manage Azure identities and governance | 20–25% | [MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md) |
| 2. Implement and manage storage | 15–20% | [IMPLEMENT_AND_MANAGE_STORAGE.md](Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md) |
| 3. Deploy and manage Azure compute resources | 20–25% | [DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md](Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md) |
| 4. Implement and manage virtual networking | 15–20% | [NETWORKING_MONITORING_AND_RECOVERY.md](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md) — PART I |
| 5. Monitor and maintain Azure resources | 10–15% | [NETWORKING_MONITORING_AND_RECOVERY.md](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md) — PARTS II & III |

These weights follow the [official AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104), skills measured as of April 17, 2026.

## Repository Structure

```
azure_104_summarize/
│
├── README.md (this file)
│
├── AZ-104_MASTER_MENTAL_MAP.md
│   └── Core architecture, decision chain, concept pairs
│
├── AZ-104_HIGH_YIELD_RECALL.md
│   └── Fast review tables (Trigger → Think → Don't confuse)
│
├── AZ-104_OBJECTIVE_MAP.md
│   └── Maps every Microsoft objective to repository section
│
├── Manage_Azure_identities_and_governance/
│   └── MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md
│
├── Implement_and_manage_storage/
│   └── IMPLEMENT_AND_MANAGE_STORAGE.md
│
├── Deploy_and_manage_Azure_compute_resources/
│   └── DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md
│
├── Networking_monitoring_and_recovery/
│   └── NETWORKING_MONITORING_AND_RECOVERY.md
│       ├── PART I — Virtual Networking
│       ├── PART II — Monitoring and Observability
│       └── PART III — Backup and Recovery
│
├── sources/
│   └── official-sources.md
│
├── LICENSE
└── THIRD_PARTY_NOTICES.md
```

## Design Philosophy

### Why Not Many Small Files?

A common mistake is creating one file per Azure service (nsg.md, route-table.md, vnet.md, etc.). This creates fragmentation and teaches services in isolation.

Instead, this repository:

- Groups related concepts into **four substantial domain guides**
- Reuses **one connected reference architecture** throughout all guides
- Teaches **decision logic**, not just definitions
- Emphasizes **concept relationships**, not feature lists

This approach mirrors the successful [azure_103_summarize](https://github.com/hiepdang1908/azure_103_summarize) repository structure.

### Why Combine Networking, Monitoring, and Recovery?

Microsoft officially defines **five domains**. This repository intentionally consolidates domains 4 and 5 into a single guide to avoid excessive fragmentation.

The guide clearly separates:

- **PART I — Virtual Networking** (Microsoft's official Domain 4)
- **PART II — Monitor and Observability** (part of Domain 5)
- **PART III — Backup and Recovery** (part of Domain 5)

The [AZ-104_OBJECTIVE_MAP.md](AZ-104_OBJECTIVE_MAP.md) still represents all five Microsoft domains separately, so nothing is lost from the official blueprint.

## Source and Currency Policy

- **Official Microsoft documentation is the source of truth.** Repository explanations and scenarios are independently written and paraphrased.
- **Preview claims are managed carefully.** Preview and availability claims are kept out of core memory rules where possible.
- **Model, API, region, deployment, role, quota, and feature support must be verified before implementation.** Never assume stability of Preview features.
- **Current Microsoft terms are preferred.** Older or legacy terms are labeled when relevant.
- **Practice exams do not define Azure behavior.** If a practice exam answer conflicts with official Microsoft documentation, Microsoft documentation wins.
- **Terminology and service behavior change.** Verify current Microsoft documentation before relying on older course material or fixed values.

## Common Exam Pitfalls

This repository explicitly teaches you to distinguish:

| Concepts | Why Confused | How to Distinguish |
|---|---|---|
| Entra ID vs. Azure RBAC | Both involve access control | Entra ID = who you are; RBAC = what you can do |
| RBAC vs. Azure Policy | Both enforce rules | RBAC is identity-based; Policy is configuration-based |
| Policy vs. Resource Lock | Both restrict changes | Policy can be overridden; Locks cannot |
| Service Endpoint vs. Private Endpoint | Both secure access to PaaS | Service Endpoint = network policy on service's public endpoint; Private Endpoint = private IP in your VNet |
| Managed Identity vs. Private Endpoint | Both make services "secure" | Managed Identity = workload authentication; Private Endpoint = network connectivity |
| NSG vs. Route Table | Both affect network traffic | NSG = filter (allow/deny); Route Table = direct (where to send) |
| Load Balancer vs. Application Gateway | Both distribute traffic | Load Balancer = Layer 4 (TCP/UDP); Application Gateway = Layer 7 (HTTP/HTTPS) |
| Availability Set vs. Availability Zone | Both provide redundancy | Availability Set = logical grouping in one datacenter; Availability Zone = physically separate locations |
| Scale Up vs. Scale Out | Both handle load | Scale Up = bigger machine; Scale Out = more machines |
| Backup vs. Site Recovery | Both handle failure | Backup = restore data point; Site Recovery = fail over region |
| Metric vs. Log | Both used in monitoring | Metric = numeric value over time; Log = detailed event record |

Understanding these distinctions is **critical for the exam**. The repository emphasizes them repeatedly.

## Disclaimer

> **Independent study resource**
>
> This is an independent study resource and is neither affiliated with, nor authorized, sponsored, or approved by Microsoft Corporation.
>
> The repository is based on publicly available Microsoft Learn documentation and the public AZ-104 skills blueprint. It does not contain actual, reconstructed, remembered, confidential, or otherwise protected certification exam questions, answers, tasks, or diagrams.
>
> Microsoft, Azure, and related product names are trademarks of Microsoft Corporation. Microsoft documentation and certification materials remain the property of their respective owners.

## License

Original repository material is available under the [MIT License](LICENSE), subject to the boundaries in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

---

## How to Use This Repository

1. **Start here:** [AZ-104_MASTER_MENTAL_MAP.md](AZ-104_MASTER_MENTAL_MAP.md)
2. **Learn deeply:** Read the four domain guides in any order
3. **Exam prep:** Use [AZ-104_HIGH_YIELD_RECALL.md](AZ-104_HIGH_YIELD_RECALL.md) during final review
4. **Verify coverage:** Check [AZ-104_OBJECTIVE_MAP.md](AZ-104_OBJECTIVE_MAP.md)

Good luck on AZ-104.
