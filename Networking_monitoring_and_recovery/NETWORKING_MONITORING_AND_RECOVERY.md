# Networking, Monitoring, and Recovery

---

**← [Previous: Compute](../Deploy_and_manage_Azure_compute_resources/DEPLOY_AND_MANAGE_AZURE_COMPUTE_RESOURCES.md)** | **[Home](../README.md)** | **[Next: High-Yield Recall →](../AZ-104_HIGH_YIELD_RECALL.md)**

---

This guide covers three official Microsoft domains consolidated into one learning resource:

- **PART I — Virtual Networking** (Microsoft Domain 4)
- **PART II — Monitoring and Observability** (part of Microsoft Domain 5)
- **PART III — Backup and Recovery** (part of Microsoft Domain 5)

---

# PART I — VIRTUAL NETWORKING

## Big Picture

An Azure administrator is responsible for:

- **How resources communicate** (VNets, subnets, routing)
- **Who can reach what** (NSG, firewall, filters)
- **Private vs. public** (endpoints, DNS, access)
- **Connectivity** (on-premises, multi-region)
- **Performance** (load balancing, CDN)
- **Troubleshooting** (diagnostics, monitoring)

---

## Mental Model

```text
Address Space
├── VNet (10.0.0.0/16)
│   └── Subnets (10.0.1.0/24, 10.0.2.0/24)
│
Routing
├── System routes (default)
└── User-defined routes (custom next-hop)
    ↓
Filtering (Security)
├── NSG (inbound/outbound allow/deny)
├── Application Security Groups (grouping)
└── Azure Firewall (centralized)
    ↓
Private Connectivity
├── VNet Peering (VNET-to-VNET within Azure)
├── Service Endpoint (access PaaS via public endpoint)
├── Private Endpoint (access PaaS via private IP)
└── Private DNS Zone (name resolution)
    ↓
Management Access
└── Azure Bastion (RDP/SSH without public IP)
    ↓
Load Balancing
├── Load Balancer (Layer 4: TCP/UDP)
└── Application Gateway (Layer 7: HTTP/HTTPS)
    ↓
Name Resolution
├── Azure DNS (public domains)
└── Private DNS Zone (internal)
```

---

## VNet and Subnets

### Virtual Network

**What it is:** Private network namespace for Azure resources

```text
VNet: 10.0.0.0/16
├── Can contain: VMs, storage, databases, app services
├── Subnets: Logical divisions
├── Route table: How traffic flows
├── NSG: Filtering rules
└── DNS: Name resolution
```

**Key points:**

- **CIDR notation:** 10.0.0.0/16 = 65,536 IP addresses
- **Region-bound:** VNet in one region (but can peer across regions)
- **Subscription-bound:** VNet in one subscription (but can peer across subscriptions)
- **Overlap:** Cannot have duplicate address spaces (unless separate subscriptions)

### Subnets

**What it is:** Subdivision of VNet's address space

```text
VNet: 10.0.0.0/16
├── Subnet 1: 10.0.1.0/24 (256 IPs)
├── Subnet 2: 10.0.2.0/24 (256 IPs)
├── Subnet 3: 10.0.3.0/24 (256 IPs)
└── Subnet 4: 10.0.4.0/24 (256 IPs)
```

**Subnet properties:**

- **Name** — User identifier
- **Address range** — CIDR block
- **Network Security Group** — Filtering rules
- **Route table** — Traffic routing
- **Service endpoints** — Allow access to PaaS services
- **Private Endpoint** — PaaS resources in this subnet

**Reserved IPs in each subnet:**

```text
Subnet: 10.0.1.0/24
├── 10.0.1.0: Network address (reserved)
├── 10.0.1.1: Gateway (reserved)
├── 10.0.1.2: DNS resolver (reserved)
├── 10.0.1.3: First available for resources
│   ...
└── 10.0.1.255: Broadcast address (reserved)

Usable IPs: 10.0.1.3 to 10.0.1.254 (252 IPs)
```

---

## Public IP Addresses

**What it is:** Public-facing IP for inbound/outbound traffic

**Static vs. Dynamic:**

| Type | Cost | Use Case |
|---|---|---|
| **Dynamic** | Free | Temporary, okay if changes |
| **Static** | $ | DNS records, SSL certificates need stable IP |

**Scopes:**

```text
Resource level: Public IP attached to:
├── Network Interface (NIC)
├── Load Balancer
└── Application Gateway
```

**Example:**

```text
Public IP: 40.123.45.67
    ↓
Attached to: VM Network Interface
    ↓
VM private IP: 10.0.1.5
    ↓
Internet → 40.123.45.67 → NAT → 10.0.1.5
```

