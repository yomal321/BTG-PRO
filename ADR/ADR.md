# btg-devops — Architecture Decision Records

**Project:** btg-devops  
**Last updated:** 2026-06-11

---

# ADR-0001: Three-Path Architecture for System Access

## 1. Status
Accepted  
Date: 2026-06-11

## 2. Context
btg-devops started as a CLI-only tool. Two problems were identified as the project grew:

- **Accessibility problem** — only engineers with a terminal, the binary installed, and Azure credentials configured could run audits. Team leads, managers, and non-technical stakeholders had no way to access findings without asking someone to run the tool for them.
- **AI assistance problem** — every audit required the engineer to know the exact command to run and then manually read and interpret raw JSON or table output. There was no way to ask a question in plain English and get an intelligent response.

The team needed to decide: add one unified new interface that replaces the CLI, or design multiple independent access paths that share the same core engine.

Project facts:
- 12 analyzer modules already built and tested
- CLI usage must not be disrupted — existing engineers rely on it
- Non-technical users need browser-based access
- AI-assisted querying is a distinct feature with different requirements from normal dashboard access
- All paths must produce the same quality of findings using the same analysis logic

## 3. Decision
btg-devops will support three separate access paths, all sharing the same `12 Analyzer Modules (internal/analyzers)` and `Azure SDK Clients (azure-sdk-for-go)` underneath:

**Path 1 — CLI**
Engineer runs `btg-devops analyze [module]` in a terminal. `CLI Interface (cmd/analyze_*.go)` routes to the analyzer. `Output Formatter` renders findings as table or JSON.

**Path 2 — Dashboard Normal Mode**
User opens the Next.js Dashboard in a browser, selects a module, and clicks Analyze. The dashboard calls `HTTP API Server (cmd/server.go)` via REST. Live findings render visually in the dashboard.

**Path 3 — Dashboard Agent Mode**
User types a question in plain English in the dashboard. The dashboard sends the prompt to `Claude Agent`. Claude calls `MCP Server (cmd/mcp.go)` tools, which invoke the analyzers and return findings. Claude explains results in plain English back to the dashboard.

## 4. Consequences

**Benefits**
- Each path is optimised for its user — the CLI stays fast and scriptable for engineers, the dashboard is accessible for non-technical users, and the agent mode removes the need to know any commands at all
- All three paths share the same analysis logic — no duplication of severity rules or Azure SDK calls
- The CLI is completely unaffected by the dashboard and MCP additions — existing workflows continue unchanged
- Each path can be built, tested, and deployed independently

**Risks**
- Three entry points to maintain — changes to the analyzer interface must be reflected in CLI, HTTP API, and MCP paths
- More components increase the surface area for bugs
- A new contributor must understand all three paths to have a complete picture of the system

## 5. Alternatives Considered

**Option 1: Single REST API replacing the CLI**
Rejected because it would break existing CLI usage. Engineers who script `btg-devops analyze storage` in automation pipelines would need to rewrite their workflows. The CLI is the simplest, most reliable interface for programmatic access.

**Option 2: Dashboard only — no CLI, no MCP**
Rejected because a web dashboard requires a browser and a running server. CLI access is needed for headless environments, CI pipelines, and quick terminal use. Removing the CLI would reduce the tool's utility for its primary users.

**Option 3: One unified API that the CLI and dashboard both call**
Rejected for this release because it would require refactoring the existing CLI to become a thin client over an API — adding significant complexity for no immediate benefit. The three-path model achieves the same outcome while keeping each path independent.

## 6. Simple Summary
btg-devops uses three separate access paths — CLI, Dashboard Normal Mode, and Dashboard Agent Mode — because each path serves a different user in a different context, and all three can share the same analysis engine without interfering with each other.

---

# ADR-0002: HTTP API Server for Dashboard Normal Mode

## 1. Status
Accepted  
Date: 2026-06-11

## 2. Context
The Next.js Dashboard needs to fetch live Azure audit data without requiring the user to touch a terminal. Several options were evaluated for how the dashboard should connect to the analysis system.

Project facts:
- The analysis logic already exists in `12 Analyzer Modules (internal/analyzers)` — it must not be duplicated
- Azure credentials must never reach the browser — they must stay server-side
- The dashboard is deployed on Vercel — it has a Next.js server-side runtime available
- Findings must be returned in real time — not from a pre-generated file
- The connection must be simple enough for the dashboard frontend to call with a standard HTTP request

## 3. Decision
A Go HTTP API Server (`cmd/server.go`) will be added to the existing Go binary. It exposes one REST endpoint per analyzer module:

```
GET /analyze/storage
GET /analyze/iam
GET /analyze/nsg
... (one per module)
```

