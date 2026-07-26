# Zero-Trust-Architecture-in-Microsoft-Azure
A hands-on cloud security project focused on implementing IAM best practices in Azure. The project covers user and group management, RBAC, MFA, access policies, and secure permission design following the principle of least privilege.


1. Create Departmental Security Groups: Microsoft Entra ID.

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

2. Provision Cloud-Only Users: Microsoft Entra ID.

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

Step 1: Create Departmental Resource GroupsFirst, we need "containers" to hold the future cloud resources for each department. 

Navigate to Resource groups in the Azure Portal.

Click Create.

Select your Subscription and use the following naming convention to create three distinct groups:
- RG-IT-Core
- RG-Engineering-Dev
- RG-Finance-DataChoose a region closest to you and click Review + create, then Create.

Step 2: Assign Built-In Roles at the Resource Group ScopeNow we map the Security Groups to the Resource Groups.

Navigate to the newly created RG-Engineering-Dev resource group.

Click Access control (IAM) in the left menu.

Click Add $\rightarrow$ Add role assignment.

Role: Search for and select Virtual Machine Contributor (this allows managing VMs but not changing their access controls). Click Next.

Members: Select User, group, or service principal. Click Select members and search for SG-Engineering-Devs.

Click Review + assign.

Repeat this process for Finance: Navigate to RG-Finance-Data, go to Access control (IAM), and assign the Reader role to SG-Finance-Users.

Step 3: Create a Custom RBAC RoleBuilt-in roles sometimes grant too much access. Let's create a custom role for the IT team that only allows them to restart virtual machines, nothing else.

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