---

## VNet Peering

### What It Means

VNet peering connects two VNets for direct communication

```text
VNet-A: 10.0.0.0/16
    ↓ (peering)
VNet-B: 10.1.0.0/16
    ↓
Result: VMs in A can talk to VMs in B directly
```

### Peering Types

**Regional Peering:**

```text
VNet-A (East US) ← Peering → VNet-B (East US)
    ↓
Free peering
```

**Global Peering:**

```text
VNet-A (East US) ← Peering → VNet-B (West US)
    ↓
Small charge for cross-region peering
```

### Peering Properties

- **Allow forwarded traffic** — Forward traffic from outside the peering relationship
- **Allow gateway transit** — Use VPN gateway for on-premises access
- **Use remote gateway** — Use peer's VPN gateway

### Key Rules

```text
1. Non-transitive
   VNet-A ←→ VNet-B ←→ VNet-C
   But A cannot talk to C directly
   
2. No overlapping address spaces
   Cannot peer VNet-A: 10.0.0.0/16 with VNet-B: 10.0.0.0/16
   
3. Bidirectional
   A can talk to B
   B can talk to A
```

---

## NSG (Network Security Group)

### What It Means

NSG controls **whether traffic is allowed** (inbound/outbound filtering)

```text
Network Security Group
├── Inbound rules (FROM Internet TO Resource)
├── Outbound rules (FROM Resource TO Internet)
└── Allow/Deny decisions
```

### NSG Rules

**Rule structure:**

```text
Rule: Allow HTTP from Internet
├── Priority: 100 (lower = evaluated first)
├── Source: Internet (0.0.0.0/0)
├── Destination: Any
├── Protocol: TCP
├── Port: 80
└── Action: Allow
```

**Processing:**

```text
Packet arrives
    ↓
Check inbound rules (priority 100, 101, 102, ...)
    ↓
First matching rule wins
    ↓
Action: Allow or Deny
```

**Default rules (cannot be deleted):**

```text
Inbound default: Deny (everything denied unless explicitly allowed)
Outbound default: Allow (everything allowed unless explicitly denied)
```

### NSG Scope

**Network Interface (NIC):**

```
Applies to single VM's NIC
```

**Subnet:**

```
Applies to all VMs/resources in subnet
```

**Both:**

```
Packet filtered by both (most restrictive wins)
```

### Effective Security Rules

Tool to see which rules actually apply:

```text
VM: web-server
    ↓
NIC NSG rules: 3 rules
Subnet NSG rules: 5 rules
    ↓
Effective rules = combination of both NSGs
    ↓
What actually filters traffic
```

---

## Application Security Groups (ASG)

### What It Means

ASG groups resources for easier NSG rule management

```text
Traditional NSG:
├── Allow traffic from 10.0.1.4 (need IP)
├── Allow traffic from 10.0.1.7 (need IP)
└── If IPs change: Update rule

Application Security Group:
├── Create ASG: "WebServers"
├── Add members: web-vm-1, web-vm-2, web-vm-3
└── NSG rule: Allow from "WebServers" ASG
    ↓
Result: If add/remove VMs to ASG, rule unchanged
```

**Benefit:**

```text
Add new VM to "WebServers" ASG
    ↓
New VM automatically gets all ASG's permissions
    ↓
No NSG rule changes needed
```

---

## Route Tables and User-Defined Routes

### What It Means

Route table controls **where traffic goes** (routing decisions)

```text
Packet destination: 192.168.1.5
    ↓
Route table lookup
    ├── Does 192.168.1.5 match any route?
    ├── If yes: Send to next hop specified
    └── If no: Discard (drop)
```

### System Routes (Automatic)

```text
Every VNet gets automatic routes:
├── 10.0.0.0/16: Local (within VNet)
├── 0.0.0.0/0: Internet (default route)
└── For any subnet: Local delivery
```

### User-Defined Routes

Custom routes to override system routes:

```text
Route: 192.168.0.0/16 → 10.0.1.4
    ↓
Meaning: Any traffic to 192.168.0.0/16 send to 10.0.1.4
    ↓
Next hop IP: 10.0.1.4 (must be reachable)
```

**Next hop types:**

| Type | Example | Purpose |
|---|---|---|
| **Virtual Appliance** | Firewall VM (10.0.1.4) | Force traffic through firewall |
| **VPN Gateway** | On-premises gateway | Route to on-premises networks |
| **Virtual Network Gateway** | ExpressRoute gateway | Route via ExpressRoute |
| **None** | Drop traffic | Explicitly discard traffic |
| **VNet Peering** | Peer VNet | Route to peered VNet |

**Common pattern:**

