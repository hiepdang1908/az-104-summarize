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
- **Overlap:** Address spaces can overlap in isolated VNets, but peering, VPN, and other connected-network designs generally require non-overlapping ranges. Plan CIDR ranges before connecting networks.

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
├── 10.0.1.3: Reserved by Azure
├── 10.0.1.4: First available for resources
│   ...
└── 10.0.1.255: Broadcast address (reserved)

Usable IPs: 10.0.1.4 to 10.0.1.254 (251 IPs). Azure reserves the first four and last address in every subnet.
```

---

## Public IP Addresses

**What it is:** Public-facing IP for inbound/outbound traffic

**Static vs. Dynamic:**

| Type | Cost | Use Case |
|---|---|---|
| **Dynamic** | Pricing depends on SKU, region, and association state | Temporary, okay if changes |
| **Static** | Pricing depends on SKU, region, and association state | Stable allowlists and DNS mappings |

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
Regional peering; check current data-transfer pricing
```

**Global Peering:**

```text
VNet-A (East US) ← Peering → VNet-B (West US)
    ↓
Data-transfer charges depend on the peering type, traffic direction, and current regional pricing
```

### Peering Properties

- **Allow forwarded traffic** — Forward traffic from outside the peering relationship
- **Allow gateway transit** — Use VPN gateway for on-premises access
- **Use remote gateway** — Use peer's VPN gateway

**Gateway transit dependency:** On the VNet that owns the gateway, enable gateway transit. On the peered VNet that consumes it, enable use remote gateways. A VNet can use only one remote gateway, and it cannot use its own gateway at the same time. Peering remains non-transitive: gateway transit provides a path to the gateway, not automatic transitive peering between every VNet.

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
First matching rule wins; lower priority numbers are evaluated first
    ↓
Action: Allow or Deny
```

**Stateful behavior:** An NSG evaluates a new flow against its five-tuple (source, source port, destination, destination port, and protocol). When it allows the initiating flow, response traffic is automatically allowed for that established flow; a reverse rule is not required solely for the response. Rules affect new connections, not an already established connection.

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
Packet must be allowed by both the subnet and NIC NSG. Evaluate the matching rules at each association; a deny at either layer blocks a new flow.
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
    └── If no: Use the best matching system route (often the default route)
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
Result: The route determines the path and the NSG independently determines whether the flow is allowed. Do not use an NSG to choose a next hop or a UDR to filter a port.
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
- **Pricing** — No separate service-endpoint charge; normal service and data-transfer charges can still apply

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
Private DNS Zone: privatelink.blob.core.windows.net
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
- **Cost** — Private Link pricing varies by region and data processed; check current pricing
- **Complex** — Requires Private DNS setup
- **Strict isolation** — Complete private network isolation

### Service Endpoint vs. Private Endpoint

| Aspect | Service Endpoint | Private Endpoint |
|---|---|---|
| **IP type** | Service's public IP | Private IP in VNet |
| **DNS** | resolves to public IP | Requires Private DNS Zone |
| **Network** | Public endpoint, private routing | Complete private networking |
| **Cost** | No separate service-endpoint charge | Private Link pricing varies by region and data processed |
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

**VNet links:** Link a private DNS zone to every VNet that must resolve its records. Enable auto-registration only for a VNet whose VM records should be automatically created in that zone. For Private Link, use the service-specific `privatelink` private DNS zone name; private endpoint creation can create and manage the matching A record. On-premises clients require a DNS forwarding design to resolve that private zone.

## VPN Gateway

Azure VPN Gateway provides encrypted connectivity through a virtual network gateway deployed in the dedicated `GatewaySubnet`. It requires a public IP address for the gateway and, for on-premises connections, a local network gateway that represents the on-premises address prefixes and VPN device public IP.

| Connection | Purpose | Peer |
|---|---|---|
| **Point-to-Site (P2S)** | Individual user/device connection to a VNet | VPN client |
| **Site-to-Site (S2S)** | Persistent branch/on-premises network connection | On-premises VPN device through a local network gateway |
| **VNet-to-VNet** | Encrypted connection between Azure VNets | Another Azure VPN gateway |

Use VNet peering for Azure-to-Azure private connectivity when its constraints fit. Use VPN Gateway when encrypted tunnel connectivity to an on-premises network, remote clients, or a gateway-based VNet connection is required.

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
├── Interval and threshold: Configured for the load-balancing rule
    ↓
Example: HTTP probe on port 80, path /health
    ↓
If VM returns 200 OK: Healthy (receives traffic)
If VM returns error: Unhealthy (traffic stopped)
```

**Troubleshooting rule:** If a load-balanced service is unreachable, verify the frontend IP and rule, backend-pool membership, probe protocol/port/path, NSG rules, guest firewall, and that the application is listening. An unhealthy probe removes a backend from new load-balanced flows; it does not prove that the frontend rule or network path is correct.

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
| **Pricing model** | Load-balancer rules and processed data | Gateway capacity/usage, processed data, and optional WAF features |

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
| **Retention** | Platform-metric and log retention policies differ | Configurable by workspace, table, and retention tier; verify the current policy |
| **Cost** | Depends on metric features and alerting | Depends on ingestion, retention, query, and export choices |
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