`cmd/root.go` starts the HTTP server alongside the existing CLI and MCP commands. The server calls `12 Analyzer Modules (internal/analyzers)` directly — reusing all existing analysis logic. The Next.js Dashboard calls the server via a standard HTTP request and renders the JSON response.

## 4. Consequences

**Benefits**
- The dashboard frontend makes a simple HTTP GET call — no special protocol or library needed
- Azure credentials stay server-side inside the Go binary — never exposed to the browser
- All existing analyzer logic is reused exactly — no duplication of severity rules or Azure SDK calls
- The REST interface is easy to test with any HTTP client
- Compatible with Vercel deployment — the dashboard can call the Go server from its API routes

**Risks**
- CORS must be configured on the Go server to allow requests from the dashboard origin
- The Go server must be running and accessible when the dashboard makes a request — adds an operational dependency
- A new component (`cmd/server.go`) must be maintained alongside the existing CLI and MCP entry points

## 5. Alternatives Considered

**Option 1: Static JSON file upload**
The user runs the CLI locally, exports a JSON file, and uploads it to the dashboard. Rejected because it is not real-time — findings are only as fresh as the last file uploaded — and it requires the user to know CLI commands, which defeats the purpose of the dashboard.

**Option 2: Shell out to the CLI binary from Next.js**
The Next.js API route runs `exec.Command("btg-devops", "analyze", "storage")` server-side and captures stdout. Rejected because executing subprocesses from a Next.js server is fragile, hard to test, and not compatible with serverless deployments like Vercel.

**Option 3: Call the Azure SDK directly from Next.js**
The dashboard backend calls Azure Resource Manager directly using the Azure JavaScript SDK. Rejected because it would duplicate all analysis and severity logic in a second language (TypeScript), and Azure credentials would need to be configured in the Next.js environment separately from the Go binary.

## 6. Simple Summary
The dashboard connects to the analysis system through a Go REST API server (`cmd/server.go`) because it keeps Azure credentials server-side, reuses all existing analyzer logic, and gives the frontend a simple standard HTTP interface to call.

---

# ADR-0003: MCP + Claude Agent for Dashboard Agent Mode

## 1. Status
Accepted  
Date: 2026-06-11

## 2. Context
Dashboard Agent Mode allows a user to type a question in plain English and receive an intelligent explanation of their Azure audit findings — without knowing which module to run or how to interpret raw output.

The team needed to decide how the dashboard should connect to an AI that can both call the analyzers and explain the results. Several options were evaluated.

Project facts:
- The MCP Server (`cmd/mcp.go`) already exposes all 12 analyzers as callable tools
- Claude already supports MCP tool calling natively
- The dashboard is a Next.js application — it can call external APIs from its server-side routes
- The AI must be able to decide which analyzer to call based on the user's question — not just call a fixed endpoint
- The response must be in plain English, not raw JSON

## 3. Decision
Dashboard Agent Mode will use the existing `MCP Server (cmd/mcp.go)` as the tool backend and `Claude Agent` as the AI layer. The flow is:

1. User types a prompt in the Next.js Dashboard
2. Dashboard sends the prompt to Claude Agent via the Anthropic API
3. Claude Agent reads the prompt, decides which MCP tools to call, and calls `MCP Server (cmd/mcp.go)`
4. `MCP Server` invokes `12 Analyzer Modules (internal/analyzers)` and returns structured findings
5. Claude Agent reads the findings and composes a plain English explanation
6. Dashboard displays the AI response

## 4. Consequences

**Benefits**
- The `MCP Server (cmd/mcp.go)` built for Agent Mode is reused directly — no new backend component needed for the AI path
- Claude handles tool selection automatically — the dashboard does not need to know which analyzer matches which question
- Users get a natural language interface without learning any commands or interpreting raw data
- Claude can call multiple analyzers in one conversation turn if the question spans resource types
- The MCP protocol is an open standard — the same server can be used by Claude Desktop and Claude Code in addition to the dashboard

**Risks**
- Agent Mode depends on Claude API availability — if the Anthropic API is down, Agent Mode is unavailable while Normal Mode continues working
- Claude API calls add latency compared to the direct HTTP call in Normal Mode
- Claude API usage incurs cost per token — high usage by many users could become expensive
- The quality of the AI explanation depends on Claude's interpretation of the findings — occasional misinterpretations are possible

## 5. Alternatives Considered

**Option 1: Call the Anthropic API directly without MCP**
The dashboard sends the question and raw Azure data directly to the Claude API in one prompt, without using MCP tools. Rejected because it would require fetching all Azure data upfront before knowing what the user is asking, and it bypasses the severity classification logic in the analyzers. MCP tool calling allows Claude to fetch only what it needs.

