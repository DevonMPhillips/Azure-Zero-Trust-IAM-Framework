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
