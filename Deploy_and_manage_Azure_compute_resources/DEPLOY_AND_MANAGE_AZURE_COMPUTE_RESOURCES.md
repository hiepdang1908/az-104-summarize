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

**Interpretation rule:** The resource `type`, `apiVersion`, `name`, `location`, `sku`, and `properties` describe the desired resource state. `dependsOn` controls order only when Azure Resource Manager cannot infer a dependency. Use an API version that supports the properties used by the template; copied export templates often need review before reuse.

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

**Modify or convert:** Update Bicep parameters, resource properties, modules, and symbolic references, then validate the file before deployment. To convert an ARM JSON template, use the Bicep decompiler (for example, `az bicep decompile --file template.json`) as a starting point, then review and refactor the generated Bicep before using it in production.

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

**Constraint:** An exported template is a starting point, not a production-ready source of truth. It can contain hard-coded values, generated resource names, secure settings that cannot be exported, and resource definitions that must be separated or parameterized before redeployment.

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

### Modifying ARM Templates

**Common modification scenarios:**

**1. Change VM size:**

Before:
```json
"hardwareProfile": {
  "vmSize": "Standard_D2s_v3"
}
```

After (upgrade to larger):
```json
"hardwareProfile": {
  "vmSize": "Standard_D4s_v3"
}
```

**2. Parameterize hardcoded values:**

Before (hardcoded):
```json
"name": "myStorageAccount123"
```

After (parameterized):
```json
"parameters": {
  "storageAccountName": {
    "type": "string",
    "defaultValue": "mystorageaccount"
  }
},
"resources": [{
  "name": "[parameters('storageAccountName')]"
}]
```

**3. Add a new resource:**

Adding a new storage account to existing template:
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2024-01-01",
  "name": "[parameters('newStorageName')]",
  "location": "[resourceGroup().location]",
  "kind": "StorageV2",
  "sku": {
    "name": "Standard_LRS"
  },
  "properties": {
    "accessTier": "Hot"
  }
}
```

**4. Add a dependency (if resources must deploy in order):**

```json
"dependsOn": [
  "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]"
]
```

**5. Modify resource properties:**

Enable encryption for storage account:
```json
"properties": {
  "encryption": {
    "services": {
      "blob": {
        "enabled": true
      }
    },
    "keySource": "Microsoft.Storage"
  }
}
```

**6. Remove a resource:**

Removing a resource from a template does **not** delete an already deployed resource when the deployment uses the normal incremental mode. Use a deliberate deletion process, or complete mode only after understanding its scope and risk.

**7. Convert parameter to variable:**

Before (parameter):
```json
"parameters": {
  "location": {
    "type": "string",
    "defaultValue": "eastus"
  }
}
```

After (calculated variable):
```json
"variables": {
  "location": "[if(equals(resourceGroup().location, ''), 'eastus', resourceGroup().location)]"
}
```

**Common modification workflow:**

```
1. Export existing template from Resource Group
2. Open template.json in editor
3. Identify section to modify (usually in "resources" array)
4. Make changes (update properties, add parameters, etc.)
5. Save modified template
6. Deploy modified template: New resources created, existing updated
7. Test in non-production first
```

**Testing modifications:**

```
Use: Validate-AzResourceGroupDeployment
  -ResourceGroupName myResourceGroup
  -TemplateFile template.json

