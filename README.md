# Zero-Trust-Architecture-in-Microsoft-Azure
A hands-on cloud security project focused on implementing IAM best practices in Azure. The project covers user and group management, RBAC, MFA, access policies, and secure permission design following the principle of least privilege.

Project Objective
Design and implement a scalable, secure Identity and Access Management (IAM) framework using Microsoft Entra ID (formerly Azure AD). This project establishes a Zero Trust identity perimeter, enforcing the Principle of Least Privilege (PoLP) across human and non-human identities while maintaining a frictionless user experience.

Real-World Business Scenario
CloudNova Solutions is a mid-sized software company migrating from an on-premises, legacy Active Directory environment to a cloud-native infrastructure in Microsoft Azure.

Historically, employees had overly broad access, resulting in compliance violations and security vulnerabilities. As the Lead IAM Engineer, your mandate is to build a new cloud identity architecture from scratch. You must securely onboard five distinct departments—IT, Human Resources, Finance, Marketing, and Engineering—ensuring that each team can only access the specific Azure resources and enterprise applications required for their job functions.

Identity Architecture Overview
The architecture relies on a centralized identity model using Microsoft Entra ID as the primary Identity Provider (IdP).

- Authentication is secured at the perimeter using Conditional Access Policies and Multi-Factor Authentication (MFA).
- Authorization is handled via Azure Role-Based Access Control (RBAC) applied at the Resource Group level.
- Segregation of Duties is achieved through dynamic Security Groups and Custom Roles.
- Non-Human Access for the Engineering team's deployment pipelines is managed via Service Principals and Managed Identities.
- Visibility is established by routing sign-in and audit logs to a centralized Log Analytics workspace.

## Technologies & Services

| Service Category | Azure / Microsoft Services |
|------------------|----------------------------|
| **Identity Provider (IdP)** | Microsoft Entra ID (Free & P2 Trial Features) |
| **Authorization** | Azure Role-Based Access Control (RBAC), Custom Roles |
| **Access Control** | Conditional Access Policies, Multi-Factor Authentication (MFA) |
| **Non-Human Identities** | App Registrations, Service Principals, Managed Identities |
| **Privileged Access** | Privileged Identity Management (PIM) |
| **Monitoring & Auditing** | Microsoft Entra ID Sign-in Logs, Audit Logs, Azure Log Analytics |

Project Roadmap
- Phase 1: Foundational Directory Setup
Tenant configuration, department structuring, and baseline user/group lifecycle management.

- Phase 2: Authorization & RBAC Implementation
Designing the permission model, mapping Azure Resource Groups to departmental Security Groups, and creating custom RBAC roles.

- Phase 3: Securing the Perimeter (Authentication & Access Control)
Deploying MFA, building robust Conditional Access Policies, and testing block/allow scenarios.

- Phase 4: Securing Non-Human Identities (Workloads & Apps)
Implementing Service Principals and Managed Identities for the Engineering department's automated infrastructure deployments.

- Phase 5: Identity Governance, Auditing, and Privileged Access
Deploying Log Analytics for sign-in auditing, configuring Access Reviews for the Finance team, and setting up Privileged Identity Management (PIM) for IT administrators.

- Phase 6: Portfolio Packaging & Documentation
Finalizing diagrams, compiling screenshots, and building out the GitHub repository structure.


## Phase 1: Foundational Directory Setup

### Objectives
- Establish the foundational directory structure for CloudNova Solutions.
- Create standardized departmental Security Groups.
- Provision realistic cloud-only user accounts and map them to their respective groups.

### Skills Being Learned
- Microsoft Entra ID (formerly Azure AD) administration.
- User and Group lifecycle management.
- Implementing organizational naming conventions.
- Understanding the difference between Assigned and Dynamic group memberships.

### Business Justification
Before we can secure access to cloud resources or applications, we must have a structured way to identify who is requesting access. By building a group-centric architecture from day one, CloudNova ensures that future access policies and permissions scale effortlessly as the company grows, adhering to the Principle of Least Privilege.

### Architecture Overview
We are operating in a single, cloud-native Microsoft Entra ID tenant. The identity model is flat but logically segmented using Security Groups based on departmental boundaries.

### Cloud Services Used
Microsoft Entra ID (Free Tier is sufficient for this phase)

#### 1. Create Departmental Security Groups: Microsoft Entra ID.

Groups">
Before creating users, we build the logical containers they will reside in.

