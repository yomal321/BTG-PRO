# Architecture Design — BTG DevOps

## 1. Purpose

This document describes the proposed architecture for the BTG DevOps project.

The architecture explains the major system layers, components, data flow, integrations, and responsibilities. It is intended to support the project initiation phase and guide development planning.

---

## 2. Architecture Goals

The architecture should support:

- Cloud infrastructure audit automation.
- Best-practice and security checks.
- Cost and drift analysis workflows.
- AI-assisted infrastructure review.
- Dashboard-based visibility for DevOps stakeholders.
- Markdown-based runbooks and operational documentation.
- Safe, least-privilege access to cloud resources.
- Clear separation between CLI/backend and dashboard/frontend tracks.

---

## 3. High-Level Architecture

```text
+----------------------+
|  DevOps User / Team  |
+----------+-----------+
           |
           v
+----------------------+
|   BTG DevOps CLI     |
| audit | cost | drift |
| runbook              |
+----------+-----------+
           |
           v
+----------------------+
| Azure Integration    |
| IAM | Storage | NSG  |
| Key Vault | CosmosDB |
| Functions | ACR      |
+----------+-----------+
           |
           v
+----------------------+
| Analysis & Findings  |
| Rules | AI | Reports |
+----------+-----------+
           |
           +--------------------+
           |                    |
           v                    v
+----------------------+  +----------------------+
| Markdown Runbooks    |  | React Dashboard      |
| Findings Reports     |  | Health | Cost | Drift|
+----------+-----------+  +----------+-----------+
           |                    |
           v                    v
+----------------------+  +----------------------+
| GitHub / Confluence  |  | Slack Notifications  |
+----------------------+  +----------------------+
```

---

## 4. Architecture Layers

### 4.1 CLI Layer

The CLI layer is the main automation interface.

Planned commands:

| Command | Responsibility |
|---|---|
| `btg-ops audit` | Run best-practice checks and produce audit findings |
| `btg-ops cost` | Pull cloud cost data and generate savings recommendations |
| `btg-ops drift` | Compare IaC state against live cloud configuration |
| `btg-ops runbook` | Support operational runbook workflows |

The CLI should produce structured outputs, preferably JSON, so that dashboard panels and integrations can consume the same data.

---

### 4.2 Azure Integration Layer

Azure is the primary initial cloud provider.

The integration layer will connect to Azure services using least-privilege credentials. In early phases, read-only access should be used for audit checks.

Initial Azure services in scope:

- IAM / RBAC
- Storage Accounts
- Network Security Groups
- Key Vault
- Cosmos DB
- Azure Functions
- Azure Container Registry
- Public IPs
- Resource Groups
- App Services
- App Service Plans

---

### 4.3 AI Analysis Layer

AI support will be used for infrastructure analysis and anomaly investigation.

Approved AI usage:

| Tool | Purpose |
|---|---|
| `claude -m` CLI | Reasoning-heavy audit analysis and infrastructure review |
| OpenRouter REST | Bulk log scanning and low-cost anomaly analysis |

Important constraint:

- No Anthropic SDK should be used.
- Routine AI usage should remain below $5/month.

---

### 4.4 IaC Layer

The IaC layer will support Terraform/OpenTofu templates and drift detection workflows.

Responsibilities:

- Store reusable infrastructure templates.
- Compare expected IaC configuration with live cloud infrastructure.
- Detect drift between repository-defined infrastructure and deployed resources.
- Avoid applying production changes without review and approval.

---

### 4.5 Dashboard Layer

The dashboard will be built using React + Vite.

Planned panels:

| Panel | Description |
|---|---|
| Infrastructure Map | Shows resource groups, services, dependencies, and health |
| Anomaly Feed | Shows detected anomalies, severity, details, and suggested fixes |
| Cost Trends | Shows monthly spend by service and environment |
| Drift Detector | Shows IaC vs live infrastructure status |
| Runbook Library | Provides searchable operational runbooks |
| Audit History | Shows previous audit results and summaries |
| Resume Work | Allows users to continue in-progress audits or reports |

---

### 4.6 Documentation Layer

Documentation will be stored in Markdown and versioned in GitHub.

Documentation includes:

- README files.
- Project initiation documents.
- Project scope document.
- Architecture design document.
- Full task list.
- Runbooks.
- Audit findings.
- Anomaly investigation notes.

---

### 4.7 Notification Layer

Slack integration will be used to send audit summaries and important findings to the DevOps team.

Initial notification types:

