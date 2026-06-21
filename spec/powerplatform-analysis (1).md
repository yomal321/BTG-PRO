# Power Platform Analysis Spec — btg-devops

## 1. Problem Statement

`btg-devops` currently analyses Azure infrastructure resources such as App Services, Network Security Groups, Key Vaults, Storage Accounts, IAM/RBAC, Cosmos DB, Public IPs, and related Azure resources.

However, BISTEC teams also use Microsoft Power Platform services such as Power Apps, Power Automate, Dataverse, and Power BI. At the moment, `btg-devops` has no visibility into Power Platform resources.

This creates several problems:

- Unused Power Apps may continue consuming licences.
- Power Apps may be overshared with too many users.
- Power Automate flows may be failing, suspended, orphaned, or misconfigured.
- Dataverse capacity usage may become high without early visibility.
- Production environments may be missing Data Loss Prevention policies.
- Power BI datasets may have stale or failed refreshes.
- DevOps engineers and architects do not have one consolidated view of Azure and Power Platform health.

Because of this, issues may only become visible after something breaks, after a licence cost increases, or during a manual compliance review.

### Affected Users

| User | Problem |
|---|---|
| DevOps Engineer | No automated CLI check for Power Platform resources |
| Architect | No central compliance and governance visibility |
| Team Lead | No easy way to monitor Power Automate flow health |
| Finance / Admin | Licence waste and unused resources are difficult to identify |

---

## 2. Goals and Non-Goals

### Goals

1. Add a new CLI command: `btg-devops analyze powerplatform`.
2. Analyse Power Platform resources for security, licence waste, idle resources, DLP gaps, and health issues.
3. Keep the command consistent with the existing `btg-devops analyze [module]` pattern.
4. Reuse the existing Go CLI architecture.
5. Classify findings using the existing severity model: Critical, Warning, and Info.
6. Support table output and JSON output.
7. Prepare the findings so they can be displayed in a dashboard or Grafana later.

### Non-Goals

1. No automatic remediation or auto-fix.
2. No Dataverse business data/content inspection.
3. No multi-tenant Power Platform analysis.
4. No real-time alerting.
5. No write or manage permissions.
6. No custom Power Platform management portal in Phase 1.

This feature is read-only analysis only.

---

## 3. Users / Personas

| Persona | Role | Needs |
|---|---|---|
| DevOps Engineer | Runs `btg-devops` CLI | Same CLI experience as existing Azure commands and JSON output for scripting |
| Architect | Reviews security and governance | Visibility into DLP gaps, overshared apps, and unused environments |
| Team Lead | Monitors team automation | Know which flows are failing, suspended, or orphaned |
| Finance / Admin | Reviews licence usage | Identify idle apps, unused flows, and licence waste |

---

## 4. Proposed Solution

Add Power Platform analysis as a new module inside the existing `btg-devops` tool.

```bash
btg-devops analyze powerplatform
```

Optional component filtering:

```bash
btg-devops analyze powerplatform --component apps
btg-devops analyze powerplatform --component flows
btg-devops analyze powerplatform --component dlp
btg-devops analyze powerplatform --component powerbi
btg-devops analyze powerplatform --environment-id <environment-id>
btg-devops analyze powerplatform --output json
```

The command will authenticate using a Service Principal, call Power Platform and Power BI APIs, collect metadata, apply analysis rules, and return structured findings.

---

## 5. Main Components

| Component | Responsibility |
|---|---|
| Go CLI Extension | Adds the new `analyze powerplatform` command |
| Power Platform Analyzer | Applies rules for apps, flows, DLP, environments, Dataverse metadata, and Power BI health |
| Power Platform Admin API | Provides environment, app, flow, Dataverse, and DLP metadata |
| Power BI REST API | Provides workspace, dataset, report, and refresh health data |
| Output Formatter | Reuses existing table and JSON output pattern |
| Dashboard / Grafana | Future visualisation layer for findings |

---

## 6. Analysis Checks

### 6.1 Power Apps Checks

| Check | Severity | Description |
|---|---|---|
| App shared with Everyone | Critical | App is overshared and may expose business data |
| App has zero launches in 30 days | Warning | App may be unused and wasting licences |
| App has no clear owner | Warning | App may become unmaintained |
| App uses risky connectors | Warning | App may connect to uncontrolled external services |
| App is active and properly owned | Info | App appears healthy |

### 6.2 Power Automate Checks

| Check | Severity | Description |
|---|---|---|
| Flow is failed or suspended | Critical | Business automation may not be working |
| Flow has no owner | Warning | Flow may be orphaned |
| Flow uses deprecated connector | Warning | Flow may break in future |
| Flow has repeated failures | Warning | Flow is unstable |
| Flow is active and healthy | Info | Flow appears healthy |

