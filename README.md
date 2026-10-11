# 🚀 Agent Control Tower
### Multi-Agent AI Orchestration, Observability & Governance Platform

[![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![React](https://img.shields.io/badge/React-TypeScript-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Backend-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Redis](https://img.shields.io/badge/Redis-Event_Processing-DC382D?logo=redis&logoColor=white)](https://redis.io/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Agent Control Tower** is a centralized control plane concept for observing, understanding, governing, and measuring multi-agent AI workflows. It brings agent activity, execution lineage, anomalies, approvals, cost attribution, and audit/replay into one dashboard.

> **Project status:** Update this section to accurately describe your current implementation (prototype, in development, or production-ready). Features listed below are intended capabilities; only mark a feature as implemented after you have tested it.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Problem Statement](#-problem-statement)
- [Goals](#-goals)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Technology Stack](#-technology-stack)
- [Agent Workflow](#-agent-workflow)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
- [Environment Variables](#-environment-variables)
- [Running Locally](#-running-locally)
- [API Overview](#-api-overview)
- [Event Schema](#-event-schema)
- [Anomaly Detection](#-anomaly-detection)
- [Governance and Human Approval](#-governance-and-human-approval)
- [Security Notes](#-security-notes)
- [Testing](#-testing)
- [Deployment](#-deployment)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

## 🌐 Overview

As organizations adopt autonomous AI agents, monitoring them across different frameworks becomes difficult. Logs may be inconsistent, workflows can involve many agent-to-agent handoffs, and unexpected retries or tool loops can increase latency and model costs.

Agent Control Tower aims to provide one interface for:

- Monitoring agent status and active runs.
- Visualizing agent, tool, and task relationships.
- Detecting suspicious or inefficient execution patterns.
- Estimating and attributing model usage costs.
- Routing sensitive actions to a human for review.
- Reviewing execution history through trace and audit views.

The platform is designed around a framework-neutral event format so integrations can be added for systems such as LangChain, LangGraph, CrewAI, and AutoGen.

## ❗ Problem Statement

Multi-agent systems are powerful but can be difficult to operate safely and efficiently. Common challenges include:

- Fragmented observability across frameworks and services.
- Limited visibility into agent handoffs and tool usage.
- Repeated calls, excessive retries, silent failures, and cost spikes.
- Difficulty attributing usage to an agent, team, or task.
- Need for human approval before high-impact operations.
- Limited ability to reconstruct what happened during a run.

## 🎯 Goals

1. Centralize agent and run monitoring.
2. Normalize events from multiple agent frameworks.
3. Show execution lineage and handoffs visually.
4. Surface anomalies and operational alerts.
5. Attribute token usage and estimated cost.
6. Provide policy-based controls and human approvals.
7. Preserve trace history for investigation and replay.

## ✨ Key Features

### 1. Unified Monitoring Dashboard
- Overview of registered agents and their statuses.
- Run/task activity, failures, token usage, and estimated cost.
- Alerts and pending approvals at a glance.
- Live updates through WebSockets when configured.

### 2. Agent Registry
- Register agents and record their framework, team, and metadata.
- Inspect status and recent activity.
- Filter agents by framework, status, or team.

### 3. Agent Lineage Graph
- Visualize agent-to-agent handoffs and tool calls.
- Trace parent and child runs.
- Identify dependencies and possible failure propagation.
- Use React Flow (`@xyflow/react`) for graph visualization.

### 4. Anomaly Detection
Potential rules include:
- Token or cost thresholds exceeded.
- Repeated calls to the same tool.
- Excessive retries or failed runs.
- Abnormally high latency.
- Suspected loops or stalled workflows.

Rule-based detection is explainable and should be the initial safety layer. Statistical methods such as Isolation Forest can be added as a supplementary signal after collecting suitable data.

### 5. Cost Attribution
- Track input/output tokens and estimated model costs.
- Aggregate usage by agent, team, task, model, and time period.
- Support configurable model pricing rather than assuming a fixed price.
- Clearly distinguish estimates from provider-reported billing.

### 6. Human-in-the-Loop (HITL)
- Send configured high-impact actions for review.
- Allow authorized operators to approve or reject a pending request.
- Record the reviewer, decision, timestamp, and reason.
- Keep the final action execution behind a server-side authorization and policy check.

### 7. Policy Engine
Example policies:
- Pause a run when its configured cost limit is exceeded.
- Flag a run after too many retries.
- Require approval for selected tools or sensitive actions.
- Stop a workflow after a repeated-call threshold.

Policies should be configurable and enforced by the backend, not only by frontend controls.

### 8. Execution Replay and Audit
- Browse a run's chronological events.
- Inspect tool calls, errors, handoffs, and model usage.
- Reconstruct the recorded sequence for debugging.
- Maintain an audit trail for important operator and policy decisions.

**Replay note:** Replaying a recorded trace for inspection is different from re-executing an agent. Re-execution can have external side effects and should require explicit safeguards.

### 9. AI-Assisted Investigation
An optional language-model integration can summarize traces and explain likely causes of alerts. The model should assist investigation, not override deterministic security policies or approval requirements.

## 🏗️ Architecture

```text
        Agent Frameworks / SDK Integrations
       ┌──────────┬──────────┬─────────┬─────────┐
       │LangChain │LangGraph │ CrewAI  │ AutoGen │
       └──────────┴──────────┴─────────┴─────────┘
                           │
                           ▼
                Universal Agent Event Schema
                           │
                           ▼
                   FastAPI Ingestion API
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
          Redis Streams         PostgreSQL
          (event queue)       (durable records)
                 │                   ▲
                 ▼                   │
           Event Worker ─────────────┘
                 │
      ┌──────────┼───────────┬────────────┐
      ▼          ▼           ▼            ▼
  Anomaly     Cost Engine  Policy/HITL  Trace/Audit
  Detection
                 │
                 ▼
        REST API + WebSockets
                 │
                 ▼
        React + TypeScript UI
```

### Architecture principles
- **Framework-neutral ingestion:** normalize events at the boundary.
- **Durable records:** use PostgreSQL as the source of truth for persisted application data.
- **Asynchronous processing:** use Redis Streams or another queue for event processing where configured.
- **Real-time UI:** publish validated updates through WebSockets.
- **Server-side enforcement:** authorization and policies are checked by the backend.
- **Observable failure handling:** record processing errors and provide retry/dead-letter handling as appropriate.

## 🧰 Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React, TypeScript, Vite | Dashboard UI |
| Styling | Tailwind CSS | Responsive interface |
| Graph visualization | `@xyflow/react` | Agent lineage |
| Charts | Recharts | Usage and cost analytics |
| API | Python, FastAPI, Pydantic | REST APIs and validation |
| Database | PostgreSQL, SQLAlchemy, Alembic | Persistent data and migrations |
| Event processing | Redis Streams | Asynchronous event pipeline |
| Live updates | WebSockets | Dashboard updates |
| Agent orchestration/integration | LangGraph, LangChain adapters | Workflow integration |
| Optional AI explanations | OpenAI API | Trace summaries and alert explanations |
| Optional statistical detection | scikit-learn | Supplementary anomaly signals |
| Local environment | Docker Compose | Reproducible development setup |

Choose compatible package versions and pin them in dependency files before sharing or deploying the project.

## 🔄 Agent Workflow

Example invoice-processing workflow:

```text
Orchestrator
    │
    ▼
Invoice Agent
    │
    ▼
Fraud Risk Agent
    │
    ▼
Policy Agent
    │
    ▼
Decision Agent
    │
    ├── Low risk / permitted ──► Continue workflow
    │
    └── High risk / sensitive ─► Human Approval
                                      │
                             Approve / Reject
                                      │
                                      ▼
                              Payment Agent
```

The control tower observes and records the workflow. It should not treat a model's output alone as authorization to execute a sensitive operation.

## 📁 Repository Structure

Suggested structure (adjust this tree to match your actual repository):

```text
agent-control-tower/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── features/
│   │   ├── hooks/
│   │   ├── lib/
│   │   └── types/
│   ├── package.json
│   └── Dockerfile
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── core/
│   │   ├── db/
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── workers/
│   │   └── main.py
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── sdk/
│   └── README.md
├── docs/
│   ├── architecture.md
│   └── screenshots/
├── docker-compose.yml
├── .env.example
├── .gitignore
├── LICENSE
└── README.md
```

## 🚀 Getting Started

### Prerequisites

Install the following tools:

- Git
- Python 3.12 or a version supported by the project dependencies
- Node.js LTS and npm
- Docker Desktop with Docker Compose (recommended)
- PostgreSQL and Redis, if not using Docker
- An OpenAI API key only if AI-assisted explanations are enabled

### 1. Clone the repository

Replace the placeholder URL with your actual GitHub repository URL:

```bash
git clone https://github.com/YOUR_USERNAME/agent-control-tower.git
cd agent-control-tower
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

On Windows Command Prompt, copy the file with:

```bat
copy .env.example .env
```

Edit `.env` with your local configuration. Do not commit secrets.

### 3. Start dependencies with Docker Compose

If your repository includes a configured `docker-compose.yml`:

```bash
docker compose up --build
```

The exact services and ports depend on your Compose configuration. Check the startup logs and service health checks before opening the app.

### 4. Run the backend manually

Example commands, assuming the FastAPI application is under `backend/app/main.py`:

```bash
cd backend
python -m venv .venv
```

Activate the environment:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install dependencies and start the API:

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

API documentation should be available at:

- Swagger UI: `http://localhost:8000/docs`
- OpenAPI schema: `http://localhost:8000/openapi.json`
- Health endpoint: `http://localhost:8000/health` (if implemented)

### 5. Run the frontend manually

In a separate terminal:

```bash
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite, commonly `http://localhost:5173`.

> These commands assume the folders and scripts described above exist. Adapt them to your actual project structure and `package.json` scripts.

## 🔐 Environment Variables

Example `.env.example`:

```dotenv
# Backend
APP_ENV=development
APP_HOST=0.0.0.0
PORT=8000
JWT_SECRET=replace-with-a-long-random-secret
CORS_ORIGINS=http://localhost:5173

# Data services — use values appropriate for your environment
DATABASE_URL=postgresql+psycopg://postgres:postgres@localhost:5432/agent_control_tower
REDIS_URL=redis://localhost:6379/0

# Optional AI explanations; keep this key on the server only
OPENAI_API_KEY=
OPENAI_MODEL=

# Frontend
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000/ws/dashboard
```

Use the connection URL format required by your database driver and hosting provider. If your provider supplies a different PostgreSQL URL format, convert or configure it appropriately.

**Important:** `.env.example` should contain placeholders only. Never commit real API keys, database passwords, JWT secrets, access tokens, or production credentials.

## 🔌 API Overview

The following is a suggested API design. Implemented routes may differ.

| Method | Example endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Service health |
| `GET` | `/api/v1/agents` | List registered agents |
| `POST` | `/api/v1/agents` | Register an agent |
| `GET` | `/api/v1/agents/{agent_id}` | Agent details |
| `GET` | `/api/v1/runs` | List runs |
| `GET` | `/api/v1/runs/{run_id}` | Run details |
| `GET` | `/api/v1/runs/{run_id}/events` | Run event timeline |
| `POST` | `/api/v1/events` | Ingest an agent event |
| `GET` | `/api/v1/lineage/{run_id}` | Retrieve lineage data |
| `GET` | `/api/v1/alerts` | List anomaly alerts |
| `POST` | `/api/v1/approvals/{approval_id}/decision` | Submit an approval decision |
| `GET` | `/api/v1/costs/summary` | Cost and token summary |
| `GET` | `/api/v1/audit-logs` | Query audit records |
| `WS` | `/ws/dashboard` | Live dashboard updates |

Check the running API's `/docs` page for the routes actually exposed by your implementation. Protect state-changing routes with authentication, authorization, validation, and audit logging.

## 📨 Universal Agent Event Schema

A normalized event allows the control tower to process telemetry from different frameworks.

Example event:

```json
{
  "event_id": "evt_01",
  "timestamp": "2026-10-11T12:00:00Z",
  "event_type": "tool_call",
  "agent_id": "fraud-agent",
  "agent_name": "Fraud Risk Agent",
  "framework": "langgraph",
  "team": "claims",
  "run_id": "run_123",
  "task_id": "task_456",
  "parent_agent_id": "orchestrator",
  "parent_run_id": "run_parent",
  "tool_name": "risk_lookup",
  "status": "started",
  "input_tokens": 120,
  "output_tokens": 35,
  "total_tokens": 155,
  "cost_usd": 0.0012,
  "latency_ms": 240,
  "retry_count": 0,
  "error_message": null,
  "metadata": {}
}
```

Treat this as an illustrative schema. Define and validate required fields, permitted event types, timestamp formats, payload size limits, and event versioning in your implementation. Avoid placing secrets or unnecessary personal data in event inputs and outputs.

Suggested event types:

- `agent_registered`, `agent_started`, `agent_completed`, `agent_failed`
- `task_started`, `task_completed`, `task_failed`
- `llm_call`, `tool_call`, `tool_result`
- `agent_handoff`
- `approval_requested`, `approval_completed`
- `policy_triggered`, `anomaly_detected`

## 🚨 Anomaly Detection

An initial rule-based engine can evaluate signals such as:

| Signal | Example condition | Suggested response |
|---|---|---|
| Repeated tool calls | Same tool called repeatedly in a short window | Raise alert; inspect trace |
| Token spike | Usage exceeds configured run threshold | Alert or pause according to policy |
| Retry storm | Retry count exceeds policy limit | Pause or stop the run |
| High latency | Duration exceeds a configured threshold | Warn and investigate |
| Repeated failures | Error rate exceeds a configured threshold | Alert and isolate the cause |
| Cost spike | Estimated run cost exceeds budget | Alert and apply configured controls |

Thresholds should be configurable, tested, and appropriate to the workload. Detection indicates a possible problem; it is not proof of malicious behavior.

## 🛡️ Governance and Human Approval

A policy may request approval before an agent performs a sensitive operation. A safe approval lifecycle should:

1. Create a pending approval record.
2. Show the requester, run context, action, and reason.
3. Verify the operator's identity and permissions.
4. Record the approval or rejection and its timestamp.
5. Re-check the policy immediately before executing the action.
6. Record the outcome in the audit log.

Do not trust a frontend-only check or an LLM-generated decision as the sole authorization mechanism. Enforce permissions and policies on the server.

## 🔒 Security Notes

Before using this project with real agents or production data:

- Store secrets in environment variables or a managed secrets service.
- Use strong authentication and role-based access control.
- Validate and limit incoming event payloads.
- Authenticate agent SDKs and restrict ingestion access.
- Apply rate limits and request-size limits.
- Use TLS for public API and WebSocket traffic.
- Restrict CORS to trusted frontend origins.
- Redact credentials, personal information, and sensitive tool outputs from logs.
- Apply least-privilege permissions to database and infrastructure accounts.
- Protect approval, pause, resume, and termination actions with server-side checks.
- Treat agent event payloads as untrusted input.
- Define retention, deletion, and access policies for traces and audit data.
- Do not claim compliance or production security without an appropriate review.

## 🧪 Testing

Add tests appropriate to the modules you implement:

```bash
# Backend example
cd backend
pytest

# Frontend example
cd frontend
npm test
```

These commands require the corresponding test dependencies and scripts to be configured. Recommended test areas:

- Event schema validation and ingestion.
- Duplicate event handling and worker retries.
- Cost calculations and model pricing configuration.
- Anomaly thresholds and false-positive cases.
- Policy enforcement and approval permissions.
- Run lineage and trace ordering.
- Authentication, authorization, and input validation.
- WebSocket reconnect and update behavior.

## ☁️ Deployment

A possible deployment layout is:

- **Frontend:** Vercel or another static frontend host.
- **Backend API:** Render, a container platform, or a cloud service.
- **Worker:** a separate background worker process.
- **Database:** managed PostgreSQL.
- **Event queue:** managed Redis-compatible service.

Configure the following for your actual hosting provider:

- Production `DATABASE_URL` and `REDIS_URL`.
- Backend secrets and optional AI API key.
- Frontend API URL and secure WebSocket URL.
- Explicit CORS allowlist for the deployed frontend.
- Database migrations and health checks.
- Logs, backups, resource limits, and monitoring.

Never put server secrets in frontend variables prefixed with `VITE_`. Those variables are bundled into client-side code.

## 🗺️ Roadmap

- [ ] Complete agent registry and event ingestion API.
- [ ] Add SDK and integration examples for multiple frameworks.
- [ ] Build real-time dashboard and agent status views.
- [ ] Implement lineage visualization.
- [ ] Add configurable anomaly rules.
- [ ] Implement token and cost attribution.
- [ ] Add role-based access and human approval workflow.
- [ ] Add trace inspection, replay, and audit views.
- [ ] Add worker retry handling and dead-letter processing.
- [ ] Add automated tests and CI.
- [ ] Add deployment configuration and operational documentation.
- [ ] Evaluate optional statistical anomaly detection and AI-assisted explanations.

Update the checklist as features are completed and verified.

## 🤝 Contributing

Contributions, bug reports, and suggestions are welcome.

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`.
3. Make focused changes and add tests where practical.
4. Run the relevant tests and checks.
5. Open a pull request describing the change and any limitations.

For significant changes, open an issue first to discuss the approach.

## 📄 License

This project is intended to use the MIT License. Add a `LICENSE` file containing the appropriate license text before representing the repository as MIT-licensed. If you choose another license, update this section and the badge.

## 👤 Author

**Your Name**

- GitHub: [@YOUR_USERNAME](https://github.com/YOUR_USERNAME)
- LinkedIn: [Add your LinkedIn profile](https://www.linkedin.com/)

Replace the placeholders with your public profile links.

---

⭐ If you find this project useful, consider starring the repository and sharing feedback.

**Positioning:** Agent Control Tower gives teams a centralized way to observe, understand, govern, and measure their autonomous AI agents.
