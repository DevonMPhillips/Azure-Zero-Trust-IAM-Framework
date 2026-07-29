##Full Walkthrough
Detailed technical breakdown.

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

#### 1. Create Departmental Security Groups:
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
  
<br>
<img width="956" height="617" alt="image" src="https://github.com/user-attachments/assets/e77d180d-359b-4f23-a495-662f15b63ef6" />
<br>

Why this matters: Prefixing groups with SG- (Security Group) identifies the group's purpose when searching.

#### 2. Provision Cloud-Only Users:
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

#### 3. Assign Users to Security Groups:
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

#### Step 1: Create Departmental Resource Groups
First, we need "containers" to hold the future cloud resources for each department. 

Navigate to Resource groups in the Azure Portal.

Click Create.

Select your Subscription and use the following naming convention to create three distinct groups:
- RG-IT-Core
- RG-Engineering-Dev
- RG-Finance-Data

Choose a region closest to you and click Review + create, then Create.

<br>
<img width="955" height="616" alt="image" src="https://github.com/user-attachments/assets/b70fa751-c7ee-4f5c-bd17-5ab925d80a35" />
<br>

#### Step 2: Assign Built-In Roles at the Resource Group Scope
Now we map the Security Groups to the Resource Groups.

Navigate to the newly created RG-Engineering-Dev resource group.

Click Access control (IAM) in the left menu.

Click Add $\rightarrow$ Add role assignment.

Role: Search for and select Virtual Machine Contributor (this allows managing VMs but not changing their access controls). 

Click Next.

Members: Select User, group, or service principal. Click Select members and search for SG-Engineering-Devs.

Click Review + assign.

Repeat this process for Finance: Navigate to RG-Finance-Data, go to Access control (IAM), and assign the Reader role to SG-Finance-Users.

<br>
<img width="952" height="615" alt="image" src="https://github.com/user-attachments/assets/a4665c08-07a2-458e-8a1e-16921ffee3b0" />
<br>

#### Step 3: Create a Custom RBAC Role
Built-in roles sometimes grant too much access. Let's create a custom role for the IT team that only allows them to restart virtual machines, nothing else.

Navigate back to your Subscription or a specific Resource Group, and go to Access control (IAM).

Click Add $\rightarrow$ Add custom role.

Name: Cloud VM Restart Operator.

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

<br>
<img width="952" height="612" alt="image" src="https://github.com/user-attachments/assets/58c1bb47-7342-4fd8-b4e7-61aff03b106e" />
<br>

Finally, assign this new Cloud VM Restart Operator role to the SG-IT-Admins group at the Subscription or RG-IT-Core scope.

###Verification and Testing
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

#### 1. Create a Break-Glass Account: Best Practice.
Before creating MFA policies, we must ensure you never get locked out of your tenant if MFA services go down or a policy is misconfigured.

Go to Microsoft Entra ID > Users > New user.

Name it something inconspicuous, like svc_emergency_admin@[yourtenant].onmicrosoft.com.

Generate a highly complex, 20+ character password and store it safely (offline or in a secure vault).

Assign this user the Global Administrator role.

<br>
<img width="952" height="616" alt="image" src="https://github.com/user-attachments/assets/d2354182-5cb5-4b73-b2a1-656161ac99b8" />
<br>

Crucial: We will exclude this user from all Conditional Access policies.

#### 2. Disable Security Defaults: Tenant Properties.
Azure enables "Security Defaults" on new tenants, which enforces a rigid, non-customizable MFA requirement. We must turn this off to use our own Conditional Access policies.

Navigate to Microsoft Entra ID > Properties.

Click Manage security defaults at the bottom of the page.

Set it to Disabled (not recommended).

Provide a reason (e.g., "My organization is using Conditional Access") and save.

<br>
<img width="956" height="617" alt="image" src="https://github.com/user-attachments/assets/c76250af-e09c-41a9-8ebc-3321f6430df5" />
<br>

#### 3. Policy 1: 
Require MFA for IT Admins:
Let's enforce MFA for any user in the IT Admins group.

Search for and navigate to Microsoft Entra ID Conditional Access. 

Click Create new policy. 

Name it CA-RequireMFA-ITAdmins.

Assignment: 
Include > Select users and groups > Choose SG-IT-Admins. 

Exclude > Users and groups > Select your break-glass account.

