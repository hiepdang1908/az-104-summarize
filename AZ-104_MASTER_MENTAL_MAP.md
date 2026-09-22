# AZ-104 Master Mental Map

---

**← [Previous: README](README.md)** | **[Home](README.md)** | **[Next: Identity & Governance →](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md)**

---

This file connects all Azure administration concepts. Use the domain guides for detailed explanations and the High-Yield sheet for last-minute recall.

## AZ-104 Core Decision Chain

An Azure administrator's job follows this thinking sequence:

```text
1. WHO?
   Identity (Entra ID)
   ↓
2. WHAT CAN THEY DO?
   Authorization (RBAC)
   ↓
3. WHERE?
   Scope (Management Group → Subscription → Resource Group → Resource)
   ↓
4. WHAT WILL THEY ACCESS?
   Storage or Compute
   ↓
5. CAN TRAFFIC REACH IT?
   Networking (VNet, NSG, Route, Endpoint)
   ↓
6. WHAT HAPPENS IF SOMETHING FAILS?
   Availability (Zones, Sets, Scale Sets, Redundancy)
   ↓
7. WHAT HAPPENS IF DATA IS DELETED?
   Protection (Backup, Site Recovery, Soft Delete, Versioning)
   ↓
8. HOW DO I KNOW SOMETHING IS WRONG?
   Monitoring (Metrics, Logs, Alerts)
   ↓
9. HOW DO I GOVERN EVERYTHING?
   Policy, Locks, Tags, Management Groups, Cost Alerts
```

## One Reference Environment

Use this architecture throughout your learning:

```text
Management Group
│
└── Production Subscription
    │
    └── rg-contoso-prod (Resource Group)
        │
        ├── Identity Layer
        │   ├── Managed Identity (VM/App)
        │   └── Microsoft Entra ID (Users, Groups)
        │
        ├── Authorization Layer
        │   ├── RBAC role assignments
        │   └── Scope (RG, Subscription, MG)
        │
        ├── Governance Layer
        │   ├── Azure Policy
        │   ├── Resource Locks
        │   └── Tags
        │
        ├── Network Layer
        │   ├── VNet (10.0.0.0/16)
        │   ├── Web subnet (10.0.1.0/24)
        │   ├── App subnet (10.0.2.0/24)
        │   ├── Private Endpoint subnet (10.0.3.0/24)
        │   │
        │   ├── NSG (allow/deny rules)
        │   ├── Route Table (where traffic goes)
        │   ├── Public IP (inbound entry point)
        │   ├── Private Endpoint (PaaS private access)
        │   ├── Service Endpoint (PaaS network policy)
        │   └── Azure Bastion (secure management)
        │
        ├── Compute Layer
        │   ├── VM (web server)
        │   │   ├── OS Disk
        │   │   └── Data Disks
        │   ├── VM Scale Set (auto-scaling)
        │   ├── Availability Set
        │   ├── Availability Zone distribution
        │   │
        │   ├── Container Registry (images)
        │   ├── Container Instances (quick run)
        │   ├── Container Apps (managed platform)
        │   │
        │   ├── App Service Plan
        │   ├── App Service (PaaS web)
        │   │   ├── VNet Integration (outbound)
        │   │   ├── Private Endpoint (inbound)
        │   │   └── Application Gateway (Layer 7)
        │   │
        │   └── Load Balancer (Layer 4)
        │
        ├── Storage Layer
        │   ├── Storage Account
        │   ├── Blob containers (hot/cool/archive)
        │   ├── Azure Files (SMB shares)
        │   ├── Queues
        │   └── Access model (key/SAS/Entra)
        │
        ├── Monitoring Layer
        │   ├── Azure Monitor
        │   ├── Metrics (numbers)
        │   ├── Logs (events)
        │   ├── Log Analytics Workspace (KQL)
        │   ├── Alerts (rules + actions)
        │   └── Azure Monitor Insights
        │
        └── Recovery Layer
            ├── Recovery Services Vault
            ├── Backup Policy (schedule + retention)
            ├── Azure Backup (point-in-time restore)
            ├── Azure Site Recovery (regional failover)
            ├── RPO (acceptable data loss)
            └── RTO (acceptable recovery time)
```

## Five Official Domains

### Domain 1 — Manage Azure Identities and Governance (20–25%)

**Who has access and what can they do?**

```text
Entra ID
├── Authentication (who are you?)
│   └── Credentials
│
├── Authorization (what can you do?)
│   └── RBAC
│       └── Role assignment = Principal + Role + Scope
│
Governance
├── Policy (enforce configuration)
├── Locks (prevent delete/change)
├── Tags (organize/track)
│
Hierarchy
├── Management Group (organize subscriptions)
├── Subscription (billing boundary, resource limit)
├── Resource Group (lifecycle boundary)
└── Resource (actual Azure object)
```