```text
Requirement: All outbound traffic must go through firewall
    ↓
Solution:
1. Create Firewall VM in DMZ subnet
2. Create route: 0.0.0.0/0 (all traffic) → Firewall VM
3. Apply to application subnets
    ↓
Result: All traffic forced through firewall for inspection
```

---

## NSG vs. Route Table

| Aspect | NSG | Route Table |
|---|---|---|
| **Controls** | Can traffic pass? | Where does traffic go? |
| **Question** | Is this allowed? | Which interface/gateway? |
| **Action** | Allow/Deny | Specify next hop |
| **Example** | Block port 23 (Telnet) | Route 192.168.0.0/16 to firewall |
| **Applies to** | NIC or Subnet | Subnet |

**Scenario clarification:**

```text
NSG rule: Deny port 23 (Telnet)
Route: Send 192.168.0.0/16 to firewall
    ↓
Result: NSG denies telnet even if route tried to send it
    ↓
NSG is evaluated first (filtering), then routing
```

---

## Service Endpoints

### What It Means

Service endpoint allows VNet resources to access Azure PaaS services via their public endpoint

```text
VNet Subnet
    ↓
Service Endpoint (special routing rule)
    ↓
Azure PaaS Service (Storage, SQL, etc.)
    ↓
Traffic: From subnet to service's public endpoint
    ↓
Network: Stays on Azure backbone (not internet)
```

### Configuration

**Step 1: Enable Service Endpoint on subnet**

```text
Subnet: app-servers
    ↓
Enable Service Endpoint: Microsoft.Storage
    ↓
Routing: Traffic to Storage goes via backbone
```

**Step 2: Allow subnet in Storage firewall**

```text
Storage Account Firewall:
    ↓
Add: app-servers subnet
    ↓
Result: Storage allows traffic from this subnet via service endpoint
```

**Step 3: Application uses storage normally**

```text
Application code:
    storage.blob.core.windows.net
    ↓
Service endpoint routes to storage's public endpoint
    ↓
Storage firewall: Recognized as coming from allowed subnet
    ↓
Access: Granted
```

### Service Endpoint Characteristics

- **Public endpoint** — Still uses service's public DNS name
- **Private routing** — But traffic goes via Azure backbone (not internet)
- **No private IP** — Service doesn't get private IP in your VNet
- **Free** — No additional charges

### Supported Services

```text
Service Endpoints available for:
├── Azure Storage
├── Azure SQL Database
├── Azure Database for MySQL
├── Azure Cosmos DB
├── Azure Key Vault
├── Azure Event Hubs
├── Azure Service Bus
└── More coming
```

---

## Private Endpoints

### What It Means

Private endpoint creates a **private IP** for an Azure PaaS service inside your VNet

```text
VNet Subnet
    ↓
Private Endpoint (service gets private IP in subnet)
    ↓
Example: Storage Account private IP: 10.0.3.5
    ↓
Application: Connects to 10.0.3.5 (private network)
    ↓
Result: Complete private connectivity
```

### Configuration

**Step 1: Create private endpoint for service**

```text
Service: Storage Account (myaccount)
    ↓
Create Private Endpoint
├── VNet: Select your VNet
├── Subnet: Select subnet
└── Resource: myaccount
    ↓
Result: Private IP assigned (e.g., 10.0.3.5)
```

**Step 2: Create private DNS zone**

```text
Private DNS Zone: blob.core.windows.net
    ↓
Zone Resolution:
├── blob.core.windows.net → 10.0.3.5 (private IP)
└── Link to VNet
    ↓
Result: Applications resolve storage name to private IP
```

**Step 3: Application connects**

```text
Application tries: myaccount.blob.core.windows.net
    ↓
Private DNS: Resolves to 10.0.3.5
    ↓
Connection: Via private network
    ↓
Access: Granted
```

### Private Endpoint Characteristics

- **Private IP** — Service gets private IP in your VNet
- **Private DNS** — Usually requires Private DNS Zone for name resolution
- **Cost** — Charged per endpoint (~$0.01/hour)
- **Complex** — Requires Private DNS setup
- **Strict isolation** — Complete private network isolation

### Service Endpoint vs. Private Endpoint

| Aspect | Service Endpoint | Private Endpoint |
|---|---|---|
| **IP type** | Service's public IP | Private IP in VNet |
| **DNS** | resolves to public IP | Requires Private DNS Zone |
| **Network** | Public endpoint, private routing | Complete private networking |
| **Cost** | Free | Paid (~$0.01/hour) |
| **Setup** | Simple | Complex |
| **Use case** | VNet isolation sufficient | Strict private access required |
| **On-premises access** | Via ExpressRoute | Via ExpressRoute/VPN + Private DNS |

