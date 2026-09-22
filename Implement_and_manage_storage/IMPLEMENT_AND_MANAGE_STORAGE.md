# Implement and Manage Storage

---

**← [Previous: Identity & Governance](../Manage_Azure_identities_and_governance/MANAGE_AZURE_IDENTITIES_AND_GOVERNANCE.md)** | **[Home](../README.md)** | **[Next: Compute →](../Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md)**

---

## Big Picture

An Azure administrator is responsible for:

- **Where** data gets stored
- **How** that data gets protected
- **Who** can access it
- **How much** it costs
- **How resilient** it is to failure

This domain covers the core data persistence mechanisms in Azure, from unstructured blobs to file shares to queues.

---

## Mental Model

```text
Data Requirement
    ↓
Which storage service?
├── Blob Storage (unstructured: images, videos, documents)
├── Azure Files (SMB/NFS file shares)
├── Queue Storage (application-to-application messages)
└── Table Storage (key-value structured data)
    ↓
Create storage account
    ↓
Configure redundancy
├── LRS (local)
├── ZRS (zones in region)
├── GRS (primary + secondary region)
└── GZRS (zones + regions)
    ↓
Configure authentication
├── Account key
├── Shared Access Signature (SAS)
├── Managed Identity + Entra ID
└── Storage account access roles
    ↓
Configure network access
├── Public (unrestricted)
├── Firewall (specific IPs/VNets)
├── Service Endpoint (from VNet to service's public endpoint)
└── Private Endpoint (service private IP in your VNet)
    ↓
Configure data protection
├── Soft Delete
├── Versioning
├── Snapshots
├── Lifecycle Management
├── Replication
└── Azure Backup (vault and tier depend on the workload)
    ↓
Result: Secure, redundant, accessible data storage
```

---

## Azure Storage Account

### What It Means

A storage account is:

- **Container** for all storage services (Blob, Files, Queues, Tables)
- **Unique namespace** — determines the DNS name
- **Authentication boundary** — credentials authenticate to the whole account
- **Billing unit** — all storage costs charged together

### Key Properties

- **Account name** — must be globally unique (3-24 alphanumeric characters)
- **DNS name** — storageaccountname.blob.core.windows.net
- **Account key** — primary and secondary keys (each 88 characters)
- **Default online access tier** — Hot, Cool, or Cold for new eligible block blobs. Archive is set per eligible blob; it is not an account default tier.
- **Redundancy** — LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS

### Storage Account Types, Performance, and SKUs

Do not treat these terms as interchangeable:

- **Account type / kind** determines the supported storage services and features.
- **Performance** is Standard or Premium. Standard general-purpose v2 is the normal choice for general workloads; Premium accounts are SSD-backed and workload-specific.
- **Redundancy** is the replication choice, such as LRS or ZRS. In templates and tools, the SKU name combines performance and redundancy, for example `Standard_ZRS` or `Premium_LRS`.

| Account type | ARM account kind | Performance | Supported workload | Typical choice |
|---|---|---|---|---|
| **General-purpose v2 (GPv2)** | `StorageV2` | Standard | Blobs, files, queues, and tables | Recommended for most Azure Storage workloads |
| **Premium block blobs** | `BlockBlobStorage` | Premium | Block and append blobs | High transaction rates or consistently low blob latency |
| **Premium file shares** | `FileStorage` | Premium | Azure Files only | High-performance SMB or NFS file shares |
| **Premium page blobs** | `StorageV2` | Premium | Page blobs only | Specialized high-performance page-blob workloads |

**Legacy boundary:** `BlobStorage` is the kind for a legacy Standard Blob Storage account; it is not the name of the current premium block-blob option. General-purpose v1 and legacy Blob Storage accounts should be migrated to GPv2 rather than selected for new deployments. Available redundancy options vary by account type and region, so validate the combination before deployment.

---

## Storage Services: Which One?

### Blob Storage

**What it is:** Unstructured data container

**Stores:**

- Images (JPG, PNG, SVG)
- Videos (MP4, WebM)
- Backups (entire VM snapshots)
- Logs (application, diagnostic)
- Large documents (PDF, DOCX)
- Archive (infrequently accessed)

**Access pattern:**

```
Create container
    ↓
Upload blob
    ↓
Reference by: https://storageaccount.blob.core.windows.net/container/blob-name
```

**Redundancy options:**

- **Hot tier** — Online and immediately accessible; highest storage cost and lowest access cost
- **Cool tier** — Online and immediately accessible; lower storage cost, higher access cost, 30-day minimum duration for GPv2 accounts
- **Cold tier** — Online and immediately accessible; lower storage cost and higher access cost than Cool, 90-day minimum duration for GPv2 accounts
- **Archive tier** — Offline; rehydration is required before blob data can be read, and the minimum duration is 180 days

