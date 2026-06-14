
# MedSecLab

**MedSecLab** is a portfolio-grade reference architecture for securely deploying clinical AI applications in a simulated healthcare environment.

The project connects four related GitHub repositories into one clear story: a security-hardened clinical AI gateway, detection engineering for LLM workloads, adversarial testing, and the homelab infrastructure that ties everything together.

> Goal: build a realistic clinical AI security lab using only synthetic healthcare data, then document the architecture, controls, detections, red-team findings, and lessons learned.

## Demo Video

End-to-end walkthrough of the clinical AI security pipeline (~2 min).

https://github.com/user-attachments/assets/e31164a2-6abf-4c0c-8143-fafe04147924



The demo covers:

1. Health check and RAG data ingest (synthetic patients → Chroma)
2. Normal clinical queries (including patient lookup by name)
3. Prompt injection blocks → Wazuh rules **100100**, **100101**, **100102**, **100200**
4. PHI probing (allowed by gateway) → Wazuh rule **100300**
5. Structured audit logging in `security.log`

Reproducible scripts: [`clinical-ai-gateway/demo/`](https://github.com/MohsenBah/clinical-ai-gateway/tree/main/demo) (`05-run-full-demo.sh`)

## Why This Exists

Most homelab projects stop at tool installation: Wazuh, Kasm, OpenEMR, Ollama, or Suricata.

MedSecLab is different. The focus is not simply installing tools. The focus is building a defensible security story around clinical AI:

- How should a local clinical AI system be exposed safely?
- How can synthetic patient records be queried without leaking PHI?
- How can prompt injection and abnormal LLM usage be logged and detected?
- How can red-team findings be mapped to real mitigations?
- How can a small lab simulate enterprise-style healthcare AI security?

## Portfolio Repositories

MedSecLab is the umbrella repo. The technical work is split into focused repositories so each hiring audience can quickly evaluate the part they care about.

| Repository | Purpose | Status |
|---|---|---|
| [`MedSecLab`](https://github.com/MohsenBah/MedSecLab) | Portfolio landing page, architecture, roadmap, demo video | Active |
| [`clinical-ai-gateway`](https://github.com/MohsenBah/clinical-ai-gateway) | Secure FastAPI gateway for clinical LLM/RAG workloads | Active |
| [`clinical-ai-detections`](https://github.com/MohsenBah/clinical-ai-detections) | Wazuh rules, Grafana dashboards, MITRE ATLAS mapping | Active |
| [`clinical-ai-redteam`](https://github.com/MohsenBah/clinical-ai-redteam) | Attack methodology, campaigns, Garak/PyRIT, red team report | Active |

## Main Project Tracks

### Track 1: Secure Clinical AI Inference Pipeline

**Story:** I built and secured an end-to-end RAG pipeline that lets clinicians query synthetic patient records using a local LLM, with PHI redaction, audit logging, and OWASP LLM Top 10 controls.

Components:

- **Data layer:** Synthea synthetic patient JSON (`data/synthetic_patients.json`)
- **Ingestion:** Presidio anonymization → Chroma vector store
- **Inference:** Ollama local LLM
- **Gateway:** FastAPI with validation, rate limiting, output filtering, and audit logging
- **Access:** Demo scripts and curl (Streamlit/Kasm documented as homelab extensions)

Primary repository:

```text
clinical-ai-gateway
```

Main deliverables:

- ✅ Working secure AI gateway with RAG (Chroma, Presidio, Ollama)
- ✅ Demo video and reproducible demo scripts
- ✅ Threat model using STRIDE — [`docs/threat-model.md`](docs/threat-model.md)
- Security controls mapped to OWASP LLM Top 10, NIST AI RMF, and HIPAA Security Rule technical safeguards

## Track 2: SOC Detection Engineering for AI Workloads

**Story:** I developed and tested custom Wazuh rules that detect prompt injection attempts, model exfiltration behavior, and anomalous API usage patterns specific to clinical LLM deployments.

Detections implemented:

- Prompt injection signatures (100100–100102)
- Role override and jailbreak-style attempts (100100, 100200)
- Unusual prompt/token volume (100400, 100401)
- PHI probing from clinical AI endpoints (100300)

Primary repository:

```text
clinical-ai-detections
```

Main deliverables:

- ✅ Wazuh decoders and 7 detection rules (100100–100401)
- ✅ Example logs, logtest notes, and validation samples
- ✅ 3 Grafana dashboards (security overview, prompt injection, RAG ingestion)
- ✅ MITRE ATLAS mapping (`clinical-ai-detections/docs/mitre-atlas-mapping.md`)
- ✅ Compliance matrix (HIPAA / OWASP / NIST)

## Track 3: Adversarial Testing and Hardening

**Story:** I conducted a structured red-team exercise against my own clinical AI deployment, documented findings using MITRE ATLAS, implemented mitigations, and retested.

Testing delivered:

- Garak LLM vulnerability scans mapped to CAI IDs
- PyRIT / stdlib multi-turn scenarios (CAI-005)
- Manual prompt injection tests (CAI-001–006)
- PHI leakage attempts (CAI-003)

Primary repository:

```text
clinical-ai-redteam
```

Main deliverables:

- ✅ Red-team methodology, campaign, Garak, PyRIT
- ✅ Findings report — [`clinical-ai-redteam/docs/red-team-report-v1.md`](https://github.com/MohsenBah/clinical-ai-redteam/blob/main/docs/red-team-report-v1.md)
- ✅ MITRE ATLAS mapping (via detections + attack catalog)
- ✅ Mitigations and retest results documented in red team report

## Lab Architecture

The final lab simulates a small healthcare provider network. It does not need to run all services at the same time.

| Zone | Purpose | Example Services |
|---|---|---|
| Clinical | Simulated healthcare user environment | Win11 workstation, OpenEMR, Synthea data |
| DMZ | Controlled access layer | Kasm, reverse proxy, AI app gateway |
| SOC | Monitoring and detection | Wazuh, Suricata, Grafana, Loki |
| AI/ML | Local AI workload | Ollama/vLLM, FastAPI gateway, Presidio, vector DB |
| Attacker | Red-team testing | Kali Purple, Garak, PyRIT, Atomic Red Team |
| Mgmt/Infra | Management services | OPNsense, DNS, Vault, Gitea |

## Repo Layout

```text
MedSecLab/
├── README.md
├── docs/
│   ├── roadmap.md
│   ├── threat-model.md
│   └── portfolio-story.md
└── diagrams/
    ├── network.md
    └── data-flow.md
```

The end-to-end demo video is published as a GitHub asset and embedded above; it is not stored in the repository.


## Important Rules

- No real patient data will be used.
- All healthcare data must be synthetic.
- This is a lab/reference architecture, not a production healthcare system.
- Red-team content targets only the author’s own lab environment.
- Failures and limitations will be documented honestly.

## Portfolio Status

| Deliverable | Status |
|---|---|
| Landing repo (MedSecLab) | ✅ Active |
| Secure clinical AI gateway + RAG | ✅ [`clinical-ai-gateway`](https://github.com/MohsenBah/clinical-ai-gateway) |
| Wazuh / Grafana detection stack | ✅ [`clinical-ai-detections`](https://github.com/MohsenBah/clinical-ai-detections) |
| End-to-end demo video | ✅ [README embed](https://github.com/user-attachments/assets/e31164a2-6abf-4c0c-8143-fafe04147924) |
| MITRE ATLAS rule mapping | ✅ [`mitre-atlas-mapping.md`](https://github.com/MohsenBah/clinical-ai-detections/blob/main/docs/mitre-atlas-mapping.md) |
| Compliance matrix (HIPAA / OWASP / NIST) | ✅ [`compliance-matrix.md`](https://github.com/MohsenBah/clinical-ai-detections/blob/main/docs/compliance-matrix.md) |
| Structured red-team report | ✅ [`red-team-report-v1`](https://github.com/MohsenBah/clinical-ai-redteam/blob/main/docs/red-team-report-v1.md) |
| STRIDE threat model | ✅ [`docs/threat-model.md`](docs/threat-model.md) |
| Architecture diagrams | ✅ [`diagrams/`](diagrams/) |
| Portfolio writeup | ✅ [`docs/portfolio-story.md`](docs/portfolio-story.md) |
