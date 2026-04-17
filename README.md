# 🔐 LLM-Security-Lab

> Experimenting with open-source LLMs and tools to build AI-assisted cybersecurity workflows — fully self-hostable, no external API required.

![License](https://img.shields.io/badge/license-MIT-green)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![LLM](https://img.shields.io/badge/LLM-Ollama%20%7C%20Llama3%20%7C%20Mistral-blueviolet)
![Security](https://img.shields.io/badge/domain-cybersecurity-red)

---

## 📖 Table of Contents

- [About](#about)
- [Goals](#goals)
- [Tools & LLMs](#tools--llms)
- [Tech Stack](#tech-stack)
- [Use Cases](#use-cases)
  
  - [01 — ATT&CK-mapped threat hunt assistant](#02--attck-mapped-threat-hunt-assistant)

- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## About

**LLM-Security-Lab** is a research sandbox for exploring how open-source large language models and established security tooling can be combined into practical, AI-assisted security workflows.Framework, KPIs and EVALS are on high focus in this lab.

Every component in this repo is open-source and self-hostable. 

---

## Goals

- Provide working, reproducible example pipelines using open models (Llama 3, Mistral, CodeLlama, Phi-3)

---

## Tools & LLMs

### Large Language Models

| Model | Best For | Runtime |
|---|---|---|
| [Llama 3 (Meta)](https://ollama.com/library/llama3) | General reasoning, log analysis, summarization | Ollama |
| [Mistral 7B](https://ollama.com/library/mistral) | Instruction following, rule generation | Ollama |
| [CodeLlama](https://ollama.com/library/codellama) | Code vulnerability analysis, SIEM queries | Ollama |
| [Phi-3 Mini](https://ollama.com/library/phi3) | Lightweight, fast classification tasks | Ollama |
| [GPT4All](https://gpt4all.io/) | Offline assistant, cross-platform alternative | GPT4All runtime |

### Security Tools

| Tool | Purpose | Category |
|---|---|---|
| [Wazuh](https://wazuh.com/) | SIEM, log collection, alert generation | Detection & Response |
| [Suricata](https://suricata.io/) | Network IDS/IPS, PCAP analysis | Network Detection |
| [Zeek](https://zeek.org/) | Network traffic analysis & logging | Network Monitoring |
| [Nuclei](https://github.com/projectdiscovery/nuclei) | Template-based vulnerability scanning | Vulnerability Scanning |
| [Semgrep](https://semgrep.dev/) | Static analysis, code pattern detection | SAST |
| [OpenCTI](https://www.opencti.io/) | Threat intelligence platform | CTI |
| [MISP](https://www.misp-project.org/) | Threat intel sharing, IOC management | CTI |

### Orchestration & Data

| Tool | Purpose |
|---|---|
| [LangChain](https://www.langchain.com/) | LLM chaining, agents, tool use |
| [LlamaIndex](https://www.llamaindex.ai/) | RAG pipelines, document indexing |
| [ChromaDB](https://www.trychroma.com/) | Local vector database for embeddings |
| [MITRE ATT&CK](https://attack.mitre.org/) | Adversary TTP knowledge base |
| [Sigma](https://github.com/SigmaHQ/sigma) | Generic detection rule format |
| [NVD / CVE](https://nvd.nist.gov/) | Vulnerability data feed |

---

## Tech Stack

```
┌─────────────────────────────────────────────────────────┐
│                     LLM LAYER                           │
│         Ollama · Llama 3 · Mistral · CodeLlama          │
├─────────────────────────────────────────────────────────┤
│                  ORCHESTRATION                          │
│          LangChain · LlamaIndex · ChromaDB              │
├─────────────────────────────────────────────────────────┤
│                  SECURITY TOOLS                         │
│       Wazuh · Suricata · Nuclei · Semgrep · MISP        │
├─────────────────────────────────────────────────────────┤
│               KNOWLEDGE BASES                           │
│        MITRE ATT&CK · Sigma Rules · NVD / CVE           │
└─────────────────────────────────────────────────────────┘
```

---

## Use Case



### 01 — ATT&CK-mapped threat hunt assistant

**The problem:** Turning raw observables into mapped TTPs requires deep familiarity with MITRE ATT&CK — expertise that's hard to scale.

**The approach:** Build a RAG (Retrieval-Augmented Generation) system over the full MITRE ATT&CK dataset. Ask natural-language questions like *"What TTPs involve PowerShell-based lateral movement?"* and receive mapped technique IDs, tactic context, and suggested detection queries.

**Tools:** LlamaIndex · Mistral · ChromaDB · MITRE ATT&CK STIX feed · Python

**Example pipeline:**
```
ATT&CK STIX bundle (JSON)
    → LlamaIndex ingestion + ChromaDB vector store
    → User query (natural language)
    → Semantic search → Mistral (reasoning + context)
    → Response: technique IDs, descriptions, detection hints
```
---

## Repository Structure

```
llm-security-lab/
├── README.md
├── .env.example                  # Environment variable template
├── requirements.txt              # Python dependencies
├── docker-compose.yml            # Spin up Ollama + ChromaDB + Wazuh stack
│
├── use-cases/
│   ├── 01-attck-rag/
│   │   ├── README.md
│   │   ├── ingest_attck.py
│   │   └── query_attck.py
│
├── shared/
│   ├── llm_client.py             # Ollama wrapper
│   ├── vector_store.py           # ChromaDB helpers
│   └── prompt_templates/         # Reusable prompt templates
│
├── data/
│   ├── attck/                    # ATT&CK STIX bundle (gitignored if large)
│   ├── sigma-rules/              # Cloned SigmaHQ rule set
│   └── sample-logs/              # Sanitized sample log data
│
└── docs/
    ├── architecture.md
    └── model-selection-guide.md
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- [Ollama](https://ollama.com/) installed and running
- Docker (optional, for full stack)
- 8GB+ RAM recommended (16GB for larger models)

### 1. Clone the repository

```bash
git clone https://github.com/your-username/llm-security-lab.git
cd llm-security-lab
```

### 2. Pull models via Ollama

```bash
ollama pull llama3
ollama pull mistral
ollama pull codellama
ollama pull phi3
```

### 3. Install Python dependencies

```bash
python -m venv venv
source venv/bin/activate       # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Configure environment

```bash
cp .env.example .env
# Edit .env with your paths and any optional API keys (e.g. NVD API key)
```

### 5. Run your first use case

```bash
# Example: log triage
cd use-cases/01-log-triage
python triage_pipeline.py --input sample_alerts.json
```

### Optional: full Docker stack

```bash
docker-compose up -d
# Starts Ollama, ChromaDB, and a local Wazuh demo instance
```

---

## Roadmap

- [x] Repository scaffold and use case documentation
- [ ] Working code for use case 01 (log triage)
- [ ] Working code for use case 02 (ATT&CK RAG)
- [ ] Working code for use case 05 (Sigma generator)
- [ ] Docker Compose full-stack setup
- [ ] Evaluation harness for LLM output quality per use case
- [ ] Fine-tuning guide for security-domain adaptation
- [ ] Integration with OpenCTI via API
- [ ] Web UI (Streamlit/Gradio) for analyst-facing demos

---

## Contributing

Contributions are welcome. To get started:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-use-case`
3. Commit your changes with clear messages
4. Open a pull request with a description of what you've added or changed

Please keep all sample data sanitized and anonymized. Do not commit real credentials, IP addresses, or personally identifiable information.

---

## Disclaimer

This repository is intended for **research, education, and defensive security** purposes only. All example data is synthetic or publicly available. The techniques and tools documented here should only be used in environments you are authorized to test and monitor.

The authors are not responsible for any misuse of the information or code contained in this repository.

---

## License

[MIT License](./LICENSE) — free to use, modify, and distribute with attribution.

---

> Built with open-source tools, for open-source defenders. 🛡️