**Key decisions:**
- Who needs access?
- What specific role? (Contributor? Owner? Custom?)
- At what scope? (Resource? RG? Subscription? MG?)
- Should this resource be modifiable? (Lock)
- Must all resources follow a configuration? (Policy)

### Domain 2 — Implement and Manage Storage (15–20%)

**How does data get stored and accessed?**

```text
Data requirement
    ↓
Choose storage service
├── Blob (unstructured, large)
├── Azure Files (SMB/NFS shares)
├── Queue (messaging)
└── Table (key-value)
    ↓
Choose redundancy
├── LRS (local only)
├── ZRS (zones in one region)
├── GRS (primary + secondary region)
└── GZRS (zones + regions)
    ↓
Choose access method
├── Storage account key (primary/secondary)
├── Shared Access Signature (time-limited, permissions)
├── Managed Identity (workload identity)
└── Entra ID (user identity)
    ↓
Choose network access
├── Public (unrestricted)
├── Service Endpoint (from VNet to service's public endpoint)
├── Private Endpoint (service private IP in your VNet)
└── Firewall (allow specific IPs/networks)
    ↓
Choose data protection
├── Soft Delete (recover accidentally deleted)
├── Versioning (recover old version)
├── Snapshots (point-in-time copy)
├── Backup (restore data to Recovery Services Vault)
└── Replication (copy to secondary region)
```

**Key decisions:**
- Blob or Files?
- How many redundancy zones needed?
- Who/what authenticates? (Managed Identity? SAS? Entra?)
- Private or public? (Private Endpoint? Service Endpoint?)
- What if deleted accidentally? (Soft Delete?)

### Domain 3 — Deploy and Manage Azure Compute Resources (20–25%)

**How do applications run?**

```text
Deployment method
├── ARM template (declarative infrastructure)
├── Bicep (ARM template shorthand)
└── Portal / CLI
    ↓
Compute choice
├── Virtual Machine (IaaS)
├── Container (ACR/ACI/Container Apps)
└── App Service (PaaS)
    ↓
Sizing and scaling
├── Vertical = bigger instance (Scale Up/Down)
└── Horizontal = more instances (Scale Out/In)
    ↓
Availability strategy
├── Single VM (no redundancy)
├── Availability Set (same datacenter, different fault/update domains)
├── Availability Zone (different physical datacenters)
└── VM Scale Set (auto-scale + availability)
    ↓
Network attachment
├── Subnet assignment
├── Public IP (optional)
├── NSG rules
├── Load Balancer (Layer 4)
└── Application Gateway (Layer 7 HTTP/HTTPS)
    ↓
Data persistence
├── OS Disk (system)
├── Data Disks (attachable)
├── Temporary Disk (ephemeral)
└── Managed Disk (Azure-managed, shared/dedicated)
    ↓
Security
├── Encryption at rest
├── Encryption at host (temporary disk and cache encryption)
├── Access via Bastion
└── Managed Identity (for service-to-service)
```

**Key decisions:**
- VM, container, or App Service?
- How many copies for availability?
- What size / tier?
- Manual or auto-scale?
- How to access securely? (Bastion? Public IP? Private?)

### Domain 4 — Implement and Manage Virtual Networking (15–20%)

**How do resources communicate?**

```text
Address space
├── VNet (10.0.0.0/16)
└── Subnets (10.0.1.0/24, 10.0.2.0/24)
    ↓
Routing
├── System routes (default)
└── User-defined routes (custom)
    ↓
Filtering (NSG)
├── Inbound rules (allow/deny)
└── Outbound rules (allow/deny)
    ↓
Private connectivity
├── VNet peering (VNET-to-VNET)
├── Service Endpoint (VNet to PaaS public endpoint)
├── Private Endpoint (PaaS private IP in VNet)
└── Private DNS (name resolution)
    ↓
Name resolution
├── Azure DNS (public domains)
└── Private DNS Zone (internal resolution)
    ↓
Management access
└── Azure Bastion (RDP/SSH without public IP)
    ↓
Load balancing
├── Load Balancer (Layer 4)
└── Application Gateway (Layer 7)
```

**Key decisions:**
- What addresses do I use? (CIDR planning)
- Where should traffic go? (UDR)
- What traffic is allowed? (NSG)
- Should resources talk privately? (Private Endpoint? VNet peering?)
- How do names resolve? (Azure DNS? Private DNS?)
- How is traffic distributed? (Load Balancer? Application Gateway?)

