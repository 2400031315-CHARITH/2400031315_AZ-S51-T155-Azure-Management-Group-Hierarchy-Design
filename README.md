# 2400031315_AZ-S51-T155-Azure Management Group Hierarchy Design
Designing an Azure Management Group (MG) hierarchy requires balancing organizational alignment with governance guardrails. When designing for scale, Microsoft Cloud Adoption Framework (CAF) guidance prioritizes archetype-based or environment-based boundaries over purely organizational ones, though departmental splits are common in enterprise structures.
4-Department Azure Hierarchy Architecture
To support four departments (e.g., HR, Finance, Engineering, Operations) while adhering to Azure governance best practices, avoid placing departments directly under the Tenant Root Group. Instead, group them under a dedicated Landing Zones archetype layer.
<img width="962" height="481" alt="image" src="https://github.com/user-attachments/assets/dba15f40-4419-4f84-9937-b997bd38da26" />
Hierarchy Breakdown & Subscriptions
<img width="736" height="721" alt="image" src="https://github.com/user-attachments/assets/4ce4af48-f71b-4ad1-bd4e-17f61144269b" />
Policy Inheritance Implementation
Azure Policy applies additive inheritance downward: a policy assigned at a parent level automatically applies to all child management groups, subscriptions, resource groups, and resources.
<img width="1102" height="334" alt="image" src="https://github.com/user-attachments/assets/2f61d57c-a66a-4adf-a004-2ccc1802897d" />
Strategic Assignment Steps
Top-Level Baseline (Tenant Root Group):
Policy: Allowed Locations restricted to approved cloud regions.
Policy: Require Security Contact Email for Microsoft Defender.
Archetype Baseline (Landing Zones MG):
Policy: Require Tag and Value (e.g., CostCenter, Owner).
Policy: Audit Unencrypted Storage Accounts.
Department-Specific Rules (Engineering MG):
Policy: Allowed Virtual Machine SKUs (restricting GPU/high-cost SKUs to specific tiers).
Operations Rules (Operations MG):
Policy: Enforce Backup Configuration on all managed disks.
Student Bottlenecks & Governance Pitfalls
Pitfall 1: Disruptive Structural Changes Post-Assignment
Once management groups have RBAC roles, Azure Policy assignments, and Defender plans attached, moving subscriptions or restructuring nodes is highly disruptive.
The Problem:
Moving a subscription from Finance MG to Decommissioned MG immediately removes inherited policies and access permissions, potentially leaving resources exposed or breaking operational scripts.
Mitigation Strategy:
Design hierarchy by archetype and life cycle (e.g., Platform vs. Workload, Prod vs. Non-Prod) rather than volatile internal org charts.
Use Infrastructure as Code (IaC) (Terraform modules or Bicep) to define MG trees so changes can be planned and dry-run before deployment.
Pitfall 2: Unexpected Effective Policy Results
Students often expect lower-level policy assignments to overwrite parent policies. In Azure Policy, Deny rules at a parent level always win, and policy evaluations are evaluated as a logical AND.
The Problem:
f Tenant Root MG assigns a policy restricting VM sizes to Standard_B2s via Deny, assigning an Allowed SKUs policy at Engineering MG that includes Standard_D4s_v3 will not grant access to the D4s size. The parent Deny block blocks it.
Mitigation Strategy:
Use Policy Exemptions (Waiver or Mitigated) at the child scope rather than attempting to override parent rules with duplicate policy assignments.
Prefer Audit or AuditIfNotExists over hard Deny during initial governance rollout to observe impact via Azure Policy Compliance dashboards without blocking workloads.
Use Policy Initiatives (Definitions Sets) to bundle related compliance rules together, making tracking effective policy inheritance simpler across levels.