Result: Errors and warnings before deployment
```

---

## Virtual Machines

### VM Sizing

**Common size families:**

| Family | Design focus | Typical use case |
|---|---|---|
| **B-series** | Burstable CPU | Dev/test and variable light workloads |
| **D-series** | Balanced CPU and memory | General-purpose workloads |
| **E-series** | High memory-to-CPU ratio | Memory-intensive applications and databases |
| **F-series** | High CPU-to-memory ratio | Compute-intensive workloads |
| **M-series** | Very large memory configurations | Large databases and enterprise applications |
| **H-series** | High-performance computing | Scientific and engineering workloads |

Available sizes, processors, disk support, quotas, and prices vary by family, generation, and region. Select from the sizes currently available for the target region and subscription.

**Naming example:** `Standard_D2s_v3`

```text
Standard = Azure VM SKU prefix
D = series (D-series)
2 = vCPU count for this SKU
s = supports Premium Storage
v3 = generation
```

### VM Disks

**OS Disk:**

```text
├── Contains operating system
├── Cannot be shared with other VMs
└── Current Azure VM deployments use managed OS disks; supported size depends on disk and VM generation
```

**Data Disks:**

```text
├── Additional storage
├── Supported count and size depend on the VM size and disk type
├── Some managed disk types support shared-disk configurations
└── Attach/detach behavior depends on VM, disk, and guest OS support
```

**Temporary Disk:**

```text
├── Ephemeral storage on host hardware when the selected VM size includes it
├── Not persisted (lost on stop/deallocate)
├── Fast access, no replication cost
├── Don't rely for persistent data
└── Use for cache, temp files only
```

### Managed Disks

**What it is:** Disk storage managed by Azure

**Advantages:**

- Azure manages disk durability and platform replication; point-in-time backup is configured separately with Azure Backup
- Simpler RBAC model
- LRS and supported ZRS options let you match disk redundancy to the workload
- Can be encrypted at rest

**Disk types:**

| Type | Performance profile | Typical use case |
|---|---|---|
| **Standard HDD** | Lowest-cost disk option; latency-sensitive work is not the target | Infrequent access, dev/test, non-critical workloads |
| **Standard SSD** | Balanced cost and performance | General workloads with moderate I/O |
| **Premium SSD** | Lower latency and higher performance | Production and I/O-intensive workloads |
| **Premium SSD v2** | Tunable performance independent of capacity | Production workloads needing flexible IOPS and throughput |
| **Ultra Disk** | Configurable high IOPS and throughput | Data-intensive, latency-sensitive workloads |

**Shared disks:**

```text
Multiple VMs can attach one supported managed data disk
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
If host maintenance affects the VM: Application can be unavailable
```

**Cost:** Lowest

**Availability:** A single VM is a single point of failure. Any applicable SLA depends on the VM and disk configuration and the current Azure SLA terms.

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

**Important:** Place two or more VMs in the availability set to distribute them across fault and update domains. VM sizes and current SLA eligibility depend on the deployed configuration; verify the current SLA rather than treating one percentage as universal.

**Cost:** The availability set has no separate resource charge; the VMs and related resources are still billed.

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
Availability Zone: Physical separation, different datacenters (same region)
```

**Availability decision:** Availability Zones provide physical separation within one region. SLA eligibility depends on the current service SLA and the exact deployment, so validate the applicable SKU and architecture.

**Cost:** Potential outbound data transfer costs between zones

**Important:** Availability Zones protect against ZONE failure, not total REGION failure

**When to use:** Mission-critical workloads requiring resilience against datacenter/zone failure within a region

**For region failure protection:** Use geo-redundant strategies such as:
- Azure Site Recovery (cross-region failover)
- Geo-redundant storage (GRS/GZRS)
- Multi-region deployment
- Paired regions architecture

### VM Scale Set

**What it is:** Managed group of load-balanced VMs that can scale automatically or on a schedule.

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

- **Consistent base configuration** — Scale-set instances normally start from the same image and model
- **Auto-scaling** — Add/remove instances based on metrics
- **Load balancing** — Built-in load balancer for distribution
- **Zone redundancy** — Can spread across zones

#### Orchestration Mode: Uniform vs. Flexible

| Mode | Core idea | Choose when |
|---|---|---|
| **Uniform** | Scale-set-managed instances use a common VM model and configuration. | You need a homogeneous, large-scale stateless fleet. |
| **Flexible** | Provides a scale-set management experience across broader VM scenarios and more flexible VM lifecycle/configuration. | You need features such as mixing VM types or Spot and on-demand VMs, or support for stateful/quorum-style workloads. |

The orchestration mode is chosen when the scale set is created and cannot be changed later. Do not assume every scale set is a set of identical clones; that describes the common Uniform model, not every Flexible deployment.

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

**Availability decision:** A scale set can use availability zones, but availability and SLA depend on the configured orchestration, instance count, zones, and application design.

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
- **On-demand execution** — Run a container group without managing VMs; create a new group when more independent capacity is required
- **Isolated** — Each container in separate environment
- **Consumption billing** — Charges depend on requested CPU, memory, and execution duration; verify current regional pricing

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

- **Auto-scaling** — Scale to zero or out to the configured and service-supported replica limit
- **Environment** — Managed compute with multiple apps
- **Revisions** — Versioning and traffic splitting
- **Dapr** — Distributed application runtime (service-to-service)
- **Container workload profiles** — Choose supported Consumption or Dedicated compute profiles for the workload

**Comparison:**

| Feature | ACI | Container Apps | AKS |
|---|---|---|---|
| **Management** | Minimal | Managed platform | Manual |
| **Scaling** | Manual | Auto | Manual or custom |
| **Networking** | Per-container | Environment-level | Cluster-level |
| **Cost model** | Requested resources and execution duration | Consumption or workload-profile resources | Cluster node resources plus supporting services |