### Azure Files

**What it is:** SMB/NFS file share (like Windows Share or NAS)

**Stores:**

- User home directories
- Application configuration files
- Shared documents
- Database backups (SMB native support)

**Access pattern:**

```
Mount share on VM / application
    ↓
Access files as local path
    ↓
Example: Z:\documents\report.docx
```

**Protocols:**

- **SMB** — Windows file share (port 445)
- **NFS** — Linux/Unix standard (port 2049)

**Access:**

- **Storage account key** — Full access to share
- **Shared Access Signature** — Time-limited access
- **Microsoft Entra ID** — Identity-based access (identity-based authentication)
- **NTFS permissions** — On-share permissions (SMB only)

### Queue Storage

**What it is:** Application-to-application message queue

**Stores:**

- Messages from producer to consumer
- Async task queues
- Buffer between fast producer and slow consumer

**Access pattern:**

```
App 1 (Producer)
    ↓
Puts message: "Process order #123"
    ↓
Queue Storage
    ↓
App 2 (Consumer)
    ↓
Retrieves message
    ↓
Process order #123
```

**Life of a message:**

```
1. Producer adds message to queue
2. Consumer receives message; it is temporarily invisible for the configured visibility timeout
3. Consumer processes message
4. Consumer explicitly deletes the message after successful processing; otherwise it becomes visible again when the timeout expires
```

### Table Storage

**What it is:** NoSQL key-value table storage

**Stores:**

- Structured data with flexible schema
- Employee records
- Product catalog
- Sensor data (time-series)

**Structure:**

```
Table: Employees
├── Partition: Sales
│   ├── Row: emp001 (EmpID) = name, email, phone
│   ├── Row: emp002
│   └── Row: emp003
└── Partition: Engineering
    ├── Row: emp010
    └── Row: emp011
```

**Key concept:** Partition Key + Row Key uniquely identify a record

---

## Storage Redundancy

### Rule of Thumb

```text
LRS
→ Copies in one physical datacenter; protects hardware failures, not datacenter failure

ZRS
→ Synchronous copies across three or more availability zones in one region

GRS / GZRS / RA-GRS / RA-GZRS
→ Region focus (resilient to region failure)
```

### LRS (Locally Redundant Storage)

**What it does:** Replicates data within a single physical datacenter in the primary region. Azure does not let you select an availability zone for LRS.

```text
One Physical Datacenter
├── Copy 1
├── Copy 2
└── Copy 3
```

**Resilience:**

- ✅ Survives disk failure
- ✅ Survives temporary node failure
- ❌ Cannot survive zone failure (earthquake, major power outage)

**Cost:** Lowest

**When to use:** Non-critical data, dev/test, data that can be recreated

### ZRS (Zone Redundant Storage)

**What it does:** 3 copies across three availability zones in one region

```text
Region (e.g., East US)
├── Zone 1: Copy 1
├── Zone 2: Copy 2
└── Zone 3: Copy 3
```

**Resilience:**

- ✅ Survives disk failure
- ✅ Survives zone failure (one zone down, other two still work)
- ❌ Cannot survive region failure

**Cost:** Higher than LRS, lower than GRS

**When to use:** Critical data in one region, minimum zone redundancy required

### GRS (Geo-Redundant Storage)

**What it does:** 3 copies in primary region + 3 copies in secondary region

```text
Primary Region (e.g., East US)
├── Copy 1
├── Copy 2
└── Copy 3

Secondary Region (e.g., West US)
├── Copy 1
├── Copy 2
└── Copy 3
```

**Resilience:**

- ✅ Survives disk failure
- ✅ Replicates asynchronously to a paired secondary region
- ❌ The primary replica is LRS, so GRS alone does not provide zone redundancy in the primary region

**Cost:** Higher than ZRS

**Limitation:** Secondary copy is read-only (access requires failover)

**When to use:** Critical business data, disaster recovery requirement

### RA-GRS (Read-Access Geo-Redundant Storage)

**What it does:** Same as GRS, but secondary copy is readable without failover

```text
Primary Region (East US)
├── Can read AND write

Secondary Region (West US)
├── Can read ONLY (no failover needed)
```

**Use case:**

```text
Application reads from primary (East US)
If primary fails, application switches to reading from secondary (West US)
    ↓
Result: Faster read failover than GRS (no admin intervention to failover)
```

**When to use:** Read-heavy scenarios where secondary region needs immediate read access

### GZRS (Geo-Zone-Redundant Storage)

**What it does:** Zones in primary region + 3 copies in secondary region

```text
Primary Region (East US)
├── Zone 1: Copy 1
├── Zone 2: Copy 2
└── Zone 3: Copy 3

Secondary Region (West US)
├── Copy 1
├── Copy 2
└── Copy 3
```

