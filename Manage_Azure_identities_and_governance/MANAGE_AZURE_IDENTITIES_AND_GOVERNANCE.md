# Manage Azure Identities and Governance

---

**← [Previous: Master Mental Map](../AZ-104_MASTER_MENTAL_MAP.md)** | **[Home](../README.md)** | **[Next: Storage →](../Implement_and_manage_storage/IMPLEMENT_AND_MANAGE_STORAGE.md)**

---

## Big Picture

An Azure administrator is responsible for **who can access what and why**.

This domain controls:

- **Identification** — Who are you? (Entra ID users, groups, service accounts)
- **Authentication** — Prove your identity (credentials)
- **Authorization** — What can you do? (RBAC role assignments)
- **Governance** — Are all resources following compliance rules? (Policy, locks, tags)
- **Organization** — How are subscriptions and resource groups structured? (Management Groups, resource organization)
- **Cost** — How much is being spent? (Budgets, alerts, Azure Advisor)

Without this domain, anyone could access anyone else's resources, and there would be no way to track who changed what or prevent accidental deletion.

---

## Mental Model

```text
Identity Foundation
    ↓
    Entra ID (users, groups, service principals, managed identities)
    ↓
Authentication
    ↓
    Verify identity (credentials, MFA, passwordless)
    ↓
Authorization (RBAC)
    ↓
    Principal + Role + Scope = Permission
    ↓
    Example: john@contoso.com (Principal)
             + Contributor role (Role)
             + Resource Group scope (Scope)
             = Can modify resources in that RG
    ↓
Governance Enforcement
    ├── Azure Policy (configuration rules)
    ├── Resource Locks (prevent deletion/modification)
    ├── Tags (organize and track)
    │
Organizational Hierarchy
    ├── Management Group (organize subscriptions)
    ├── Subscription (billing + resource limit)
    ├── Resource Group (lifecycle + access)
    └── Resource (actual Azure object)
    ↓
Cost & Compliance
    ├── Budgets (spending alerts)
    ├── Azure Advisor (optimization)
    ├── Policies (compliance enforcement)
    └── Locks (compliance protection)
```

---

## Microsoft Entra ID (Identity Management)

### What It Means

Microsoft Entra ID (formerly Azure AD) is Azure's identity directory service. It stores information about **who** can access Azure.

It is:

- **Cloud-based directory** — Users and groups live in the cloud
- **Authentication service** — Proves identity before granting access
- **Authorization provider** — Works with RBAC to say what users can do
- **Hybrid-capable** — Can sync with on-premises Active Directory

### Users and Groups

**Users:**

- Cloud-only users (created directly in Entra ID)
- Synced users (synchronized from on-premises AD)
- Guest users (external users with limited access)

**Groups:**

- **Security groups** — Used to assign directory and Azure RBAC permissions. They are not Application Security Groups (ASGs), which group NICs for NSG rules.
- **Microsoft 365 groups** — Used for collaboration (teams, shared mailboxes)

### User Properties

Key properties:

- **User Principal Name (UPN)** — Format: user@domain.onmicrosoft.com or user@customdomain.com
- **Display Name** — Full name
- **Object ID** — Unique Azure identifier (used internally)
- **Account enabled** — Whether the user can sign in
- **MFA status** — Multi-factor authentication requirement
- **License** — What features the user can access

### License Management

Licenses can be assigned directly to a user or through a supported group. Before assigning a license, set the user's **usage location** because service availability is location-dependent. A direct assignment is appropriate for an exception; group-based licensing is easier to manage when users with the same role need the same service plan.

| Task | Knowledge rule |
|---|---|
| Assign a license | Select a product license and optionally enable or disable individual service plans. |
| Change a license | Add the new entitlement before removing the old one when uninterrupted service matters. |
| Remove a license | Removes access to the licensed service; understand service data-retention behavior separately. |
| License a group | Members inherit the selected licenses; nested groups are not processed for group-based licensing. |