**When to use:**

```text
ACI: One-off jobs, simple containers
Container Apps: Microservices, auto-scaling needed
AKS: Complex orchestration, large scale
```

### Container Sizing and Resource Limits

**Azure Container Instances (ACI) sizing:**

```text
CPU and memory combinations, quotas, regions, and pricing vary. Choose a supported combination based on measured application need, then verify the current regional limits and pricing.
```

**Sizing rule:** Azure Container Instances uses CPU and memory requested for the container group. Container Apps has its own supported replica resources and scaling model; do not apply Kubernetes request/limit semantics unless you are specifically working in AKS.

**Memory sizing strategy:**

```text
Estimate application memory
    ↓
├── Measure normal and peak CPU/memory usage
├── Include runtime and initialization overhead
├── Choose a supported resource combination
    ↓
Load-test and observe throttling, restarts, and latency
    ↓
Set as container memory request
```

**Azure Container Apps scaling:**

**Automatic scaling rules:**

```text
Scale rules can use HTTP concurrency, event-driven signals, or custom/KEDA-supported scalers. Configure minimum and maximum replicas, then choose a scaler that represents demand; do not assume fixed CPU or memory thresholds apply to every app.
```

**Scale-to-zero:**

```text
Configuration: Allow scale to zero
    ↓
When no traffic: Scale down to 0 instances
    ↓
Compute consumption can fall substantially; other configured resources can still incur charges
    ↓
Tradeoff: A cold start can add latency on the first request; the duration depends on the image, workload profile, and initialization work.
    ↓
When traffic returns: Scale back up automatically
```

**Example Container Apps configuration:**

```yaml
Scaling rules:
├── Min replicas: 1
├── Max replicas: 10
├── CPU target: 70%
├── Memory target: 80%
    ↓
When deployed:
├── 3 AM (no traffic): 1 instance running
├── 9 AM (peak): Auto-scales to 8 instances
├── Evening: Auto-scales back to 2 instances
├── Cost follows the selected plan, resources, replicas, and current pricing
```

**Comparison:**

| Aspect | ACI | Container Apps | AKS |
|---|---|---|---|
| **Sizing** | Per-container specified | Per-replica resources within a supported workload profile | Per-pod requests/limits on cluster nodes |
| **Auto-scaling** | None | Built-in | Via Horizontal Pod Autoscaler (HPA) |
| **Scale to zero** | No built-in autoscale | Supported for eligible profiles and configurations | Requires workload and node-pool autoscaling configuration; limits apply |
| **Startup time** | Workload dependent | Workload and profile dependent | Workload and node-capacity dependent |
| **Cost model** | Requested container resources and duration | Selected plan, workload profile, replicas, and usage | Cluster node resources plus supporting services |

---

## App Service

### App Service Plan

**What it is:** Compute resource where apps run

**SKUs:**

| Tier family | Compute model | Typical capability boundary |
|---|---|---|
| **Free / Shared** | Shared compute | Development or evaluation; limited scale and features |
| **Basic** | Dedicated compute | Entry dedicated hosting; backup is supported, but only for the production slot |
| **Standard** | Dedicated compute | Adds production features such as deployment slots and autoscale |
| **Premium** | Higher-performance dedicated compute | More scale and advanced production features |
| **Isolated** | Dedicated App Service Environment | Network-isolated enterprise hosting |

**Key concept:**

```text
Plan: Selected App Service tier and worker size
    ├── Cost depends on region, operating system, tier, size, and instance count
    ├── Scale limits depend on the selected tier and current service limits
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
Clients can access the app through the private endpoint
```

**Public access:** A private endpoint is inbound access only. Public access can coexist with it; disable public network access separately when private-only access is required. VNet Integration is still the separate outbound feature and cannot use the same subnet as the private endpoint.

**Key distinction:**

```text
VNet Integration
├── Outbound from app
├── App can reach private resources

Private Endpoint
├── Inbound to app
├── Clients access app privately
```

### Custom DNS Names and Domain Mapping

**Default URL:**

```text
App Service created: contoso-app
    ↓
Default domain: contoso-app.azurewebsites.net
    ↓
Anyone can access this URL (no custom domain)
```

**Custom domain and private endpoint flow:**