**Common tables and prerequisites:** `AzureActivity` contains subscription control-plane events. Resource logs reach a Log Analytics workspace only after the resource's diagnostic settings send them there. `SecurityEvent` requires supported Windows security-event collection through Azure Monitor Agent and a data collection rule. Verify the available tables and columns in the target workspace before reusing a query.

**Basic Query Structure:**

```kusto
TableName
| where Condition (filter)
| project Columns (choose or rename columns)
| summarize Aggregation (count, sum, avg)
| sort by Column desc
```

**Example 1: Find all Create operations in last 24 hours**

```kusto
AzureActivity
| where TimeGenerated > ago(1d)
| where OperationName contains "Create"
| summarize count() by ResourceType
```

**Example 2: Investigate recent failed Azure control-plane operations**

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| where ActivityStatusValue == "Failure"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup, ResourceId, Properties
| order by TimeGenerated desc
```

**Flow-log note:** Virtual network flow logs are stored as JSON flow tuples, not as a universal `AzureDiagnostics` schema. For a network-flow investigation, first configure virtual network flow logs, then parse the flow-log records in the selected storage or analytics destination. A denied tuple has flow state `D`.

**Example 3: Monitor application errors over time**

```kusto
AppTraces
| where SeverityLevel >= 2 // Error or Critical
| summarize ErrorCount = count() by bin(TimeGenerated, 1h), Message
| order by TimeGenerated desc
```

**Example 4: Audit failed login attempts**

```kusto
SecurityEvent
| where EventID == 4625 // Failed sign-in
| summarize FailureCount = count() by TargetAccount, SourceIpAddress
| where FailureCount > 5
| order by FailureCount desc
```

**Example 5: Check ARM operation failures (troubleshooting deployments)**

```kusto
AzureActivity
| where Status == "Failed"
| where TimeGenerated > ago(7d)
| project TimeGenerated, Caller, ResourceType, OperationName, Authorization, ActivityStatus
| sort by TimeGenerated desc
```

**Common KQL operators:**

| Operator | Purpose | Example |
|---|---|---|
| `where` | Filter rows | `where Severity == "Error"` |
| `project` | Choose/rename columns | `project Time=TimeGenerated, Msg=Message` |
| `summarize` | Aggregate data | `summarize count() by Category` |
| `sort` or `order` | Sort results | `sort by TimeGenerated desc` |
| `top` | Get top N rows | `top 10 by Duration` |
| `count()` | Count rows | `summarize TotalCount = count()` |
| `sum()` | Sum column values | `summarize Total = sum(BytesSent)` |
| `avg()` | Average column values | `summarize AvgTime = avg(Duration)` |
| `dcount()` | Distinct count | `summarize UniqueUsers = dcount(UserId)` |
| `bin()` | Group by time | `bin(TimeGenerated, 1h)` (hourly) |

**Admin/troubleshooting KQL quick reference:**

| Use Case | Query |
|---|---|
| What VMs had failed backups? | `AzureDiagnostics \| where Category == "Backup" \| where BackupStatus_s == "Failed"` |
| Which storage accounts accessed? | `AzureActivity \| where ResourceType == "Microsoft.Storage/storageAccounts" \| summarize count() by CallerIpAddress` |
| Most active users? | `SecurityEvent \| summarize LoginCount = count() by Account \| top 10 by LoginCount` |
| Resources deleted last 7 days? | `AzureActivity \| where OperationName contains "Delete" \| where TimeGenerated > ago(7d)` |

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
Processing rule evaluates scope, filters, and schedule
    ↓
Suppress action groups OR add action groups
    ↓
Alert remains visible even when notifications are suppressed
```

| Component | Decides | Use it for |
|---|---|---|
| **Alert rule** | When a signal creates an alert | CPU threshold, failed log query, or activity event |
| **Action group** | What happens after an alert fires | Email, SMS, push, webhook, automation, or integration |
| **Alert processing rule** | Whether to suppress or add action groups to fired alerts | Maintenance windows, after-hours routing, or centralized action-group assignment |

An alert processing rule can apply to resources, resource groups, or a subscription in the same subscription as the rule. Filters narrow the affected alerts and schedules can be one-time or recurring. Use it instead of disabling broadly scoped alert rules during maintenance; it does not change an alert's condition or arbitrary alert details.

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
Azure Monitor Agent (AMA):
    ├── Collect guest data through data collection rules
    ├── Send supported data to Log Analytics or Azure Monitor
    └── Replaces the legacy Log Analytics agent for new designs

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

VM Insights uses Azure Monitor data collection and is intended for VM performance, health, and dependency visibility. Use the Performance view for CPU, memory, disk, and network analysis; use dependency data where it is enabled and supported. Do not confuse VM Insights with Azure Activity Log: Activity Log records control-plane events, while VM Insights analyzes monitored guest and platform performance data.