**Trap:** Azure RBAC grants access to Azure resources. A Microsoft 365 or Microsoft Entra license grants product/service entitlement; it does not grant Contributor, Reader, or other Azure RBAC permissions.

### External Users (Microsoft Entra B2B)

Microsoft Entra B2B collaboration lets an organization share apps and Azure resources with external partners while they authenticate using their own work, school, or supported external identity.

```text
Invite external user
    ↓
Guest user object created in the resource tenant
    ↓
Guest redeems invitation and signs in with home identity
    ↓
Assign group, application, or Azure RBAC access as needed
```

- A **Guest** user is normally an external B2B collaborator. A **Member** user is normally internal to the tenant; the user type alone is not an authorization grant.
- After redemption, the guest has a user object in the resource tenant and can receive Azure RBAC role assignments at the appropriate scope, directly or through a group.
- External collaboration and cross-tenant access settings control who can be invited and authenticated. Use guest lifecycle controls to remove access when the collaboration ends.
- B2B collaboration is for workforce-to-partner access. It is not the same purpose as a customer-facing identity implementation.

### Groups

Benefits:

- **Bulk RBAC assignment** — Assign one role to a group instead of individual users
- **Easy management** — Add/remove users from groups without touching role assignments
- **Scalability** — Adding 100 users? Add them to a group once instead of 100 role assignments

Common pattern:

```text
Security Group: "WebAdmins"
├── Contains users: alice@contoso.com, bob@contoso.com
└── Assigned Contributor role on: resource-group-prod
```

Result: All group members automatically have Contributor access.

### Self-Service Password Reset (SSPR)

Allows users to reset their own password without calling help desk.

**Configuration decisions:**

- Select the users or groups in scope; do not assume every tenant user is enabled.
- Choose the authentication methods users must register and the number required to reset.
- Confirm the tenant's current Microsoft Entra licensing and feature entitlement in Microsoft documentation before enabling a production policy.

**Scope:**

- Can reset their own password
- Cannot change password of other users
- Cannot reset if their account is locked by admin

---

## Microsoft Entra Roles vs. Azure RBAC

### Critical Distinction

| Aspect | Microsoft Entra Role | Azure RBAC |
|---|---|---|
| **What it controls** | Entra ID objects (users, groups, devices) | Azure resources (VMs, storage, etc.) |
| **Who is affected** | Entra ID directory administrators | Resource users and operators |
| **Scope** | Entire directory | Specific subscription/RG/resource |
| **Example** | "Global Administrator" can modify all Entra ID users | "Contributor" can modify resources in a RG |
| **Used for** | Identity management tasks | Resource access control |

**Key rule:**

```text
Entra Role
→ manage Entra ID (users, groups, devices, domains)

RBAC Role
→ manage Azure resources (VMs, storage, networks)
```

An Entra Global Administrator can manage all Entra users, but might have **zero** Azure resource access.

---

## Azure RBAC (Role-Based Access Control)

### Core Concept

RBAC = **Principal** + **Role** + **Scope** = Permission

```text
Principal (WHO)
    ↓ is assigned
Role (WHAT CAN THEY DO)
    ↓ at this
Scope (WHERE)
    ↓
= Permission

Example:
alice@contoso.com (Principal)
+ Contributor (Role)
+ /subscriptions/abc123/resourceGroups/rg-prod (Scope)
= Alice is Contributor in rg-prod
```

### Principal Types

- **User** — Alice, Bob, actual people
- **Service Principal** — Applications (enterprise app)
- **Managed Identity** — Workload identity (system-assigned or user-assigned)
- **Security Group** — Collection of users

### Role Types

**Built-in roles** (most common):

| Role | Capabilities | Cannot Do |
|---|---|---|
| **Owner** | Create, modify, delete resources + manage RBAC access | N/A (can do everything) |
| **Contributor** | Create, modify, delete resources | Cannot grant RBAC access to others (assign roles) |
| **Reader** | Read-only access to resources | Modify anything |
| **User Access Administrator** | Manage RBAC assignments only | Modify resources themselves |
| **Specific roles** | Limited to one service (e.g., "Virtual Machine Contributor") | Beyond the specific service |

