# Implement and Manage Storage

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
└── Backup to Recovery Services Vault
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
- **Access tier** — Hot (frequently accessed) or Cold (infrequent)
- **Redundancy** — LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS

### Storage Account SKUs

| SKU | Details | Use Case |
|---|---|---|
| **Standard** | General-purpose, older | Most workloads |
| **Premium** | Higher performance, IOPS focus | High-transaction scenarios |
| **Blob Storage** | Optimized for blobs | Legacy, now use Standard |

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

- **Hot tier** — Immediately accessible, highest cost
- **Cool tier** — Delayed access (30 days minimum), lower cost
- **Archive tier** — Longest delay (90 days minimum), lowest cost
- **Rehydrate** — Upgrade cold/archive to hot when needed

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
2. Consumer reads message (message stays in queue, invisible for 30 seconds)
3. Consumer processes message
4. Consumer deletes message (optional, automatic after timeout)
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
LRS / ZRS
→ Zone focus (resilient to zone failure)

GRS / GZRS / RA-GRS / RA-GZRS
→ Region focus (resilient to region failure)
```

### LRS (Locally Redundant Storage)

**What it does:** 3 copies within one availability zone

```text
One Datacenter (Availability Zone)
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
- ✅ Survives zone failure
- ✅ Survives region failure (primary region destroyed, secondary is still available)

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

| Type | Zones Protected | Regions Protected | Secondary Readable | Cost |
|---|---|---|---|---|
| LRS | Same zone | N/A | N/A | $ |
| ZRS | ✅ Same region | N/A | N/A | $$ |
| GRS | ✅ | ✅ | Only via failover | $$$ |
| RA-GRS | ✅ | ✅ | ✅ Without failover | $$$ |
| GZRS | ✅ | ✅ | Only via failover | $$$$ |
| RA-GZRS | ✅ | ✅ | ✅ Without failover | $$$$ |

---

## Storage Authentication and Authorization

### Account Key (Primary/Secondary)

**What it is:** 88-character key granting full access

```text
Primary Key: DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=abcd1234...;EndpointSuffix=core.windows.net
```

**Access:**

- ✅ Read all data
- ✅ Write all data
- ✅ Delete all data
- ✅ Modify account settings

**Risk:** If key is exposed, attacker has full account access

**Rotation:**

```text
1. Generate new secondary key
2. Update applications to use secondary key
3. Delete primary key
4. Generate new primary key
5. Rotate back to primary when ready
```

**When to use:** Server-to-server communication, trusted applications only

### Shared Access Signature (SAS)

**What it is:** Time-limited, permission-limited access token

**Structure:**

```text
SAS Token = Credentials + Permissions + Expiration

Example:
https://myaccount.blob.core.windows.net/container/blob?sv=2024-02-04&se=2024-12-31T23:59:59Z&sp=r
                                                       ↑                    ↑                    ↑
                                                   API Version         Expiration            Permission (r=read only)
```

**Types of SAS:**

| Type | Who creates | Use case |
|---|---|---|
| **Account SAS** | Storage account owner | Full account delegation with limited permissions |
| **Service SAS** | Storage account owner | Limited to one service (Blob, Queue, File, Table) |
| **User delegation SAS** | Storage account owner (but uses Entra creds) | Supports Azure AD authorization |

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
Define SAS once in policy
    ↓
Reference policy when creating SAS
    ↓
Revoke all SAS at once by deleting policy (faster than individual revocation)
```

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

**Two-layer access:**

```text
Step 1: Do you have RBAC permission to access the storage account?
        → Need Storage Account Contributor or higher (control plane)

Step 2: Do you have data plane role to read/write blobs?
        → Need Storage Blob Data Reader or higher (data plane)

Both layers required for access.
```

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
Application access storage via 10.0.1.5 (private network)
```

**Configuration:**

```text
1. Create private endpoint for storage account
2. Specify VNet and subnet
3. Private IP created in that subnet
4. Storage gets private IP, public endpoint disabled (optional)
5. Create Private DNS Zone to resolve storage.blob.core.windows.net → 10.0.1.5
```

**Name resolution:**

```
Application tries: storage.blob.core.windows.net
    ↓
Private DNS Zone resolves → 10.0.1.5 (private IP)
    ↓
Connection via private network
```

### Service Endpoint vs. Private Endpoint

| Aspect | Service Endpoint | Private Endpoint |
|---|---|---|
| **Network model** | Service still public endpoint | Service gets private IP |
| **IP address** | Service's public IP | Private IP in your VNet |
| **DNS** | resolves.blob.core.windows.net | Private DNS zone (custom) |
| **Cost** | Free | Paid (~$0.01/hour per endpoint) |
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
- Access: 30+ seconds
- Use: Infrequent access (at least once per 30 days)
- Minimum billing: 30 days

**Archive Tier:**

- Cost: Lowest storage, highest access cost
- Access: Hours (rehydration needed)
- Use: Rarely accessed (kept for compliance/backup)
- Minimum billing: 90 days

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

**What it does:** Recover accidentally deleted blobs within retention period

**Retention period:** 1-365 days (configurable)

**How it works:**

```text
1. Blob deleted
2. Blob marked as "soft deleted" (not immediately removed)
3. Within retention period: Can recover
4. After retention expires: Permanently deleted
```

**Difference from versioning:**

```text
Soft Delete = recover accidentally deleted blobs
Versioning = recover overwritten blobs
Both = comprehensive data protection
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

IF blob NOT accessed for 180 days
THEN move to Archive
```

**Automation:** Rules run daily at midnight UTC

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

**What it means:** Use Entra ID or on-premises AD instead of storage key

**Requirements:**

- Storage account in Standard tier (not Premium)
- Enable "Identity-based access for Azure file shares"
- Azure RBAC role assignment for user/group

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

**Retention:** 1-365 days

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

**Certificate management:**

- **Microsoft-managed keys** — Azure manages encryption certificates (default)
- **Customer-managed keys** — You manage keys in Azure Key Vault (advanced)

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
| **Cost** | Free | Paid |
| **Setup** | Simple | Complex (Private DNS needed) |
| **Access** | From specific VNets | From specific VNets + on-premises (via VPN) |

### Soft Delete vs. Versioning

| Aspect | Soft Delete | Versioning |
|---|---|---|
| **Protects against** | Accidental deletion | Accidental overwrite |
| **Retention** | Configurable (1-365 days) | Indefinite |
| **Cost** | Soft-deleted data still counts | Each version counts |

### Backup vs. Replication

| Aspect | Backup | Replication |
|---|---|---|
| **Purpose** | Point-in-time restore | Continuous redundancy |
| **Target** | Recovery Services Vault | Another storage account |
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
| **LRS** | Local redundancy | Cost-sensitive | ZRS | Single zone |
| **ZRS** | Zone redundancy | Zone resilience needed | LRS | Three zones, one region |
| **GRS** | Region redundancy | Disaster recovery | GZRS | Secondary read-only |
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
- LRS = local copy
- ZRS = zone copies
- GRS = region copy (secondary read-only)
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
- LRS vs. ZRS = single zone vs. three zones
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
