# AZ-104 2026 High-Yield Recall

---

**← [Previous: Networking](Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md)** | **[Home](README.md)**

---

Use this file for final exam revision only. Each row is a decision trigger, not a full explanation.

For learning, start with [AZ-104_MASTER_MENTAL_MAP.md](AZ-104_MASTER_MENTAL_MAP.md) and domain guides.

---

## Identity and Governance

| Trigger | Think | Don't Confuse |
|---|---|---|
| Who can access this resource? | RBAC role assignment + scope | Azure Policy (configuration) |
| Can this user assign roles? | Owner or appropriate access-management role | Contributor (cannot assign) |
| Need to manage directory users? | Entra ID | Azure RBAC (controls resource access) |
| Must prevent accidental deletion? | CanNotDelete lock | ReadOnly lock (also prevents modify) |
| Must prevent modification? | ReadOnly lock | CanNotDelete (allows modify) |
| Need to enforce configuration? | Azure Policy | RBAC (access control) |
| Can policy be overridden? | Yes (via exemption) | Lock (remove it with lock-management permission) |
| Scope inheritance? | Yes (down hierarchy only) | RBAC (no upward inheritance) |
| Need billing separation? | New subscription | Resource Group (same billing) |
| Bulk policy across subs? | Management Group | Subscription (one sub only) |
| Track spending per team? | Tags + Cost Analysis | Budgets (alerts, not tracking) |

---

## Storage

| Trigger | Think | Don't Confuse |
|---|---|---|
| Unstructured data (images, videos)? | Blob Storage | Files (structured, file shares) |
| File shares (SMB/NFS)? | Azure Files | Blob (no mount, HTTP access) |
| Need app-to-app messaging? | Queue Storage | Topic (publish-subscribe) |
| How many copies? | Redundancy level (LRS/ZRS/GRS) | Backup (separate service) |
| One physical location copies? | LRS | ZRS (multiple availability zones) |
| Multiple zones, one region? | ZRS | LRS (one physical location) or GZRS (zones + regions) |
| Region failure resilience? | GRS / GZRS / RA-GRS | ZRS (zones only) |
| Secondary readable without failover? | RA-GRS / RA-GZRS | GRS / GZRS (failover needed) |
| Recover deleted blob (30 days)? | Soft Delete | Versioning (recover overwritten) |
| Recover overwritten blob? | Versioning | Soft Delete (recover deleted) |
| Time-limited external access? | SAS token | Account key (full access, no expiry) |
| Full account access? | Account key | SAS (limited permissions) |
| Workload authentication? | Managed Identity + role | Account key (manual rotation) |
| VNet to storage private? | Private Endpoint + DNS | Service Endpoint (public endpoint) |
| VNet to storage simple? | Service Endpoint | Private Endpoint (complex DNS) |
| Auto tier/delete by age? | Lifecycle Management | Manual tagging |
| Rehydrate archive blob? | Change tier to Hot/Cool | Download and re-upload |

---

## Compute

| Trigger | Think | Don't Confuse |
|---|---|---|
| Standardize deployments? | ARM Template / Bicep | Portal (one-time deployment) |
| Simplified ARM syntax? | Bicep | ARM Template (JSON) |
| Prevent downtime during maintenance? | Availability Set or Zone | Single VM (no protection) |
| Datacenter failure protection? | Availability Zone | Availability Set (same datacenter) |
| Auto-scaling based on load? | VM Scale Set | Manual VMs (no auto-scaling) |
| Layer 4 load balancing? | Load Balancer | Application Gateway (Layer 7) |
| Layer 7 HTTP routing? | Application Gateway | Load Balancer (TCP/UDP only) |
| Containers, one-off? | Azure Container Instances | Container Apps (managed platform) |
| Containers, auto-scaling? | Container Apps | ACI (no scaling) |
| Web application hosting? | App Service | VM (full control, more management) |
| Bigger machine? | Scale Up | Scale Out (more machines) |
| More machines? | Scale Out | Scale Up (bigger machine) |
| Encrypt temporary disk and caches? | Encryption at Host | Disk encryption (all layers) |
| Move VM to different RG? | Validate dependent-resource move support | Cross-subscription move can be supported, but never assume it |

---

## Networking

