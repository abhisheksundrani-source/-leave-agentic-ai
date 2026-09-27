# Leave Agentic AI

**Project**: Leave Agentic AI — multi‑agent system to intake, validate, predict, and submit employee leave requests with automated handling for casual/one‑day sick leaves and manager escalation for higher‑impact cases.

**Repository**: `leave-agentic-ai`

---

## Overview

This project implements an **agentic, multi‑agent** leave application system that:

- Provides a **user‑facing chatbot** to collect leave requests.
- Splits work across specialized agents: intake, balance validation, calendar/conflict analysis, policy enforcement, document verification, prediction/recommendation, and submission.
- Supports **auto‑grant** for trivial cases (e.g., casual 1‑day leave) when checks pass.
- Validates medical documents (OCR + classifier) for sick/extended leaves and uses a confidence score to decide auto‑submission vs manager escalation.
- Uses **RAG** (vector DB) to ground LLM responses with policy excerpts and past cases.
- Maintains **auditability, explainability, and human‑in‑the‑loop** review for ambiguous or high‑impact requests.

---

## Goals

1. Reduce time to submit and approve routine leaves.
2. Reduce manager overhead for trivial requests while preserving control for high‑impact cases.
3. Provide transparent, explainable recommendations with provenance.
4. Maintain strict security and privacy for PII and documents.

---

## Architecture (high level)

- **User Agent (Chatbot UI)**: React web app / Teams / Slack app.
- **MCP Orchestrator**: Central workflow manager (Temporal + Kubernetes microservice).
- **Agents**:
  - **Intake Agent**: slot filling, classification.
  - **Balance Validator**: HRIS integration.
  - **Calendar Analyzer**: calendar + project tools.
  - **Policy Engine**: deterministic rules (OPA/Drools).
  - **Doc Verifier**: OCR + classifier for medical docs.
  - **Predictor**: ML model for approval probability and business impact.
  - **Submission Agent**: HRIS write / notifications.
- **RAG Layer**: Vector DB (Pinecone/Weaviate) for policy and past cases.
- **LLMs**: intent parsing (small), generation/explanation (larger), with RAG grounding.
- **Message Bus**: Kafka for events.
- **Storage**: Redis for state, secure DB for audit logs.
- **Monitoring**: Prometheus/Grafana, ELK.

---

## Key flows

1. **Intake**: user requests leave → Intake Agent collects fields and attachments.
2. **Parallel checks**: Balance Validator + Calendar Analyzer run concurrently.
3. **Policy & Doc verification**: Policy Engine applies rules; Doc Verifier validates attachments.
4. **Prediction**: Predictor computes approval probability and business impact.
5. **Decision**:
   - **Auto‑grant** for trivial cases (e.g., 1‑day casual) if confidence ≥ 0.95 and no critical conflicts.
   - **Auto‑submit** for short planned leaves if confidence ≥ 0.80.
   - **Escalate** to manager/HR if confidence < 0.80 or business impact high.
6. **Submission**: Submission Agent writes to HRIS and notifies stakeholders.
7. **Audit & Learning**: store decision, RAG snippets, doc hashes, and outcomes for retraining.

---

## Confidence scoring (example)

Final confidence = weighted sum:
- **balance_score** (0.35)
- **doc_score** (0.25)
- **calendar_penalty** (1 − conflict_score) (0.20)
- **history_score** (0.15)
- **policy_penalty** (0.05)

Thresholds:
- **Auto‑grant**: ≥ 0.95
- **Auto‑submit**: ≥ 0.80
- **Manager review**: < 0.80

---

## Security & compliance

- OAuth2/OIDC for auth; RBAC for service accounts.
- Field‑level encryption for PII; DLP for attachments.
- Immutable audit logs for decisions and RAG provenance.
- Consent flow for calendar access.

---

## Getting started (developer)

1. Clone the repo.
2. Provision infra (see `infra/terraform`).
3. Deploy vector DB and LLM endpoints (or configure Azure OpenAI).
4. Configure HRIS sandbox credentials and calendar API keys.
5. Start orchestrator and agents locally (Docker Compose or minikube).
6. Run tests: `./ci/run-tests.sh`.

---

## Project board & milestones

- **MVP**: Intake, Balance Validator, Submission Agent, basic policy engine, calendar free/busy.
- **Phase 2**: Calendar Analyzer, Doc Verifier, RAG ingestion, LLM responses.
- **Phase 3**: Predictor model, explainability, canary rollout.
- **Phase 4**: Fine‑tuning LLMs, proactive suggestions, federated learning.

---

## Contributing

- Use the issue templates in `.github/ISSUE_TEMPLATE`.
- Follow CODEOWNERS and PR checklist.
- All infra changes require `terraform plan` in CI.

---

## Files of interest

- `api/openapi.yaml` — MCP endpoints and agent contracts.
- `docs/architecture.md` — architecture diagram and component details.
- `services/*` — microservice skeletons for each agent.
- `rags/` — ingestion scripts and schema.

---

## Contact

Project owner: **abhishek** (repository under `abhisheksundrani-source`)