**Resilience:** Best of both worlds (zone + region protection)

**Cost:** Highest

**When to use:** Critical production data requiring both zone and region redundancy

### RA-GZRS (Read-Access Geo-Zone-Redundant Storage)

**What it does:** GZRS + secondary copy is readable without failover

**Cost:** Highest

**When to use:** Critical production requiring both zone/region redundancy AND immediate secondary read access

### Comparison Table

| Type | Zones Protected | Regions Protected | Secondary Readable | Main cost driver |
|---|---|---|---|---|
| LRS | No: one physical datacenter | N/A | N/A | Local copies only |
| ZRS | Yes: three or more zones in primary region | N/A | N/A | Cross-zone replication |
| GRS | No: LRS primary | Yes | Only after failover | Cross-region replication |
| RA-GRS | No: LRS primary | Yes | Yes, without failover | Cross-region replication plus read access |
| GZRS | Yes | Yes | Only after failover | Cross-zone and cross-region replication |
| RA-GZRS | Yes | Yes | Yes, without failover | Cross-zone and cross-region replication plus read access |

**Decision boundary:** Redundancy copies data; it does not provide an application failover plan or a point-in-time recovery point. Choose ZRS for resilience to an availability-zone failure in the primary region. Choose GRS or RA-GRS for asynchronous replication to a paired secondary region, understanding that the primary replica is LRS. Choose GZRS or RA-GZRS when the primary region also requires zone redundancy. Read access to the secondary is an RA option, not an automatic write failover.

---

## Storage Authentication and Authorization

### Account Key (Primary/Secondary)

**What it is:** One of two storage account access keys used for Shared Key authorization.

```text
Primary Key: DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=abcd1234...;EndpointSuffix=core.windows.net
```

**Access:**

- ✅ Authorize supported storage **data-plane** requests for the account's services
- ✅ Read, write, and delete data when the requested operation is allowed by the storage service
- ✅ Generate account SAS and service SAS tokens signed with that key

**Risk:** If key is exposed, the holder has broad data-plane access for the storage services supported by that key. Treat it as a high-value secret and rotate it safely.

**Memorable boundary:**

```text
Storage account key → storage data access (Shared Key)
Azure management RBAC → resource configuration through Azure Resource Manager
```

An account key does not itself grant Azure Resource Manager permission such as `Microsoft.Storage/storageAccounts/write`. A management role such as Storage Account Contributor controls account configuration; a storage key or a data-plane role controls storage data operations.

**Rotation:**

```text
1. Regenerate the **secondary** key while applications still use the primary key.
2. Update and verify applications to use the new secondary key.
3. Regenerate the primary key only after no application depends on it.
4. Optionally repeat the process to return applications to the primary key.
```

**When to use:** Use only where Shared Key is required. Prefer Microsoft Entra ID, managed identities, or a user delegation SAS where supported; Shared Key can be disabled at account level when it is not needed.

### Shared Access Signature (SAS)

**What it is:** Time-limited, permission-limited access token

**Structure:**

```text
SAS Token = Credentials + Permissions + Expiration

Example:
https://myaccount.blob.core.windows.net/container/blob?sv=<version>&se=<expiry>&sp=r
                                                       ↑            ↑       ↑
                                                  API version      Expiration  Permission (read)
```

**Types of SAS:**

| Type | Who creates | Use case |
|---|---|---|
| **Account SAS** | Principal or service with Shared Key access | Scoped access across one or more storage services |
| **Service SAS** | Principal or service with Shared Key access | Scoped access to a resource in one storage service |
| **User delegation SAS** | Microsoft Entra principal authorized to request a user delegation key | Delegates supported storage data access using Entra credentials |

**Permissions:**

```text
r = Read
a = Add (append)
c = Create
w = Write
d = Delete
l = List
```

**Stored Access Policy:**

```text
Define start time, expiry, and permissions on a container, file share, queue, or table
    ↓
Reference the policy from one or more service SAS tokens
    ↓
Change or delete the policy to update or revoke the associated service SAS tokens
```

Stored access policies apply only to **service SAS**. Account SAS and user delegation SAS are ad hoc and cannot reference a stored access policy.

**When to use:**

- Temporary access (download link expires in 1 hour)
- External partners (limit what they can access)
- Client-side operations (browser directly accesses storage)

### Microsoft Entra ID (Identity-Based Access)

**What it is:** Use Entra user identity or managed identity for authentication

**Flow:**

```text
Application authenticates with Entra ID
    ↓
Gets access token
    ↓
Uses token to access storage
```

**Advantages:**

- No keys to rotate
- Audit trail shows which user/app accessed data
- Can disable access instantly by revoking role

**Role assignment:**

```text
Assign: alice@contoso.com + "Storage Blob Data Reader" + storage account scope
    ↓
Result: alice can read blobs, cannot modify
```