```text
1. Add and validate the custom domain in App Service using public DNS validation.
2. Create the TLS binding after the custom domain is validated.
3. For private clients, make the custom DNS name resolve to the private endpoint:
    - Preferred: private CNAME → <app-name>.azurewebsites.net
    - Private-only custom zone: A record → private endpoint IP
4. Link the private DNS zone privatelink.azurewebsites.net to the client VNet
    so <app-name>.azurewebsites.net resolves through Private Link.
```

Private DNS does not replace the public DNS validation required to add a custom domain. The TLS certificate must match the custom hostname, whether clients reach it privately or publicly.

**Custom domain setup without private endpoint:**

```text
Goal: Access app via myapp.contoso.com (owned domain)

Step 1: Verify domain ownership
    ├── Create DNS verification record in domain registrar
    └── Azure confirms you own contoso.com

Step 2: Map custom domain to App Service
    ├── In App Service: Add custom domain
    ├── Point DNS to Azure
    └── Azure adds SSL binding

Step 3: DNS records (choose one)
    ├── A record: points to App Service IP
    │   └── contoso.com (root) → App Service IP
    │   └── www.contoso.com → App Service IP
    ├── CNAME record: points to azurewebsites.net
    │   └── www.contoso.com → contoso-app.azurewebsites.net
    └── ALIAS record (Azure DNS only): hybrid approach
```

**A Record vs. CNAME:**

| Record Type | Use | Limitation |
|---|---|---|
| **A Record** | Point domain root (contoso.com) | Requires IP (may change) |
| **CNAME** | Point subdomain (www.contoso.com) | Cannot use for root domain |
| **ALIAS** | Azure DNS: root domain | Only in Azure DNS |

**Typical configuration:**

```text
contoso.com (root)
    ├── A record: Points to 40.117.40.200 (App Service IP)
    └── Result: contoso.com → App Service

www.contoso.com (subdomain)
    ├── CNAME record: Points to contoso-app.azurewebsites.net
    └── Result: www.contoso.com → contoso-app.azurewebsites.net
```

### TLS/SSL Certificates

**What it is:** Encryption for HTTPS connections

**Why needed:**

```text
User browser → App Service
    ↓
Without cert: HTTP (unencrypted, browser warns user)
    ↓
With cert: HTTPS (encrypted, browser shows green lock)
```

**Certificate sources:**

| Source | Renewal behavior | Key point |
|---|---|---|
| **App Service Managed Certificate** | Automatically renews while prerequisites remain valid | Suitable for supported custom domains |
| **Azure Key Vault certificate** | App Service can synchronize renewed Key Vault certificate versions when configured correctly | Certificate lifecycle is managed in Key Vault |
| **Uploaded private certificate** | Administrator uploads a renewed certificate and updates the binding | Existing certificate from a trusted CA |
| **Self-signed certificate** | Administrator-managed | Testing, not public browser trust |

**Certificate prerequisites:** A TLS binding for a custom domain requires an App Service plan in Basic, Standard, Premium, or Isolated. The free App Service managed certificate also requires the domain to be mapped to the app. It does not support wildcard names, private DNS, export, or App Service Environment. A root-domain managed certificate requires the app to remain publicly reachable for certificate issuance and renewal.

**App Service Managed Certificate (simplest):**

```text
1. In App Service: Custom domains
2. Select domain (contoso.com)
3. Click "Add binding"
4. Select "Managed certificate"
5. Save
6. Azure automatically creates and renews the certificate while its prerequisites remain valid
```

**Bring-your-own certificate:**

```text
1. Purchase or create certificate
2. Export as .pfx file (includes private key)
3. In App Service: TLS/SSL settings
4. Upload .pfx file
5. Select certificate for binding
6. Save
```

**Key distinction:**

```text
HTTP binding: No cert, no encryption
    ├── contoso-app.azurewebsites.net (HTTP)
    └── Users see browser warning

HTTPS binding: With cert, encrypted
    ├── contoso-app.azurewebsites.net (HTTPS)
    └── Secure connection, green lock
```

**Force HTTPS:**

```text
Configuration: Force HTTPS
    ↓
User tries: http://contoso-app.azurewebsites.net
    ↓
App Service redirects: https://contoso-app.azurewebsites.net
    ↓
Result: All traffic forced to HTTPS
```

### App Service Backup

**What it is:** Full offline backups of supported app content and restorable configuration. Backups are not incremental, and not every App Service setting is restored.

**Backup choices:**

