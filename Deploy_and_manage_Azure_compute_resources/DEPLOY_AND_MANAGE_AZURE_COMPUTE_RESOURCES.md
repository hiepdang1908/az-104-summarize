# Deploy and Manage Azure Compute Resources

---

**← [Previous: Storage](../Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md)** | **[Home](../README.md)** | **[Next: Networking →](../Networking_monitoring_and_recovery/NETWORKING_MONITORING_AND_RECOVERY.md)**

---

## Big Picture

An Azure administrator is responsible for:

- **How applications run** (VM, Container, PaaS)
- **Scaling** (up, down, out, in)
- **Availability** (replicas, zones, redundancy)
- **Deployment** (how to standardize deployments)
- **Resource size** (choosing right tier and size)
- **Cost** (expensive resources for short-lived workloads)

This domain covers everything from virtual machines to containers to fully managed application platforms.

---

## Mental Model

```text
Application Requirement
    ↓
Compute Choice
├── Infrastructure as Code (ARM/Bicep)
├── Virtual Machine (IaaS - full control)
├── Container (ACI - simple) or Container Apps (managed)
└── App Service (PaaS - fully managed)
    ↓
Sizing Decision
├── Small / Medium / Large / Extra Large
└── Depends on: CPU, memory, network, storage needs
    ↓
Availability Strategy
├── Single instance (no redundancy)
├── Availability Set (logical, one datacenter)
├── Availability Zone (physical, separate datacenters)
├── VM Scale Set (auto-scaling)
└── Multi-region (extreme resilience)
    ↓
Scaling Pattern
├── Manual (change size manually)
├── Scheduled (different sizes on schedule)
└── Metrics-based (auto-scale based on CPU/memory)
    ↓
Network Attachment
├── Subnet assignment
├── Public IP (optional)
├── NSG rules
└── Load Balancer / Application Gateway
    ↓
Deployment
├── ARM Template (declarative)
├── Bicep (simplified ARM)
└── Portal / CLI (one-time)
```

---

## Infrastructure as Code (ARM Templates and Bicep)

### ARM Templates

**What it is:** JSON file declaring Azure resources

**Structure:**

```json
{
  "$schema": "...",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myvm"
    }
  },
  "variables": {
    "storageAccountName": "[concat('storage', uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[parameters('vmName')]",
      "properties": {
        "hardwareProfile": { "vmSize": "Standard_D2s_v3" }
      }
    }
  ],
  "outputs": {
    "vmId": { "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]" }
  }
}
```

**Key sections:**

- **Parameters** — Input variables
- **Variables** — Reusable values computed from parameters
- **Resources** — Azure objects to deploy
- **Outputs** — Values returned after deployment
- **Dependencies** — Declare resource order

### Bicep

**What it is:** Simplified language that compiles to ARM template

**Bicep syntax:**

```bicep
param vmName string = 'myvm'
param vmSize string = 'Standard_D2s_v3'
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2024-01-01' = {
  name: 'storage${uniqueString(resourceGroup().id)}'
  location: location
  kind: 'StorageV2'
  sku: {
    name: 'Standard_LRS'
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2024-03-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
  }
}

output vmId string = vm.id
```

**Benefits over ARM:**

- Simpler syntax (less JSON boilerplate)
- Functions and loops built-in
- Easier to read and maintain

### ARM Template Export

**How it works:**

```text
1. Deploy resources manually via portal
2. Export template from Resource Group
3. Get JSON template representing deployed resources
4. Modify template for reuse
5. Deploy via template next time
```

**Use case:**

```text
"I built everything manually; now I want to recreate it"
    ↓
Export template
    ↓
Parameterize hardcoded values
    ↓
Redeploy in another environment
```

### Deployment

**PowerShell:**

```powershell
New-AzResourceGroupDeployment `
  -ResourceGroupName myResourceGroup `
  -TemplateFile template.json `
  -TemplateParameterFile parameters.json
```

**Azure CLI:**

```bash
az deployment group create \
  --resource-group myResourceGroup \
  --template-file template.json \
  --parameters parameters.json