**When to use:**

- Applications running in Azure (use managed identity)
- Human users accessing storage
- Any scenario where you want audit trails

### Storage Account Roles (Data Plane)

**Important:** Data plane roles are **different** from RBAC control plane roles

| Role | What it allows | Access |
|---|---|---|
| **Storage Blob Data Owner** | Read/write/delete blobs, manage permissions | Full blob access |
| **Storage Blob Data Contributor** | Read/write/delete blobs | Modify blobs only |
| **Storage Blob Data Reader** | Read blobs | Read-only |
| **Storage Queue Data Contributor** | Read/write/delete queue messages | Full queue access |
| **Storage File Data SMB Share Contributor** | Read/write/delete files (SMB) | Modify share files |

**Key distinction:**

```text
Control Plane (RBAC):
├── "Storage Account Contributor" = can delete storage account

Data Plane (Storage roles):
├── "Storage Blob Data Reader" = can read blobs inside storage account
```

**Exam trap:** Control-plane roles manage the storage resource; data-plane roles authorize data operations. A workload that uses Microsoft Entra authentication to read a blob needs the appropriate data-plane role, not `Storage Account Contributor`. Portal navigation can separately require management-plane read access.

---

## Storage Networking Security

### Public Endpoint (Default)

**What it is:** Storage account accessible from public internet

```
Internet
    ↓
Storage Account Public Endpoint
```

**Access:** Anyone with account key or SAS can access

**Risk:** No network isolation, only credential-based protection

### Storage Firewall

**What it does:** Block access from unauthorized networks

**Configuration:**

```text
Allowed Networks:
├── Azure services (check box)
├── Specific VNets (list)
└── Specific IP addresses (list)

All other traffic: Denied
```

**Effect:**

```text
Internet client tries to access storage
    ↓
Firewall: "Your IP not in allowed list"
    ↓
Result: Connection denied
```

### Service Endpoint

**What it does:** Allow traffic from a VNet to the service's public endpoint via Azure backbone network

```
VNet Subnet
    ↓
Service Endpoint (special network rule)
    ↓
Service Public Endpoint (storage.azure.com)
```

**Configuration:**

```text
1. Enable Service Endpoint on VNet subnet
2. Storage account firewall: Add VNet to allowed networks
3. VM in subnet can now access storage via service endpoint
```

**Key characteristic:** Still uses service's public endpoint, but traffic stays within Azure backbone (doesn't go through internet)

**When to use:**

- VNet resources need storage access
- Don't need private IP resolution
- Want to restrict to specific VNets

### Private Endpoint

**What it does:** Create a private IP address for the service inside your VNet

```
VNet Subnet
    ↓
Private Endpoint (storage gets private IP in subnet)
    ↓
Example: 10.0.1.5 (private IP)
    ↓
Application uses account.blob.core.windows.net; DNS returns 10.0.1.5
```

**Configuration:**

```text
1. Create private endpoint for storage account
2. Specify VNet and subnet
3. Private IP created in that subnet
4. Storage gets private IP; restrict or disable public network access separately if required
5. Link the service-specific Private DNS zone, `privatelink.blob.core.windows.net`, to the VNet
```

**Name resolution:**

```
Application tries: account.blob.core.windows.net
    ↓
Public service FQDN resolves through the Private Link alias
    ↓
Private DNS Zone: privatelink.blob.core.windows.net
    ↓
Private DNS A record resolves → 10.0.1.5 (private IP)
    ↓
Connection via private network
```

**DNS rule:** Clients continue to use the normal service FQDN. The Private Link DNS chain directs the lookup to the service-specific `privatelink` zone, whose A record returns the private endpoint IP. Creating a private endpoint does not automatically disable the service public endpoint.

### Service Endpoint vs. Private Endpoint

| Aspect | Service Endpoint | Private Endpoint |
|---|---|---|
| **Network model** | Service still public endpoint | Service gets private IP |
| **IP address** | Service's public IP | Private IP in your VNet |
| **DNS** | Standard service hostname resolves publicly | `privatelink.blob.core.windows.net` maps the Blob service name to the private endpoint IP |
| **Cost** | No separate service-endpoint charge | Private Link pricing varies by region and data processed; check current pricing |
| **Configuration complexity** | Simple (enable on VNet) | Complex (need private DNS) |
| **Access from on-premises** | Via ExpressRoute | Via ExpressRoute/VPN + Private DNS |
| **Use case** | Simple VNet-to-service access | Strict private network isolation |

---

## Azure Blob Storage

### Blob Containers and Structure

**Container:** Folder-like collection

```text
Storage Account: myaccount
    ↓
Container: photos
    ├── 2024/vacation/beach.jpg
    ├── 2024/vacation/sunset.jpg
    └── 2024/mountain/peak.jpg

Container: documents
    ├── contract.pdf
    └── invoice.docx
```

