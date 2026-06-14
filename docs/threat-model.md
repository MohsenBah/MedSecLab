# Threat Model — Clinical AI Gateway (STRIDE)

**MedSecLab portfolio** · Lab / reference architecture · Synthetic data only  
**Version:** 1.0 · June 2026  
**Scope:** `clinical-ai-gateway` + `clinical-ai-detections` + `clinical-ai-redteam` lab stack

Related: [roadmap.md](roadmap.md) · [Red team report v1](https://github.com/MohsenBah/clinical-ai-redteam/blob/main/docs/red-team-report-v1.md) · [Compliance matrix](https://github.com/MohsenBah/clinical-ai-detections/blob/main/docs/compliance-matrix.md)

---

## 1. Purpose

This document models threats to the MedSecLab clinical AI pipeline using **STRIDE** (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege).

It connects:

- Architecture controls (gateway)
- Detection rules (Wazuh / Grafana)
- Red team findings (CAI-001–006)

This is a **design and portfolio artifact**, not a production risk assessment.

---

## 2. System Context

```
┌─────────────┐     POST /query      ┌──────────────────────┐
│  Clinician  │ ──────────────────►│ clinical-ai-gateway  │
│  (lab user) │     POST /data/*     │  FastAPI + validation│
└─────────────┘                      └──────────┬───────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────────┐
                    ▼                         ▼                         ▼
             ┌────────────┐           ┌────────────┐           ┌────────────┐
             │   Chroma   │           │   Ollama   │           │ security.log│
             │    RAG     │           │    LLM     │           │  (audit)    │
             └────────────┘           └────────────┘           └──────┬─────┘
                                                                       │
                                                                       ▼
                                                              ┌────────────────┐
                                                              │ Wazuh + Grafana │
                                                              │ (detections)    │
                                                              └────────────────┘
```

### Trust boundaries

| Boundary | Inside | Outside |
|----------|--------|---------|
| B1 | Gateway process, validation middleware | Untrusted user queries |
| B2 | Synthetic patient corpus (Chroma) | User-supplied ingest paths |
| B3 | Audit log → SIEM pipeline | Direct log tampering by client |
| B4 | Ollama inference | Direct client → Ollama (blocked in lab) |

### Primary assets

| Asset | Sensitivity | Location |
|-------|-------------|----------|
| Synthetic patient records | Medium (simulated PHI) | Chroma |
| System prompt / RAG instructions | High | Gateway + LLM context |
| Audit logs (`security.log`) | High (forensics) | Gateway volume |
| Model weights | Medium | Ollama |
| Detection rules | Medium | Wazuh |

---

## 3. STRIDE Analysis

### 3.1 Spoofing (identity)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| S-01 | Attacker forges `user_id` / `session_id` on `/query` | High (lab) | Medium | Client-supplied IDs logged as-is | **100200** correlates by `user_id` | ⚠️ No API auth |
| S-02 | Spoofed ingest source path | Medium | Medium | Path validation on `/data/ingest` | Ingestion audit `event_type=ingestion` | 🟡 Partial |
| S-03 | Impersonate clinical vs admin user | Medium | Medium | `query_category` classification | Grafana category panels | 🟡 No RBAC |

**Red team:** CAI-004 admin abuse — no identity verification that caller is authorized for admin queries.

**Residual risk:** HIPAA §164.312(d) authentication not implemented in the lab gateway.

---

### 3.2 Tampering (data integrity)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| T-01 | RAG poisoning via malicious ingest | Medium | High | Presidio at ingest; `clear_existing` option | Ingestion telemetry only | Documented |
| T-02 | Prompt injection alters model behavior | High | High | Input blocklist (`input_validation.py`) | **100100–100102** | ✅ Tested CAI-001/002 |
| T-03 | Encoded injection bypasses blocklist | Medium | High | URL/Base64 normalization before blocklist | **100100–100102** (`decode_method`) | ✅ Remediated CAI-006 |
| T-04 | Tamper with audit logs | Low | High | File permissions in container | Immutable log shipping (Loki) | 🟡 Lab only |

**Red team:** CAI-006 encoding bypass found and remediated (input normalization). CAI-001/002 blocked at gateway.

---

### 3.3 Repudiation (deny action)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| R-01 | User denies submitting adversarial query | Medium | Medium | `request_id`, timestamp, full audit JSON | All Wazuh rules consume audit log | ✅ |
| R-02 | Deny RAG ingest operation | Low | Medium | Ingestion events with `data_path`, `status` | RAG ingestion Grafana dashboard | ✅ |
| R-03 | Deny PHI probing attempt | Medium | Medium | `query` field on allowed events | **100300** | ✅ CAI-003 |

**Red team:** Every CAI scenario produces auditable `request_id` + `decision` + `reason`.

---

### 3.4 Information disclosure (confidentiality)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| I-01 | System prompt extraction | High | High | Blocklist patterns | **100100**, **100101** | ✅ CAI-002 |
| I-02 | PHI leakage via LLM answers | Medium | Critical | Presidio at ingest; patient ID indirection | **100300** on query keywords | 🟡 CAI-003 |
| I-03 | Synthetic PHI in RAG responses | Medium | Medium | Name→ID lookup; redacted ingest | Query-side detection only | 🟡 Partial |
| I-04 | Admin/config exfiltration via query | Medium | High | None for admin-scope | None dedicated | ❌ CAI-004 gap |
| I-05 | Model extraction / weights access | Low | Medium | No direct Ollama exposure | Not deployed (100600) | Documented |

**Red team:** CAI-003 allowed at gateway, detected at SIEM. CAI-004 pure admin abuse undetected.

---

### 3.5 Denial of service (availability)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| D-01 | Token flooding / long prompts | Medium | Medium | Rate limit placeholder; length bucket | **100400**, **100401** | 🟡 |
| D-02 | Repeated injection probes | High | Low | Per-request block (fast) | **100200** correlation | ✅ CAI-005 |
| D-03 | Ingestion DoS (large files) | Low | Medium | Ingest path validation | Ingestion `status=failed` events | 🟡 |
| D-04 | Ollama resource exhaustion | Medium | Medium | Single-worker lab | Grafana latency panels | 🟡 |

**Red team:** CAI-005 repeated blocks trigger **100200** without exhausting Ollama (blocked pre-LLM).

---

### 3.6 Elevation of privilege (authorization)

| Threat | Description | Likelihood | Impact | Controls | Detection | Status |
|--------|-------------|------------|--------|----------|-----------|--------|
| E-01 | Instruction override / jailbreak | High | High | Blocklist | **100100**, **100102** | ✅ CAI-001 |
| E-02 | Bypass safety via encoding | Medium | High | URL/Base64 normalization | **100100**, **100102** | ✅ Remediated CAI-006 |
| E-03 | Administrative privilege abuse | Medium | High | No scope separation | None (PHI hybrid → **100300** only) | ❌ CAI-004 |
| E-04 | Multi-turn jailbreak (stateful) | Low | Medium | Stateless gateway | **100200** (repeated blocks only) | 🟡 CAI-005 |

**Note:** Gateway does not maintain server-side conversation memory. Multi-turn tests measure per-turn blocking and SIEM correlation, not contextual jailbreak memory.

---

## 4. Attack ID → STRIDE Mapping

| CAI ID | STRIDE | Threat IDs | Gateway | Wazuh | Result |
|--------|--------|------------|---------|-------|--------|
| CAI-001 | T, E | T-02, E-01 | Blocked | 100100, 100102 | ✅ |
| CAI-002 | I, T | I-01, T-02 | Blocked | 100100, 100101 | ✅ |
| CAI-003 | I, R | I-02, R-03 | Allowed | 100300 | ✅ |
| CAI-004 | I, E, S | I-04, E-03, S-03 | Allowed | Partial | ⚠️ Gap |
| CAI-005 | D, E | D-02, E-04 | Blocked | 100100, 100200 | ✅ |
| CAI-006 | T, E | T-03, E-02 | Blocked | 100100, 100102 | ✅ Remediated |

Source: [`clinical-ai-redteam/docs/red-team-report-v1.md`](https://github.com/MohsenBah/clinical-ai-redteam/blob/main/docs/red-team-report-v1.md)

---

## 5. Control Summary

### Preventive (gateway)

| Control | Mitigates | STRIDE |
|---------|-----------|--------|
| Input validation blocklist | T-02, E-01, I-01 | T, E, I |
| Input normalization (URL/Base64 decode) | T-03, E-02 | T, E |
| Presidio at ingest | I-02, T-01 | I, T |
| Rate limiting (placeholder) | D-01 | D |
| Patient ID indirection in RAG | I-03 | I |

### Detective (SIEM)

| Rule | Mitigates | STRIDE |
|------|-----------|--------|
| 100100–100102 | Injection, extraction | T, E, I |
| 100200 | Repeated probing | D, E |
| 100300 | PHI probing | I, R |
| 100400–100401 | Abnormal length | D |

### Gaps (documented residual risk)

| Gap | CAI | Priority | Recommended mitigation |
|-----|-----|----------|------------------------|
| Admin abuse undetected | CAI-004 | Medium | Admin-scope rules or gateway RBAC |
| Nested / multi-layer encoding | CAI-006 | Low | Single-layer URL/Base64 remediated; ML classifier for deeper obfuscation |
| RAG poisoning rules | T-01 | Medium | Wazuh rules on ingestion anomalies |
| No API authentication | S-01, S-03 | High | API keys / OIDC / mTLS |
| Output-side PHI filter | I-02 | Medium | Strengthen `output_filter` |

---

## 6. Data Flow (threat-relevant)

```
User query
    │
    ▼ [B1] Input validation ──── block ──► audit (decision=blocked) ──► Wazuh 100xxx
    │ pass
    ▼
RAG retrieve [B2] ──► Chroma (synthetic records)
    │
    ▼
Ollama generate [B4]
    │
    ▼
Output filter (placeholder)
    │
    ▼
audit (decision=allowed, query=...) ──► Wazuh 100300 if PHI keywords
```

Ingest path:

```
POST /data/ingest [B2]
    │
    ▼
Validate path ──► ingest_data() ──► Presidio ──► Chroma
    │
    ▼
audit (event_type=ingestion) ──► Grafana ingestion dashboard
```

---

## 7. Compliance Cross-Reference

| Framework | STRIDE relevance | Portfolio mapping |
|-----------|------------------|-------------------|
| HIPAA §164.312 | S, R, I, T | [compliance-matrix.md](https://github.com/MohsenBah/clinical-ai-detections/blob/main/docs/compliance-matrix.md) |
| OWASP LLM Top 10 | T, I, E | LLM01 (injection), LLM02 (output), LLM06 (excessive agency) |
| NIST AI RMF | Map, Measure, Manage | Threat model = **Map**; detections = **Measure**; gateway blocks = **Manage** |
| MITRE ATLAS | T, I, E | AML.T0051, AML.T0057 on Wazuh rules |

---

## 8. Validation Evidence

| Layer | Artifact | Proves |
|-------|----------|--------|
| Manual campaign | `run_campaign.sh` + manifest | End-to-end control + detection |
| Red team report | `red-team-report-v1.md` | CAI findings → mitigations |
| Garak | `run_garak.sh` + probe map | Automated probe alignment |
| PyRIT | `run_pyrit.sh` + CAI-005 scenarios | Multi-turn correlation |
| Offline validation | `validate_rules.py --offline` | Rule regression |

---

## 9. References

| Document | Repository |
|----------|------------|
| Attack catalog (CAI-001–006) | `clinical-ai-redteam` |
| Wazuh rules 100100–100401 | `clinical-ai-detections` |
| MITRE ATLAS mapping | `clinical-ai-detections/docs/mitre-atlas-mapping.md` |
| Demo video | `MedSecLab/README.md` |

---

*STRIDE threat model v1 — lab environment, synthetic data only.*