### 6.3 DLP Policy Checks

| Check | Severity | Description |
|---|---|---|
| Production environment has no DLP policy | Critical | High governance and data leakage risk |
| Environment has weak DLP configuration | Warning | Connectors may not be controlled properly |
| DLP policy exists and is assigned | Info | Environment has a governance policy |

### 6.4 Dataverse Checks — Phase 2

| Check | Severity | Description |
|---|---|---|
| Capacity usage above 95% | Critical | Environment may stop accepting data soon |
| Capacity usage above 80% | Warning | Capacity needs review |
| Environment has no apps | Warning | Environment may be idle |
| Capacity usage is healthy | Info | Environment capacity is within normal range |

### 6.5 Power BI Checks — Phase 2

| Check | Severity | Description |
|---|---|---|
| Dataset refresh failed repeatedly | Critical | Reports may show outdated or incorrect data |
| Dataset refresh stale for more than 7 days | Warning | Dataset may not be updated |
| Workspace has no activity in 30 days | Warning | Workspace may be unused |
| Report shared externally | Critical | Possible data exposure risk |
| Dataset refresh healthy | Info | Dataset appears healthy |

---

## 7. High-Level Workflow

```text
Engineer runs command
        ↓
btg-devops analyze powerplatform
        ↓
Authenticate using Service Principal
        ↓
Call Power Platform Admin API
        ↓
Call Power BI REST API where required
        ↓
Collect environments, apps, flows, DLP policies, and dataset metadata
        ↓
Apply analysis rules
        ↓
Classify findings as Critical / Warning / Info
        ↓
Render output as table or JSON
        ↓
Dashboard / Grafana can use JSON output later
```

---

## 8. Authentication Model

The feature should use Service Principal authentication.

This is required because `btg-devops` is a CLI tool and may run unattended in CI jobs, scheduled jobs, or automation environments. It should not depend on an interactive user login.

### Required Environment Variables

```bash
AZURE_TENANT_ID=
AZURE_CLIENT_ID=
AZURE_CLIENT_SECRET=
AZURE_SUBSCRIPTION_ID=
```

Additional Power Platform / Power BI scopes may be required depending on implementation.

### Binding Constraint

Before implementation, the team must confirm whether the BISTEC tenant allows app permission consent for Power Platform read-only analysis.

If tenant policy blocks the required application permissions, Phase 1 cannot proceed until an admin grants the required consent.

---

## 9. Required APIs and Permissions

| Data Needed | API / Source | Permission / Scope |
|---|---|---|
| Power Platform environments | Power Platform Admin API | Power Platform read/admin permission |
| Power Apps metadata | Power Apps API | Power Platform read permission |
| Power Automate flows | Power Automate / Flow API | Flow read permission |
| DLP policies | Power Platform Governance API | Admin read permission |
| Dataverse capacity | Power Platform Admin Center API | Admin read permission |
| Power BI workspaces | Power BI REST API | Power BI read permission |
| Dataset refresh history | Power BI REST API | Dataset read permission |

Permissions must be read-only. No write/manage permission should be requested for this phase.

---

## 10. Output Format

The command should support the existing output style used by `btg-devops`.

### Table Output

```bash
btg-devops analyze powerplatform
```

Example table columns:

| Resource | Component | Severity | Issue | Recommendation |
|---|---|---|---|---|
| Leave Request App | Power Apps | Critical | Shared with Everyone | Restrict sharing to required users/groups |
| Invoice Flow | Power Automate | Warning | Flow failed multiple times | Review run history and fix connector/auth issue |
| Production Env | DLP | Critical | No DLP policy assigned | Assign approved DLP policy |

### JSON Output

```bash
btg-devops analyze powerplatform --output json
```

Example JSON:

```json
[
  {
    "resourceName": "Leave Request App",
    "component": "Power Apps",
    "severity": "Critical",
    "issue": "App is shared with Everyone",
    "recommendation": "Restrict app sharing to required users or security groups"
  },
  {
    "resourceName": "Invoice Approval Flow",
    "component": "Power Automate",
    "severity": "Warning",
    "issue": "Flow has repeated failures",
    "recommendation": "Review flow run history and fix failing action"
  }
]
```

---

## 11. Dashboard Approach

The dashboard approach must be confirmed before implementation.

### Option A — Existing Dashboard

Power Platform findings are added to the existing `btg-devops` dashboard.

Benefits:

- One dashboard for Azure and Power Platform findings.
- Consistent user experience.
- Fits existing dashboard architecture.

Risks:

- More frontend work may be required.
- Dashboard scope may increase.

### Option B — Grafana Dashboard

`btg-devops` outputs JSON or metrics, and Grafana renders the Power Platform dashboard.

Benefits:

- Reuses existing BISTEC monitoring stack.
- Avoids building custom UI.
- Grafana already supports dashboards, alerting, and access control.

Risks:

- Needs JSON/metrics integration.
- May become separate from existing dashboard design.

### Recommendation

For Phase 1, the CLI should output table and JSON only. Dashboard rendering should be handled after the architect confirms whether Power Platform findings should use the existing dashboard or Grafana.

---

## 12. Constraints and Risks

| Constraint / Risk | Mitigation |
|---|---|
| Power Platform API permissions may require admin consent | Confirm tenant permission before coding |
| Service Principal may need broader Power Platform admin role | Document this clearly in setup guide |
| API rate limits may affect large tenants | Add retry and exponential backoff |
| Power BI API requires separate permission scope | Acquire separate token using same Service Principal |
| Dataverse capacity APIs may be less stable | Mark capacity checks as Phase 2 |
| Too much scope in one release | Build Apps, Flows, and DLP first |
| Dashboard approach is not confirmed | Keep Phase 1 CLI-only and JSON-output first |

---

## 13. Success Criteria

This feature is successful when:

1. `btg-devops analyze powerplatform` runs successfully.
2. The command supports `--component apps`, `--component flows`, and `--component dlp`.
3. The command supports table output and JSON output.
4. The analyzer detects failed or suspended flows.
5. The analyzer flags apps shared with Everyone.
6. The analyzer flags production environments missing DLP policies.
7. No credentials are stored in source code.
8. All credentials are loaded from environment variables.
9. The command runs in under 60 seconds for a tenant with fewer than 20 environments.
10. The output can be reused by a dashboard or Grafana in a future phase.

---

## 14. Phase Plan

### Phase 1 — Core Power Platform Analysis

Build first:

```bash
btg-devops analyze powerplatform --component apps
btg-devops analyze powerplatform --component flows
btg-devops analyze powerplatform --component dlp
btg-devops analyze powerplatform --output json
```

Phase 1 checks:

- Unused Power Apps.
- Overshared Power Apps.
- Failed or suspended Power Automate flows.
- Orphaned Power Automate flows.
- Missing DLP policies.

### Phase 2 — Extended Analysis

Add later:

- Dataverse capacity checks.
- Power BI dataset refresh health.
- Power BI workspace activity.
- Dashboard or Grafana visualisation.
- Pre-built dashboard JSON under documentation.

### Phase 3 — Monitoring and Automation

Future work:

- Prometheus metrics output.
- Grafana scrape endpoint.
- Scheduled checks.
- Alerting.
- Trend history.
- Report export.

---

## 15. Architecture Impact

The container diagram should be updated to show the new Power Platform analysis path.

New items to add:

| Item | Type |
|---|---|
| Power Platform Analyzer | Internal component/container inside btg-devops |
| Power Platform Admin API | External system |
| Power BI REST API | External system |
| Microsoft Entra ID / Azure AD | External identity provider |
| JSON / Dashboard / Grafana Output | Output target |

Suggested flow:

```text
DevOps Engineer
        ↓
btg-devops CLI
        ↓
Power Platform Analyzer
        ↓
Microsoft Entra ID authentication
        ↓
Power Platform Admin API
        ↓
Power BI REST API
        ↓
Critical / Warning / Info findings
        ↓
Table / JSON output
        ↓
Dashboard or Grafana
```

---

## 16. ADR Recommendation

A new ADR should be created after the architect confirms the dashboard direction.

Suggested ADR title:

```text
ADR-0004: Add Power Platform Analysis to btg-devops
```

Decision summary:

```text
btg-devops will add a new read-only Power Platform analysis command using Service Principal authentication. The command will analyse Power Apps, Power Automate, and DLP policy metadata first, then extend to Dataverse and Power BI in later phases.
```

---

## 17. Recommendation

The recommended next step is not coding.

The next step is:

1. Confirm whether the BISTEC tenant allows Service Principal access for Power Platform read-only analysis.
2. Confirm dashboard direction: existing dashboard or Grafana.
3. Add this spec file to the new feature branch.
4. Update only the container diagram to include the Power Platform analysis path.
5. Ask for architect review.
6. Start implementation only after approval.

For the first implementation, focus on Power Apps, Power Automate, DLP policies, table output, and JSON output.