- Weekly audit summary.
- Critical anomaly notification.
- Link to detailed report or runbook.

---

## 5. Component Relationships

| Component | Connects To | Purpose |
|---|---|---|
| DevOps User | CLI / Dashboard | Runs audits and reviews findings |
| CLI | Azure | Reads infrastructure configuration |
| CLI | AI Analysis | Sends audit context for deeper analysis |
| CLI | JSON Output | Produces data for reports and dashboard |
| JSON Output | Dashboard | Powers live panels |
| CLI | Markdown Reports | Stores findings and runbooks |
| Dashboard | JSON Output | Displays audit, anomaly, cost, and drift information |
| Notification Layer | Slack | Sends audit summaries and alerts |
| GitHub | Documentation | Versions runbooks, reports, and project docs |
| Confluence | Documentation | Makes documents accessible to stakeholders |

---

## 6. Data Flow

### 6.1 Audit Flow

1. User runs `btg-ops audit`.
2. CLI authenticates with Azure using least-privilege credentials.
3. CLI collects cloud resource configuration.
4. CLI applies audit rules and best-practice checks.
5. Optional AI analysis is performed using `claude -m`.
6. Findings are written as JSON and Markdown report.
7. Dashboard consumes JSON result.
8. Slack summary is sent if configured.
9. Findings are reviewed by the DevOps team.

### 6.2 Cost Flow

1. User runs `btg-ops cost`.
2. CLI collects cloud cost data.
3. CLI groups cost by service, resource group, and environment.
4. AI or rule-based logic identifies possible savings.
5. Cost summary is exported as JSON.
6. Dashboard displays cost trends.

### 6.3 Drift Flow

1. User runs `btg-ops drift`.
2. CLI reads Terraform/OpenTofu desired state.
3. CLI compares desired state with live cloud configuration.
4. Drift findings are generated.
5. Dashboard displays drift status.

### 6.4 Runbook Flow

1. User runs or opens a runbook workflow.
2. CLI or dashboard displays relevant Markdown runbook.
3. Operator follows documented steps.
4. Findings and resolution steps are added back to the repository.

---

## 7. JSON Schema Dependency

Before the dashboard uses live data, Dev A and Dev B must agree on JSON output schemas.

Required schemas:

1. Audit result schema.
2. Anomaly finding schema.
3. Cost summary schema.
4. Drift detection schema.
5. Runbook metadata schema.

Example audit result format:

```json
{
  "audit_id": "audit-001",
  "timestamp": "2026-05-24T10:00:00Z",
  "cloud_provider": "azure",
  "scope": "bistec-subscription",
  "summary": {
    "total_findings": 10,
    "critical": 1,
    "high": 3,
    "medium": 4,
    "low": 2
  },
  "findings": [
    {
      "id": "IAM-001",
      "service": "IAM/RBAC",
      "severity": "high",
      "title": "Over-permissioned role assignment",
      "description": "A principal has broad access at subscription scope.",
      "recommendation": "Apply least-privilege role assignment."
    }
  ]
}
```

---

## 8. Security Considerations

The architecture must follow these security rules:

- Use least-privilege credentials.
- Prefer read-only access for audit tasks.
- Do not apply production infrastructure changes without review.
- Do not commit secrets, API keys, or credentials.
- Store sensitive values in secure secret managers or GitHub secrets.
- Review all infrastructure remediation actions before execution.
- Keep audit logs and findings traceable.

---

## 9. CI/CD Considerations

GitHub Actions should be used for CI/CD.

Initial CI checks:

- Go build validation.
- Go test execution.
- Markdown/documentation validation if needed.
- Pull request checks before merge.

Suggested workflow file:

```text
.github/workflows/ci.yml
```

---

## 10. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Cloud scope unclear | Wrong services may be implemented | Confirm cloud scope with project owner |
| JSON schema not agreed early | Dashboard integration may be blocked | Define schemas before live panel work |
| Missing credentials | Real audits cannot run | Use mock data until access is available |
| AI cost exceeds target | Operational cost issue | Use OpenRouter for bulk work and track spend |
| Unsafe infrastructure change | Production impact | Enforce review before applying changes |
| Poor documentation | Team confusion | Maintain Markdown docs and Confluence links |

---

## 11. Summary

The BTG DevOps architecture is designed around a CLI-first approach, Azure-first cloud integration, AI-assisted analysis, Markdown documentation, and a React dashboard for visibility.

The architecture supports parallel work by Dev A and Dev B, while requiring early agreement on JSON schemas and project scope before live implementation.