Navigate to Microsoft Entra ID and select Groups from the left menu. 

Click New Group.

Group type: Security.

Membership type: Assigned (we will use manual assignment for this foundational phase).

Create the following five groups. Use a clear naming convention: 
- SG-IT-Admins
- SG-HR-Users
- SG-Finance-Users
- SG-Marketing-Users
- SG-Engineering-Devs

Why this matters: Prefixing groups with SG- (Security Group) instantly identifies the group's purpose when searching through hundreds of directory objects later.

#### 2. Provision Cloud-Only Users: Microsoft Entra ID.

Users">

Now we will create the employee identities.

Navigate to Users > All users > New user > Create new user.

Uncheck "Auto-generate password" and set a temporary password you can remember (e.g., TempPassword123!).

Create one user for each department to act as your test subjects:

## Test Users

| Name | Department | User Principal Name (UPN) |
|------|------------|---------------------------|
| Alice IT | IT | `alice.it@[yourtenant].onmicrosoft.com` |
| Bob HR | Human Resources | `bob.hr@[yourtenant].onmicrosoft.com` |
| Charlie Finance | Finance | `charlie.finance@[yourtenant].onmicrosoft.com` |
| Diana Marketing | Marketing | `diana.marketing@[yourtenant].onmicrosoft.com` |
| Evan Engineering | Engineering | `evan.engineering@[yourtenant].onmicrosoft.com` |

Assign Users to Security Groups: Microsoft Entra ID.

Groups">

Map the users to their respective departmental roles.

Navigate back to Groups and click on SG-IT-Admins.

Click Members on the left menu, then Add members.

Search for "Alice" and add her to the group.

Repeat this process to place Bob in HR, Charlie in Finance, Diana in Marketing, and Evan in Engineering.

Why this matters: When we lock down the cloud environment in Phase 2, we will apply permissions to SG-Finance-Users, not to Charlie. If Charlie leaves and is replaced by Chloe, you simply add Chloe to the group, and she instantly inherits the exact access she needs.

#### Verification and Testing
Navigate to Microsoft Entra ID > Users.

Click on Charlie Finance.

In Charlie's profile, click on Groups in the left menu. Verify that SG-Finance-Users is listed. This confirms the identity mapping is successful.

Open an incognito/private browser window, go to portal.azure.com, and attempt to log in as one of your new users to verify the credentials work. You should be prompted to update the temporary password.


## Phase 2: Authorization & RBAC Implementation

### Objectives
- Build a simulated infrastructure environment using Resource Groups.
- Apply Azure RBAC assignments to departmental Security Groups.
- Design and deploy a Custom RBAC role.
- Test and validate access boundaries between departments.

### Business Justification
If an Engineering user's account is compromised, the blast radius must be contained to Engineering resources. By scoping permissions to specific Resource Groups rather than the entire Azure Subscription, CloudNova limits lateral movement capabilities for attackers and prevents accidental infrastructure deletion by unauthorized internal staff.

### Architecture Overview
To fully grasp this configuration, it is critical to break down the architecture of an Azure RBAC role assignment. Every assignment requires three fundamental components: 
- Security Principal (Who): The entity requesting access. In our case, these are the SG- Security Groups we created in Phase 1.
- Role Definition (What): A collection of permissions (actions and not-actions). We will use Built-in roles (like Contributor or Reader) and build a Custom role.
- Scope (Where): The boundary where the access applies. The Azure hierarchy flows from Management Group $\rightarrow$ Subscription $\rightarrow$ Resource Group $\rightarrow$ Resource. We will explicitly target the Resource Group scope to enforce tight boundaries.

Critical Architectural Distinction: We are configuring Azure RBAC Roles (which govern access to Azure resources like Virtual Machines and Storage Accounts). This is distinctly different from Microsoft Entra ID Roles (which govern access to directory resources like user passwords and domain settings).

Cloud Services Used
- Azure Subscriptions
- Azure Resource Groups
- Azure RBAC (Role Assignments & Custom Roles)

#### Step 1: Create Departmental Resource GroupsFirst, we need "containers" to hold the future cloud resources for each department. 

Navigate to Resource groups in the Azure Portal.

Click Create.

Select your Subscription and use the following naming convention to create three distinct groups:
- RG-IT-Core
- RG-Engineering-Dev
- RG-Finance-DataChoose a region closest to you and click Review + create, then Create.