**Blob types:**

- **Block Blob** — Files built from blocks (images, documents)
- **Page Blob** — Fixed-size pages (VM disks)
- **Append Blob** — Optimized for append-only (logs)

### Access Tiers

**Hot Tier:**

- Cost: Highest storage cost
- Access: Immediate
- Use: Frequently accessed data
- Minimum duration: None

**Cool Tier:**

- Cost: Lower storage, higher access cost
- Access: Online, milliseconds to first byte
- Use: Infrequent access (at least once per 30 days)
- Minimum billing: 30 days

**Cold Tier:**

- Cost: Lower storage and higher access cost than Cool
- Access: Online, milliseconds to first byte
- Use: Rarely accessed data that still needs immediate retrieval
- Minimum billing: 90 days

**Archive Tier:**

- Cost: Lowest storage, highest access cost
- Access: Hours (rehydration needed)
- Use: Rarely accessed (kept for compliance/backup)
- Minimum billing: 180 days

**Constraint:** Archive is supported only with LRS, GRS, or RA-GRS. It is not supported with ZRS, GZRS, or RA-GZRS.

**Selection example:**

```text
New data: Hot (developer needs immediate access)
    ↓ 30 days later
Move to: Cool (occasional reports need it)
    ↓ 90 days later
Move to: Archive (compliance requires 7-year retention)
```

### Rehydration

**What it is:** Move data from Archive back to Hot/Cool for access

**Process:**

```text
1. Blob in Archive tier
2. Request rehydration (change tier to Hot or Cool)
3. Rehydration starts (several hours)
4. Blob available in new tier
5. Cost: Rehydration fee + new tier storage cost
```

**When to use:**

```
"We need that 2020 log file for an audit"
    ↓
Request rehydration
    ↓
Wait hours
    ↓
Access rehydrated data
    ↓
Move back to Archive when done
```

### Blob Versioning

**What it does:** Keep multiple versions of a blob automatically

```text
Blob: report.xlsx
    ├── Version 2024-12-20T15:00:00Z
    ├── Version 2024-12-20T14:00:00Z
    ├── Version 2024-12-20T13:00:00Z
    └── Version 2024-12-19T10:00:00Z
```

**Restore pattern:**

```text
Current version corrupted
    ↓
Restore from previous version
    ↓
Old version becomes current
```

**Cost:** Each version stored counts toward account storage quota

### Soft Delete

**What it does:** Recover accidentally deleted blob data or containers within a configured retention period.

**Blob soft-delete retention period:** currently 1–365 days (configurable)

**How it works:**

```text
1. Enable the required protection at the storage account: blob soft delete, container soft delete, or both.
2. Deleted data remains recoverable during the configured retention period.
3. Restore the soft-deleted blob or the whole deleted container before retention expires.
4. After retention expires, the data is permanently deleted.
```

**Scope boundary:** Blob soft delete protects individual blobs, snapshots, and versions. Container soft delete is a separate setting that restores a deleted container and its contents to their state at deletion; the original container name must be available. Neither feature restores a deleted storage account—use an Azure Resource Manager lock to reduce that risk.

**Difference from versioning:**

```text
Blob soft delete = recover deleted blob data
Container soft delete = recover a deleted container and its contents
Versioning = recover overwritten blobs
Together = more complete in-account data protection
```

### Snapshots

**What it is:** Point-in-time copy of a blob

```text
Blob: backup.vhd (Version 1)
    ↓
Create snapshot
    ↓
Blob: backup.vhd (Version 2, different content)
    ↓
Snapshot: Captures original state
    ↓
Can restore snapshot anytime
```

**Cost:** Incremental (only changed blocks stored)

### Blob Lifecycle Management

**What it does:** Automatically move blobs between tiers or delete them

**Example rule:**

```text
IF blob age > 30 days
THEN move to Cool tier

IF blob age > 90 days
THEN move to Archive tier

IF blob age > 365 days
THEN delete

IF blob has not been accessed for 180 days
THEN move to Archive (requires last-access-time tracking to be enabled)
```

**Automation:** Lifecycle policies run asynchronously. Do not depend on an exact execution time for an individual blob.

**Cost savings:** Automatic tiering without manual intervention

---

## Azure Files

### File Shares

**What it is:** SMB/NFS file share accessible by multiple clients

**Mount:**

```text
Windows VM:
├── net use Z: \\storageaccount.file.core.windows.net\sharename /user:Azure\storageaccount [key]
    ↓
Z: drive available

Linux VM:
├── mount -t nfs storageaccount.file.core.windows.net:/storageaccount/sharename /mnt/files
    ↓
/mnt/files available
```

### Identity-Based Authentication