**Custom roles** (advanced):

- Created when built-in roles don't match your needs
- `Actions`, `NotActions`, `DataActions`, and `NotDataActions` define the role's permissions
- `AssignableScopes` define where the role is available for assignment, such as supported management group, subscription, or resource group scopes
- A role assignment grants the custom role to a principal at a selected allowed scope; assignments at parent scopes are inherited by child scopes

Custom roles are not inherently subscription-scoped. `AssignableScopes` control where the role can be assigned, including supported child scopes.

### Critical Distinction: Contributor vs. Owner

```text
Contributor
├── CAN create, modify, delete resources
├── CAN use resources
└── CANNOT assign RBAC roles to others

Owner
├── CAN create, modify, delete resources
├── CAN use resources
└── CAN assign RBAC roles to others
```

**Real scenario:**

> Problem: Give your database administrator permission to manage VMs, but NOT grant access to others.
>
> Solution: Assign "Contributor" role (allows resource modification), not "Owner"

### Control Plane vs. Data Plane

**Control plane** = managing the resource itself

```
RBAC controls this:
- Create VM
- Delete VM
- Resize VM
- Configure disks
→ Requires "Contributor" or "Owner" or specific role
```

**Data plane** = accessing data inside the resource

```
Data-plane access can use service-specific authentication and/or Azure RBAC roles
that include DataActions:
- Read/write blobs or files
- Connect to a database
- Execute service data operations
→ Example: Storage Blob Data Reader + Microsoft Entra authentication
```

**Example:**

```text
Requirement: Alice should manage the storage account (control plane)
Solution: Assign RBAC "Storage Account Contributor" role

Requirement: Alice's application should read blobs (data plane)
Solution: Assign storage "Blob Data Reader" role + use managed identity
```

### Scope Hierarchy

Scope determines **where** the role applies:

```text
Management Group
    ↓ (affects all subscriptions below)
Subscription
    ↓ (affects all resource groups below)
Resource Group
    ↓ (affects all resources below)
Resource
    ↓ (affects only this resource)
```

**Inheritance rule:** A role assigned at a higher scope is inherited by lower scopes.

```text
Example:
Alice = Contributor at Subscription scope
    ↓
Result: Alice is Contributor in ALL resource groups in that subscription
Result: Alice is Contributor for ALL resources in ALL RGs
```

**Scope rule:** Azure RBAC grants accumulate down the hierarchy. A role assignment at a child scope can add permissions there, but a normal child-scope assignment cannot subtract a grant inherited from a parent scope. Azure RBAC deny assignments are exceptional system-managed controls, not the normal least-privilege design tool. Grant the narrowest role at the narrowest scope that satisfies the requirement.

---

## Azure Subscriptions

### What It Means

A subscription is:

- **Billing boundary** — All costs roll up to one subscription
- **Resource limit boundary** — Quotas are evaluated by resource type, region, SKU, and subscription. Check current quota and availability rather than memorizing a universal VM count.
- **Trust boundary** — Resources in one subscription are isolated from others
- **Scale unit** — One "container" for managing resources

### Subscription Components

- **Subscription ID** — Unique identifier (GUID format)
- **Subscription name** — User-friendly label
- **Owner account** — Account Service Account administrator account
- **Resource groups** — Collections within the subscription
- **Offers** — Billing model (Pay-As-You-Go, Enterprise Agreement, etc.)

### Multiple Subscriptions Pattern

Common enterprise scenario:

```text
Company Organization
│
├── Production Subscription
│   └── Production resources
│
├── Development Subscription
│   └── Dev/test resources
│
├── Finance Subscription
│   └── Financial systems (separate for compliance)
│
└── Management Subscription
    └── Central monitoring, backup vaults, policy
```

Benefits:

- **Cost tracking** — See costs per environment/team
- **Access control** — Different teams access different subscriptions
- **Resource limits** — Each subscription has independent quotas
- **Blast radius** — If dev subscription is compromised, production is unaffected

---

## Resource Groups

### What It Means

A resource group is:

- **Lifecycle container** — Delete the RG = delete all resources in it
- **Organization unit** — Group related resources
- **Scope for RBAC** — Assign permissions at RG level
- **Deployment unit** — ARM templates often deploy to an RG

### Key Properties

- **Name** — User-friendly identifier (per subscription)
- **Location** — Where resource group metadata is stored (not where resources are)
- **Access** — Inherit RBAC from subscription or assign specific roles

### Resource Group Scope

**RBAC at RG level:**

```text
Assign: alice@contoso.com + Contributor + rg-prod RG
Result: Alice can modify any resource inside rg-prod
Result: Alice cannot access rg-dev
```

**Important:** Resource group location ≠ resource location

```text
RG Location = East US (metadata storage)
Resource locations = can be anywhere
    ├── VM in West US
    ├── Storage in East Asia
    └── Database in South Africa
```

### When to Use Multiple Resource Groups

```text
Scenario: Company with multiple applications
Solution:
├── rg-app-billing
├── rg-app-crm
├── rg-app-erp
└── rg-shared-infrastructure

Each RG can have:
- Different RBAC access
- Different cost tracking
- Different backup policies
- Different lifecycle (one app may sunset before another)
```

---

## Management Groups

### What It Means

Management Groups organize subscriptions.

```text
Management Group (top level)
├── Subscription (prod)
├── Subscription (dev)
├── Subscription (finance)
└── Child Management Group
    ├── Subscription (partner1)
    └── Subscription (partner2)
```

### Benefits

- **Bulk policy application** — Apply policy to all subscriptions at once
- **Bulk role assignment** — Grant access to all subscriptions for a team
- **Organization** — Logical grouping (by region, by team, by environment)
- **Cost tracking** — Sum costs across subscriptions

### Common Hierarchy

```text
Root Management Group (Tenant Root Group)
│
├── Management Group: Production
│   ├── Subscription: prod-east
│   └── Subscription: prod-west
│
├── Management Group: Development
│   ├── Subscription: dev-engineers
│   └── Subscription: dev-testing
│
└── Management Group: Finance
    └── Subscription: financial-systems
```

### Policy at Management Group Level

Apply policy once to MG, affects all subscriptions below:

```
Policy Assignment: "Require Encryption at Rest"
Scope: Management Group: Production
    ↓
Applied to:
├── prod-east subscription
└── prod-west subscription
    ↓
Result: All resources in both subscriptions must have encryption
```

---

## Azure Policy

### What It Means

Azure Policy is:

- **Compliance enforcement** — Ensures resources follow organization rules
- **Declarative** — You define rules, Azure enforces them
- **Preventive** — Can block non-compliant resource creation
- **Remedial** — Can automatically fix non-compliant resources

### Core Components

**Policy definition:**

```text
Rule: Required resources must have an approved configuration

Definition includes:
- What resource type does the definition target?
- Which documented alias or property is evaluated?
- What compliant and noncompliant values are permitted?
```

**Policy assignment:**

```text
Take the "encryption at rest" rule
Apply it to: Subscription scope
Effect: Deny (block creation if not compliant)
```

**Effect options:**

- **Audit** — Log non-compliance, allow resource creation
- **Deny** — Block resource creation if non-compliant
- **Append** — Add fields to the request when the definition supports it; use `Modify` for modern property/tag remediation patterns
- **Modify** — Automatically change properties to comply
- **DeployIfNotExists** — Automatically add missing configuration
- **AuditIfNotExists** — Audit when a related resource or configuration is absent

**Policy building blocks:** A definition contains the rule and optional parameters. An initiative groups definitions under one assignment. An assignment applies a definition or initiative at a management group, subscription, resource group, or resource scope. An exclusion removes a child scope from the assignment; an exemption records an approved waiver for resources that remain in scope.

