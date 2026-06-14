# Network Architecture

MedSecLab clinical AI security lab — logical zones.

```mermaid
flowchart TB
    subgraph Clinical["Clinical Zone"]
        User[Clinician / Demo scripts]
    end

    subgraph DMZ["Application Zone"]
        GW[clinical-ai-gateway<br/>FastAPI :8000]
    end

    subgraph AI["AI / Data Zone"]
        Chroma[(Chroma RAG)]
        Ollama[Ollama LLM]
        Presidio[Presidio]
    end

    subgraph SOC["SOC Zone"]
        Loki[Loki]
        Wazuh[Wazuh Manager]
        Grafana[Grafana]
    end

    subgraph RedTeam["Red Team Zone"]
        Campaign[run_campaign.sh]
        Garak[Garak scanner]
        PyRIT[Multi-turn orchestrator]
    end

    User -->|POST /query| GW
    Campaign --> GW
    Garak --> GW
    PyRIT --> GW
    GW --> Chroma
    GW --> Ollama
    GW --> Presidio
    GW -->|security.log| Loki
    Loki --> Grafana
    Loki --> Wazuh
```

## Zone Summary

| Zone | Components | Role |
|------|------------|------|
| Clinical | Demo scripts, curl | Synthetic clinical queries |
| Application | `clinical-ai-gateway` | Validation, RAG, audit logging |
| AI / Data | Chroma, Ollama, Presidio | Inference and de-identified RAG |
| SOC | Loki, Wazuh, Grafana | Detection and investigation |
| Red Team | Campaign, Garak, PyRIT | Adversarial validation |

All data is synthetic. No production PHI.
