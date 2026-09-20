# Source Conflicts and Terminology Changes

This document records potentially conflicting information learners may encounter from:
- Older practice materials
- Older Microsoft documentation
- Deprecated service names
- Terminology evolution

When conflicts arise, **always refer to current official Microsoft documentation**.

---

## Terminology Changes

| Older/Conflicting Term | Current Microsoft Term | Status | Notes |
|---|---|---|---|
| Azure AD | Microsoft Entra ID | **Use current** | Renamed in 2023; all documentation now uses Entra ID |
| Azure Key Vault (classic) | Azure Key Vault (current) | **Use current** | Updated management and access model |
| Blob Storage tiers (hot/cool/archive) | Access tiers (hot/cool/archive) | **Both valid** | Same feature, updated naming in 2024 |
| Virtual Network (VNet) | Azure Virtual Network | **Both valid** | Abbreviated form widely used |
| NSG | Network Security Group | **Both valid** | Abbreviated form is standard |
| SAS | Shared Access Signature | **Both valid** | Abbreviated form is standard |
| RBAC | Azure RBAC / Role-based access control | **Both valid** | Formal term is Azure RBAC |
| Resource group | Resource Group | **Both valid** | Capitalization varies |
| Storage Account Key | Account Key / Primary Key | **Both valid** | Various naming used |
| Provisioned throughput | Reserved capacity | **Likely change pending** | Monitor Microsoft docs |

---

## Service Name/Feature Changes

| Older Name | Current Name | Status | Explanation |
|---|---|---|---|
| Azure SQL Database | Azure SQL | **Use current** | Rebranded 2019, includes SQL Managed Instance |
| Cosmos DB (legacy SDK) | Cosmos DB (current SDK) | **Use current** | SDK v3+ is current (v2 deprecated) |
| Azure DevOps Services | Azure DevOps | **Use current** | Formerly Visual Studio Team Services |
| Backup Vault | Azure Backup (new Backup Vault) | **Distinction required** | Two different vault types now exist |
| App Service on Linux | App Service (Linux container) | **No change needed** | Supported since 2016; same service |

---

## Deprecated Features (Do Not Use)

| Feature | Status | Replacement | Notes |
|---|---|---|---|
| Unmanaged Disks | **DEPRECATED** | Managed Disks | Microsoft no longer recommends; legacy only |
| Classic Network (ASM) | **DEPRECATED** | Azure Resource Manager (ARM) VNets | Classic model has been in deprecation since 2016 |
| Azure AD B2C Directory | **Still supported** | Azure AD B2C (current) | Some older names persist in legacy systems |
| Classic Storage Account | **DEPRECATED** | Standard/Premium Storage Accounts | Classic accounts no longer recommended |
| Azure Service Management API | **DEPRECATED** | Azure Resource Manager API | ASM is legacy; use ARM API only |

---

## Region and SKU Availability

**Important:** Always verify current availability before implementation.