**Choose the effect:** Use `Audit` to discover impact before enforcement, `Deny` when a configuration must not be created, and `AuditIfNotExists` when the requirement depends on a related resource or setting. Use `Modify` or `DeployIfNotExists` only when the definition supports remediation. Those effects can require a managed identity on the assignment and role permissions at the remediation scope; existing resources are remediated by a remediation task, not merely by creating the assignment.

**Decision rule:** Use an initiative for a repeatable standard such as a production baseline. Use an exclusion when a child scope should not inherit that assignment. Use an exemption for a documented, time-bounded exception to a requirement that still applies at the resource's scope.

### Policy vs. RBAC

| Policy | RBAC |
|---|---|
| **Controls:** Configuration of resources | **Controls:** Who can access |
| **Question:** Does this resource follow rules? | **Question:** Does this user have permission? |
| **Example:** All VMs must have backup enabled | **Example:** Only alice can create VMs |
| **Applied to:** All resources regardless of user | **Applied to:** Users/service principals |
| **Scope behavior:** Applicability can change through exclusions, exemptions, or rule conditions | **Scope behavior:** Parent grants inherit; normal lower-scope grants do not subtract access |

### Policy vs. Lock

| Policy | Lock |
|---|---|
| **Controls:** Configuration compliance | **Controls:** Modification/deletion |
| **Enforcement:** Preventive or audit | **Enforcement:** Takes precedence over normal RBAC for protected control-plane operations |
| **Can be bypassed by:** Exemption rules | **Can be removed by:** A principal with lock-management permission |
| **Example:** Require tags on all resources | **Example:** Prevent accidental production database deletion |
| **Scope:** Across subscriptions via MG | **Scope:** Subscription, Resource Group, or Resource; parent locks are inherited |

---

## Resource Locks

### What It Means

A lock restricts Azure Resource Manager modification or deletion until it is removed by a principal with lock-management permission.

**Two lock types:**

| Lock Type | Prevents | Allows |
|---|---|---|
| **CanNotDelete** | Deletion only | Modification of properties |
| **ReadOnly** | Modification AND deletion | Reading |

### Lock Behavior

**Owner with CanNotDelete lock:**

```text
Owner tries to delete resource
    ↓
System: "This resource is locked"
    ↓
Result: Cannot delete, even though Owner can normally delete anything

To delete:
1. Remove the lock (requires lock permission)
2. Then delete
```

**Owner with ReadOnly lock:**

```text
Owner tries to modify tags
    ↓
System: "This resource is locked as ReadOnly"
    ↓
Result: Cannot modify (even read-only name can be changed by reading it)
```

### When to Use Locks

> **Control-plane boundary:** Locks protect Azure Resource Manager operations. They do not block data-plane operations such as reading a blob with a valid data credential. Use data-plane authorization and service-level protection for the data itself.

```text
Scenario: Production database account
Risk: Developer with Contributor role accidentally deletes
Solution: CanNotDelete lock on database
    ↓
Result: Even Contributor cannot delete
```

```text
Scenario: Critical storage account configuration
Risk: Script accidentally modifies account settings
Solution: ReadOnly lock on storage account
    ↓
Result: Azure Resource Manager cannot modify the account until the lock is removed
```

### Lock Scope

```text
Lock at Subscription level
    ↓
Inherited by resource groups and resources in that subscription

Lock at Resource Group level
    ↓
Inherited by all resources in RG

Lock at individual Resource level
    ↓
Applies only to that resource
```

---

## Tags

### What It Means

Tags are metadata labels attached to resources.

**Format:**

```text
Key: Value
    ↓
Example:
Environment: Production
Owner: alice@contoso.com
CostCenter: Finance-2024
Application: Billing
```

### Why Tags Matter

**Cost tracking:**

```text
Tag all resources with Department: Finance
    ↓
Run cost analysis
    ↓
See: "Finance department spent $45,000 this month"
```

**Organization:**

```text
Tag all production resources with Environment: Prod
    ↓
Policy rule: Only Prod resources can use certain SKUs
    ↓
Enforce expensive resources only in production
```

