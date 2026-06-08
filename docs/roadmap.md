# MedSecLab – Execution Roadmap

This document outlines how the system is built, expanded, and validated over time.

It is not a task checklist. It captures architectural intent, sequencing decisions, and why each layer exists.

**Last updated:** June 2026 — demo video recorded; Phases 3.1A–3.2A complete across gateway and detections repos.

---

## 1. Core Objective

Building a secure, auditable clinical AI system that supports:

- Querying synthetic patient data via LLM
- Enforcing PHI protection boundaries
- Detecting misuse and adversarial behavior
- Simulating real-world deployment constraints

---

## 2. System Evolution Strategy

The system is developed in layers, each adding a new dimension:

| Layer | Focus | Outcome | Status |
|------|------|--------|--------|
| AI Gateway | Secure inference interface | Controlled LLM access | ✅ Active |
| Data Integration | Clinical + synthetic data | Realistic RAG workload | ✅ Chroma + Presidio |
| Observability | Logs, metrics, dashboards | Visibility into behavior | ✅ Promtail → Loki → Grafana |
| Detection | Security rules & alerts | Threat awareness | ✅ Wazuh 100100–100401 + MITRE |
| Adversarial Testing | Attack simulation | System hardening | ⏳ `clinical-ai-redteam` |
| Compliance | Regulatory mapping | Audit-ready documentation | ✅  |

Each layer builds on the previous one. No layer is isolated.

---

## 3. Initial System (AI Gateway)

The starting point is a minimal but secure inference service.

### Components

- FastAPI service (entry point)
- Local LLM (via Ollama)
- Structured logging (JSON logs)
- Basic input validation
- Controlled response handling

### Design Principles

- No direct model exposure
- Every request is logged
- Every response is observable
- Security is enforced at the gateway, not the model