**Option 2: Custom AI agent with function calling**
Build a custom agent that maps user questions to analyzer calls using a hand-written routing layer. Rejected because this re-implements what MCP already provides — a standard protocol for connecting AI models to tools. Using MCP means the same tool definitions work with any MCP-compatible client, not just this dashboard.

**Option 3: Use a different AI model**
Use an open-source model (e.g. Llama, Mistral) with tool calling support instead of Claude. Rejected for this release because Claude has native MCP support and the MCP Server is already built around it. Other models can be evaluated in a future release if cost or availability becomes a concern.

## 6. Simple Summary
Agent Mode uses `Claude Agent` calling the existing `MCP Server (cmd/mcp.go)` because Claude natively supports MCP tool calling, the MCP server is already built, and this approach lets Claude decide what to analyze and explain the findings in plain English without any custom routing logic.

---

# ADR-0004: Next.js, Tailwind CSS and Vercel for the Dashboard Frontend

## 1. Status
Accepted  
Date: 2026-06-11

## 2. Context
The btg-devops dashboard needs a frontend that serves two types of users and two interaction modes in a single web application. The team needed to choose a frontend framework, a styling approach, and a hosting platform.

Project facts:
- The dashboard has a **server-side requirement** — the API route that calls the Go HTTP API Server must run server-side to keep Azure credentials out of the browser
- The dashboard has a **client-side requirement** — the UI must update in real time as findings arrive without full page reloads
- Two modes must be supported in the same application — Normal Mode (button-driven) and Agent Mode (chat interface)
- The application must be deployable with minimal infrastructure setup — the team is focused on the Go binary, not frontend DevOps
- TypeScript is preferred across the project for type safety
- The dashboard should reflect BISTEC branding — a generic out-of-the-box component library look is not acceptable

Options compared:

**Framework:** Next.js, React with Vite, Vue.js, Angular  
**Styling:** Tailwind CSS, Material UI (MUI), Chakra UI, plain CSS modules  
**Hosting:** Vercel, Azure Static Web Apps, self-hosted on Azure App Service

## 3. Decision

**Framework — Next.js 14+ (App Router)**
The dashboard will be built with Next.js 14 using the App Router. Next.js is the only framework that provides both a React-based frontend and a server-side API layer (`/api/analyze`) in one project — the API route calls the Go HTTP API Server securely without exposing credentials to the browser.

**Styling — Tailwind CSS**
The dashboard will use Tailwind CSS for all styling. Components are built from scratch using Tailwind utility classes to match BISTEC branding rather than adopting the default visual identity of a component library.

**Hosting — Vercel**
The dashboard will be deployed on Vercel. Vercel is purpose-built for Next.js — deployment requires zero configuration beyond connecting the repository.

**Full tech stack:**

| Layer | Technology |
|---|---|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Charts | Recharts |
| API layer | Next.js API Routes (`/api/analyze`) |
| Deployment | Vercel |

## 4. Consequences

**Benefits**
- Next.js provides both the frontend UI and the server-side API route in one project — no separate backend service is needed for the dashboard layer
- The API route runs on Vercel's server — Azure credentials stay server-side and never reach the browser
- App Router supports React Server Components — data fetching is clean and performant without extra state management libraries
- Tailwind gives full control over the visual design — the dashboard can match BISTEC colours, typography and spacing exactly
- Vercel deployment is a single `git push` — no Docker, no Azure App Service configuration, no pipeline setup needed
- TypeScript across the entire dashboard catches type errors before runtime — especially important for the MCP and REST API response parsing

**Risks**
- Next.js App Router is a newer pattern — some team members may need time to learn the server/client component boundary
- Tailwind requires discipline — without a component system, CSS can become inconsistent if multiple developers style the same components differently
- Vercel is a third-party hosting platform — if BISTEC requires all infrastructure on Azure, Vercel would need to be replaced with Azure Static Web Apps
- Building UI components from scratch with Tailwind takes more initial time than dropping in a pre-built MUI component library

## 5. Alternatives Considered

**Framework — React with Vite**
Rejected because React with Vite is a pure client-side framework. It has no server-side runtime, which means the API route that calls the Go HTTP API Server would not exist — Azure credentials would either need to be embedded in the browser bundle (a security risk) or a separate Express server would need to be added (extra infrastructure). Next.js eliminates this problem by including the server layer in the same project.

**Framework — Vue.js**
Rejected because the team's existing TypeScript experience is React-based. Vue would introduce a new template syntax and ecosystem with no benefit over Next.js for this use case. Nuxt.js (the Vue equivalent of Next.js) would provide similar server-side capabilities but at the cost of a framework switch the team is not familiar with.

**Framework — Angular**
Rejected because Angular is significantly more complex for a dashboard of this scope. Its strong opinions on architecture, dependency injection, and module structure add overhead that is not justified for a focused internal tool.

