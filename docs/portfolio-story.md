# Portfolio Story — Securing Clinical AI in the Lab

**MedSecLab** demonstrates how to secure a clinical LLM + RAG deployment using synthetic healthcare data only.

## The Problem

Clinical AI systems combine three high-risk surfaces:

1. **User prompts** — prompt injection and PHI probing
2. **RAG corpora** — synthetic or real patient context
3. **Audit trails** — required for healthcare security investigations

Tooling alone does not tell a defensible story. This portfolio shows **methodology → detection → adversarial proof**.

## What Was Built

### 1. Secure gateway (`clinical-ai-gateway`)

- FastAPI inference API with input validation blocklist
- Presidio-backed RAG ingest with patient ID indirection
- Structured JSON audit logging on every decision
- Demo scripts and recorded walkthrough

### 2. Detection engineering (`clinical-ai-detections`)

- Seven Wazuh rules (100100–100401) mapped to MITRE ATLAS
- Three Grafana dashboards (security, injection, RAG ingestion)
- Compliance matrix: HIPAA §164.312, OWASP LLM, NIST AI RMF
- Automated validation harness (`validate_rules.py --offline`)

### 3. Red team program (`clinical-ai-redteam`)

- Six curated attack IDs (CAI-001–006), not random payload sprawl
- Manual campaign with offline Wazuh validation
- Garak automated scanning aligned to CAI IDs
- Multi-turn orchestration for CAI-005 session correlation

### 4. Threat model (`MedSecLab/docs/threat-model.md`)

- STRIDE analysis across gateway, RAG, LLM, and SIEM
- CAI findings mapped to threats and controls

## The Security Story (One Sentence)

**Attack → Gateway decision → Audit log → Wazuh rule → Grafana panel**

Every tested CAI ID traces through this pipeline with evidence in `red-team-report-v1.md`.

## Key Results

| Attack | Gateway | Detection |
|--------|---------|-----------|
| Direct injection (CAI-001/002) | Blocked | Wazuh 100100–102 |
| PHI probing (CAI-003) | Allowed | Wazuh 100300 |
| Repeated blocks (CAI-005) | Blocked | Wazuh 100200 |
| Encoded injection (CAI-006) | Blocked (after remediation) | Wazuh 100100–102 |
| Admin abuse (CAI-004) | Blocked (after remediation) | Wazuh 100310 (+100300 on PHI hybrid) |
| RAG ingestion abuse | Detected | Wazuh 100320 / 100321 |

## Remediation Cycle (find → fix → verify)

The red team didn't just find gaps — it closed two end-to-end.

**CAI-006 — Encoded injection**

1. **Found** — Base64 and URL-encoded overrides bypassed the literal blocklist (HTTP 200, no alert).
2. **Fixed** — `validate_input()` now decodes URL/Base64 variants before the blocklist check and records a `decode_method` audit field.
3. **Retested** — gateway unit tests + the `blocked-encoded-injection` detection case pass; both variants now return HTTP 400 and fire Wazuh 100100/100102.

**CAI-004 — Administrative privilege abuse**

1. **Found** — credential, account-enumeration, and config/API-key dump requests were answered by the model with no dedicated alert.
2. **Fixed** — added an admin-scope blocklist (`ADMIN_ABUSE_PATTERNS`) that blocks these requests (`reason=blocked_admin_scope:*`) and a new Wazuh rule **100310**.
3. **Retested** — gateway unit tests + the `blocked-admin-credential-exfiltration` detection case pass; both variants now return HTTP 400 and fire 100310. The admin-framed PHI variant stays allowed by design and is detected via 100300.

This is the full find → fix → verify loop a security engineer is expected to own.

## Reproduce It

```bash
# Gateway
cd clinical-ai-gateway && docker compose up

# Full manual campaign
cd clinical-ai-redteam && ./scripts/run_campaign.sh

# Demo video script
cd clinical-ai-gateway && ./demo/05-run-full-demo.sh
```

## Repositories

| Repo | Role |
|------|------|
| [MedSecLab](https://github.com/MohsenBah/MedSecLab) | Portfolio hub |
| [clinical-ai-gateway](https://github.com/MohsenBah/clinical-ai-gateway) | Secure inference |
| [clinical-ai-detections](https://github.com/MohsenBah/clinical-ai-detections) | SIEM + dashboards |
| [clinical-ai-redteam](https://github.com/MohsenBah/clinical-ai-redteam) | Adversarial validation |

## Diagrams

- [Network zones](../diagrams/network.md)
- [Data flow](../diagrams/data-flow.md)
- [Demo video](https://github.com/user-attachments/assets/e31164a2-6abf-4c0c-8143-fafe04147924)