#### Step 2: Assign Built-In Roles at the Resource Group ScopeNow we map the Security Groups to the Resource Groups.

Navigate to the newly created RG-Engineering-Dev resource group.

Click Access control (IAM) in the left menu.

Click Add $\rightarrow$ Add role assignment.

Role: Search for and select Virtual Machine Contributor (this allows managing VMs but not changing their access controls). Click Next.

Members: Select User, group, or service principal. Click Select members and search for SG-Engineering-Devs.

Click Review + assign.

Repeat this process for Finance: Navigate to RG-Finance-Data, go to Access control (IAM), and assign the Reader role to SG-Finance-Users.

#### Step 3: Create a Custom RBAC RoleBuilt-in roles sometimes grant too much access. Let's create a custom role for the IT team that only allows them to restart virtual machines, nothing else.

Navigate back to your Subscription or a specific Resource Group, and go to Access control (IAM).

Click Add $\rightarrow$ Add custom role.

Name: CloudNova VM Restart Operator.

Baseline permissions: Start from scratch.

Go to the JSON tab and click Edit. Replace the "actions": [] array with:

JSON

```
"actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action"
]
```

Save and create the role.

Finally, assign this new CloudNova VM Restart Operator role to the SG-IT-Admins group at the Subscription or RG-IT-Core scope.

Verification and Testing

We must verify that our boundaries hold up.

Within any Resource Group's Access control (IAM) blade, use the Check access tab.

Search for Evan Engineering and review his effective access. You should see he has Virtual Machine Contributor rights exclusively over RG-Engineering-Dev.

Real-world test: Open an incognito window, log in as charlie.finance@[yourtenant], and try to view resources in RG-Engineering-Dev. Charlie should be explicitly blocked by the Azure portal (returning an "Unauthorized" or "No access" error).

## Phase 3: Securing the Perimeter (Authentication & Access Control)

### Objectives
- Configure a highly privileged "Break-Glass" emergency access account.
- Disable basic Security Defaults to unlock advanced granular policies.
- Author and deploy Conditional Access Policies to enforce MFA and block risky sign-ins.
- Validate policy enforcement using Entra ID Sign-in logs.

### Business Justification
A blanket MFA requirement can cause alert fatigue and frustrate users. Conditional Access allows CloudNova Solutions to balance security and productivity by enforcing strict controls on highly privileged users (like SG-IT-Admins) while applying context-aware rules (like blocking logins from outside the country) to standard employees.

### Architecture Overview
When a user attempts to authenticate, Entra ID intercepts the request and evaluates Signals (Who is the user? What is their IP address? What device are they using?). It runs these signals through our Conditional Access Policies, which make a Decision (Allow, Block, or Require MFA). Only if the conditions are met does the user receive a token to access the application.

### Cloud Services Used
- Microsoft Entra ID P2 (Requires a free trial or Microsoft 365 Developer tenant)
- Microsoft Authenticator App (For testing)

#### 1. Create a Break-Glass Account: Best Practice.Before creating MFA policies, we must ensure you never get locked out of your tenant if MFA services go down or a policy is misconfigured.

Go to Microsoft Entra ID > Users > New user.Name it something inconspicuous, like svc_emergency_admin@[yourtenant].onmicrosoft.com.Generate a highly complex, 20+ character password and store it safely (offline or in a secure vault).Assign this user the Global Administrator role.

Crucial: We will exclude this user from all Conditional Access policies.

#### 2. Disable Security Defaults:Tenant Properties.Azure enables "Security Defaults" on new tenants, which enforces a rigid, non-customizable MFA requirement. We must turn this off to use our own Conditional Access policies.

Navigate to Microsoft Entra ID > Properties.

Click Manage security defaults at the bottom of the page.

Set it to Disabled (not recommended).Provide a reason (e.g., "My organization is using Conditional Access") and save.

#### 3. Policy 1: Require MFA for IT Admins: Conditional Access.

Let's enforce MFA for any user in the IT Admins group.

Search for and navigate to Microsoft Entra ID Conditional Access. 

Click Create new policy. Name it CA-RequireMFA-ITAdmins.

Users: Include > Select users and groups > Choose SG-IT-Admins. Exclude > Users and groups > Select your break-glass account.

Target resources: Include > All cloud apps.

Grant: Select Require multifactor authentication.

Enable policy: Switch from Report-only to On.Click Create.

#### 4. Policy 2: Block Non-US Sign-ins (Simulated): Conditional Access. Let's create a geographic boundary to simulate blocking foreign login attempts.