```

---

## Virtual Machines

### VM Sizing

**Size categories:**

| Category | CPU | RAM | Network | Use Case |
|---|---|---|---|---|
| **B-series** | 1-4 | 0.5-16 GB | Low | Dev/test, light workloads |
| **D-series** | 2-64 | 4-256 GB | Medium | General purpose |
| **E-series** | 2-128 | 16-3500 GB | High | Memory-intensive |
| **F-series** | 2-72 | 4-192 GB | Medium | CPU-intensive |
| **M-series** | 2-416 | 32-11400 GB | High | Enterprise, SAP |
| **H-series** | 8-120 | 16-240 GB | High | HPC |

**Naming example:** `Standard_D2s_v3`

```text
Standard = category (Standard vs Premium)
D = series (D-series)
2 = number of vCPUs
s = high I/O optimization
v3 = generation (v1, v2, v3)
```

### VM Disks

**OS Disk:**

```text
├── Contains operating system
├── Max size: 2048 GiB (limited by OS)
├── Cannot be shared with other VMs
└── Must be managed disk in production
```

**Data Disks:**

```text
├── Additional storage
├── Can have multiple (up to 64)
├── Can be shared between VMs (premium only)
├── Sizes: 1 GiB - 32 TiB
└── Can attach/detach while running
```

**Temporary Disk:**

```text
├── Ephemeral storage on host hardware
├── Not persisted (lost on stop/deallocate)
├── Fast access, no replication cost
├── Don't rely for persistent data
└── Use for cache, temp files only
```

### Managed Disks

**What it is:** Disk storage managed by Azure

**Advantages:**

- Azure handles replication/backup
- Simpler RBAC model
- Better availability (regional/zone redundancy)
- Can be encrypted at rest

**Disk types:**

| Type | Performance | Cost | Use Case |
|---|---|---|---|
| **Standard HDD** | Low IOPS | Cheap | Dev/test, non-critical |
| **Standard SSD** | Medium IOPS | Medium | General |
| **Premium SSD** | High IOPS | Expensive | Database, high-traffic |
| **Ultra Disk** | Extreme IOPS | Very expensive | High-performance database |

**Shared disks:**

```text
Multiple VMs can attach same premium/ultra disk
    ↓
Requires: Cluster-aware software (SQL Server, Oracle)
    ↓
Use case: Clustered databases requiring shared storage
```

---

## Virtual Machine Availability

### Single VM (No Redundancy)

**What it means:**

```text
1 VM
    ↓
If VM fails: Application down
If host maintenance: Application down for ~1 hour
```

**Cost:** Lowest

**SLA:** 99.5% (99.5% uptime guaranteed by Microsoft)

**When to use:** Non-critical workloads, dev/test

### Availability Set

**What it does:** Logical grouping of VMs for redundancy within one datacenter

**Key concepts:**

```text
Availability Set: MySet
├── Fault Domain 2
│   ├── VM1 (in Fault Domain 2)
│   └── VM2 (in Fault Domain 2)
├── Fault Domain 1
│   ├── VM3 (in Fault Domain 1)
│   └── VM4 (in Fault Domain 1)
└── Update Domain 3
    ├── VM1 (Update Domain 3)
    ├── VM2 (Update Domain 3)
    └── VM5 (Update Domain 3)
```

**Fault Domain:**

```text
Protects against: Hardware failure (power, network, storage)
    ↓
VMs in different fault domains on different hardware
    ↓
If one hardware rack fails: Other VMs still running
```

**Update Domain:**

```text
Protects against: Azure maintenance (OS updates, hypervisor)
    ↓
Azure updates one update domain at a time
    ↓
VMs in other update domains stay running during maintenance
```

**Important:** All VMs in set must be same size (for balance)

**SLA:** 99.95% (with 2+ VMs)

**Cost:** Only pay for VMs (availability set itself is free)

**When to use:** On-premises-like setup, same datacenter acceptable

### Availability Zone

**What it does:** Spread VMs across physically separate datacenters

```text
Region: East US
├── Zone 1: DC1 (Physical datacenter 1)
│   └── VM1
├── Zone 2: DC2 (Physical datacenter 2)
│   └── VM2
└── Zone 3: DC3 (Physical datacenter 3)
    └── VM3
```

**Resilience:**

```text
If entire Zone 1 datacenter fails (earthquake, power outage):
├── VM1 down
├── VM2 running (different physical location)
└── VM3 running (different physical location)
```

**Key difference from Availability Set:**

```text
Availability Set: Logical grouping, same datacenter
Availability Zone: Physical separation, different datacenters
```

**SLA:** 99.99% (with 3+ VMs across zones)

**Cost:** Potential outbound data transfer costs between zones

**When to use:** Mission-critical workloads, region failure protection

### VM Scale Set

**What it is:** Auto-scaling group of identical VMs

```text
Scale Set: WebServers
├── VM Instance 1
├── VM Instance 2
├── VM Instance 3
└── Auto-scaling rules
    ├── If CPU > 80%: Add more instances
    └── If CPU < 20%: Remove instances