---

## Azure Bastion

### What It Means

Azure Bastion provides **RDP/SSH access to VMs without public IP**

```text
Traditional approach:
├── VM has public IP
├── Internet accessible
├── Risk: Exposed to attacks
└── Port 22/3389 open

Azure Bastion approach:
├── VM no public IP
├── Accessed via Bastion (secure gateway)
├── Portal, CLI, or native client
└── No direct internet exposure
```

### How It Works

```text
1. Deploy Azure Bastion in VNet
2. VM without public IP in same VNet
3. User portal → Bastion → VM RDP/SSH
4. Bastion handles encryption, authentication
5. VM logs show: User accessed at time X from Bastion
```

**Access methods:**

```text
Azure Portal:
    VM → Connect → Bastion
    ↓
Browser RDP/SSH opens

Azure CLI:
    az network bastion rdp --name bastion --resource-group rg --target-resource-id <vm-id>
    ↓
Native RDP client

Native SSH:
    ssh user@vm-private-ip (via Bastion tunnel)
```

### Benefits

```text
Security:
├── No public IP exposure
├── Encrypted tunnels
└── Centralized access control

Operations:
├── Access without SSH keys pre-shared
├── Audit trail (who accessed when)
├── MFA support
```

---

## Azure DNS

### What It Means

Azure DNS hosts DNS zones for public domains

```text
Domain: contoso.com
    ↓
Create DNS Zone in Azure
    ↓
Add records:
├── A: www.contoso.com → 40.123.45.67
├── MX: mail.contoso.com → mail-server
├── CNAME: api.contoso.com → www.contoso.com
└── TXT: verification records
```

### DNS Records

| Type | Purpose | Example |
|---|---|---|
| **A** | IPv4 address | www → 40.123.45.67 |
| **AAAA** | IPv6 address | www → 2001:db8::1 |
| **CNAME** | Alias | api → www (redirect) |
| **MX** | Mail server | mail priority 10 |
| **TXT** | Text (verification) | "v=spf1..." |
| **NS** | Nameserver | Delegates zone |
| **SOA** | Start of Authority | Zone metadata |

### Name Resolution Process

```text
Client: Looks up www.contoso.com
    ↓
Recursive resolver:
    "I don't know, let me ask root nameserver"
    ↓
Root nameserver:
    "Go ask .com TLD nameserver"
    ↓
TLD nameserver:
    "Go ask Azure DNS nameservers (contoso.com's nameservers)"
    ↓
Azure DNS:
    "www.contoso.com = 40.123.45.67"
    ↓
Result: 40.123.45.67 returned to client
```

---

## Private DNS Zones

### What It Means

Private DNS zones resolve internal names within your VNet

```text
Private DNS Zone: internal.contoso.com
    ↓
Records:
├── db.internal.contoso.com → 10.0.2.5 (private IP)
├── api.internal.contoso.com → 10.0.1.4 (private IP)
└── cache.internal.contoso.com → 10.0.3.8 (private IP)
    ↓
Link to VNet
    ↓
VMs in VNet resolve internal names to private IPs
    ↓
External clients cannot resolve these names
```

### Common Use Cases

**Private Endpoint DNS:**

```text
Storage Account: myaccount
Private Endpoint: Private IP 10.0.3.5
Private DNS Zone: blob.core.windows.net
Record: myaccount.blob.core.windows.net → 10.0.3.5
    ↓
Application resolves: myaccount.blob.core.windows.net → 10.0.3.5
```

**Hybrid on-premises:**

```text
On-premises DC: tries to resolve internal.contoso.com
    ↓
Azure Private DNS Zone: internal.contoso.com
    ↓
Resolution: Via VPN/ExpressRoute
```

---

## Load Balancer

### What It Means

Layer 4 (TCP/UDP) load balancer distributes traffic across backend pools

```text
Load Balancer (public IP: 40.123.45.67)
    ↓
Inbound rule:
├── Listen on: Port 80 (HTTP)
├── Protocol: TCP
└── Forward to: Backend Pool
    ↓
Backend Pool:
├── VM1: 10.0.1.4 (port 80)
├── VM2: 10.0.1.5 (port 80)
└── VM3: 10.0.1.6 (port 80)
    ↓
Traffic distribution:
├── Client 1 → 40.123.45.67:80 → VM1
├── Client 2 → 40.123.45.67:80 → VM2
└── Client 3 → 40.123.45.67:80 → VM3
```

### Internal vs. Public

| Type | IP | Access | Use Case |
|---|---|---|---|
| **Public** | Public IP (internet) | External traffic | Web servers |
| **Internal** | Private IP (VNet only) | Internal traffic | Database balancing |

