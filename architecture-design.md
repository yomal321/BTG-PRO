# Project Scope — BTG DevOps

## 1. Purpose

BTG DevOps is a DevOps intelligence and infrastructure management project for Bistec Global.

The project focuses on analyzing cloud infrastructure, identifying best-practice issues, detecting anomalies, supporting cost optimization, and maintaining operational documentation and runbooks.

This document defines the agreed project scope before development begins. It covers objectives, deliverables, team responsibilities, boundaries, constraints, and success criteria.

---

## 2. Project Objectives

The main objectives of BTG DevOps are:

1. Build a DevOps tooling platform to support cloud infrastructure audits.
2. Identify security, configuration, cost, and best-practice issues in cloud environments.
3. Provide CLI-based automation for audit, cost, drift, and runbook workflows.
4. Provide a dashboard for infrastructure health, anomaly visibility, cost trends, drift detection, runbook access, and audit history.
5. Support Bistec Global’s internal Azure adoption by running regular infrastructure reviews.
6. Improve operational documentation through Markdown-based runbooks and findings reports.
7. Maintain least-privilege access and safe infrastructure review practices.

---

## 3. Success Metrics

| Success Metric | Target |
|---|---|
| Weekly infrastructure audit | Weekly audit process is live for Bistec Azure subscription |
| Incident resolution | At least 3 infrastructure incidents resolved using BTG DevOps tooling by Q3 2026 |
| AI tooling cost | Routine AI-assisted audit cost remains below $5/month |
| Documentation quality | README, project scope, runbooks, and findings are maintained in Markdown |
| Operational adoption | Bistec infrastructure team can use the tool for audit and review activities |
| Security practice | All cloud credentials follow least-privilege access principles |

---

## 4. In-Scope Deliverables

### 4.1 CLI Deliverables

| Command | Purpose |
|---|---|
| `btg-ops audit` | Run best-practice checks against cloud infrastructure |
| `btg-ops cost` | Analyze cloud cost data and generate savings recommendations |
| `btg-ops drift` | Detect differences between Infrastructure as Code and live cloud configuration |
| `btg-ops runbook` | Support runbook-related operational workflows |

### 4.2 Cloud Infrastructure Analysis

The project will focus on cloud infrastructure checks, mainly for Azure in the initial phase.

In-scope analysis areas include:

- Azure IAM / RBAC
- Azure Storage Accounts
- Azure Network Security Groups
- Azure Key Vault
- Azure Cosmos DB
- Azure Functions
- Azure Container Registry
- Azure Public IPs
- Azure Resource Groups
- App Services and App Service Plans
- General cloud best-practice checks
- Configuration anomaly detection
- Infrastructure drift checks

### 4.3 React Dashboard

A React + Vite dashboard is in scope.

The dashboard should include the following 7 panels:

| Panel | Purpose |
|---|---|
| Infrastructure Map | Show resource groups, services, dependencies, and health status |
| Anomaly Feed | Show detected anomalies with severity and suggested fixes |
| Cost Trends | Show monthly spend by service and environment |
| Drift Detector | Show IaC vs live infrastructure drift status |
| Runbook Library | Provide searchable operational runbooks |
| Audit History | Show previous audit results and summary reports |
| Resume Work | Allow users to continue in-progress audits or reports |

### 4.4 Bistec Azure Adoption

In-scope adoption activities include:

- Weekly automated audit of the Bistec Azure subscription.
- Posting audit findings to the DevOps communication channel.
- Monthly review of audit quality with the infrastructure team.
- Training session for the infrastructure team.
- Documenting findings and resolution steps.

### 4.5 Slack Integration

Slack integration is in scope for posting audit findings and critical summaries to the DevOps team channel.

Initial integration should focus on:

- Weekly audit summary notifications.
- Critical anomaly notifications.
- Links to reports or runbooks.

### 4.6 Documentation and Runbooks

The following documentation deliverables are in scope:

- Project scope document.
- README updates.
- Infrastructure topology documentation.
- Existing scripts and runbooks inventory.
- Markdown runbooks.
- Audit findings reports.
- Anomaly investigation notes.
- Setup and usage documentation.