| Trigger | Think | Don't Confuse |
|---|---|---|
| Allow/deny traffic? | NSG | Route Table (direction) |
| Direct traffic to destination? | Route Table / UDR | NSG (filtering) |
| Private network for Azure resources? | VNet | Subnet (subdivision of VNet) |
| VNet subdivision? | Subnet | VNet (entire network) |
| Connect two VNets? | VNet Peering | Site-to-Site VPN (on-premises) |
| Access PaaS via private IP? | Private Endpoint | Service Endpoint (public endpoint) |
| Access PaaS via public endpoint? | Service Endpoint | Private Endpoint (private IP) |
| VNet to service simple setup? | Service Endpoint | Private Endpoint (needs Private DNS) |
| VNet to service strict isolation? | Private Endpoint | Service Endpoint (less private) |
| Private IP resolution? | Private DNS Zone | Azure DNS (public) |
| Public domain? | Azure DNS | Private DNS Zone (internal only) |
| Secure RDP/SSH without public IP? | Azure Bastion | Jumphost VM (manual) |
| Distribute TCP/UDP traffic? | Load Balancer | Application Gateway (HTTP/HTTPS) |
| Distribute HTTP traffic with routing? | Application Gateway | Load Balancer (basic) |
| Filter traffic at NIC or subnet? | NSG | Network Watcher (monitoring) |

---

## Monitoring

| Trigger | Think | Don't Confuse |
|---|---|---|
| Numeric value over time? | Metric | Log (event record) |
| Detailed event record? | Log | Metric (just a number) |
| Real-time signal? | Metric | Log (historical) |
| Investigation / troubleshooting? | Log | Metric (trends only) |
| Condition-triggered notification? | Alert | Metric (no action) |
| Alert + routing to multiple channels? | Action Group | Single channel email |
| Central log repository? | Log Analytics Workspace | Application Insights (APM) |
| Application performance monitoring? | Application Insights | Azure Monitor (broad platform) |
| Multi-VM performance dashboard? | VM Insights | Single VM metrics |
| Network diagnostics? | Network Watcher | Azure Monitor (higher level) |
| Can client reach server? | Connection Monitor | Ping (simple test) |

---

## Recovery

| Trigger | Think | Don't Confuse |
|---|---|---|
| Recover accidentally deleted file? | Soft Delete / Backup | Replication (continuous) |
| Recover from regional failure? | Site Recovery | Backup (point-in-time) |
| Recover from data corruption? | Backup from clean point | Replication (copies corruption) |
| How much data loss acceptable? | RPO | RTO (recovery time) |
| How long recovery acceptable? | RTO | RPO (data loss) |
| Point-in-time restore? | Backup | Site Recovery (failover) |
| Replicate to secondary region? | Site Recovery | Backup (same region vault) |
| Long-term retention (years)? | Backup with policy | Soft Delete (30-365 days) |
| Managed restore infrastructure? | Recovery Services Vault | DIY backup server |
| Backup VM to vault? | Recovery Services Vault | Backup to storage account (old) |
| Continuous replication to region? | Site Recovery | GRS redundancy (passive) |
| Test failover? | Site Recovery test | Site Recovery production failover |
| Need production unchanged during test? | Test Failover | Planned Failover |

---

## Control Plane vs. Data Plane

| Layer | Controlled By | Example |
|---|---|---|
| **Control Plane** | RBAC role | Contributor can create/delete VMs |
| **Data Plane** | Data plane role + credential | Storage Blob Reader can read blobs |

**Exam pattern:**

```
Question: Alice should manage databases but not read customer data
Answer: Database Admin (control plane) + Data Plane Viewer excluded
```

---

## Scope Hierarchy

| Scope | Contains | Access | RBAC Inheritance |
|---|---|---|---|
| **Management Group** | Subscriptions | MG-level | Down to subscriptions |
| **Subscription** | Resource Groups | Sub-level | Down to RGs |
| **Resource Group** | Resources | RG-level | Down to resources |
| **Resource** | N/A | Resource-level | N/A |

**Rule:** Role assignments inherit downward. Design least privilege at the narrowest scope; normal lower-scope assignments do not subtract an inherited grant.

---

## Role Decision Tree

```
Can modify resources?
    ├─ YES, can manage role assignments → Owner or appropriate access-management role
    ├─ YES, cannot grant access → Contributor
    └─ NO → Reader or specific role
```

---

## Redundancy Decision Tree

```
Must survive zone failure?
    ├─ YES → ZRS or GZRS
    ├─ NO → LRS or GRS

Must survive region failure?
    ├─ YES → GRS or GZRS or RA-GRS or RA-GZRS
    └─ NO → LRS or ZRS
```

---

## Network Access Decision Tree

```
Need private IP for resource?
    ├─ YES → Private Endpoint
    └─ NO → Service Endpoint

Need complex routing?
    ├─ YES → Route Table
    └─ NO → NSG suffices
```