```

**Key features:**

- **Identical VMs** — All instances are clones
- **Auto-scaling** — Add/remove instances based on metrics
- **Load balancing** — Built-in load balancer for distribution
- **Zone redundancy** — Can spread across zones

**Scaling rules:**

```text
Rule 1: If average CPU > 75% for 5 minutes
        → Add 1 instance (max 10 total)

Rule 2: If average CPU < 25% for 5 minutes
        → Remove 1 instance (min 2 total)
```

**Use case:**

```text
Web application with variable traffic
├── Morning: 2 instances (low traffic)
├── Midday: 5 instances (peak traffic)
└── Evening: 2 instances (low traffic)
```

**SLA:** 99.95% (with multiple zones) or 99.99% (with 3+ zones)

### Availability Decision Tree

```
Can tolerate downtime during host maintenance?
    ↓ NO
    Must use: Availability Set or Zone
    
Can tolerate Zone 1 complete failure?
    ↓ NO
    Must use: Availability Zone (spread across multiple zones)
    
Load changes frequently?
    ↓ YES
    Must use: VM Scale Set (auto-scale capability)
    ↓ NO
    Use: Availability Set or Zone
```

---

## Containers

### Azure Container Registry (ACR)

**What it is:** Private Docker image repository

```text
Push container image to ACR
    ↓
Tag: myregistry.azurecr.io/myapp:latest
    ↓
Pull from ACI / App Service / AKS
```

**Components:**

- **Registry** — Stores images
- **Repository** — Collection within registry
- **Image** — Container snapshot
- **Tag** — Version label

### Azure Container Instances (ACI)

**What it is:** Run container without managing infrastructure

```text
Push image to ACR
    ↓
Deploy to ACI
    ↓
Container runs (serverless, no VM management)
```

**Characteristics:**

- **Serverless** — No VM to manage
- **Instant scaling** — Fast startup
- **Isolated** — Each container in separate environment
- **Per-second billing** — Minimal cost

**Use case:**

```text
Quick script execution
Dev/test containers
One-off batch jobs
Lightweight microservices
```

### Azure Container Apps

**What it is:** Managed container application platform

```text
Push image
    ↓
Deploy to Container Apps
    ↓
Platform handles: Scaling, networking, load balancing, monitoring
```

**Features:**

- **Auto-scaling** — Scale to zero or up to thousands
- **Environment** — Managed compute with multiple apps
- **Revisions** — Versioning and traffic splitting
- **Dapr** — Distributed application runtime (service-to-service)
- **Container workload profiles** — Choose spot or dedicated

**Comparison:**

| Feature | ACI | Container Apps | AKS |
|---|---|---|---|
| **Management** | Minimal | Managed platform | Manual |
| **Scaling** | Manual | Auto | Manual or custom |
| **Networking** | Per-container | Environment-level | Cluster-level |
| **Cost** | Per-second | Reserved CPU hours | Compute + storage |

**When to use:**

```text
ACI: One-off jobs, simple containers
Container Apps: Microservices, auto-scaling needed
AKS: Complex orchestration, large scale
```

---

## App Service

### App Service Plan

**What it is:** Compute resource where apps run

**SKUs:**

| SKU | Cost | Features | Scale |
|---|---|---|---|
| **Free** | $0 | Shared resources, 1 GB RAM | 1 instance |
| **Shared** | $ | Shared resources, 1 GB RAM | 1 instance |
| **Basic** | $$ | Dedicated, 1.75 GB - 7 GB RAM | Up to 3 instances |
| **Standard** | $$$ | Dedicated, slots, scaling | Up to 10 instances |
| **Premium** | $$$$ | Dedicated, App Service Environment | Up to 20 instances |

**Key concept:**

```text
Plan: Standard_P1V2
    ├── Cost: $145/month
    ├── Instances available: 1-20
    ├── Each instance: 1 vCPU, 3.5 GB RAM
    │
    Apps running on this plan:
    ├── App 1 (web API)
    ├── App 2 (scheduling job)
    └── App 3 (worker function)
```

**Scaling:**

```text
Manual scale: Change number of instances
    ├── 1 instance
    ├── 3 instances
    └── 5 instances

Auto scale: Based on metrics
    ├── CPU > 80%: Add instance
    └── CPU < 20%: Remove instance
```

### App Service

**What it is:** Web application hosting (PaaS)

**Supports:**

- ASP.NET, Node.js, Python, Java, Ruby, PHP
- Windows or Linux container
- Built-in deployment (Git, GitHub, Azure DevOps)

**Features:**

- **Custom domain** — myapp.contoso.com
- **TLS/SSL** — Automatic or bring-your-own certificate
- **Authentication** — Built-in Entra ID
- **Backup** — Automated backups
- **Deployment slots** — Blue-green deployments
- **VNet Integration** — Outbound access to VNet
- **Private Endpoint** — Inbound private access

### App Service Networking

**VNet Integration:**

```text
App Service
    ↓ (outbound connection)