Target resources: 
Include > All resource (formerly all cloud apps')

Grant: 
Select Require multifactor authentication.

Enable policy: 
Switch from Report-only to On.

Click Create.

<br>
<img width="957" height="866" alt="image" src="https://github.com/user-attachments/assets/c1f14bb1-92b7-4298-8772-b556e6e15e51" />
<br>

#### 4. Policy 2: Block Non-US Sign-ins (Simulated):

Let's create a geographic boundary to simulate blocking foreign login attempts.

In the Conditional Access menu, go to Named locations > Countries location > Create a location named Allowed Countries and select your home country.

<br>
<img width="947" height="867" alt="image" src="https://github.com/user-attachments/assets/dfbc1f13-b7fa-45e2-ae8f-00b1f273f8c0" />
<br>

Create a new Conditional Access policy named CA-Block-Foreign-Logins. 

Assignment: 
Include > All users. Exclude > Break-glass account.

Target resources:
Include > All resource (formerly all cloud apps')

Conditions (Locations):
Include > Any location. 
Exclude > Selected locations > Choose Allowed Countries. 

Grant: Block access.
Enable policy: On, then create.

Verification and Testing

Open an incognito browser and log in as Alice IT. You should be immediately interrupted and prompted to register for Microsoft Authenticator (MFA).

Log in as Charlie Finance. Because Charlie is not in the IT group, he should bypass the MFA requirement (though in a real enterprise, we would eventually enforce MFA for everyone, perhaps based on risk level).

## Validating the Block Policy: 
To test the geographic block, you would need a VPN routing through a different country. If you attempt to log in while on a foreign VPN, Entra ID will return a specific error stating your login was blocked by Conditional Access.

## Phase 4: Securing Non-Human Identities
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

#### 1. Create an App Registration (Service Principal):

We will create an identity that the Engineering team's external GitHub deployment pipeline can use.

Navigate to Microsoft Entra ID > App registrations.

Click New registration.

Name: SP-Engineering-GitHub-Deploy.

Supported account types: Accounts in this organizational directory only (Single tenant).

Click Register.

Note the Application (client) ID and Directory (tenant) ID on the overview page. The pipeline will need these to identify itself.

<br>
<img width="936" height="867" alt="image" src="https://github.com/user-attachments/assets/1d4250a9-26f2-47ba-80f2-b41359c7667c" />
<br>

#### 2. Generate a Client Secret:

To prove it is the legitimate pipeline, the Service Principal needs a secret key.

On the SP-Engineering-GitHub-Deploy blade, click Certificates & secrets in the left menu.

Click New client secret.

Description: GitHub Actions Secret. 

Set the expiration to 6 months (best practice: keep lifetimes short).

Click Add.

CRITICAL: Copy the Value immediately. Once you navigate away from this page, Azure masks the value forever, and you would have to generate a new one.

<br>
<img width="956" height="867" alt="image" src="https://github.com/user-attachments/assets/043b66f6-d0d9-47ff-8b6c-38ea150477f6" />
<br>

#### 3. Create a Managed Identity: Azure Portal.
Now we will create a User-Assigned Managed Identity. Engineering will attach this to their Azure Virtual Machines so the VMs can securely read from an Azure Key Vault without needing passwords.

In the global top search bar, search for Managed Identities.

Click Create.

Subscription: Your subscription. 

Resource group: Select RG-Engineering-Dev.

Region: The same region as your resource group.

Name: MI-Engineering-AppServer.

Click Review + create, then Create.

<br>
<img width="976" height="862" alt="image" src="https://github.com/user-attachments/assets/ebdb800d-212d-4da6-ad6b-581d759c5cac" />
<br>

#### 4. Assign Permissions to the Workload Identities:
Right now, these identities exist, but they have no access. We must authorize them using RBAC, just like we did for human users.

Navigate to your RG-Engineering-Dev resource group.

Click Access control (IAM) > Add > Add role assignment.

Role: Select Contributor. Click Next.

Members: Choose User, group, or service principal. Click Select members.

Search for and select SP-Engineering-GitHub-Deploy.

Click Review + assign.

<br>
<img width="976" height="861" alt="image" src="https://github.com/user-attachments/assets/db493038-754c-48b1-ad2c-eed0d245e9d2" />
<br>

Repeat this process, but this time search for and assign the Reader role to the Managed Identity: MI-Engineering-AppServer.

<br>
<img width="977" height="867" alt="image" src="https://github.com/user-attachments/assets/61a3ed2d-f582-4906-9ec3-88347062d7a4" />
<br>

## Verification and Testing
Navigate to Microsoft Entra ID > Enterprise applications.

Remove the default filter (which says Application type == Enterprise Applications) and search for SP-Engineering-GitHub-Deploy. This confirms the Service Principal object was successfully instantiated in your directory.

Go back to RG-Engineering-Dev > Access control (IAM) > Role assignments. Verify that both the Service Principal (Contributor) and Managed Identity (Reader) are listed alongside your human Security Groups from Phase 2.

## Phase 5: Identity Governance, Auditing, and Privileged Access

Objectives
- Centralize identity telemetry by routing Entra ID logs to Azure Log Analytics.
- Implement Just-In-Time (JIT) access using Privileged Identity Management (PIM).
- Create an Access Review policy to audit departmental group memberships.


Business Justification
If an attacker compromises Alice IT's account, they instantly gain her administrative powers if those powers are permanent. By implementing PIM, Alice operates as a standard user most of the time. She must explicitly "activate" her admin rights, provide a justification, and pass MFA — and those rights automatically expire after a few hours. Meanwhile, Access Reviews ensure CloudNova remains compliant with security frameworks (like SOC2 or ISO 27001) by proving that access to sensitive data (like Finance) is routinely verified.

Architecture Overview
- Auditing: Entra ID generates Sign-in and Audit logs, but they only stick around for 30 days by default. We will export these to a Log Analytics Workspace for long-term retention and querying.
- PIM (Authorization): Instead of making Alice a permanent User Administrator, we make her Eligible. When she needs it, PIM grants the role temporarily.
- Governance: We will schedule an automated Access Review for the Finance group, forcing Charlie to attest that he still needs his access.

Cloud Services Used
- Microsoft Entra ID P2 (Required for PIM and Access Reviews)
- Azure Log Analytics

### 1. Create a Log Analytics Workspace: 
We need a secure vault for our identity logs. 

In the global search bar, search for Log Analytics workspaces.

Click Create.

Select your Subscription and choose the RG-IT-Core resource group.

Name: LAW-CloudNova-SecurityLogs.

Region: Select your region, then click Review + Create > Create.

<br>
<img width="960" height="866" alt="image" src="https://github.com/user-attachments/assets/4c6ebe3f-998c-4502-ac64-fbd73d71826d" />
<br>

### 2. Route Logs to Log Analytics: 
Now we tell Entra ID to send its telemetry to our new workspace.

Navigate to Microsoft Entra ID > Diagnostic settings (under the Monitoring menu).

Click Add diagnostic setting.

Name: Entra-to-LAW.

Under Categories, check AuditLogs, SignInLogs, and NonInteractiveUserSignInLogs.

Under Destination details, check Send to Log Analytics workspace and select LAW-CloudNova-SecurityLogs.

Click Save.

<br>
<img width="956" height="862" alt="image" src="https://github.com/user-attachments/assets/0886d55b-8d90-4b29-bb24-97d42a8317b6" />
<br>

### 3.Configure Just-In-Time Access (PIM): 
We will grant Alice IT eligible administrative rights. (Note: In Phase 1, we created standard groups. To use PIM on a group, it must be created as a "role-assignable" group. For this lab, we will assign PIM directly to the user, though enterprise environments use role-assignable groups).

Search for and open Microsoft Entra Privileged Identity Management.

Click Microsoft Entra roles (under Manage).

Click Roles on the left menu, search for User Administrator, and click on it.

Click Add assignments.

Select member: Search for and select Alice IT. Click Next.

Assignment type: Ensure it is set to Eligible (not Active). Set the duration for 1 year.

Click Assign.

<br>
<img width="977" height="862" alt="image" src="https://github.com/user-attachments/assets/ef23213a-45e8-4c87-b724-eefee6274f51" />
<br>

### 4. Create an Access Review:
Let's audit the Finance team to ensure no unauthorized users have slipped into the group.

Search for and open Identity Governance.

Click Access reviews > New access review.

Select what to review: Teams + Groups.

Scope: Select Teams + groups > Choose SG-Finance-Users.

Reviewers: Select Users review their own access (Self-review).

Recurrence: Monthly. 

Click Next.

Under Upon completion settings, enable Auto apply results to resource. (If Charlie doesn't respond, he loses access automatically!).

Review name: Monthly Finance Access Audit. Click Create.

### Verification and Testing
Testing PIM: Open an incognito window and log in as Alice IT. Navigate to Microsoft Entra ID > Users. Try to create a new user or reset a password. You should be blocked because she doesn't have active rights.

Search for Privileged Identity Management in Alice's portal. Go to My roles > Microsoft Entra roles. She will see User Administrator listed as Eligible. Click Activate, provide a reason ("Need to provision a new hire"), and submit.

Log out and log back in as Alice. She can now successfully manage users!

Testing Access Reviews: Log in as Charlie Finance. Go to myaccess.microsoft.com. Charlie will see a pending task asking him to review his membership in SG-Finance-Users. He can approve it and type a justification ("Still working in Finance").