### Storage and Network Monitoring

| Area | What to review | Main tools |
|---|---|---|
| **Storage** | Availability, capacity, transactions, latency, errors, and throttling | Storage metrics, resource logs through diagnostic settings, and Azure Monitor workbooks |
| **Network** | Reachability, routing, NSG decisions, topology, and flow behavior | Network Watcher, Connection Monitor, effective routes/security rules, and virtual network flow logs |

Metrics are available without routing every platform metric to Log Analytics. Configure diagnostic settings when resource logs must be retained or queried in Log Analytics, Storage, or Event Hubs.

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
├── Virtual network flow logs: Record network flows for new designs
├── Packet Capture: Detailed packet analysis
└── VPN Troubleshoot: VPN connection issues
```

**Flow-log currency:** Network Security Group flow logs are retiring and no longer support new creation. Use virtual network flow logs for new designs. They collect Layer 4 IP-flow records at VNet scope and can show the rule that allowed or denied a flow. Do not enable both types over the same workload unless duplicate logging and cost are intentional.

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
| **Purpose** | Backup and Site Recovery vault for established workloads | Backup vault for supported newer Azure Backup workloads |
| **Typical workloads** | Azure VMs, Azure Files, SQL Server in Azure VM, MARS, and Site Recovery | Azure Blobs, Azure Disks, Azure Database for PostgreSQL, and other supported V2 workloads |
| **Site Recovery** | Yes | No |
| **Selection rule** | Choose when the protected workload or Site Recovery requires it | Choose only when the workload is supported by Backup vault |

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

**Retention decision:** A policy controls when recovery points are created and how long each retention tier is retained. Daily, weekly, monthly, and yearly rules serve different recovery and compliance needs; they do not create the same number of restore points. Longer retention increases protected-storage consumption. Use soft delete and immutable-vault features when supported and required, but do not confuse either with a backup schedule.

### Backup and Restore Flow

```text
1. Select a supported workload and vault.
2. Assign a backup policy and enable protection.
3. Azure creates recovery points according to the workload's backup implementation and policy.
4. When restoring, choose the recovery point and the restore option supported by that workload.
5. Validate the restored data or workload before returning it to service.
```

**Restore options vary by workload:** Azure VM backup can restore the VM, restore disks for manual VM creation, or provide file-level recovery where supported. Other protected workloads expose their own restore choices. Select the recovery point, destination, and overwrite behavior deliberately; restoring is not limited to replacing the original resource.

### Azure Backup Monitoring, Alerts, and Reports

- **Backup jobs:** Use Resiliency, the Recovery Services vault, or the Backup vault to inspect backup and restore jobs, their status, start and end times, and failure details. Confirm successful jobs and a usable recent recovery point; policy assignment alone does not prove recoverability.
- **Backup alerts:** Azure Backup provides Azure Monitor-based built-in alerts for supported security events and backup or restore failures. Azure Monitor alert rules can also evaluate supported metrics or logs for custom conditions.
- **Notifications:** An alert rule defines the condition. An Azure Monitor action group routes the fired alert to email, SMS, ITSM, webhook, Logic Apps, or other supported actions. Alert processing rules can add or suppress action groups for scoped alerts, such as during maintenance.
- **Backup reports:** Backup Reports use Azure Monitor Logs and Azure workbooks. Configure vault diagnostic settings to send the required data to a Log Analytics workspace, then use reports for historical job trends, protected-item inventory, storage usage, policy and backup health, and operational monitoring.

> **Requirement:** Operations must receive an email when backup failures occur. **Think:** Azure Backup monitoring/alerts + an Azure Monitor action group containing the email receiver.

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

**Azure-to-Azure prerequisites:** Replication applies only to supported source and target regions, VM configurations, operating systems, disks, and networking. Azure VMs use managed disks for this scenario. The source VM needs outbound connectivity for Site Recovery and replication traffic, and the target region needs sufficient quota and supported target resources. Site Recovery can create or use target resource groups, virtual networks, and replica disks; validate them before enabling replication.

### Key Concepts

**Replication:**

```text
Change in primary VM
    ↓
Automatically replicated to secondary
    ↓
RPO (Recovery Point Objective): How much data can be lost?
├── Determined by the source, target, workload, replication configuration, and current service capability
├── Validate the achievable RPO rather than assuming a fixed interval
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

**Failover choices:**

| Operation | Purpose | Production impact |
|---|---|---|
| **Test failover** | Validate a recovery point and recovery plan | Uses an isolated test network; production replication continues |
| **Planned failover** | Controlled migration when the source is available | Shuts down the source before failover to minimize data loss |
| **Unplanned failover** | Recover after an outage | Uses the selected available recovery point; data loss can occur within the achieved RPO |
| **Reprotect and fail back** | Return protection and later workload operation to the original region | Requires the workload to be stable in the recovery direction |

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
├── Design Site Recovery replication and the recovery plan to meet the stated targets
├── Test failover to measure the realistic RTO
└── Use backup separately when point-in-time recovery and retention are required
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