**Styling — Material UI (MUI)**
Rejected because MUI components have a strong default visual identity (Google Material Design) that does not match BISTEC branding without significant theme overriding — which often takes more effort than building components directly with Tailwind. MUI also adds a large bundle size for a relatively simple dashboard UI.

**Styling — Chakra UI**
Rejected for the same branding reason as MUI. Chakra is well-designed but its default colour system and component shapes would need extensive customisation to reflect BISTEC's visual identity.

**Hosting — Azure Static Web Apps**
Remains a viable fallback if BISTEC requires all infrastructure on Azure. Not chosen for v0.14.0 because Azure Static Web Apps requires additional configuration for Next.js server-side rendering and API routes, whereas Vercel supports both with zero setup.

**Hosting — Self-hosted on Azure App Service**
Rejected because it introduces infrastructure management overhead that is unnecessary for a frontend application. Vercel handles scaling, CDN, and deployment automatically.

## 6. Simple Summary
The dashboard is built with Next.js, styled with Tailwind CSS, and deployed on Vercel because Next.js is the only single-project solution that provides both the React frontend and the server-side API layer needed to securely call the Go backend, Tailwind gives full control over BISTEC branding, and Vercel deploys Next.js with zero configuration.


## 1. Status
Accepted  
Date: 2026-06-11

## 2. Context
Dashboard Agent Mode allows a user to type a question in plain English and receive an intelligent explanation of their Azure audit findings — without knowing which module to run or how to interpret raw output.

The team needed to decide how the dashboard should connect to an AI that can both call the analyzers and explain the results. Several options were evaluated.

Project facts:
- The MCP Server (`cmd/mcp.go`) already exposes all 12 analyzers as callable tools
- Claude already supports MCP tool calling natively
- The dashboard is a Next.js application — it can call external APIs from its server-side routes
- The AI must be able to decide which analyzer to call based on the user's question — not just call a fixed endpoint
- The response must be in plain English, not raw JSON

## 3. Decision
Dashboard Agent Mode will use the existing `MCP Server (cmd/mcp.go)` as the tool backend and `Claude Agent` as the AI layer. The flow is:

1. User types a prompt in the Next.js Dashboard
2. Dashboard sends the prompt to Claude Agent via the Anthropic API
3. Claude Agent reads the prompt, decides which MCP tools to call, and calls `MCP Server (cmd/mcp.go)`
4. `MCP Server` invokes `12 Analyzer Modules (internal/analyzers)` and returns structured findings
5. Claude Agent reads the findings and composes a plain English explanation
6. Dashboard displays the AI response

## 4. Consequences

**Benefits**
- The `MCP Server (cmd/mcp.go)` built for Agent Mode is reused directly — no new backend component needed for the AI path
- Claude handles tool selection automatically — the dashboard does not need to know which analyzer matches which question
- Users get a natural language interface without learning any commands or interpreting raw data
- Claude can call multiple analyzers in one conversation turn if the question spans resource types
- The MCP protocol is an open standard — the same server can be used by Claude Desktop and Claude Code in addition to the dashboard

**Risks**
- Agent Mode depends on Claude API availability — if the Anthropic API is down, Agent Mode is unavailable while Normal Mode continues working
- Claude API calls add latency compared to the direct HTTP call in Normal Mode
- Claude API usage incurs cost per token — high usage by many users could become expensive
- The quality of the AI explanation depends on Claude's interpretation of the findings — occasional misinterpretations are possible

## 5. Alternatives Considered

**Option 1: Call the Anthropic API directly without MCP**
The dashboard sends the question and raw Azure data directly to the Claude API in one prompt, without using MCP tools. Rejected because it would require fetching all Azure data upfront before knowing what the user is asking, and it bypasses the severity classification logic in the analyzers. MCP tool calling allows Claude to fetch only what it needs.

**Option 2: Custom AI agent with function calling**
Build a custom agent that maps user questions to analyzer calls using a hand-written routing layer. Rejected because this re-implements what MCP already provides — a standard protocol for connecting AI models to tools. Using MCP means the same tool definitions work with any MCP-compatible client, not just this dashboard.

**Option 3: Use a different AI model**
Use an open-source model (e.g. Llama, Mistral) with tool calling support instead of Claude. Rejected for this release because Claude has native MCP support and the MCP Server is already built around it. Other models can be evaluated in a future release if cost or availability becomes a concern.

## 6. Simple Summary
Agent Mode uses `Claude Agent` calling the existing `MCP Server (cmd/mcp.go)` because Claude natively supports MCP tool calling, the MCP server is already built, and this approach lets Claude decide what to analyze and explain the findings in plain English without any custom routing logic.