**Repo:** [`clinical-ai-gateway`](https://github.com/MohsenBah/clinical-ai-gateway)

---

## 4. Data Layer Integration

Introduce clinical context using synthetic data.

### Components

- Synthea-style synthetic patient JSON (`data/synthetic_patients.json`)
- Ingestion pipeline:
  - PHI detection (Presidio)
  - Record-ID-based de-identification for RAG retrieval
  - Embedding generation (sentence-transformers)
  - Storage (Chroma)
- Patient name → record ID lookup for demo queries (e.g. Sarah Johnson → PAT002)

### Key Constraint

No real patient data is ever used.

**Open / later:** OpenEMR FHIR integration, Qdrant option.

---

## 5. Security Controls (Gateway Hardening)

The gateway evolves into a security boundary.

### Controls Introduced

- ✅ Prompt injection filtering
- ✅ Rate limiting per user/session
- ✅ Structured audit logs (query + ingestion events)
- ✅ Query classification (`medical`, `administrative`, `adversarial`, `unknown`)
- ✅ Model performance telemetry (tokens, latency)
- Output PHI leakage detection (Presidio at ingest; output filter placeholder)
- Service-to-service authentication (mTLS or token-based) — planned

### Goal

Shift from “API” → “Security enforcement layer”

---

## 6. Observability and Logging

All system activity becomes traceable.

### Signals Collected

- API requests (decision, reason, `query_category`, token counts)
- Ingestion events (`event_type=ingestion`, success/failure)
- Block decision latency on allowed and blocked paths
- PHI probing queries (logged with `query` field for SIEM)

### Output

- `security.log` → Promtail → Loki
- 3 Grafana dashboards (security overview, prompt injection, RAG ingestion)
- Input for Wazuh detection engineering

**Docs:** [`clinical-ai-gateway/docs/audit-logging.md`](https://github.com/MohsenBah/clinical-ai-gateway/blob/main/docs/audit-logging.md)

---

## 7. Detection Engineering

Detection logic is built on top of observed behavior.

### Detection Targets

| Scenario | Rule(s) | Status |
|----------|---------|--------|
| Prompt injection blocked | 100100 | ✅ |
| System prompt extraction | 100101 | ✅ |
| Instruction override | 100102 | ✅ |
| Repeated probing | 100200 | ✅ |
| PHI probing | 100300 | ✅ |
| Abnormal query length | 100400, 100401 | ✅ |
| Off-hours access | 100500 | ⏳ Planned |
| RAG data poisoning | TBD | 🔄 Telemetry ready |
| Model tampering | 100600 | ⏳ Planned |

### Implementation

- Wazuh rules + JSON decoder (primary)
- Grafana dashboards for investigation
- MITRE ATLAS mapping (`AML.T0051`, `AML.T0057`)

**Repo:** [`clinical-ai-detections`](https://github.com/MohsenBah/clinical-ai-detections)

---

## 8. Adversarial Testing

The system is tested from an attacker’s perspective.

### Attack Surface

- Prompt injection
- RAG data poisoning
- PHI extraction attempts
- Model extraction
- API abuse

### Tooling

- Garak
- PyRIT
- Custom attack scenarios (`clinical-ai-gateway/demo/` scripts as baseline)

### Output

- Documented findings
- Mapped to MITRE ATLAS
- Mitigations implemented and verified

**Repo:** [`clinical-ai-redteam`](https://github.com/MohsenBah/clinical-ai-redteam) — planned

---

## 9. Documentation as a First-Class Output

Every layer produces documentation:

- Architecture diagrams
- Threat model (STRIDE / LINDDUN)
- Compliance mapping:
  - OWASP LLM Top 10
  - NIST AI RMF
  - HIPAA §164.312
- Detection coverage matrix
- Red team findings

The documentation is part of the deliverable, not an afterthought.

---

## 10. Demo and Portfolio Artifacts

The end-to-end story is demonstrated, not only documented.

### Demo Video

**Recording:** [GitHub asset](https://github.com/user-attachments/assets/e31164a2-6abf-4c0c-8143-fafe04147924) (embedded in `README.md`)


Shows: RAG ingest → clinical query → administrative query → prompt injection (Wazuh 100100–102, 100200) → PHI probing (Wazuh 100300) → audit log evidence.

### Reproducible Scripts

```bash
# clinical-ai-gateway repo
./demo/05-run-full-demo.sh
```

Individual steps: `01-health-check.sh` through `04-phi-probing.sh`.

---

## 11. Operational Constraints

This lab is intentionally designed with real-world limitations:

- Limited RAM → selective service activation
- Local-only models → performance trade-offs
- Segmented network → controlled access paths

These constraints are part of the design, not problems to eliminate.

---

## 12. Success Criteria

The system is considered complete when:

| Criterion | Status |
|-----------|--------|
| Clinician can query synthetic patient data through a secured interface | ✅ |
| All requests are logged, auditable, and observable | ✅ |
| Security controls actively prevent misuse | ✅ |
| Detection rules identify abnormal behavior | ✅ |
| Demo video shows full pipeline | ✅ [`demo.webm`](demo.webm) |
| MITRE ATLAS mapping for active rules | ✅ |
| Compliance matrix (HIPAA / OWASP / NIST) | ✅ [`compliance-matrix.md`](https://github.com/MohsenBah/clinical-ai-detections/blob/main/docs/compliance-matrix.md) |
| Red team testing produces findings and verified mitigations | ⏳ |
| All components documented and reproducible | 🔄 |

---

## 13. Current Focus (Phase 3.4+)

1. **Validation harness** — automated Wazuh logtest regression
2. **RAG poisoning detections** — Wazuh rules on ingestion telemetry
3. **Red team phase** — Garak/PyRIT campaigns in `clinical-ai-redteam`
4. **Architecture diagrams** — network, data-flow, threat model in `MedSecLab/diagrams/`

---

## 14. Guiding Principle

This is not a collection of tools.

This is a **reference architecture** for securing clinical AI systems.

Every component must answer:

> What risk does this mitigate?

If it doesn’t, it doesn’t belong.