---

## Most Important Concept Pairs

| Pair | Distinction | Exam Frequency |
|---|---|---|
| RBAC vs. Policy | Access control vs. Configuration | Very High |
| NSG vs. Route | Filter vs. Direction | Very High |
| Soft Delete vs. Versioning | Deletion vs. Overwrite | Very High |
| Availability Set vs. Zone | Logical vs. Physical | Very High |
| Scale Up vs. Out | Size vs. Count | Very High |
| Service Endpoint vs. Private Endpoint | Public vs. Private IP | Very High |
| Backup vs. Site Recovery | Point-in-time vs. Failover | High |
| Metric vs. Log | Numeric vs. Event | High |
| RPO vs. RTO | Data loss vs. Recovery time | High |
| LRS vs. ZRS | Single vs. Multiple Zones | High |

---

## Exam Reasoning Pattern

When stuck on a scenario question:

```
1. WHO is the question about?
   → Look for: Users, roles, permissions
   → Decide: Identity or Authorization

2. WHAT are they trying to do?
   → Look for: Action (create, delete, read, etc.)
   → Decide: Control plane or data plane

3. WHERE is the constraint?
   → Look for: Region, zone, scope, network
   → Decide: Scope or networking

4. WHEN is timing important?
   → Look for: Failure recovery, downtime
   → Decide: Availability or recovery

5. WHY is this requirement needed?
   → Look for: Compliance, resilience, security
   → Decide: Policy, lock, encryption, or redundancy
```

---

## Common Exam Traps

| Trap | Correct Answer | Why Tempting |
|---|---|---|
| Contributor can assign roles | False; Owner or an appropriate access-management role can | Contributor sounds powerful |
| Policy can be enforced absolutely | False (has exemptions) | Lock can be absolute |
| Soft Delete vs. Versioning are same | False (deletion vs. overwrite) | Both recover data |
| Service Endpoint = Private IP | False (public endpoint) | Sounds like private |
| RA-GRS secondary always readable | True (without failover) | GRS secondary not readable |
| NSG deny always wins over allow | False | The first matching rule by priority wins; lower number is evaluated first |
| Availability Set protects zone failure | False (one datacenter) | Availability Zone does |
| Scale Up = Scale Out | False (size vs. count) | Both handle load |
| Metric = Log | False (number vs. event) | Both monitoring |
| Backup = Site Recovery | False (restore vs. failover) | Both recovery |

---

## 30-Second Decisions

**Sees "prevent modification":**
→ Think: ReadOnly Lock

**Sees "prevent deletion":**
→ Think: CanNotDelete Lock

**Sees "enforce config":**
→ Think: Azure Policy

**Sees "who can do what":**
→ Think: RBAC

**Sees "images and videos":**
→ Think: Blob Storage

**Sees "file shares":**
→ Think: Azure Files

**Sees "multiple copies across zones":**
→ Think: ZRS

**Sees "multiple regions":**
→ Think: GRS / GZRS / RA-GRS

**Sees "filter traffic":**
→ Think: NSG

**Sees "route traffic":**
→ Think: Route Table

**Sees "VNet to PaaS private":**
→ Think: Private Endpoint (+ Private DNS)

**Sees "recover deleted":**
→ Think: Soft Delete or Backup

**Sees "recover overwritten":**
→ Think: Versioning

**Sees "regional failure":**
→ Think: Site Recovery

**Sees "data loss acceptable":**
→ Think: RPO

---

## Remember for Test Day

1. **RBAC vs. Policy** — know what each controls
2. **NSG vs. Route** — filtering is not routing
3. **Service Endpoint vs. Private Endpoint** — public endpoint policy vs. private IP connectivity
4. **Scope matters** — inheritance, don't forget
5. **Networking is interconnected** — NSG + Route + DNS all matter
6. **Redundancy options** — LRS, ZRS, GRS decisions are common
7. **Recovery ≠ Replication** — backup is point-in-time, Site Recovery is failover
8. **Availability is zone-critical** — Set vs. Zone distinction very important
9. **Scaling direction matters** — Up/Down is size, Out/In is count
10. **Lock is control-plane protection** — Remove it only with the required lock permission; it does not block data-plane access

---

## Time Management Tip

If unsure on a question:

1. First 30 seconds: Identify the layer (identity / network / storage / compute / recovery)
2. Next 15 seconds: Rule out most obvious wrong answers
3. Remaining time: Choose closest remaining answer

Spend no more than 2 minutes per question. Come back to flagged questions at end if time.
