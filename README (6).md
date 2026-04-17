# 🔐 llm-security-lab

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
  - [01 — Intelligent log triage & alert summarization](#01--intelligent-log-triage--alert-summarization)
  - [02 — ATT&CK-mapped threat hunt assistant](#02--attck-mapped-threat-hunt-assistant)
  - [03 — Vulnerability report explainer](#03--vulnerability-report-explainer)
  - [04 — Phishing email classifier](#04--phishing-email-classifier)
  - [05 — Sigma rule generator from natural language](#05--sigma-rule-generator-from-natural-language)
  - [06 — LLM-augmented code vulnerability scanner](#06--llm-augmented-code-vulnerability-scanner)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## About

**llm-security-lab** is a research sandbox for exploring how open-source large language models and established security tooling can be combined into practical, AI-assisted security workflows.

The goal is not to replace analysts — it's to explore where LLMs can reduce toil, surface context faster, and help teams scale detection, response, and analysis capabilities without relying on closed, cloud-hosted AI services.

Every component in this repo is open-source and self-hostable. Sensitive security data never leaves your environment.

---

## Goals

- Demonstrate practical LLM integration patterns for blue team and security engineering use cases
- Provide working, reproducible example pipelines using open models (Llama 3, Mistral, CodeLlama, Phi-3)
- Bridge the gap between security tooling (SIEMs, scanners, CTI platforms) and modern LLM orchestration frameworks
- Serve as a reference for security teams evaluating AI-assisted workflows

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

## Use Cases

---

### 01 — Intelligent log triage & alert summarization

**The problem:** Modern SIEMs generate thousands of alerts daily. Most are noise. Analysts spend hours manually triaging before any real investigation begins.

**The approach:** Pipe Wazuh or Suricata alert JSON into a local LLM to auto-summarize events, group related alerts, classify by severity, and produce a human-readable incident digest — all on-premise.

**Tools:** Ollama · Llama 3 · Wazuh · LangChain · Python

**Example pipeline:**
```
Wazuh alert feed (JSON)
    → LangChain document loader
    → Llama 3 (summarize + classify)
    → Structured incident digest (Markdown / JSON)
```

**Sample prompt pattern:**
```
You are a security analyst. Given the following SIEM alerts, summarize what happened,
identify the most critical events, and assign a severity level (low/medium/high/critical).

Alerts:
{alerts_json}
```

**Getting started:** See [`/use-cases/01-log-triage/`](./use-cases/01-log-triage/)

---

### 02 — ATT&CK-mapped threat hunt assistant

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

**Getting started:** See [`/use-cases/02-attck-rag/`](./use-cases/02-attck-rag/)

---

### 03 — Vulnerability report explainer

**The problem:** CVE descriptions are terse and technical. Translating them into business risk or actionable remediation steps takes time and experience.

**The approach:** Feed CVE details from NVD or Nuclei scan output into an LLM to generate plain-language impact summaries, exploitation context, CVSS score interpretation, and stack-specific remediation guidance.

**Tools:** Nuclei · NVD API · LangChain · GPT4All · Python

**Example output for a given CVE:**
```
CVE-2024-XXXXX — Critical (CVSS 9.8)

Plain-language summary: This vulnerability allows an unauthenticated attacker to
execute arbitrary code on affected Apache servers via a malformed HTTP request header.

Affected versions: Apache HTTP Server 2.4.0 – 2.4.55
Exploitation likelihood: High — public PoC exists
Recommended fix: Upgrade to 2.4.56 immediately. Apply WAF rule to block...
```

**Getting started:** See [`/use-cases/03-vuln-explainer/`](./use-cases/03-vuln-explainer/)

---

### 04 — Phishing email classifier

**The problem:** Email-based threats remain the most common initial access vector. Automated classification is often rule-based and misses novel patterns.

**The approach:** Prompt-engineer or fine-tune a local LLM to analyze email headers, body content, and metadata for phishing indicators. Output structured verdicts with reasoning chains — keeping all email data on-premise.

**Tools:** Ollama · Phi-3 · Python · Open phishing datasets (PhishTank, CISA feeds)

**Verdict schema:**
```json
{
  "verdict": "phishing",
  "confidence": 0.94,
  "indicators": [
    "sender domain registered 2 days ago",
    "urgency language detected",
    "mismatched display name vs envelope-from",
    "link redirects to lookalike domain"
  ],
  "reasoning": "..."
}
```

**Getting started:** See [`/use-cases/04-phishing-classifier/`](./use-cases/04-phishing-classifier/)

---

### 05 — Sigma rule generator from natural language

**The problem:** Writing Sigma detection rules requires knowledge of the Sigma schema, log field names, and correct condition syntax — a barrier for many analysts.

**The approach:** Describe a suspicious behavior in plain English and have an LLM generate a syntactically valid Sigma rule, ready to compile and deploy to your SIEM.

**Tools:** Mistral · Sigma · LangChain · Wazuh · Python

**Example:**

Input:
```
Detect lateral movement where cmd.exe is spawned by a remote service (services.exe)
and connects outbound to a non-standard port.
```

Output:
```yaml
title: Suspicious cmd.exe spawned by services.exe with outbound connection
status: experimental
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage|endswith: '\services.exe'
        Image|endswith: '\cmd.exe'
    condition: selection
level: high
tags:
    - attack.lateral_movement
    - attack.t1021
```

**Getting started:** See [`/use-cases/05-sigma-generator/`](./use-cases/05-sigma-generator/)

---

### 06 — LLM-augmented code vulnerability scanner

**The problem:** Static analysis tools (SAST) produce many findings with limited context — developers need to understand exploitability and fix guidance, not just line numbers.

**The approach:** Run Semgrep to identify flagged code patterns, then pass each finding with surrounding code context to a local LLM for deep reasoning: Is this actually exploitable? What's the attack vector? What's the minimal fix?

**Tools:** Semgrep · CodeLlama · Python · LangChain

**Example pipeline:**
```
Source code repository
    → Semgrep scan (structured JSON findings)
    → Per-finding: code snippet + rule metadata → CodeLlama
    → Enriched report: exploitability, attack vector, fix suggestion
```

**Getting started:** See [`/use-cases/06-code-vuln-scanner/`](./use-cases/06-code-vuln-scanner/)

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
│   ├── 01-log-triage/
│   │   ├── README.md
│   │   ├── triage_pipeline.py
│   │   └── sample_alerts.json
│   ├── 02-attck-rag/
│   │   ├── README.md
│   │   ├── ingest_attck.py
│   │   └── query_attck.py
│   ├── 03-vuln-explainer/
│   │   ├── README.md
│   │   └── explain_cve.py
│   ├── 04-phishing-classifier/
│   │   ├── README.md
│   │   └── classify_email.py
│   ├── 05-sigma-generator/
│   │   ├── README.md
│   │   └── generate_sigma.py
│   └── 06-code-vuln-scanner/
│       ├── README.md
│       └── scan_and_enrich.py
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