**Compliance:**

```text
Tag all resources with DataClassification: Confidential
    ↓
Security policy: Confidential resources must have encryption
    ↓
Audit non-compliant resources
```

### Important Caveat

**Tags are NOT enforced by default.**

```text
You create a tag: Owner: alice

But then bob@contoso.com creates a resource without the Owner tag.
    ↓
Result: Resource exists untagged (no error)

To enforce: Use Azure Policy
Policy rule: "Append Owner tag if missing"
    ↓
Result: All resources automatically tagged
```

---

## Cost Management

### Budgets

A budget is a spending limit alert.

**How it works:**

```text
Set budget: $1,000 per month
    ↓
Actual spending: $750
    ↓
Alert triggers: When approaching 80% = $800 spent
    ↓
Alert recipient: Alert email sent to owner
    ↓
Action: Budget does NOT automatically stop spending
        (you must manually intervene or set policy)
```

**Budget scope:**

- Subscription
- Resource Group
- Management Group

### Azure Advisor Recommendations

Azure Advisor analyzes usage and suggests improvements:

- **Cost optimization** — Unused VMs, right-sizing recommendations
- **Security** — Missing NSGs, weak authentication
- **Reliability** — Resources without backups, single-point failures
- **Performance** — Underutilized resources, capacity issues
- **Operational excellence** — Automation opportunities, best practices

### Cost Analysis

Tool to visualize spending:

- By resource type
- By resource group
- By tag
- By service
- Over time

---

## Don't Confuse

### Entra ID vs. Azure RBAC

```text
Entra ID
├── Manages: users, groups, devices in the directory
├── Controls: who can sign in to Azure
├── Example: alice@contoso.com is a user

Azure RBAC
├── Manages: permissions for Azure resources
├── Controls: what users can do with resources
├── Example: alice has Contributor on rg-prod
```

**Scenario clarification:**

```text
Step 1: Alice is created in Entra ID
        → Entra ID domain
Step 2: Alice signs in
        → Entra ID authenticates her
Step 3: Alice tries to create a VM
        → Azure RBAC checks: does alice have permission?
        → Azure RBAC grants/denies
```

### Azure RBAC vs. Azure Policy

```text
Azure RBAC
├── Question: Does this user have permission?
├── Controls: Access
├── Example: Alice (Contributor) can create VMs
           Bob (Reader) cannot create VMs

Azure Policy
├── Question: Does this resource follow compliance rules?
├── Controls: Configuration
├── Example: All VMs must have backup enabled
            (regardless of who creates them)
```

### Policy vs. Lock

```text
Azure Policy
├── Enforces: Configuration compliance
├── Can be bypassed: Via exemption rules
├── Example: Policy requires encryption, but exemption allows unencrypted

Resource Lock
├── Enforces: Prevents protected control-plane modification/deletion
├── Takes precedence over normal RBAC until removed by a principal with lock permission
├── Example: Lock prevents deletion; no exemption mechanism exists
```

### Contributor vs. Owner

```text
Contributor
├── CAN: Create, modify, delete resources
├── CAN: Use resources
├── CANNOT: Assign roles to others

Owner
├── CAN: Create, modify, delete resources
├── CAN: Use resources
├── CAN: Assign roles to others
```

**Exam trap:**

```text
Question: "Give alice permission to manage VMs, but she cannot grant
         access to others"

Correct: Contributor (or "Virtual Machine Contributor")
Wrong:   Owner
```

---

## Real-World Scenario

### Contoso Multi-Team Setup

**Requirements:**

- Separate prod and dev environments
- Finance team manages cost center
- Operations team manages production
- Developers access only dev
- Nobody can accidentally delete production database

**Solution:**