### Health Probes

```text
Health Probe:
├── Protocol: TCP, HTTP, HTTPS
├── Interval: 15 seconds
├── Threshold: 2 failed checks = unhealthy
    ↓
Example: HTTP probe on port 80, path /health
    ↓
If VM returns 200 OK: Healthy (receives traffic)
If VM returns error: Unhealthy (traffic stopped)
```

### Outbound Rules

```text
Outbound rule:
├── Allocates outbound IP address
├── Translates outbound traffic
└── Example: VM private IP → Load Balancer public IP
    ↓
Result: Outbound traffic appears to come from LB public IP
```

---

## Application Gateway

### What It Means

Layer 7 (HTTP/HTTPS) application load balancer with advanced routing

```text
Application Gateway (public IP: 40.123.45.67)
    ↓
HTTP request: GET /api/users
    ↓
Routing rule:
├── If path = /api/*: Route to API backend pool
├── If path = /images/*: Route to image backend pool
└── Default: Route to web backend pool
    ↓
Result: Path-based routing to different backends
```

### Features

- **Path-based routing** — /api → API backend, /images → image backend
- **Host-based routing** — api.contoso.com → API backend, www.contoso.com → web
- **URL rewriting** — Modify request before forwarding
- **SSL/TLS termination** — Decrypt HTTPS, forward HTTP to backends
- **WAF (Web Application Firewall)** — Protect against SQL injection, XSS, etc.
- **Session affinity** — Sticky sessions (same client → same backend)

### Comparison: Load Balancer vs. Application Gateway

| Aspect | Load Balancer | Application Gateway |
|---|---|---|
| **OSI Layer** | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| **Protocol** | TCP, UDP | HTTP, HTTPS |
| **Routing** | Basic (port-based) | Advanced (path, host, header) |
| **WAF** | No | Yes (optional) |
| **Use case** | Generic protocols | Web apps, microservices |
| **Cost** | Lower | Higher |

---

## Network Troubleshooting Flow

When network issues occur, follow this systematic approach:

```text
1. DNS RESOLUTION
   Can client resolve hostname to IP?
   → Check: Azure DNS, Private DNS Zone
   → Tool: nslookup, dig, host command
   
2. ROUTING
   Can traffic reach destination network?
   → Check: Route table, system routes, UDRs
   → Tool: tracert, traceroute
   
3. NSG FILTERING
   Is traffic allowed by Network Security Groups?
   → Check: NSG inbound rules, effective rules
   → Tool: Portal effective rules view
   
4. FIREWALL / NVA
   Is appliance blocking traffic?
   → Check: Firewall rules, NVA configuration
   → Tool: Firewall logs
   
5. DESTINATION FIREWALL
   Is destination firewall blocking?
   → Check: Destination OS firewall, security software
   → Tool: Destination logs, firewall status
   
6. PRIVATE ENDPOINT
   If accessing via Private Endpoint:
   → Check: Private DNS resolution, endpoint status
   → Tool: dig, nslookup for private DNS
   
7. SERVICE CONFIGURATION
   Is destination service actually listening?
   → Check: Service status, listening ports
   → Tool: netstat -an, netstat -tuln (listening sockets)
```

---

# PART II — MONITORING AND OBSERVABILITY

## Mental Model

```text
Observable signals
    ↓
Metrics (numeric values)
├── CPU percentage
├── Memory usage
├── Request count
└── Response time
    
Logs (detailed events)
├── Application logs
├── Diagnostic logs
├── Audit logs
└── Security logs
    
Alerts (notifications)
├── Threshold-based
├── Action groups
└── Notifications (email, SMS, webhook)
    
Insights (curated views)
├── VM Insights
├── Container Insights
└── SQL Insights
    ↓
Health Assessment
├── Is service healthy?
├── What's the bottleneck?
└── What action to take?
```

---

## Azure Monitor

### What It Means

Azure Monitor is the umbrella platform for all observability in Azure

```text
Azure Monitor
├── Metrics (numeric time-series)
├── Logs (detailed queryable records)
├── Alerts (condition-based notifications)
├── Insights (managed dashboards)
├── Application Insights (APM for applications)
└── Log Analytics Workspace (KQL queries)
```

### Metrics vs. Logs

| Aspect | Metric | Log |
|---|---|---|
| **Data type** | Numeric value | Event record |
| **Example** | CPU: 75%, Memory: 4096MB | User logged in at 14:30:00, IP 192.168.1.5 |
| **Storage** | Time-series database | Log Analytics |
| **Query** | Chart, alert threshold | KQL (Kusto Query Language) |
| **Retention** | 30 days default | Configurable (30 days to 2 years) |
| **Cost** | Per metric | Per GB ingested |
| **Use case** | Real-time trends | Detailed investigation |