| Feature | Automatic backup | Custom backup |
|---|---|---|
| **Supported plans** | Basic, Standard, Premium, Isolated | Basic, Standard, Premium, Isolated |
| **Configuration** | Platform managed; no setup | User configured; on-demand or scheduled |
| **Storage account** | No customer account required | Required in the same subscription; must support SAS-based authorization |
| **Frequency** | Hourly; not configurable | Configurable, with a two-hour minimum interval and no more than 12 manual plus scheduled backups per day |
| **Retention** | 30 days; platform thins older hourly recovery points | Scheduled retention is 0–30 days or indefinite; on-demand backups are retained indefinitely |
| **Maximum backup size** | 30 GB | 10 GB total |
| **Linked database** | Not included | Do not use for new database protection; use the database service's native backup |
| **Download / partial backup** | Not downloadable; no partial backup | Stored as downloadable blobs; partial backup supported |
| **VNet path** | Not supported | Supported when the documented VNet integration and storage firewall prerequisites are met |

Basic supports backup and restore for the production slot only. Free and Shared plans do not support App Service backup and restore. New custom backup configurations no longer provide a durable strategy for linked databases, and existing linked-database backup support ends on March 31, 2028; use each database service's native backup capability.

**Backup configuration:**

```text
1. In App Service: Backups
2. Choose custom backup and select a Storage account/container that supports SAS authorization
3. Configure on-demand or scheduled backups and the retention requirement
4. For a firewall-protected account, use the documented VNet-integration backup path when its prerequisites are met
5. Save and verify a backup job completes
```

**Restore from backup:**

```text
App crashes or data corrupted
    ↓
In App Service: Backups
    ↓
Select the required recovery point
    ↓
Azure restores: App + files + config to that point in time
    ↓
Result: Application back to working state
```

**Restore boundary:** Restore can overwrite an existing app or target a new app or slot. The target app or slot stops during restore, and restoring to an existing slot overwrites its file-system data. Restore to a deployment slot first when minimizing production downtime matters, then validate and swap. Networking, authentication, managed identities, custom domains, TLS/SSL, scale settings, alerts, and linked databases are not restored by automatic backup; protect databases with their native service backup.

**Cost:**

```text
Custom backup storage is charged through the selected storage account.
Cost depends on backup size, frequency, retention, and current regional pricing.
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

**Swap behavior:** Production stays online while the source slot is prepared and warmed. App settings and connection strings normally swap, but can be marked as deployment-slot settings to stay with their slot. Managed identities, custom domains, scale settings, VNet integration, and several platform settings are slot-specific and do not swap. Do not assume every configuration value follows the code.

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

- Active VM memory (RAM) — use a Confidential VM when protection of memory in use is a requirement; Azure Disk Encryption encrypts disks, not active memory

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

Alternative for protection of data in use / VM memory: Use a Confidential VM. Azure Disk Encryption protects OS and data disks, not active memory.
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

**Cross-subscription moves are supported only for supported resource types and compatible source and destination subscriptions.** Validate every dependent resource with the current move-support matrix before treating a move as an option.

```text
VM in: Subscription A, rg-prod
    ↓
Target: Subscription B, rg-prod
    ↓
Azure moves: VM, disks, NICs, etc.
```

### Move to Different Region

**Different-region move:** A resource move is not the normal mechanism for changing a VM's region. Use an appropriate migration or replication approach, such as Azure Resource Mover, Azure Site Recovery, or rebuild from an image, based on the workload and downtime requirement.

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
| **Availability basis** | Fault and update domain distribution | Physical zone distribution within one region |
| **Cost** | No separate availability-set charge | Possible inter-zone data-transfer charges; verify current pricing |

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
| **Availability Zone** | Physical redundancy | Multi-datacenter | Set | Verify current SLA eligibility for the architecture |
| **Scale Set** | Auto-scaling VMs | Load changes | Manual | Spread across zones |
| **ACR** | Image repository | Store container images | Docker Hub | Private registry |
| **ACI** | Serverless containers | Quick jobs | Container Apps | No infrastructure |
| **Container Apps** | Managed platform | Auto-scale microservices | ACI | Environment-level |
| **App Service Plan** | Compute for apps | Host web apps | VM | Managed platform |
| **Deployment Slots** | Staging environment | Blue-green deploy | Backup | Zero-downtime swap |
| **Scale Up** | Bigger machine | More power | Scale Out | Downtime |
| **Scale Out** | More machines | More capacity | Scale Up | No downtime |
| **Encryption at Host** | Encrypt temporary disk and host caches | Disk encryption | Does not encrypt active VM memory | Configuration and support vary by VM |

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
- Availability Set = logical fault/update-domain distribution in one datacenter
- Availability Zone = physical separation across datacenters in one region
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