In the Conditional Access menu, go to Named locations > Countries location > Create a location named Allowed Countries and select your home country.

Create a new Conditional Access policy named CA-Block-Foreign-Logins. Users: Include > All users. Exclude > Break-glass account.

Target resources: All cloud apps. 
Conditions > Location: Include > Any location. Exclude > Selected locations > Choose Allowed Countries. 
Grant: Block access.
Enable policy: On, then create.

Verification and Testing

Open an incognito browser and log in as Alice IT. You should be immediately interrupted and prompted to register for Microsoft Authenticator (MFA).

Log in as Charlie Finance. Because Charlie is not in the IT group, he should bypass the MFA requirement (though in a real enterprise, we would eventually enforce MFA for everyone, perhaps based on risk level).

## Validating the Block Policy: To test the geographic block, you would need a VPN routing through a different country. If you attempt to log in while on a foreign VPN, Entra ID will return a specific error stating your login was blocked by Conditional Access.

## Phase 4: Securing Non-Human Identities (Workloads & Apps)
Objectives
- Register a cloud application to generate a Service Principal.
- Generate and secure client secrets for workload authentication.
- Create a User-Assigned Managed Identity.
- Apply Azure RBAC to non-human identities.

### Architecture Overview
There are two primary ways to handle non-human identities in Azure:

- Service Principals (App Registrations): Think of this as a "service account" for an external application. If a pipeline sitting outside of Azure (like GitHub) needs to deploy resources into Azure, it authenticates using a Service Principal (via a Client ID and Secret, or a federated certificate).

- Managed Identities: The ultimate security best practice. If a resource is already inside Azure (like an Azure Virtual Machine), Azure can manage its identity automatically. There are no passwords or secrets for you to handle, rotate, or accidentally leak.

Cloud Services Used
- Microsoft Entra ID (App Registrations & Enterprise Applications)
- Azure Managed Identities
- Azure RBAC

#### 1. Create an App Registration (Service Principal): Microsoft Entra ID.

We will create an identity that the Engineering team's external GitHub deployment pipeline can use.

Navigate to Microsoft Entra ID > App registrations.Click New registration.

Name: SP-Engineering-GitHub-Deploy.

Supported account types: Accounts in this organizational directory only (Single tenant).

Click Register.

Note the Application (client) ID and Directory (tenant) ID on the overview page. The pipeline will need these to identify itself.

#### 2. Generate a Client Secret:Microsoft Entra ID.

To prove it is the legitimate pipeline, the Service Principal needs a secret key.

On the SP-Engineering-GitHub-Deploy blade, click Certificates & secrets in the left menu.

Click New client secret.

Description: GitHub Actions Secret. Set the expiration to 6 months (best practice: keep lifetimes short).

Click Add.

CRITICAL: Copy the Value immediately. Once you navigate away from this page, Azure masks the value forever, and you would have to generate a new one.

#### 3. Create a Managed Identity: Azure Portal.

Now we will create a User-Assigned Managed Identity. Engineering will attach this to their Azure Virtual Machines so the VMs can securely read from an Azure Key Vault without needing passwords.

In the global top search bar, search for Managed Identities.

Click Create.

Subscription: Your subscription. 

Resource group: Select RG-Engineering-Dev (created in Phase 2).

Region: The same region as your resource group.

Name: MI-Engineering-AppServer.

Click Review + create, then Create.

#### 4. Assign Permissions to the Workload Identities:

Right now, these identities exist, but they have no access. We must authorize them using RBAC, just like we did for human users.

Navigate to your RG-Engineering-Dev resource group.

Click Access control (IAM) > Add > Add role assignment.

Role: Select Contributor. Click Next.

Members: Choose User, group, or service principal. Click Select members.

Search for and select SP-Engineering-GitHub-Deploy.

Click Review + assign.Repeat this process, but this time search for and assign the Reader role to the Managed Identity: MI-Engineering-AppServer.

## Verification and Testing
Navigate to Microsoft Entra ID > Enterprise applications.

Remove the default filter (which says Application type == Enterprise Applications) and search for SP-Engineering-GitHub-Deploy. This confirms the Service Principal object was successfully instantiated in your directory.

Go back to RG-Engineering-Dev > Access control (IAM) > Role assignments. Verify that both the Service Principal (Contributor) and Managed Identity (Reader) are listed alongside your human Security Groups from Phase 2.