### Metrics

**Built-in metrics** (no configuration):

```text
VM metrics:
├── CPU percentage
├── Network in/out
├── Disk read/write
└── Available memory

Storage metrics:
├── Used capacity
├── Request count
├── Success rate
└── Average latency
```

**Custom metrics:**

```text
Application can send custom metric:
├── "OrdersProcessed": 1000
├── "InventoryLevel": 500
└── "CheckoutAbandonmentRate": 15%
```

### Logs

**Log Analytics Workspace:**

```text
Central repository for all logs
    ↓
Tables:
├── AzureActivity (ARM operations)
├── AzureDiagnostics (service diagnostics)
├── AppTraces (application logs)
├── SecurityEvent (Windows security logs)
└── Syslog (Linux system logs)
```

**KQL (Kusto Query Language):**

```kusto
AzureActivity
| where TimeGenerated > ago(1d)
| where OperationName contains "Create"
| summarize count() by ResourceType
```

### Alerts

**Alert rule structure:**

```text
Signal (what to monitor)
    ↓
├── Metric: CPU > 80%
├── Log: Error count > 100
└── Custom: Application metric

Condition (when to trigger)
    ↓
├── Threshold: Greater than, less than, equals
├── Window: Evaluate over 5 minutes
└── Frequency: Check every 1 minute

Action (what to do)
    ↓
├── Send email
├── SMS notification
├── Call webhook
└── Trigger runbook
```

**Alert Processing Rules:**

```text
Alert fires
    ↓
Processing rule 1: Is alert suppressed? (check)
    ↓
No: Send notifications
    ↓
Processing rule 2: Modify alert details (optional)
    ↓
Send to Action Group
```

### Action Group

**What it is:** Reusable collection of notification channels

```text
Action Group: IT-Operations
    ├── Email: ops@contoso.com
    ├── SMS: +1-555-0123
    ├── Webhook: https://webhook.contoso.com/alert
    └── Runbook: Restart-Service

Alert Rule 1 → Alert High CPU → Action Group
Alert Rule 2 → Alert Failed Backup → Action Group
Alert Rule 3 → Alert Storage Full → Action Group
    ↓
Result: All notifications sent to same channels
```

### Log Analytics Workspace

**What it is:** Central repository for log data

```text
Log Analytics Workspace: central-logs
    ├── Diagnostic settings → Send logs to workspace
    ├── Data sources → Collect metrics from agents
    ├── KQL queries → Investigate data
    └── Alerts → Trigger based on queries
```

**Agents:**

```text
Log Analytics Agent (OMS agent):
    ├── Send logs to Log Analytics
    ├── Configure data collection
    └── Push metrics

Application Insights Agent:
    ├── Monitor application performance
    ├── Exception tracking
    └── Request tracing
```

---

## Azure Monitor Insights

### VM Insights

```text
Enable on VM
    ↓
Collects:
├── CPU, Memory, Network, Disk metrics
├── Running processes
├── Network connections (TCP/UDP)
├── Dependency map (which VMs talk to each other)
└── Performance data
    ↓
Dashboard: Multi-VM view
    ├── Which VMs are slow?
    ├── Dependency visualization
    └── Performance comparison
```

### Container Insights

```text
Enable on AKS cluster
    ↓
Collects:
├── Pod performance
├── Container resource usage
├── Kubernetes events
├── Node health
└── Cluster-level metrics
    ↓
Dashboard: Cluster health
    ├── Pod status
    ├── Node status
    └── Performance trends
```

### Application Insights

```text
Enable on App Service / VM application
    ↓
Collects:
├── Request tracking (incoming requests)
├── Exception tracking (errors)
├── Performance counters (CPU, memory)
├── Dependency tracking (calls to DB, external services)
└── Custom events (business metrics)
    ↓
Dashboard: Application health
    ├── Request rate & response time
    ├── Error rate
    ├── Dependency performance
    └── Availability tests
```

---

## Network Watcher

### What It Means

Diagnostics tool for network troubleshooting

```text
Network Watcher
├── Connection Monitor: Can A reach B?
├── IP Flow Verify: Why is traffic blocked?
├── NSG Flow Logs: See all traffic
├── Packet Capture: Detailed packet analysis
└── VPN Troubleshoot: VPN connection issues
```

### Connection Monitor