| Service | Note | Action |
|---|---|---|
| **All services** | Region availability changes frequently | Check [Azure Products by Region](https://azure.microsoft.com/en-us/global-infrastructure/services/) before implementation |
| **Premium SKUs** | Some SKUs unavailable in certain regions | Verify SKU availability in target region |
| **Preview features** | May not be in all regions | Verify GA status and regional availability |

---

## Practice Test Conflicts

### Known Practice Test Issues

| Topic | Practice Test Answer | Official Microsoft Position | Reason |
|---|---|---|---|
| **Availability Set scope** | Some tests say "global" | Incorrect—always region-bound | Practice material outdated |
| **RBAC inheritance** | Some tests suggest bidirectional | Only downward inheritance exists | Misunderstanding of scope model |
| **GRS endpoint** | Some tests say "both readable" | Wrong—only RA-GRS is readable | Confusing GRS with RA-GRS |
| **Private Endpoint cost** | Some say "free" | **Paid (~$0.01/hour)** | Outdated practice material |
| **Policy enforcement** | Some say "absolute" | Has exemptions possible | Incomplete understanding |

### How to Handle Conflicts

```text
If practice exam contradicts Microsoft documentation:

1. Check Microsoft official documentation first
2. If Microsoft clearly contradicts practice test:
   → Believe Microsoft (practice tests can have errors)
3. If unclear:
   → Study both perspectives
   → Understand why they differ
   → On exam, choose Microsoft-aligned answer
```

---

## Feature Consolidation

| Old Approach | New Approach | Notes |
|---|---|---|
| Multiple storage account types | Single storage account (SKU-based) | Simplified in 2019; all features in one account type now |
| Separate Backup service | Azure Backup unified platform | Consolidated backup management |
| Multiple monitoring tools | Azure Monitor unified platform | Centralized observability |
| Entra ID + B2C separate | Can now coexist in same tenant | More flexible identity models |

---

## Documentation Gaps

Some older certifications covered topics no longer emphasized in AZ-104:

| Topic | Status | Notes |
|---|---|---|
| Classic deployments (ASM) | **Removed from AZ-104** | Still in AZ-900 fundamentals; focus on ARM only |
| Azure Government clouds | **Not in AZ-104 blueprint** | Specialized certification (AZ-108) exists instead |
| China cloud variants | **Not in AZ-104 blueprint** | Specialized requirements; not mainstream AZ-104 focus |
| On-premises VM migration (Hyper-V) | **Supporting material only** | Site Recovery supports it, but not heavily tested |

---

## Recent Changes (2024-2026)

| Change | Date | Impact | Details |
|---|---|---|---|
| Entra ID rename from "Azure AD" | March 2023 | Naming convention | All documentation now uses Entra ID |
| Private Endpoint pricing clarification | 2024 | Cost models | ~$0.01/hour per endpoint confirmed |
| Backup Vault vs. Recovery Services Vault distinction | 2024 | Service clarity | Two vaults now have clearly defined roles |
| App Service VNet Integration redesign | 2024 | Networking | New integration model with regional/gateway variants |
| Azure Monitor improvements | Ongoing | Feature | Continuous additions to insights and alerts |

---

## If You See Conflicting Information

### Priority Order (High to Low)

```text
1. Current Official Microsoft Learn documentation
   (learn.microsoft.com, most authoritative)

2. Current Official Microsoft Azure Documentation
   (docs.microsoft.com)

3. Current Microsoft Study Guide (AZ-104)
   (released April 2026)

4. Recent official Microsoft blog posts
   (azure.microsoft.com/en-us/blog/)

5. Practice tests from Microsoft
   (Learn platform)

6. Third-party practice tests
   (may contain errors)

7. Community forums / Stack Overflow
   (user-contributed, not official)
```

### Best Practice

```
When you find conflicting info:

1. Note both positions
2. Research current Microsoft documentation
3. On exam: Choose Microsoft-aligned answer
4. After exam: Update your notes if Microsoft position changes
```

---

## How to Stay Current

- Subscribe to [Azure Updates](https://azure.microsoft.com/en-us/updates/)
- Follow [Microsoft Tech Community](https://techcommunity.microsoft.com/)
- Check [Azure Service Health](https://status.azure.com/)
- Review [Breaking Changes and Deprecations](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/breaking-changes)

---

## Known Exam Pitfalls from Practice Tests

| Pitfall | What Practice Tests Get Wrong | Correct Understanding |
|---|---|---|
| **GRS is readable secondary** | "GRS secondary is always readable" | False—must use RA-GRS for readable secondary |
| **All locks are the same** | "Locks prevent both modify and delete" | False—CanNotDelete allows modify, ReadOnly prevents both |
| **Policy is absolute** | "Policy cannot be violated" | False—exemptions possible |
| **RBAC inheritance is two-way** | "Child can affect parent scope" | False—only downward inheritance |
| **Service Endpoint is private** | "Service Endpoint gives private IP" | False—uses service's public endpoint, private routing only |
| **Contributor can assign roles** | "Contributor has full access except deletion" | False—cannot assign roles; Owner only |
| **Soft Delete vs. Versioning** | "Soft Delete and Versioning are interchangeable" | False—deletion vs. overwrite protection |

---

## Version History of This Document

| Date | Version | Changes |
|---|---|---|
| Sept 2026 | 1.0 | Initial documentation |
| - | 1.1 (future) | Updates as Microsoft changes services |

---

## Questions or Conflicts?

If you find information in this repository that conflicts with current Microsoft official documentation:

1. Document the conflict
2. Check official Microsoft sources
3. If Microsoft has changed, this repository should be updated

Report issues to: [Repository issues section if available]

This document will be updated as Microsoft Azure services and exam content evolve.