```text
Management Group: Contoso
├── MG: Production
│   └── Subscription: prod-ops
│       ├── RG: prod-db
│       │   ├── SQL Database
│       │   └── Lock: CanNotDelete (protects against accidents)
│       └── RG: prod-app
│           └── VMs
│           └── RBAC: ops-team = Contributor
│
├── MG: Development
│   └── Subscription: dev-eng
│       └── RG: dev-all
│           ├── VMs
│           ├── Storage
│           └── RBAC: dev-team = Contributor
│
└── MG: Finance
    └── Subscription: finance-shared
        └── RG: billing
            └── Storage account
            └── RBAC: finance-team = Contributor

Tags on all resources:
├── Environment: Prod / Dev / Shared
├── Owner: ops-team / dev-team / finance-team
├── CostCenter: 1001 / 2002 / 3003

Policies:
├── At MG level: All resources must have Owner tag
├── At MG level: All resources must have CostCenter tag
├── At prod-ops: Require encryption at rest
├── At prod-ops: Require backup for all databases
```

**Result:**

- Developers can only access dev subscription
- Operations owns production
- Finance tracks costs per team
- Production cannot be accidentally deleted
- Compliance is enforced across the org

---

## Domain Summary

| Concept | Purpose | Trigger | Nearest Confusion | Key Exam Rule |
|---|---|---|---|---|
| **Entra ID** | Identify users and manage directory | Who needs access? | Azure RBAC | Authentication, not authorization |
| **User/Group** | Organize users for RBAC | Bulk access needed | Individual RBAC | Use groups, not individual assignments |
| **RBAC** | Authorize access to resources | Can this user access that resource? | Policy | Identity-based, scope matters |
| **Owner** | Full access including role assignment | Need full control | Contributor | Can grant access to others |
| **Contributor** | Modify resources but not access | Need resource mgmt no role grant | Owner | Cannot assign RBAC roles |
| **Scope hierarchy** | Where role applies | At what level? | Inheritance | Inherited by lower scopes |
| **Policy** | Enforce configuration compliance | All resources must follow rules | RBAC | Preventive, not access control |
| **Lock** | Prevent modification/deletion | Protect from accidents | Policy | Absolute, no exemption |
| **Tag** | Organize and track resources | Need to categorize | Policy | Not enforced by default |
| **Subscription** | Billing boundary | Separate billing/teams | Resource Group | Independent quotas |
| **Resource Group** | Lifecycle container | Organize related resources | Management Group | Delete RG = delete all contents |
| **Management Group** | Organize subscriptions | Bulk policy/role assignment | Subscription | Apply policy/roles across subs |
| **Budget** | Spending alert | Set spending limit | Lock | Alerts only, doesn't stop |
| **Azure Advisor** | Recommendations | Optimize costs/security | Policy | Suggests, doesn't enforce |

---

## 30-Second Recall

**Identity Layer:**
- Entra ID = directory + authentication
- Users/Groups organized in Entra
- RBAC = Principal + Role + Scope

**Authorization:**
- Owner = full control + grant access
- Contributor = modify resources, cannot grant access
- Specific roles = limited to one service

**Governance:**
- Policy = enforce or assess configuration (applicability can include exclusions, exemptions, and conditions)
- Lock = control-plane protection; remove it with lock-management permission
- Tags = metadata for organization (not enforced)

**Organization:**
- Subscription = billing boundary
- Resource Group = lifecycle container
- Management Group = organize subscriptions

**Cost:**
- Budgets = spending alerts
- Tags = cost tracking organization
- Azure Advisor = optimization recommendations

**Key Distinctions:**
- Entra ID = identity; RBAC = authorization
- RBAC = who; Policy = compliance
- Policy = can be exempted; Lock = can be removed only with lock-management permission
- Scope behavior is service-specific: RBAC, Policy, and locks can apply downward; tags do not automatically inherit
- Tags don't enforce = need policy for enforcement

---

## Official Sources

- [Azure RBAC Documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/)
- [Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/active-directory/)
- [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/)
- [Management Groups](https://learn.microsoft.com/en-us/azure/governance/management-groups/)
- [Azure Resource Locks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)
- [Subscriptions Overview](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/create-subscription)