**What it means:** Use Kerberos-based identity authentication for SMB Azure file shares instead of the storage account key. It is separate from using an account key or SAS and is not supported for NFS file shares.

**Supported identity sources for SMB:**

| Identity source | Best fit | Important dependency |
|---|---|---|
| **On-premises Active Directory Domain Services (AD DS)** | Existing AD DS environment | Hybrid users must be synchronized to Microsoft Entra ID; clients need domain-controller connectivity. |
| **Microsoft Entra Domain Services** | Managed domain in Azure | Clients need connectivity to and usually membership in the managed domain. |
| **Microsoft Entra Kerberos** | Microsoft Entra-joined cloud-first or hybrid Windows clients | Cloud-only or hybrid identities are supported; Linux user authentication is not supported. |

Only one user identity source is configured per storage account and applies to all its file shares. A managed identity can separately provide keyless SMB access for supported Azure workloads; it does not replace the user identity-source choice.

**Access layers:**

| Layer | Decision it answers | Example |
|---|---|---|
| **Authentication** | Can this identity obtain a Kerberos ticket for SMB? | AD DS, Entra Domain Services, or Entra Kerberos |
| **Share-level authorization** | Can the identity access this share? | `Storage File Data SMB Share Contributor` role |
| **File and directory authorization** | What can the identity do after entering the share? | NTFS ACLs where supported by the chosen identity source |
| **Network access** | Can the client reach the share endpoint? | Private Endpoint, firewall, DNS, and SMB connectivity |

Cloud-only Microsoft Entra users do not automatically work with every Azure Files identity configuration. Select an identity source that supports the clients and identities in use, then configure all four layers.

**Process:**

```text
1. Enable Entra ID or on-premises AD on storage account
2. Assign user RBAC role: "Storage File Data SMB Share Contributor"
3. User mounts share without password (uses Entra token)
4. On-premises users mount with Entra hybrid identity (Entra Connect)
```

### Share-Level Roles and NTFS Permissions

**Two-layer access control:**

| Layer | Controls | How |
|---|---|---|
| **Share-level role** | Can access share? | RBAC role assignment |
| **NTFS permissions** | What files can modify? | Windows permissions on files |

**Example:**

```text
Alice:
├── Assigned role: "Storage File Data SMB Share Elevated Contributor" (share level)
├── NTFS permission on \\share\documents: Modify
├── NTFS permission on \\share\admin: Read-only
    ↓
Result: Alice can modify files in documents folder, but only read admin folder
```

### File Share Snapshots

**What it is:** Point-in-time copy of entire share

```text
Share: company-data (1000 files)
    ↓
Create snapshot
    ↓
Share modified: files added/deleted/changed
    ↓
Snapshot preserved: shows original state
```

**Restore:**

```text
Deleted important file
    ↓
Restore from snapshot
    ↓
File recovered
```

### File Share Soft Delete

**What it does:** Recover deleted file shares within retention period

**File-share soft-delete retention:** currently 1–365 days

**Scenario:**

```text
Admin accidentally deletes share containing payroll data
    ↓
Within retention period: Restore entire share
    ↓
All files recovered
```

---

## Encryption

### Encryption at Rest (Default)

**What it does:** Data encrypted on disk automatically

**How:**

```text
Data: "MyPassword123"
    ↓
Encryption: AES-256
    ↓
Encrypted on disk: "a7x8m9n2k3j4h5g6..."
```

**Important:** Enabled by default, cannot be disabled

**Encryption key management:**

- **Microsoft-managed keys** — Azure manages the encryption keys (default)
- **Customer-managed keys** — You manage encryption keys in Azure Key Vault or Managed HSM, subject to the storage feature's support and required permissions

### Encryption in Transit

**What it does:** Data encrypted while moving across network

**How:**

```text
Client → Storage
    ↓
HTTPS only (TLS 1.2 minimum)
    ↓
Data encrypted in transit
```

**Requirement:** Enforce HTTPS (disable HTTP)

---

## Blob Replication

### Object Replication

**What it does:** Automatically copy changed blobs to secondary storage account in different region

**Configuration:**

```text
Primary Storage Account
├── Blob A (created)
├── Blob B (created)
└── Blob A (modified)
    ↓
Object Replication Rule
├── Source: Primary account, container "data"
└── Destination: Secondary account, container "data-backup"
    ↓
Result:
    ├── Blob A → Secondary (creates and updates replicated)
    └── Blob B → Secondary (created)
```

**Use case:**

```text
Requirement: Keep backup copy in different region automatically
    ↓
Solution: Object Replication
    ↓
Result: Async copy of all blob changes
```

**Important:** Replication is **async** (eventual consistency, not immediate)

### Object Replication Prerequisites and Limits

Object replication copies **block blobs** asynchronously between a source and destination storage account. It is configured by policy and rules, not by the account redundancy setting.

