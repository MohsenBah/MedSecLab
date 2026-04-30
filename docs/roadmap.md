# MedSecLab – Execution Roadmap

This document outlines how the system is built, expanded, and validated over time.

It is not a task checklist. It captures architectural intent, sequencing decisions, and why each layer exists.

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

| Layer | Focus | Outcome |
|------|------|--------|
| AI Gateway | Secure inference interface | Controlled LLM access |
| Data Integration | Clinical + synthetic data | Realistic workload |
| Observability | Logs, metrics, traces | Visibility into behavior |
| Detection | Security rules & alerts | Threat awareness |
| Adversarial Testing | Attack simulation | System hardening |

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

---

## 4. Data Layer Integration

Introduce clinical context using synthetic data.

### Components

- Synthea-generated patient data
- OpenEMR as a FHIR-compatible system
- Ingestion pipeline:
  - PHI detection (Presidio)
  - Redaction
  - Embedding generation
  - Storage (Qdrant / Chroma)

### Key Constraint

No real patient data is ever used.

---

## 5. Security Controls (Gateway Hardening)

The gateway evolves into a security boundary.

### Controls Introduced

- Prompt injection filtering
- Output PHI leakage detection
- Rate limiting per user/session
- Structured audit logs (forwarded to SIEM)
- Service-to-service authentication (mTLS or token-based)

### Goal

Shift from “API” → “Security enforcement layer”

---

## 6. Observability and Logging

All system activity becomes traceable.

### Signals Collected

- API requests (input/output metadata)
- Token usage patterns
- PHI detection events
- Authentication events

### Output

- Centralized logs
- Queryable security events
- Input for detection engineering

---

## 7. Detection Engineering

Detection logic is built on top of observed behavior.

### Detection Targets

- Prompt injection attempts
- Abnormal prompt size / token usage
- Repeated PHI-related queries
- Off-hours access patterns
- Model or system tampering

### Implementation

- Wazuh rules (primary)
- Optional Suricata signals
- Grafana dashboards for visibility

### Outcome

System becomes actively monitored, not just protected.

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
- Custom attack scenarios

### Output

- Documented findings
- Mapped to MITRE ATLAS
- Mitigations implemented and verified

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

## 10. Operational Constraints

This lab is intentionally designed with real-world limitations:

- Limited RAM → selective service activation
- Local-only models → performance trade-offs
- Segmented network → controlled access paths

These constraints are part of the design, not problems to eliminate.

---

## 11. Success Criteria

The system is considered complete when:

- A clinician can query synthetic patient data through a secured interface
- All requests are logged, auditable, and observable
- Security controls actively prevent misuse
- Detection rules identify abnormal behavior
- Red team testing produces findings and verified mitigations
- All components are documented and reproducible

---

## 12. Guiding Principle

This is not a collection of tools.

This is a **reference architecture** for securing clinical AI systems.

Every component must answer:

> What risk does this mitigate?

If it doesn’t, it doesn’t belong.