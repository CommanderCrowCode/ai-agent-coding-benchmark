# AI Agent Coding Benchmark

A real-world evaluation of AI coding agents on a production-grade full-stack application. Six models, five agents each, one task.

Published analysis: [Your Benchmark Is the Only One That Matters](https://flight-notes.ghost.io/your-benchmark-is-the-only-one-that-matters/) · [tanwa.info](https://tanwa.info)

---

## The Finding

Standard AI benchmarks (HumanEval, SWE-bench, Artificial Analysis Intelligence Index) show a **2.2×** spread from weakest to strongest model. The same models on this real-world build task show a **5.1×** spread — and two models that scored *perfect marks* on Docker/Nginx configuration produced applications that cannot start.

The gap between "scores well on benchmarks" and "builds working software" is not noise.

---

## Results

| Model | Absolute Score | Grade | Relative Score | App Starts? |
|-------|---------------|-------|---------------|-------------|
| Claude Opus 4.6 (Claude Code) | 190 / 200 | S | 9.51 / 10 | ✅ Yes |
| GLM 5 | 188 / 200 | S | 9.52 / 10 | ✅ Yes |
| Kimi 2.5 | 179 / 200 | A | 8.76 / 10 | ✅ Yes |
| Codex 5.3 High | 168 / 200 | A | 7.84 / 10 | ✅ Yes |
| Nvidia Nemotron 3 | 41.5 / 200 | D | 4.26 / 10 | ❌ No |
| GPT OSS 120B | 37 / 200 | F | 3.36 / 10 | ❌ No |

Claude Opus 4.6 and GLM 5 are separated by 2 absolute points and 0.01 on the relative scale — a practical tie.

Nemotron 3 and GPT OSS 120B both scored **10/10 on Docker & Nginx configuration** — perfect marks, matching or exceeding the top performers — while producing applications that fail on launch. See: [The Docker Paradox](#the-docker-paradox).

---

## The Task

Each model team was given the same specification: build a production-grade inventory management system for a small import/wholesale business.

Full specification: [`INSTRUCTIONS.md`](INSTRUCTIONS.md)

**Scope:**
- **Database:** 20+ interconnected tables. Users with bcrypt + TOTP MFA. Suppliers, products with price history, purchase orders with 10-stage workflow, shipments, payments (Wise API), inventory ledger, multi-platform sales.
- **Backend:** FastAPI + SQLAlchemy ORM. JWT auth (HS256). Full CRUD. Atomic bulk inventory operations. Background scheduler. Audit log capturing every mutation with old/new value diffing.
- **Security:** bcrypt work factor 12. Rate limiting (100 req/min global, 5 req/min auth). bleach input sanitization. Fernet encryption. CORS whitelist. Security headers.
- **Frontend:** Next.js 14 + React 18 + TailwindCSS. Dashboard with KPI cards, order pipeline, inventory health.
- **Infrastructure:** Docker Compose (frontend, backend, nginx). SSL termination. Health checks.
- **Tests:** 117+ pytest tests across 8 files. In-memory SQLite isolation. All must pass.

---

## Agent Structure

Each model team used a **5-agent relay-mesh** architecture:

| Agent | Role | Scope |
|-------|------|-------|
| Agent 1 — Team Lead | Database & Models | Schema design, SQLAlchemy models, coordinates all agents |
| Agent 2 — Backend Auth | Config & Core Auth | `main.py`, auth router, products/suppliers routers |
| Agent 3 — Backend Logic | Business Logic | Orders, payments, inventory router, dashboard, services |
| Agent 4 — Frontend Dev | UI Layer | Next.js 14 dashboard, components, API integration |
| Agent 5 — DevOps & Testing | Infrastructure & QA | Docker Compose, Nginx config, full pytest suite |

All agents operated fully autonomously — no human input between agent handoffs. Each agent received the full task specification plus role-specific instructions.

Agent prompt templates (from the Codex team run, which can be adapted for any agentic framework): [`agent_prompts/`](agent_prompts/)

The relay-mesh pattern: Agent 1 creates the foundational layer, each subsequent agent reads prior agents' output and builds on it, Agent 5 validates the complete system.

---

## The Docker Paradox

The two lowest-scoring models — Nemotron 3 (41.5/200) and GPT OSS 120B (37/200) — each scored **10/10 on Docker & Nginx configuration**, matching or exceeding the top performers.

Their applications do not run.

This reveals something about how these models work: producing a correct `docker-compose.yml` with proper service definitions, health checks, and network configuration is a *pattern reproduction* task. A model can learn to do it from training data without any coherent understanding of what it's actually deploying.

The Docker file is plausible. The application it points to is broken. The wrapper is perfect. The contents are not there.

This also appeared in the benchmark rankings: on the Artificial Analysis Intelligence Index, Codex 5.3 (score: 49) ranks above Kimi 2.5 (score: 46). In this real-world test, Kimi (179/200) outperforms Codex (168/200) by 11 points. The standard benchmark inverted the relative ordering between these two models.

---

## Scoring Rubrics

Two independent rubrics reduce the chance of gaming either one.

### Relative Rubric — [`GRADING_RUBRIC.md`](GRADING_RUBRIC.md)
Scores each of 20 features from 0–10, weighted by business criticality. Produces a weighted average out of 10.

Example weights:
- Inventory Router: 15% (highest — core business logic)
- Auth & Security: 10%
- Email Utility: 3% (lowest)

### Absolute Rubric — [`GRADING_RUBRIC_ABSOLUTE.md`](GRADING_RUBRIC_ABSOLUTE.md)
Eight binary/functional categories. 200 points total. Unforgiving: code that exists but doesn't function scores zero.

| Category | Points |
|----------|--------|
| Does It Run? | 30 |
| Test Suite | 35 |
| API Endpoints | 50 |
| Business Logic | 25 |
| Security | 20 |
| Audit Trail | 10 |
| Frontend | 10 |
| Schema Completeness | 20 |

---

## Grading Reports

[`reports/`](reports/) contains detailed grading reports for all models on both rubrics:

| File | Description |
|------|-------------|
| `absolute_claude_code.md` | Absolute rubric — Opus 4.6 |
| `absolute_codex.md` | Absolute rubric — Codex 5.3 High |
| `absolute_opencode_GLM.md` | Absolute rubric — GLM 5 |
| `absolute_opencode_kimi.md` | Absolute rubric — Kimi 2.5 |
| `absolute_opencode_nemotron.md` | Absolute rubric — Nvidia Nemotron 3 |
| `absolute_opencode_OSS.md` | Absolute rubric — GPT OSS 120B |
| `relative_claude_code.md` | Relative rubric — Opus 4.6 |
| `relative_codex.md` | Relative rubric — Codex 5.3 High |
| `relative_opencode_GLM.md` | Relative rubric — GLM 5 |
| `relative_opencode_kimi.md` | Relative rubric — Kimi 2.5 |
| `relative_opencode_nemotron.md` | Relative rubric — Nvidia Nemotron 3 |
| `relative_opencode_OSS.md` | Relative rubric — GPT OSS 120B |
| `benchmark_presentation.html` | Interactive report with Chart.js visualizations |

Note: `absolute_emergent.md` and `relative_emergent.md` are present in the reports directory — these are grading reports for a 7th model run that was excluded from the published results.

---

## How to Replicate This Test

### 1. Choose your task
Pick something representative of your actual work — not a toy problem. A service you need to build, an API you need to implement, a migration you need to write. The task needs to be large enough that compounding failures become visible (a single isolated function won't surface the same dynamics).

### 2. Write your specification
Use [`INSTRUCTIONS.md`](INSTRUCTIONS.md) as a template. Be specific: list every feature, every constraint, every acceptance criterion. Write it as you would write a real ticket, not a coding challenge prompt.

### 3. Write your rubrics before you run
Define "works" before you see any output. Both the relative rubric (feature quality 0–10) and the absolute rubric (functional pass/fail) work well together. See [`GRADING_RUBRIC.md`](GRADING_RUBRIC.md) and [`GRADING_RUBRIC_ABSOLUTE.md`](GRADING_RUBRIC_ABSOLUTE.md) for structure.

### 4. Set up the agent structure
Adapt the prompts in [`agent_prompts/`](agent_prompts/) for your agentic framework. The 5-agent relay-mesh pattern works across most multi-agent systems. Key principles:
- Agent 1 builds the foundation layer (schema, models)
- Each subsequent agent reads prior output and builds on it
- Agent 5 validates everything and writes tests
- All agents operate fully autonomously — no human handoffs

Run 3–5 agent teams per model for statistical stability.

### 5. Grade against your rubrics
Apply both rubrics independently. The absolute rubric is the signal that matters most: does it run? do the tests pass? does the business logic produce correct results?

Appearance (README quality, code style, comment density) should not score points.

### 6. Compare
Run at least two or three models to establish a baseline. You have no reference point without comparison.

### 7. Re-run periodically
Model capabilities update frequently. A result from six months ago may not reflect current performance.

---

## Repo Structure

```
ai-agent-coding-benchmark/
├── README.md                         This file
├── INSTRUCTIONS.md                   Full task specification given to all agents
├── GRADING_RUBRIC.md                 Relative rubric (20 features, weighted 0–10)
├── GRADING_RUBRIC_ABSOLUTE.md        Absolute rubric (8 categories, 200 pts)
├── agent_prompts/
│   ├── agent1_lead_prompt.md         Team Lead: DB schema, models, coordination
│   ├── agent2_prompt.md              Backend Auth: config, main.py, auth router
│   ├── agent3_prompt.md              Backend Logic: orders, inventory, payments
│   ├── agent4_prompt.md              Frontend: Next.js 14 dashboard
│   └── agent5_prompt.md             DevOps & Testing: Docker, Nginx, pytest
└── reports/
    ├── absolute_*.md                 Absolute rubric grading for each model
    ├── relative_*.md                 Relative rubric grading for each model
    └── benchmark_presentation.html   Interactive results report
```

---

## Notes on Methodology

- All models received identical instructions and the same `INSTRUCTIONS.md` specification
- No model was fine-tuned for this task
- Each team ran autonomously; no human input between agent handoffs
- Grading was conducted independently on both rubrics
- The relative rubric was scored before the absolute rubric for each model
- February 2026

---

*Questions about methodology or results: [tanwa.info](https://tanwa.info)*