### Domain 5 — Monitor and Maintain Azure Resources (10–15%)

**Is everything healthy? How do I recover?**

```text
Monitoring
├── Metrics (numeric values: CPU, memory, requests)
├── Logs (detailed event records)
├── KQL (query language)
├── Alerts (condition + action)
└── Insights (curated dashboards)
    ↓
Health assessment
├── Application Insights (APM for apps)
├── Azure Monitor (comprehensive platform)
└── Log Analytics Workspace (centralized logs)
    ↓
Troubleshooting
├── Network Watcher (network diagnostics)
├── Connection Monitor (reachability)
└── Effective NSG rules
    ↓
Availability strategy
├── Backup
│   ├── Recovery Services Vault (long-term)
│   ├── Backup Vault (block-level)
│   └── Policy (schedule + retention)
│
└── Site Recovery
    ├── Replicate to secondary region
    ├── Test failover (no production impact)
    ├── Failover (switch to secondary)
    └── Failback (return to primary)
    ↓
Recovery metrics
├── RPO (how much data loss acceptable?)
└── RTO (how long recovery acceptable?)
```

**Key decisions:**
- What signals matter? (Metrics? Logs? Both?)
- When should I be alerted? (Thresholds? Conditions?)
- Where should logs go? (Log Analytics? Storage?)
- How do I recover deleted data? (Backup?)
- How do I handle region failure? (Site Recovery?)

## Critical Concept Pairs

Learn these **together** — they are often confused:

| Concept | Purpose | Do NOT confuse with | Reason |
|---|---|---|---|
| **Authentication** | Verify identity | Authorization | Authentication answers "WHO?", Authorization answers "WHAT?" |
| **Authorization (RBAC)** | Grant permissions | Policy | RBAC is identity-based access, Policy is configuration governance |
| **Azure Policy** | Enforce configuration | RBAC | Policy makes rules about resources, not people |
| **Resource Lock** | Prevent modification/deletion | Policy | Lock takes precedence over normal RBAC until a lock manager removes it; Policy can be exempted |
| **Managed Identity** | Workload credential | Private Endpoint | Identity is authentication, network is connectivity |
| **Service Endpoint** | Network policy to service | Private Endpoint | Service Endpoint allows traffic to public endpoint, Private Endpoint creates private IP |
| **Private Endpoint** | Private IP to service | Service Endpoint | Private Endpoint + Private DNS resolves service name to private IP |
| **NSG** | Filter traffic (yes/no) | Route Table | NSG answers "can traffic pass?", Route answers "where does traffic go?" |
| **Route Table** | Direct traffic path | NSG | Route controls next hop, NSG controls allow/deny |
| **VNet Peering** | Connect two VNets | Site-to-Site VPN | Peering is within Azure regions (or global), VPN connects on-premises |
| **Scale Up** | Bigger machine | Scale Out | Up/Down = machine size, Out/In = machine count |
| **Scale Out** | More machines | Scale Up | Out/In = machine count, Up/Down = machine size |
| **Availability Set** | Redundancy in one datacenter | Availability Zone | Availability Set uses fault/update domains, Availability Zone spans physical locations |
| **Availability Zone** | Redundancy across locations | Availability Set | Zones are physically separate, Sets are logical groupings in one location |
| **LRS** | Replication in one physical datacenter | GRS | LRS keeps copies local, GRS copies to secondary region |
| **Backup** | Point-in-time restore | Site Recovery | Backup recovers deleted/old data, Site Recovery handles regional failover |
| **Site Recovery** | Regional failover | Backup | Site Recovery switches to secondary region, Backup restores previous state |
| **Metric** | Numeric time-series | Log | Metric is a number (CPU 75%), Log is an event record |
| **Log** | Detailed event record | Metric | Log has context (which user, which operation), Metric is just a number |
| **RPO** | Acceptable data loss | RTO | RPO is "how much data can I lose?" (time), RTO is "how long recovery?" (time) |
| **RTO** | Acceptable recovery time | RPO | RTO is "how long recovery?" (time), RPO is "how much data loss?" (time) |

## Configuration Scope Rules

These rules govern **where** things can be configured:

### Scope Hierarchy

```text
Management Group
    ↓ (contains multiple)
Subscription
    ↓ (contains multiple)
Resource Group
    ↓ (contains multiple)
Resource
```

**Inheritance rule:** Lower scopes inherit settings from higher scopes, unless explicitly overridden.

### Scope Examples