- Enable **change feed** on the source account and **blob versioning** on both accounts.
- Use supported General-purpose v2 or premium block blob accounts. Page blobs, append blobs, snapshots, and accounts with hierarchical namespace enabled are not supported.
- Create the source and destination containers before creating a rule. The destination is read/delete capable but does not accept writes while its replication rule is active.
- Source and destination can be in the same or different regions, subscriptions, or Microsoft Entra tenants when the applicable cross-tenant setting permits it. They are not required to be in the same resource group or subscription.
- Archive-tier blobs are not replicated. Changing a blob's tier does not replicate its tier selection.

| Feature | Azure Storage redundancy | Object replication |
|---|---|---|
| **Who controls copies** | Azure, based on the account redundancy option | Administrator, using source/destination rules |
| **Copy target** | Azure-managed secondary replica for geo-redundant options | A separate destination storage account and container |
| **Data model** | Account-level durability | Asynchronous block-blob replication with versioning |
| **Use when** | The account needs durability/resilience | Specific blob data must be distributed or processed in another account/region |

---

## Managing Data with AzCopy and Storage Explorer

### Azure Storage Explorer

**What it is:** GUI tool for managing Azure Storage

**Access:** Download from Microsoft (free)

**Capabilities:**

- Upload/download blobs
- Manage containers
- Copy between accounts/regions
- Edit metadata
- Share via SAS token

**Use case:** One-time uploads, small file management, visual organization

### AzCopy

**What it is:** Command-line utility for high-performance data transfer

**Installation:** `azcopy` command-line tool

**Key advantages:**

- Bulk transfers
- Resumable operations
- Parallel uploads
- Scriptable/automatable
- Cross-account/region copies
- Performance optimized

**Common scenarios:**

```text
Upload local files to Blob Storage:
$ azcopy copy "C:\LocalFolder\*" "https://myaccount.blob.core.windows.net/container" --recursive

Download from Blob Storage:
$ azcopy copy "https://myaccount.blob.core.windows.net/container/file.txt" "C:\Downloads\"

Copy between storage accounts:
$ azcopy copy "https://source.blob.core.windows.net/container/*" "https://dest.blob.core.windows.net/container/" --recursive

Copy with SAS token (for authentication):
$ azcopy copy "C:\data\*" "https://account.blob.core.windows.net/container?<SAS-token>" --recursive
```

**Authentication options:**

- Managed Identity (in Azure VMs/App Service)
- Service Principal
- User credentials (interactive prompt)
- SAS token
- Storage account key

**Performance tips:**

```text
✓ Use recursive flag for folders
✓ Run from Azure VM for speed (bandwidth)
✓ Parallelization automatic
✓ Can resume interrupted transfers
✓ Compress large files before transfer
```

### Comparison: Storage Explorer vs. AzCopy

| Aspect | Storage Explorer | AzCopy |
|---|---|---|
| **Interface** | GUI | Command-line |
| **Use case** | One-off, small transfers | Bulk, automated transfers |
| **Performance** | Good for manual use | Optimized for large-scale |
| **Scriptable** | No | Yes |
| **Learning curve** | Easy | Moderate |
| **Best for** | Testing, browsing | Production, automation |

**Real scenario:**

```text
Weekly backup of on-premises data to Azure:
    ↓
Use AzCopy in scheduled script
    ↓
Bulk upload every Sunday at 2 AM
    ↓
Result: Automated, efficient, resumable
```

---

## Don't Confuse

### Blob Storage vs. Azure Files

| Aspect | Blob Storage | Azure Files |
|---|---|---|
| **Access model** | RESTful HTTP/HTTPS | SMB/NFS mount |
| **Use case** | Images, videos, backups | File shares, home directories |
| **Mount** | Not mounted, accessed by URL | Mounted like local drive |
| **Redundancy** | LRS, ZRS, GRS, etc. | Same redundancy options |

### Service Endpoint vs. Private Endpoint

| Aspect | Service Endpoint | Private Endpoint |
|---|---|---|
| **IP** | Service's public IP | Private IP in your VNet |
| **Cost** | No separate service-endpoint charge | Private Link charges depend on region and data processed |
| **Setup** | Simple | Complex (Private DNS needed) |
| **Access** | From specific VNets | From specific VNets + on-premises (via VPN) |

### Soft Delete vs. Versioning

| Aspect | Soft Delete | Versioning |
|---|---|---|
| **Protects against** | Accidental deletion | Accidental overwrite |
| **Retention** | Configurable, currently 1–365 days | Indefinite |
| **Cost** | Soft-deleted data still counts | Each version counts |

### Backup vs. Replication

