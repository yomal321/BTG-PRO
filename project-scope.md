# Project Initiation — BTG DevOps

## 1. Purpose

This document formally initiates the BTG DevOps project.

The purpose of the project initiation phase is to ensure that all stakeholders have a shared understanding of what will be built, how it will be built, who is responsible, and what must be completed before development begins.

---

## 2. Project Summary

BTG DevOps is a DevOps intelligence and infrastructure management project for Bistec Global.

The project focuses on cloud infrastructure auditing, anomaly detection, best-practice analysis, infrastructure drift detection, cost optimization, and operational runbook documentation.

The initial focus is Azure-based infrastructure analysis. AWS and GCP support can be considered later after the project owner confirms the wider cloud scope.

---

## 3. Initiation Objectives

The objectives of the initiation phase are:

- Document the full project scope.
- Design the high-level system architecture.
- Create and validate the full Jira task list.
- Confirm project deliverables and boundaries.
- Identify constraints, dependencies, and risks.
- Make the key project documents available in Confluence.
- Get review and approval from the mentor or project owner before development begins.

---

## 4. Initiation Deliverables

| Deliverable | Description | Status |
|---|---|---|
| Project Scope Document | Defines objectives, deliverables, boundaries, constraints, success criteria, and team roles | Prepared |
| Architecture Design Document | Defines CLI, Azure integrations, AI analysis, IaC, UI dashboard, and component relationships | Prepared |
| Full Task List | Breaks the project into epics, stories, tasks, sub-tasks, priorities, owners, estimates, and dependencies | Prepared |
| Confluence Documentation | Makes all initiation documents accessible to stakeholders | Pending upload/link |
| Jira Linkage | Links all initiation documents to the parent Jira story | Pending |

---

## 5. Linked Documents

| Document | File Path |
|---|---|
| Project Scope | `docs/project-scope.md` |
| Architecture Design | `docs/architecture-design.md` |
| Full Task List | `docs/full-task-list.md` |

---

## 6. High-Level Architecture Summary

BTG DevOps will include the following layers:

| Layer | Description |
|---|---|
| CLI Layer | Go-based CLI commands for audit, cost, drift, and runbook workflows |
| Azure Integration Layer | Read-only Azure infrastructure checks for IAM/RBAC, Storage, NSG, Key Vault, Cosmos DB, Functions, Public IPs, and related services |
| AI Analysis Layer | Uses `claude -m` CLI for reasoning-heavy analysis and OpenRouter for bulk log scanning |
| IaC Layer | Terraform/OpenTofu templates and drift detection workflows |
| UI Dashboard Layer | React + Vite dashboard for infrastructure health, anomalies, cost trends, drift detection, runbooks, audit history, and resume work |
| Documentation Layer | Markdown README, runbooks, project documents, findings, and investigation notes |
| Notification Layer | Slack integration for audit summaries and anomaly notifications |

---

## 7. Work Tracks

| Track | Owner | Responsibility |
|---|---|---|
| CLI / Backend Track | Dev A | Go CLI, audit automation, CI workflow, tests, JSON output schemas |
| Dashboard / Adoption Track | Dev B | React dashboard, UI panels, runbook library, BISTEC adoption |
| Review / Approval Track | Mentor / Project Owner | Scope review, architecture review, task validation, approval |

---

## 8. Key Dependencies

| Dependency | Reason |
|---|---|
| GitHub repo access | Required for commits, branches, and pull requests |
| Azure read-only credentials | Required to run real infrastructure audits |
| Cloud scope confirmation | Required to confirm Azure/AWS/GCP boundaries |
| JSON schema agreement | Required before dashboard panels consume CLI output |
| Claude CLI access | Required for AI-assisted audit analysis |
| OpenRouter API key | Required for bulk log scanning |
| Slack access/webhook | Required for audit notifications |
| Project owner approval | Required before major development starts |

---

## 9. Acceptance Criteria Mapping

| Acceptance Criteria | Evidence |
|---|---|
| Scope document written, reviewed, and approved | `docs/project-scope.md` |
| Architecture design documented with all layers and component relationships | `docs/architecture-design.md` |
| Full task list created in Jira and linked to this story | `docs/full-task-list.md` and linked Jira tasks |
| All three documents accessible in Confluence before development begins | Confluence page links to be added |

---

## 10. Next Steps

1. Upload or link the scope document in Confluence.
2. Upload or link the architecture design document in Confluence.
3. Upload or link the full task list in Confluence.
4. Link all three documents to the Jira parent story.
5. Request mentor/project owner review.
6. Start development only after approval.

---

## 11. Summary

The BTG DevOps project initiation phase prepares the project for development by documenting scope, architecture, task breakdown, ownership, dependencies, and approval process.

Development should begin only after the project scope, architecture design, and full task list are reviewed and approved.