```text
Test: Can VM-A reach VM-B on port 443?
    ↓
Connection Monitor
├── Source: VM-A (10.0.1.4)
├── Destination: VM-B (10.0.2.5)
├── Port: 443
└── Protocol: TCP
    ↓
Result:
├── Status: Reachable or unreachable
├── Latency: 5ms average
├── Packet loss: 0%
```

---

# PART III — BACKUP AND RECOVERY

## Big Picture

An Azure administrator is responsible for:

- **How to recover from deletion** — Data protection
- **How to recover from regional failure** — Disaster recovery
- **Meeting compliance** — Retention requirements
- **Meeting RTO/RPO** — Recovery targets

---

## Mental Model

```text
Failure Type
    ↓
Accidental Deletion
├── Soft Delete (quick recovery)
├── Backup (point-in-time restore)
└── Versioning (for blobs/files)
    ↓
Regional Failure
├── Redundancy (GRS/GZRS/RA-GRS)
├── Site Recovery (failover)
└── Backup copy in secondary region
    ↓
Data Corruption
├── Versioning
├── Snapshots
└── Backup from clean point-in-time
    ↓
Compliance
├── Long-term retention (years)
├── Backup policies (schedule + retention)
└── Immutable backups (cannot delete)
```

---

## Azure Backup

### What It Means

Backup service for point-in-time restore

```text
Create backup
    ↓
Stored in Recovery Services Vault
    ↓
Retention policy: Keep for X days/weeks/months/years
    ↓
Need to restore: Get backup from specific date/time
```

### Recovery Services Vault vs. Backup Vault

| Aspect | Recovery Services Vault | Backup Vault |
|---|---|---|
| **Purpose** | Traditional backup + Site Recovery | Block-level backup |
| **Backup types** | VM, File, SQL, MARS agent | Disk, Blob, Database |
| **Supports** | Longer retention | Faster backup/restore |
| **Site Recovery** | Yes | No |
| **Geo-redundancy** | GRS available | GRS available |

**Typical choice:**

```text
VM backup → Recovery Services Vault
Disk backup → Backup Vault
SQL Server backup → Recovery Services Vault
```

### Backup Policy

**Schedule:**

```text
Daily backup: Every day at 2:00 AM
Weekly backup: Sunday at 3:00 AM
Monthly backup: First Sunday at 4:00 AM
Yearly backup: January 1st at 5:00 AM
```

**Retention:**

```text
Daily backups: Keep 7 days
Weekly backups: Keep 4 weeks
Monthly backups: Keep 12 months
Yearly backups: Keep 5 years
```

### Backup and Restore Flow

```text
1. Enable backup on VM
2. First backup: Full backup (entire disk)
3. Subsequent: Incremental (only changes)
4. Stored in Recovery Services Vault
5. If need restore: Choose point-in-time
6. Download backup: Restore disk
7. Create VM from disk: Resume operations
```

---

## Azure Site Recovery

### What It Means

Replication service for regional disaster recovery

```text
Primary Region: East US
├── Application running
├── Site Recovery enabled
    
Secondary Region: West US
├── Replica (standby copy)
├── Stays in sync
    
If primary fails:
├── Failover to secondary
├── Application runs in West US
├── Traffic redirected
```

### Key Concepts

**Replication:**

```text
Change in primary VM
    ↓
Automatically replicated to secondary
    ↓
RPO (Recovery Point Objective): How much data can be lost?
├── ≤ 5 minutes (standard replication)
├── ≤ 30 seconds (Premium SSD)
```

**Failover:**

```text
Test Failover (no production impact):
├── Create copy of secondary
├── Test it
├── Delete copy
├── Primary still running

Planned Failover (scheduled):
├── Shutdown primary gracefully
├── Failover to secondary
├── Wait for completion
├── No data loss (RPO = 0)

Unplanned Failover (emergency):
├── Primary failed
├── Immediate failover
├── Possible data loss (RPO minutes)
```

**Failback:**

```text
Primary recovered
    ↓
Failback: Secondary → Primary
    ↓
Original primary becomes primary again
    ↓
Secondary continues as backup
```

### RTO and RPO

**RPO (Recovery Point Objective):**

```text
How much data loss acceptable?

RPO = 5 minutes
    ↓
Meaning: Acceptable to lose 5 minutes of data
    ↓
Scenario: Primary fails at 14:00
         Last replicated backup: 13:55
         Data loss: 14:00 - 13:55 = 5 minutes
```

**RTO (Recovery Time Objective):**

```text
How long recovery acceptable?

RTO = 2 hours
    ↓
Meaning: Must be back online within 2 hours
    ↓
Scenario: Primary fails at 14:00
         Failover completes by 16:00
         Recovery time: 2 hours
```

**Real scenario:**

