# MedSecLab

**MedSecLab** is a portfolio-grade reference architecture for securely deploying clinical AI applications in a simulated healthcare environment.

The project connects four related GitHub repositories into one clear story: a security-hardened clinical AI gateway, detection engineering for LLM workloads, adversarial testing, and the homelab infrastructure that ties everything together.

> Goal: build a realistic clinical AI security lab using only synthetic healthcare data, then document the architecture, controls, detections, red-team findings, and lessons learned.

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
| [`medseclab`](#) | Portfolio landing page, architecture, roadmap, infra notes, lessons learned | Phase 0 |
| [`clinical-ai-gateway`](https://github.com/MohsenBah/clinical-ai-gateway) | Secure FastAPI gateway for clinical LLM/RAG workloads | Active |
| [`clinical-ai-detections`](https://github.com/MohsenBah/clinical-ai-detections) | Wazuh rules, Suricata signatures, dashboards, and detection docs | Active |
| [`clinical-ai-redteam`](https://github.com/MohsenBah/clinical-ai-redteam) | Garak/PyRIT testing methodology, findings, mitigations, and MITRE ATLAS mapping | Planned later |

## Main Project Tracks

### Track 1: Secure Clinical AI Inference Pipeline

**Story:** I built and secured an end-to-end RAG pipeline that lets clinicians query synthetic patient records using a local LLM, with PHI redaction, audit logging, and OWASP LLM Top 10 controls.

Planned components:

- **Data layer:** OpenEMR seeded with Synthea synthetic patient data
- **Ingestion:** Synthea/FHIR records processed through Microsoft Presidio
- **Vector database:** Qdrant or Chroma
- **Inference:** Ollama or vLLM serving a local model
- **Gateway:** FastAPI service with validation, rate limiting, output filtering, and audit logging
- **Access layer:** Streamlit or React frontend, optionally accessed through Kasm

Primary repository:

```text
clinical-ai-gateway
```

Main deliverables:

- Working secure AI gateway
- Threat model using STRIDE and optionally LINDDUN
- Security controls mapped to OWASP LLM Top 10, NIST AI RMF, and HIPAA Security Rule technical safeguards
- Demo showing safe querying of synthetic clinical records

## Track 2: SOC Detection Engineering for AI Workloads

**Story:** I developed and tested custom Wazuh rules that detect prompt injection attempts, model exfiltration behavior, and anomalous API usage patterns specific to clinical LLM deployments.

Planned detections:

- Prompt injection signatures
- Role override and jailbreak-style attempts
- Unusual prompt/token volume
- Off-hours access to clinical AI endpoints
- Repeated PHI redaction triggers from one user
- Model file access or tampering anomalies

Primary repository:

```text
clinical-ai-detections
```

Main deliverables:

- Wazuh decoders and rules
- Example logs and validation tests
- Grafana dashboard: **Clinical AI Security Posture**
- MITRE ATLAS coverage matrix
- Blog-style detection engineering writeup

## Track 3: Adversarial Testing and Hardening

**Story:** I conducted a structured red-team exercise against my own clinical AI deployment, documented findings using MITRE ATLAS, implemented mitigations, and retested.

Planned testing:

- Garak LLM vulnerability scans
- PyRIT scenarios
- Manual prompt injection tests
- PHI leakage attempts
- Model extraction and abuse-pattern testing against the lab only

Primary repository:

```text
clinical-ai-redteam
```

Main deliverables:

- Red-team methodology
- Lab-only test scenarios
- Findings report
- MITRE ATLAS mapping
- Mitigations and retest results

## Planned Lab Architecture

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
medseclab/
├── README.md
├── ARCHITECTURE.md
├── docs/
│   ├── roadmap.md
│   ├── lessons-learned.md
│   ├── compliance-coverage.md
│   └── runbook.md
├── infra/
│   ├── proxmox/
│   ├── ansible/
│   └── networking/
├── diagrams/
│   ├── network.png
│   ├── data-flow.png
│   └── threat-model.png
└── related-repos.md
```


## Important Rules

- No real patient data will be used.
- All healthcare data must be synthetic.
- This is a lab/reference architecture, not a production healthcare system.
- Red-team content targets only the author’s own lab environment.
- Failures and limitations will be documented honestly.

## Final Portfolio Outcome

The finished project should include:

- A polished landing repo
- A working secure clinical AI gateway
- Wazuh/SOC detection content for LLM abuse cases
- A structured red-team report
- Architecture diagrams
- Threat model and compliance mapping
- A 5-minute demo video
- Blog posts or writeups explaining the build