| Object | Scope | Inherits? | Cross-Subscription? |
|---|---|---|---|
| **Role Assignment** | MG/Sub/RG/Resource | Yes (down) | An assignment at a management group can apply to descendant subscriptions |
| **Azure Policy** | MG/Sub/RG | Yes (down) | Yes (MG can enforce across subs) |
| **Resource Lock** | Subscription/RG/Resource | Yes (down) | A parent lock is inherited by child resources |
| **Tag** | Resource/RG | No (must tag each) | No |
| **VNet** | Resource Group | No | No (per RG) |
| **VNet Peering** | Subscription | No | Yes (can peer across subs) |
| **Subnet NSG** | Subnet (within VNet) | No | No |
| **Management Group** | Global | N/A | Yes (contains subs) |
| **Private Endpoint** | Resource Group | No | The private endpoint and target resource can be in different resource groups; the endpoint is created in a subnet |
| **Load Balancer** | Resource Group | No | Can route across availability zones |

## Storage Decision Tree

**Choose the right storage service:**

```
Need to store unstructured data (images, videos, backups)?
    ↓ YES → BLOB STORAGE
    ↓ NO

Need Windows file share (SMB) or Linux (NFS)?
    ↓ YES → AZURE FILES
    ↓ NO

Need message queue (application-to-application)?
    ↓ YES → QUEUE STORAGE
    ↓ NO

Need key-value table (structured)?
    ↓ YES → TABLE STORAGE
    ↓ NO

→ Not a storage account scenario
```

## Networking Decision Tree

**Choose the right network access:**

```
Need workload inside Azure to access PaaS service securely?
    ↓ YES
    Need private IP address in my VNet?
        ↓ YES → PRIVATE ENDPOINT (+ Private DNS)
        ↓ NO → SERVICE ENDPOINT (uses service's public endpoint)
    ↓ NO

Need inbound traffic filtering?
    ↓ YES → NSG
    ↓ NO

Need to direct traffic to specific locations?
    ↓ YES → USER-DEFINED ROUTE
    ↓ NO

Need to connect two VNets?
    ↓ YES → VNET PEERING (Azure-to-Azure) or VPN (on-premises)
    ↓ NO → NOTHING (resources in same VNet can communicate by default)
```

## Availability Decision Tree

**Choose the right availability strategy:**

```
Can your application tolerate downtime for OS patching?
    ↓ NO → MUST USE AVAILABILITY SET or ZONE or SCALE SET
    ↓ YES

Can your application tolerate datacenter failure?
    ↓ NO → MUST USE AVAILABILITY ZONE or SCALE SET across zones
    ↓ YES

Does load change frequently?
    ↓ YES → USE SCALE SET (auto-scale capability)
    ↓ NO → USE AVAILABILITY SET (manual + planned maintenance resilience)

Must be zone-redundant (physically separate locations)?
    ↓ YES → AVAILABILITY ZONE or ZONE-REDUNDANT (ZRS, GRS, GZRS)
    ↓ NO → AVAILABILITY SET (single datacenter)
```

## Recovery Decision Tree

**Choose backup vs. Site Recovery:**

```
Protecting against accidental deletion?
    ↓ YES → BACKUP (use Soft Delete for storage)
    ↓ NO

Protecting against regional datacenter failure?
    ↓ YES → SITE RECOVERY (replicate to secondary region)
    ↓ NO

Need to restore previous state of data?
    ↓ YES → BACKUP or VERSIONING/SNAPSHOTS
    ↓ NO

Need automated failover to secondary region?
    ↓ YES → SITE RECOVERY
    ↓ NO → BACKUP is sufficient
```

## Exam Reasoning Pattern

When you see an AZ-104 scenario, think in this order:

```
1. IDENTITY LAYER
   "Who is making this request? Do they have permission?"
   → Look for Entra ID, RBAC role, scope

2. NETWORK LAYER
   "Can traffic reach the resource?"
   → Look for NSG, route, endpoint, firewall

3. DATA LAYER
   "How is data stored and protected?"
   → Look for storage type, redundancy, backup/replication

4. GOVERNANCE LAYER
   "Are there compliance requirements?"
   → Look for Policy, locks, tags, cost controls

5. MONITORING LAYER
   "Can we detect failures?"
   → Look for alerts, metrics, logs

6. RECOVERY LAYER
   "Can we recover if something fails?"
   → Look for backup, Site Recovery, RPO/RTO
```

## Official Sources

- [AZ-104 Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/active-directory/)
- [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/)
- [Azure Storage](https://learn.microsoft.com/en-us/azure/storage/)
- [Azure Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [Azure Virtual Networks](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/)

---

## Next Steps

→ Start with [Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md](Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md)

→ Then explore other domain guides as you build your understanding

→ Use [AZ-104_HIGH_YIELD_RECALL.md](AZ-104_HIGH_YIELD_RECALL.md) during final exam review