```text
Business requirement:
├── Cannot lose more than 15 minutes of data (RPO ≤ 15 min)
├── Must be back within 4 hours (RTO ≤ 4 hours)

Solution:
├── Site Recovery: Continuous replication (RPO ≤ 5 min)
├── Automated failover: Achieves RTO within 30 min
└── Exceeds requirements (backup already included)
```

### Backup vs. Site Recovery

| Aspect | Backup | Site Recovery |
|---|---|---|
| **Protection** | Deleted data, corruption | Regional failure |
| **Scenario** | Restore from point-in-time | Fail over region |
| **RPO** | Variable (daily, hourly) | Near-continuous (minutes) |
| **RTO** | Hours (restore, start app) | Minutes (failover) |
| **Cost** | Lower (periodic) | Higher (continuous replication) |
| **Scope** | Individual resource | Entire region |
| **Testing** | Manual restore | Test failover |

---

## RPO vs. RTO Summary

**RPO = Data Loss Tolerance:**

```text
RPO = 1 hour
    ↓
"We can accept losing up to 1 hour of data"
    ↓
Backup frequency: Hourly
```

**RTO = Recovery Time Tolerance:**

```text
RTO = 30 minutes
    ↓
"We must be back online within 30 minutes"
    ↓
Failover automation: Fast switchover
```

**Business Translation:**

```text
"RPO 1 hour, RTO 30 minutes"
    ↓
Means:
├── Backup every hour
├── Latest data: 1 hour old maximum
├── Failover: Completes in 30 minutes
└── Total: Down for 30 minutes, lose 1 hour of data max
```

---

## Domain Summary

| Concept | Purpose | Trigger | Nearest Confusion | Key Exam Rule |
|---|---|---|---|---|
| **VNet** | Private network | Isolate resources | Subnet | Region-bound |
| **NSG** | Filter traffic | Can traffic pass? | Route Table | Inbound/outbound rules |
| **Route Table** | Direct traffic | Where to send | NSG | Next hop specification |
| **Service Endpoint** | Access PaaS via public | VNet to service | Private Endpoint | Service's public endpoint |
| **Private Endpoint** | Access PaaS via private | Strict isolation | Service Endpoint | Private IP in VNet |
| **Bastion** | Secure RDP/SSH | No public IP | Jumphost VM | Managed gateway |
| **Load Balancer** | Layer 4 distribution | TCP/UDP traffic | Application Gateway | Protocol-agnostic |
| **Application Gateway** | Layer 7 distribution | HTTP/HTTPS routing | Load Balancer | Path/host-based |
| **Metric** | Numeric value | Real-time trends | Log | Time-series |
| **Log** | Event record | Investigation | Metric | Detailed context |
| **Alert** | Notification trigger | Condition detected | Alert Processing | Signal + Condition + Action |
| **Backup** | Point-in-time restore | Accidental deletion | Site Recovery | Recovery from past |
| **Site Recovery** | Regional failover | Disaster recovery | Backup | Continuous replication |
| **RPO** | Acceptable data loss | How much? | RTO | In minutes/hours |
| **RTO** | Acceptable recovery time | How long? | RPO | In minutes/hours |

---

## 30-Second Recall

**Networking:**
- VNet = private network container
- NSG = filter (allow/deny)
- Route = direct traffic to next hop
- Service Endpoint = access PaaS via public endpoint
- Private Endpoint = private IP in VNet
- Bastion = secure access without public IP
- Load Balancer = Layer 4
- Application Gateway = Layer 7

**Monitoring:**
- Metric = numeric time-series
- Log = event record
- Alert = notification on condition
- Action Group = reusable notification channel
- Application Insights = APM for apps
- VM Insights = performance dashboard
- Network Watcher = troubleshooting tools

**Recovery:**
- Backup = point-in-time restore
- Site Recovery = regional failover
- RPO = acceptable data loss
- RTO = acceptable recovery time
- Recovery Services Vault = long-term retention
- Soft Delete = recover deleted quickly

**Key Distinctions:**
- NSG vs. Route = filter vs. direction
- Service Endpoint vs. Private Endpoint = public vs. private IP
- Metric vs. Log = number vs. event
- Backup vs. Site Recovery = point-in-time vs. failover
- RPO vs. RTO = data loss vs. recovery time

---

## Official Sources

- [Azure Virtual Networks](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [Network Security Groups](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Route Tables](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview)
- [Private Endpoints](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
- [Service Endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview)
- [Azure Bastion](https://learn.microsoft.com/en-us/azure/bastion/bastion-overview)
- [Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
- [Log Analytics](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
- [Azure Backup](https://learn.microsoft.com/en-us/azure/backup/backup-overview)
- [Azure Site Recovery](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-overview)