---

## 5. Out-of-Scope Items

The following items are not part of the immediate scope:

- Production infrastructure changes without review.
- Automatic infrastructure fixes without human approval.
- Full multi-cloud cost comparison in the initial phase.
- Complete incident management platform replacement.
- Full SLA/SLO monitoring implementation in the first phase.
- Advanced alerting pipelines in the first phase.
- Direct use of Anthropic SDK.
- Unapproved access to production credentials.
- Write-level cloud operations without review and approval.

---

## 6. Deferred / P3 Backlog Items

| Deferred Item | Reason |
|---|---|
| Automated alerting pipeline for critical anomalies | Requires stable audit rules and notification design |
| SLA/SLO monitoring dashboards | Requires finalized service ownership and monitoring targets |
| Incident management integration | Requires decision on Jira/Linear or other tool integration |
| Multi-cloud cost comparison | Azure is the primary initial focus |
| AI-generated runbooks from incident post-mortems | Requires incident data and review process |
| Advanced remediation automation | Requires approval workflow and safety controls |

---

## 7. Team Roles and Responsibilities

| Role | Responsibility |
|---|---|
| Dev A | CLI tooling, backend automation, audit workflows, Terraform/OpenTofu templates, CI pipelines |
| Dev B | React dashboard, cost analytics UI, runbook library, Bistec adoption support |
| Mentor / Project Owner | Scope approval, technical guidance, infrastructure access confirmation, review and merge approval |
| Infrastructure Team | Provide cloud environment details, review findings, confirm remediation actions |

---

## 8. Key Constraints

The project must follow these constraints:

1. Azure is the primary cloud environment for the initial phase.
2. AWS and GCP scope must be confirmed before implementation.
3. AI analysis must use `claude -m` CLI or OpenRouter REST.
4. No Anthropic SDK should be used.
5. OpenRouter should be used for bulk log scanning and low-cost analysis.
6. Routine AI usage must remain below $5/month.
7. Cloud credentials must follow least-privilege access.
8. Infrastructure changes must be reviewed before applying to production.
9. Runbooks and operational documentation must be written in Markdown.
10. All findings and anomaly investigations must include resolution notes.
11. GitHub changes should be reviewed through pull requests.
12. Developer access depends on GitHub repo access, cloud credentials, and required monitoring tools.

---

## 9. Assumptions

- Azure is the main cloud provider.
- Bistec Global infrastructure details will be clarified by the project owner or operator.
- The team will use GitHub for version control and pull request review.
- The CLI will be the first major automation interface.
- The dashboard will be developed after the core audit workflows are understood.
- Weekly audits will be used as the initial operational cadence.
- Documentation will be updated as the project scope becomes clearer.

---

## 10. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Cloud scope is unclear | Development may target wrong services | Confirm Azure/AWS/GCP scope with project owner |
| Limited repo access | Developer cannot push changes directly | Use fork and pull request or request contributor access |
| Missing credentials | Audits cannot run against real infrastructure | Use mock/test data until access is provided |
| AI cost exceeds target | Routine audits become expensive | Use OpenRouter for bulk tasks and keep Claude for reasoning-heavy tasks |
| Poor documentation | New developers cannot understand the project | Maintain README, runbooks, and scope documents |
| Unsafe remediation | Production infrastructure may be affected | Require review before applying infrastructure changes |

---

## 11. Approval Criteria

This scope document should be reviewed and approved by the project owner before major development starts.

Approval should confirm:

- Cloud providers in scope.
- Azure services in scope.
- CLI deliverables.
- Dashboard deliverables.
- Team responsibilities.
- Success metrics.
- Out-of-scope items.
- Constraints and access requirements.

---

## 12. Summary

BTG DevOps is intended to become an internal DevOps intelligence and automation platform for Bistec Global.

The initial focus is to understand the existing repository, document the project scope, clarify cloud infrastructure coverage, maintain runbooks, and begin building CLI-based audit automation.

Once the scope is approved, the team can continue with CLI implementation, dashboard development, CI improvements, audit scheduling, and Bistec Azure adoption.