VNet Subnet
    ↓
Can reach resources in VNet
├── Storage (private endpoint)
├── Database (private endpoint)
└── On-premises (via VPN)
```

**Result:** App can access private resources

**Private Endpoint:**

```text
Private Endpoint in VNet
    ↓ (private IP)
App Service
    ↓
Clients access app via private IP only
    ↓
No public internet access
```

**Key distinction:**

```text
VNet Integration
├── Outbound from app
├── App can reach private resources

Private Endpoint
├── Inbound to app
├── Clients access app privately
```

### Deployment Slots

**What it is:** Staging version of app for testing before production

```text
App Service: contoso-app

Slots:
├── Production
│   ├── URL: contoso-app.azurewebsites.net
│   ├── Traffic: 100%
│   └── Running
├── Staging
│   ├── URL: contoso-app-staging.azurewebsites.net
│   ├── Traffic: 0%
│   └── For testing
```

**Blue-Green Deployment:**

```text
1. Deploy new code to Staging slot
2. Test in Staging (full isolation)
3. Once validated, swap Staging ← → Production
4. New code now live (instant swap)
5. Old code rolled back in Staging if needed
```

**Cost:**

```text
Standard tier: Slots included
Basic tier: No slots
```

---

## VM Scaling Terminology

### Scale Up vs. Down

```text
Scale Up
├── Change to bigger machine (D2 → D4)
├── More CPU, more RAM
├── Causes downtime (VM restart)

Scale Down
├── Change to smaller machine (D4 → D2)
├── Less CPU, less RAM
├── Causes downtime
```

### Scale Out vs. In

```text
Scale Out
├── Add more instances (2 → 3 → 4)
├── Horizontal scaling
├── No downtime (new instances start)
├── Requires load balancer

Scale In
├── Remove instances (4 → 3 → 2)
├── Horizontal scaling
├── No downtime (instances shut down gracefully)
```

**Exam focus:**

```text
Up/Down = Machine size (vertical)
Out/In = Machine count (horizontal)
```

---

## Encryption at Host

**What it is:** Encryption of VM temporary storage and disk caches using host hardware

**Protects:**

- VM temporary disk (ephemeral storage)
- OS disk cache
- Data disk cache
- Data being processed by the host hypervisor before reaching Azure Storage

**Does NOT protect:**

- Active VM memory (RAM) — use Azure Disk Encryption or Confidential VMs for full memory encryption

**Trade-off:**

```text
Enables: Host-level encryption of temporary storage and caches
Costs: Minimal performance impact (transparent to VM)
```

**When to use:**

```text
Regulatory requirement: All temporary/cached data must be encrypted at hardware level
    ↓
Enable encryption at host

Alternative for full memory encryption: Use Confidential VM or Azure Disk Encryption
```

---

## Moving VMs

### Move Within Subscription

**Resource Group move:**

```text
VM in: rg-prod
    ↓
Target: rg-prod-v2
    ↓
Requirements:
├── Both in same subscription
├── Managed disks only
├── No public IP (or move it too)
```

### Move Between Subscriptions

**Same requirements, different subscription**

```text
VM in: Subscription A, rg-prod
    ↓
Target: Subscription B, rg-prod
    ↓
Azure moves: VM, disks, NICs, etc.
```

### Move to Different Region

**Constraints:**

```text
Region move requires:
├── Image available in target region
├── Quota in target region
├── Managed disk replication possible
```

---

## Don't Confuse

### ARM Template vs. Bicep

```text
ARM Template: JSON format, older
Bicep: Simplified language, compiles to ARM