| Aspect | Backup | Replication |
|---|---|---|
| **Purpose** | Point-in-time restore | Continuous redundancy |
| **Target** | Workload-supported vault or operational tier | Another storage account |
| **Retention** | User-defined (days/years) | Continuous |
| **Use case** | Recover deleted data, compliance | Disaster recovery |

---

## Real-World Scenario

### Contoso Finance Application

**Requirements:**

- Financial documents must be encrypted
- Access only from corporate network
- Automatically archive documents older than 1 year
- Recover accidentally deleted files within 30 days
- Replicate to secondary region for disaster recovery

**Solution:**

```text
Storage Account: financeaccount (GRS redundancy)
    ├── Encryption at Rest: Enabled (default)
    ├── Network: Private Endpoint + Private DNS
    │   └── Only accessible from finance VNet
    ├── Authentication: Managed Identity
    │   └── Finance app authenticates without keys
    │
    ├── Container: documents
    │   ├── Soft Delete: Enabled (30 days)
    │   ├── Versioning: Enabled
    │   ├── Snapshot: Daily backups
    │   └── Lifecycle: Move to Archive after 365 days
    │
    └── Object Replication
        └── Replicate to secondary region automatically

Result:
├── Encrypted storage (at rest + transit)
├── Private network only (no internet exposure)
├── Automatic archival for old documents
├── 30-day recovery window for accidents
├── Secondary region copy for DR
└── Full audit trail via managed identity
```

---

## Domain Summary

| Concept | Purpose | Trigger | Nearest Confusion | Key Exam Rule |
|---|---|---|---|---|
| **Blob Storage** | Store unstructured data | Images, videos, logs | Files | HTTP/REST access |
| **Azure Files** | File shares | Mount SMB/NFS | Blob | Like network drive |
| **Queue Storage** | App-to-app messaging | Async tasks | Topic | One-directional |
| **Account key** | Full access credential | Trusted server | SAS | High security risk |
| **SAS token** | Time-limited credential | Temporary external access | Account key | Controlled permissions |
| **Managed Identity** | Workload authentication | App needing storage access | SAS | No credentials to store |
| **LRS** | One-datacenter redundancy | Cost-sensitive, reconstructible data | ZRS | Not availability-zone resilience |
| **ZRS** | Zone redundancy | Zone resilience needed | LRS | Three or more zones in one region |
| **GRS** | Region redundancy | Disaster recovery | GZRS | Secondary is not readable before failover |
| **RA-GRS** | Region + read secondary | DR + read failover | GRS | Secondary readable |
| **GZRS** | Zone + region redundancy | High availability | GRS | Three zones + regions |
| **Service Endpoint** | Network policy to public endpoint | VNet to service | Private Endpoint | Cheaper, simpler |
| **Private Endpoint** | Private IP for service | Strict network isolation | Service Endpoint | In your VNet |
| **Soft Delete** | Recover deleted blobs | Accidental deletion | Versioning | Retention period |
| **Versioning** | Recover overwritten blobs | Accidental overwrite | Soft Delete | Indefinite |
| **Lifecycle Management** | Auto tier/delete | Age-based policies | Manual | Automatic |
| **Object Replication** | Copy to secondary region | DR redundancy | Backup | Async, continuous |

---

## 30-Second Recall

**Storage Services:**
- Blob = unstructured (images, videos)
- Files = SMB/NFS shares
- Queue = async messages
- Table = key-value

**Redundancy:**
- LRS = copies in one physical datacenter
- ZRS = synchronous copies across availability zones
- GRS = region copy (secondary is not readable before failover)
- RA-GRS = region + secondary read
- GZRS = zone + region (zones + regions)

**Authentication:**
- Account key = full access, high risk
- SAS = time-limited, controlled permissions
- Managed Identity = workload auth, no credentials
- Entra ID = user identity

**Networking:**
- Public = internet accessible
- Service Endpoint = VNet to public endpoint
- Private Endpoint = private IP in VNet
- Firewall = IP/VNet restrictions

**Data Protection:**
- Encryption at rest = default
- Soft Delete = recover deleted
- Versioning = recover overwritten
- Backup = point-in-time restore
- Lifecycle = auto tier/delete
- Replication = secondary copy

**Key Distinctions:**
- Blob vs. Files = HTTP vs. SMB/NFS mount
- LRS vs. ZRS = one datacenter vs. multiple availability zones
- GRS vs. RA-GRS = secondary not readable vs. readable
- Soft Delete vs. Versioning = deleted vs. overwritten
- Service Endpoint vs. Private Endpoint = public endpoint vs. private IP
- Backup vs. Replication = point-in-time vs. continuous

---

## Official Sources

- [Azure Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/)
- [Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Azure Files](https://learn.microsoft.com/en-us/azure/storage/files/)
- [Storage Account Security](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Azure Storage Redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy)
- [Private Endpoints](https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints)
