# btg-devops — Sequence Diagrams

**Version:** v0.12.0  
**Scope:** Current production CLI only — no upcoming tasks included

---

## Diagram 1 — Happy Path

> `btg-devops analyze storage --output json` runs successfully against a live Azure subscription.

```mermaid
sequenceDiagram
    actor Engineer as DevOps Engineer
    participant CLI as btg-devops CLI
    participant Auth as azidentity
    participant AAD as Azure Active Directory
    participant ARM as Azure Resource Manager
    participant Out as Output Formatter

    Engineer->>CLI: btg-devops analyze storage --output json

    CLI->>Auth: NewClientSecretCredential(tenantID, clientID, secret)
    Auth->>AAD: POST /oauth2/v2.0/token (HTTPS)
    AAD-->>Auth: 200 OK — access_token issued
    Auth-->>CLI: TokenCredential ready

    CLI->>ARM: armstorage.NewClient(cred).List(subscriptionID)
    ARM-->>CLI: 200 OK — []StorageAccount

    CLI->>CLI: StorageAnalyzer.Analyze(accounts) → []Finding

    CLI->>Out: Render(findings, format="json")
    Out-->>Engineer: JSON findings printed to stdout
```

---

## Diagram 2 — Missing Credentials

> One or more required environment variables are not set. The CLI exits before touching Azure.

```mermaid
sequenceDiagram
    actor Engineer as DevOps Engineer
    participant CLI as btg-devops CLI
    participant Env as OS Environment

    Engineer->>CLI: btg-devops analyze storage

    CLI->>Env: os.Getenv("AZURE_CLIENT_SECRET")
    Env-->>CLI: "" (empty — variable not set)

    CLI-->>Engineer: Error: AZURE_CLIENT_SECRET is required (exit 1)

    note over CLI,Env: CLI checks all four env vars on startup:<br/>AZURE_TENANT_ID, AZURE_CLIENT_ID,<br/>AZURE_CLIENT_SECRET, AZURE_SUBSCRIPTION_ID.<br/>Any missing value triggers this error path.
```

---

## Diagram 3 — Azure Authentication Failure

> All environment variables are present but the Service Principal credentials are invalid or expired.

```mermaid
sequenceDiagram
    actor Engineer as DevOps Engineer
    participant CLI as btg-devops CLI
    participant Auth as azidentity
    participant AAD as Azure Active Directory

    Engineer->>CLI: btg-devops analyze storage

    CLI->>Auth: NewClientSecretCredential(tenantID, clientID, secret)
    Auth->>AAD: POST /oauth2/v2.0/token (HTTPS)
    AAD-->>Auth: 401 Unauthorized — invalid_client

    Auth-->>CLI: AuthenticationFailedError
    CLI-->>Engineer: Error: authentication failed (exit 1)

    note over Auth,AAD: Azure Resource Manager is never reached.<br/>Auth must succeed before any resource<br/>data can be fetched.
```

---

## Arrow Convention

| Arrow | Meaning |
|---|---|
| `->>` | Call / Request |
| `-->>` | Response / Return |
| Red text | Error response |