Both: Deploy identical infrastructure
Use: Bicep (simpler to read/maintain)
```

### Availability Set vs. Availability Zone

| Aspect | Availability Set | Availability Zone |
|---|---|---|
| **Scope** | One datacenter | Multiple datacenters |
| **Resilience** | Hardware failure | Datacenter failure |
| **SLA** | 99.95% | 99.99% |
| **Cost** | Free | Possible data transfer costs |

### Scale Up vs. Scale Out

```text
Scale Up: Bigger machine (downtime)
Scale Out: More machines (no downtime)
```

### ACI vs. Container Apps

```text
ACI: Serverless, one-off containers
Container Apps: Managed platform, scaling, networking
```

### App Service Slots

```text
Production: Live traffic
Staging: Testing before prod swap
```

---

## Real-World Scenario

### Contoso E-Commerce Application

**Requirements:**

- Web tier scales automatically with traffic
- Database tier always available (3-zone redundancy)
- Deploy without downtime
- Infrastructure as Code

**Solution:**

```text
Infrastructure (Bicep template):
├── VM Scale Set: Web Tier
│   ├── Image: Custom Linux, pre-configured
│   ├── Min instances: 2
│   ├── Max instances: 20
│   ├── Auto-scale: CPU > 70% (add), CPU < 30% (remove)
│   ├── Zones: 1, 2, 3 (spread across zones)
│   └── Load Balancer: Distribute traffic
│
├── VMs: Database Tier (Manual)
│   ├── VM1 (Zone 1)
│   ├── VM2 (Zone 2)
│   ├── VM3 (Zone 3)
│   ├── Availability: Zones (physical redundancy)
│   └── Premium SSD: Fast I/O
│
├── App Service: Admin Portal
│   ├── Standard plan (slots available)
│   ├── Deployment slot: Staging (for testing)
│   ├── Swap: Production ← Staging (when ready)
│   └── VNet Integration: Access database privately
│
└── Deployment Process:
    1. Push Bicep template
    2. Azure creates entire infrastructure
    3. Deploy app code via GitHub Actions
    4. Scale Set auto-adjusts based on load
    5. No manual intervention needed

Result:
├── Fully automated
├── Infrastructure as Code
├── Auto-scaling
├── Multi-zone resilience
├── Zero-downtime deployments
```

---

## Domain Summary

| Concept | Purpose | Trigger | Nearest Confusion | Key Exam Rule |
|---|---|---|---|---|
| **ARM Template** | Declare infrastructure | Standardize deployments | Bicep | JSON format |
| **Bicep** | Simplified ARM | Easier to read | ARM Template | Compiles to ARM |
| **VM Size** | Choose CPU/RAM | Match workload | SKU naming | D2s_v3 format |
| **Managed Disk** | Persist data | Replace storage account | Unmanaged | Azure-managed replication |
| **Availability Set** | Logical redundancy | Same datacenter needed | Zone | Fault/update domains |
| **Availability Zone** | Physical redundancy | Multi-datacenter | Set | 99.99% SLA |
| **Scale Set** | Auto-scaling VMs | Load changes | Manual | Spread across zones |
| **ACR** | Image repository | Store container images | Docker Hub | Private registry |
| **ACI** | Serverless containers | Quick jobs | Container Apps | No infrastructure |
| **Container Apps** | Managed platform | Auto-scale microservices | ACI | Environment-level |
| **App Service Plan** | Compute for apps | Host web apps | VM | Managed platform |
| **Deployment Slots** | Staging environment | Blue-green deploy | Backup | Zero-downtime swap |
| **Scale Up** | Bigger machine | More power | Scale Out | Downtime |
| **Scale Out** | More machines | More capacity | Scale Up | No downtime |
| **Encryption at Host** | Encrypt VM memory | Full-disk requirement | Storage encryption | ~3-5% overhead |

---

## 30-Second Recall

**Infrastructure as Code:**
- ARM Template = JSON format
- Bicep = simplified, compiles to ARM
- Both deploy standardized infrastructure

**VMs:**
- Standard_D2s_v3 = D-series, 2 vCPU, SSD optimized
- Managed disks = Azure-managed, better than unmanaged
- Zones spread VMs across physical datacenters

**Availability:**
- Availability Set = logical, same datacenter, 99.95%
- Availability Zone = physical, separate datacenters, 99.99%
- Scale Set = auto-scaling instances

**Containers:**
- ACR = private image repository
- ACI = serverless one-off containers
- Container Apps = managed platform, auto-scaling

**App Service:**
- Plan = compute resource
- App = application code
- Slots = staging environment for testing

**Scaling:**
- Up/Down = machine size
- Out/In = machine count
- Auto-scaling = metrics-based

**Key Distinctions:**
- Availability Set vs. Zone = datacenter vs. zones
- Scale Up vs. Scale Out = size vs. count
- ACI vs. Container Apps = one-off vs. managed platform
- VNet Integration vs. Private Endpoint = outbound vs. inbound

---

## Official Sources

- [ARM Templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/)
- [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/)
- [VM Availability](https://learn.microsoft.com/en-us/azure/virtual-machines/availability)
- [VM Scale Sets](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/)
- [Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/)
- [Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/)
- [App Service](https://learn.microsoft.com/en-us/azure/app-service/)
