# 🎯 Use Case 01 — ATT&CK-Mapped Threat Hunt Assistant
## Evaluation Framework, KPIs & Evals

> A standalone reference for measuring the quality, reliability, and operational usefulness of the agentic RAG pipeline that maps natural-language threat hunt queries to MITRE ATT&CK techniques.

---

## 📖 Table of Contents

- [Overview](#overview)
- [Evaluation Philosophy](#evaluation-philosophy)
- [Agentic Framework](#agentic-framework)
- [KPI Definitions](#kpi-definitions)
  - [Layer 1 — Retrieval KPIs](#layer-1--retrieval-kpis)
  - [Layer 2 — Reasoning KPIs](#layer-2--reasoning-kpis)
  - [Layer 3 — Operational KPIs](#layer-3--operational-kpis)
- [Eval Stack](#eval-stack)
- [Golden Dataset](#golden-dataset)
- [Running Evals](#running-evals)
- [Eval Report Schema](#eval-report-schema)
- [Regression Testing](#regression-testing)
- [File Structure](#file-structure)

---

## Overview

The ATT&CK-mapped threat hunt assistant is an agentic RAG system that takes a natural-language query from an analyst — such as *"What TTPs are associated with PowerShell-based lateral movement?"* — and returns mapped MITRE ATT&CK technique IDs, tactic context, explanation, and suggested detection logic.

Because this system is agentic (it plans, retrieves, reasons, and produces actionable output across multiple steps), standard LLM benchmarks are insufficient. This document defines a layered evaluation framework purpose-built for this use case.

**System under evaluation:**

```
Analyst query (natural language)
    → ChromaDB semantic retrieval (ATT&CK STIX chunks)
    → Mistral / Llama 3 reasoning + TTP mapping
    → Structured response: technique IDs, tactic, detection hints
    → Optional: Sigma rule generation
```

---

## Evaluation Philosophy

An agentic RAG system has distinct failure modes at each stage of the pipeline. A response can retrieve the right techniques but reason poorly about them — or reason well from entirely wrong context. Each layer must be evaluated independently.

```
┌──────────────────────────────────────────────────────────────────┐
│  LAYER 1: Retrieval                                              │
│  Did the right ATT&CK content get surfaced from ChromaDB?        │
│  Failure mode: wrong chunks retrieved → correct reasoning,       │
│  wrong techniques                                                │
├──────────────────────────────────────────────────────────────────┤
│  LAYER 2: Reasoning                                              │
│  Did the LLM correctly map, explain, and connect TTPs?           │
│  Failure mode: hallucinated technique IDs, wrong tactic mapping  │
├──────────────────────────────────────────────────────────────────┤
│  LAYER 3: Operational                                            │
│  Is the output actually useful to a real threat hunter?          │
│  Failure mode: technically correct but unusable in practice      │
└──────────────────────────────────────────────────────────────────┘
```

**Core principle:** A green score at Layer 2 does not guarantee green at Layer 1. Run all three layers on every eval cycle.

---

## Agentic Framework

The system follows a **Plan → Retrieve → Reason → Act** loop. Each step is independently observable and testable.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AGENTIC LOOP                                 │
│                                                                     │
│  1. PLAN          Parse query intent, identify required TTPs,       │
│                   tactics, or adversary goals                       │
│                                                                     │
│  2. RETRIEVE      Semantic search over ATT&CK STIX chunks           │
│                   in ChromaDB (top-K results)                       │
│                                                                     │
│  3. REASON        LLM maps retrieved context to technique IDs,      │
│                   explains relationships, resolves ambiguity        │
│                                                                     │
│  4. ACT           Produce structured output: T#### IDs, tactic,    │
│                   detection hints, optional Sigma rule              │
│                                                                     │
│  5. VALIDATE      (Optional) Self-check — verify T#### IDs exist    │
│                   in ATT&CK; re-retrieve if confidence is low       │
└─────────────────────────────────────────────────────────────────────┘
```

### Agent tools

| Tool | Purpose | Invocation |
|---|---|---|
| `attck_search` | Semantic search over ChromaDB ATT&CK index | Every query |
| `technique_lookup` | Exact lookup of a T#### ID from STIX bundle | Validation step |
| `sigma_generator` | Generate Sigma rule from technique context | On request |
| `tactic_resolver` | Map a technique ID to its parent tactic | Reasoning aid |

---

## KPI Definitions

### Layer 1 — Retrieval KPIs

These measure whether the right ATT&CK content is being surfaced from ChromaDB before the LLM ever sees it.

| KPI | Definition | Target | Measurement Method |
|---|---|---|---|
| **Retrieval Precision@K** | % of top-K retrieved chunks relevant to the query | ≥ 80% at K=5 | Human-label a golden eval set; compare retrieved chunk IDs to labelled relevant set |
| **Technique ID Recall** | % of ground-truth technique IDs present in retrieved context | ≥ 85% | Extract T#### IDs from retrieved chunks; compare to golden dataset ground truth |
| **Mean Reciprocal Rank (MRR)** | Average of 1/rank of the first correct technique in retrieved results | ≥ 0.75 | Compute per query; average across eval set |
| **Embedding Coverage** | % of all ATT&CK techniques (current version) indexed in ChromaDB | 100% | `len(collection.get()['ids'])` vs total technique count in STIX bundle |
| **Chunk Relevance Score** | Average cosine similarity score of retrieved chunks for labelled-relevant queries | ≥ 0.72 | Read similarity scores from ChromaDB query results |

### Layer 2 — Reasoning KPIs

These measure whether the LLM correctly interprets retrieved context and produces accurate TTP mappings.

| KPI | Definition | Target | Measurement Method |
|---|---|---|---|
| **TTP Mapping Accuracy** | % of LLM-suggested T#### IDs that match ground truth | ≥ 75% | Regex-extract `T\d{4}(\.\d{3})?` from response; compare to labelled ground truth |
| **Hallucination Rate** | % of responses containing at least one non-existent T#### ID | ≤ 5% | Validate all extracted IDs against ATT&CK STIX bundle; flag any not found |
| **Tactic Alignment** | % of suggested techniques whose tactic correctly matches the query's intended tactic | ≥ 80% | Cross-reference technique-to-tactic mapping in STIX data |
| **Reasoning Coherence** | LLM-as-judge score (1–5) rating logical consistency of the explanation | ≥ 3.5 avg | Run a secondary judge prompt on each response; average scores across eval set |
| **Sub-technique Specificity** | % of responses that correctly use sub-techniques (T####.###) when relevant | ≥ 60% | Check whether ground truth sub-techniques appear when the query warrants specificity |
| **Context Faithfulness** | % of claims in the response traceable back to retrieved ATT&CK chunks | ≥ 85% | Use RAGAs `faithfulness` scorer with retrieved chunks as reference |

### Layer 3 — Operational KPIs

These measure whether the output is useful in a real analyst workflow.

| KPI | Definition | Target | Measurement Method |
|---|---|---|---|
| **Detection Query Validity** | % of generated Sigma / KQL / SPL queries that are syntactically valid | ≥ 90% | Run `sigma-cli convert` on Sigma output; use KQL parser for Sentinel queries |
| **Analyst Acceptance Rate** | % of agent responses rated useful or actionable by a human analyst | ≥ 70% | Collect structured analyst feedback (useful / partially useful / not useful) |
| **Response Latency P95** | 95th percentile end-to-end response time under normal load | ≤ 15s (local Ollama) | Instrument pipeline with `time.perf_counter()`; log per query |
| **Response Completeness** | % of responses that include all expected output fields (IDs, tactic, description, detection hint) | ≥ 90% | Parse structured output schema; check for missing required fields |
| **Tactic Coverage per Session** | Number of distinct tactics covered when analyst runs a full hunt session (10 queries) | ≥ 5 of 14 tactics | Count distinct tactic values across a session's technique outputs |

---

## Eval Stack

```
Golden dataset (JSONL)
    │
    ├── RAGAs ──────────────── faithfulness, context recall, answer relevance
    ├── DeepEval ───────────── hallucination, correctness, coherence scoring
    ├── TTP ID Validator ────── validate T#### against ATT&CK STIX bundle
    ├── Sigma CLI ──────────── syntax validation for generated detection rules
    └── LLM-as-Judge ────────── coherence and reasoning quality (1–5 score)
            │
            └── eval_report.json  (per-query + summary metrics)
```

### Tools

| Tool | Version | Purpose | Install |
|---|---|---|---|
| [RAGAs](https://github.com/explodinggradients/ragas) | ≥ 0.1 | RAG-specific metrics: faithfulness, context recall, answer relevance | `pip install ragas` |
| [DeepEval](https://github.com/confident-ai/deepeval) | ≥ 0.20 | Hallucination detection, G-Eval coherence scoring | `pip install deepeval` |
| [sigma-cli](https://github.com/SigmaHQ/sigma-cli) | ≥ 0.8 | Validate and convert generated Sigma rules | `pip install sigma-cli` |
| Custom `ttp_id_validator.py` | — | Cross-reference extracted T#### IDs against STIX bundle | See `/evals/validators/` |
| Custom `llm_judge.py` | — | LLM-as-judge prompt for coherence scoring | See `/evals/validators/` |

---

## Golden Dataset

### Format

Store at `evals/02-attck-rag/golden_dataset.jsonl`. Each line is one labelled evaluation case.

```jsonl
{
  "id": "uc02-001",
  "tactic": "lateral-movement",
  "query": "What TTPs are associated with PowerShell-based lateral movement?",
  "expected_technique_ids": ["T1059.001", "T1021.006", "T1570"],
  "expected_tactics": ["execution", "lateral-movement"],
  "difficulty": "medium",
  "notes": "Should surface PowerShell remoting and script block logging evasion"
}
{
  "id": "uc02-002",
  "tactic": "persistence",
  "query": "How do adversaries establish persistence using scheduled tasks on Windows?",
  "expected_technique_ids": ["T1053.005"],
  "expected_tactics": ["persistence", "privilege-escalation"],
  "difficulty": "easy",
  "notes": "Core Windows persistence technique — should be high-confidence retrieval"
}
{
  "id": "uc02-003",
  "tactic": "initial-access",
  "query": "What techniques are used in supply chain attacks?",
  "expected_technique_ids": ["T1195", "T1195.001", "T1195.002", "T1195.003"],
  "expected_tactics": ["initial-access"],
  "difficulty": "hard",
  "notes": "Tests multi-technique retrieval for a broad adversary goal"
}
{
  "id": "uc02-004",
  "tactic": "defense-evasion",
  "query": "How do attackers disable Windows Event Logging to avoid detection?",
  "expected_technique_ids": ["T1562.002", "T1562.006"],
  "expected_tactics": ["defense-evasion"],
  "difficulty": "medium",
  "notes": "Sub-technique specificity check — should not just return T1562 parent"
}
{
  "id": "uc02-005",
  "tactic": "exfiltration",
  "query": "What methods do threat actors use to exfiltrate data over DNS?",
  "expected_technique_ids": ["T1048.003", "T1071.004"],
  "expected_tactics": ["exfiltration", "command-and-control"],
  "difficulty": "hard",
  "notes": "Cross-tactic query — tests whether agent surfaces both exfil and C2 angles"
}
```

### Dataset composition guidelines

Build towards **100 labelled queries** with the following distribution:

| Dimension | Requirement |
|---|---|
| Tactic coverage | At least 5 queries per ATT&CK tactic (14 tactics total) |
| Difficulty split | 30% easy · 50% medium · 20% hard |
| Query types | 40% specific technique queries · 35% adversary goal queries · 25% cross-tactic queries |
| Sub-technique coverage | At least 30 queries that require sub-technique specificity |
| Platform coverage | Mix of Windows, Linux, macOS, and cloud-focused queries |

---

## Running Evals

### Prerequisites

```bash
pip install ragas deepeval sigma-cli
ollama pull mistral   # or whichever model you are evaluating
```

### Full eval run

```bash
cd evals/02-attck-rag

python run_evals.py \
  --dataset golden_dataset.jsonl \
  --model mistral \
  --top-k 5 \
  --output reports/eval_$(date +%Y%m%d_%H%M).json
```

### Run a single layer only

```bash
# Retrieval layer only (fast — no LLM calls)
python run_evals.py --dataset golden_dataset.jsonl --layer retrieval

# Reasoning layer only
python run_evals.py --dataset golden_dataset.jsonl --layer reasoning --model mistral

# Operational layer only (requires human feedback file)
python run_evals.py --layer operational --feedback analyst_feedback.json
```

### `run_evals.py` execution steps

```
1. Load golden_dataset.jsonl
2. For each query:
   a. Run ChromaDB semantic search (top-K) → record retrieved chunk IDs + scores
   b. Run LLM pipeline → record full response text + latency
   c. Extract T#### IDs from response using regex
   d. Validate extracted IDs against ATT&CK STIX bundle
   e. Score retrieval: precision@K, recall, MRR
   f. Score reasoning: TTP accuracy, hallucination flag, tactic alignment
   g. Score faithfulness via RAGAs
   h. Score coherence via LLM-as-judge
   i. If Sigma rule present: validate with sigma-cli
3. Aggregate per-layer summaries
4. Write eval_report.json
5. Print pass/fail summary to stdout
```

---

## Eval Report Schema

```json
{
  "run_id": "eval-20241101-mistral-k5",
  "timestamp": "2024-11-01T14:32:00Z",
  "model": "mistral",
  "top_k": 5,
  "attck_version": "14.1",
  "dataset_size": 100,

  "summary": {
    "layer_1_retrieval": {
      "precision_at_5": 0.83,
      "technique_id_recall": 0.87,
      "mrr": 0.79,
      "embedding_coverage": 1.0,
      "avg_chunk_relevance_score": 0.76
    },
    "layer_2_reasoning": {
      "ttp_mapping_accuracy": 0.78,
      "hallucination_rate": 0.03,
      "tactic_alignment": 0.82,
      "reasoning_coherence_avg": 3.7,
      "sub_technique_specificity": 0.64,
      "context_faithfulness": 0.88
    },
    "layer_3_operational": {
      "detection_query_validity": 0.91,
      "analyst_acceptance_rate": 0.74,
      "response_latency_p95_ms": 11400,
      "response_completeness": 0.93,
      "tactic_coverage_per_session": 6.2
    }
  },

  "kpi_pass_fail": {
    "precision_at_5": "PASS",
    "technique_id_recall": "PASS",
    "mrr": "PASS",
    "ttp_mapping_accuracy": "PASS",
    "hallucination_rate": "PASS",
    "reasoning_coherence_avg": "PASS",
    "detection_query_validity": "PASS",
    "analyst_acceptance_rate": "PASS",
    "response_latency_p95_ms": "PASS"
  },

  "per_query_results": [
    {
      "id": "uc02-001",
      "query": "What TTPs are associated with PowerShell-based lateral movement?",
      "retrieved_chunk_ids": ["T1059.001-desc", "T1021.006-desc", "T1570-desc", "T1059-desc", "T1105-desc"],
      "extracted_technique_ids": ["T1059.001", "T1021.006", "T1570"],
      "ground_truth_ids": ["T1059.001", "T1021.006", "T1570"],
      "hallucinated_ids": [],
      "ttp_accuracy": 1.0,
      "faithfulness": 0.94,
      "coherence_score": 4.0,
      "sigma_valid": true,
      "latency_ms": 8200,
      "difficulty": "medium"
    }
  ]
}
```

---

## Regression Testing

Run evals on every model change, prompt update, or ATT&CK version upgrade to catch regressions before they reach analysts.

```bash
python compare_evals.py \
  --baseline reports/eval_baseline.json \
  --candidate reports/eval_candidate.json \
  --output reports/diff_report.md
```

### Metric history tracking

Maintain a `reports/history.csv` for trend visibility:

```
run_id                        | p@5  | recall | halluc | coherence | latency_p95
------------------------------|------|--------|--------|-----------|------------
mistral-k5-baseline           | 0.79 | 0.82   | 0.06   | 3.4       | 13100ms
mistral-k5-prompt-v2          | 0.83 | 0.87   | 0.03   | 3.7       | 11400ms  ✅
llama3-k5                     | 0.81 | 0.84   | 0.04   | 3.9       | 14800ms
mistral-k7-attck-v14          | 0.85 | 0.89   | 0.03   | 3.8       | 12100ms  ✅
```

### Regression alert thresholds

Fail the eval run and block promotion if any of the following regress beyond their threshold:

| Metric | Max allowed regression |
|---|---|
| Retrieval Precision@5 | > 5 percentage points drop |
| Hallucination Rate | Any increase above 5% |
| TTP Mapping Accuracy | > 5 percentage points drop |
| Response Latency P95 | > 20% increase |
| Analyst Acceptance Rate | > 10 percentage points drop |

---

## File Structure

```
evals/
└── 02-attck-rag/
    ├── golden_dataset.jsonl          # Labelled query-answer pairs (100 entries)
    ├── run_evals.py                  # Main eval runner
    ├── compare_evals.py              # Regression diff tool
    ├── analyst_feedback.json         # Human analyst ratings (collected separately)
    │
    ├── validators/
    │   ├── ttp_id_validator.py       # Validate T#### IDs against ATT&CK STIX bundle
    │   ├── sigma_validator.py        # Run sigma-cli on generated Sigma rules
    │   ├── llm_judge.py              # LLM-as-judge coherence prompt + scorer
    │   └── schema_validator.py       # Check response completeness against output schema
    │
    └── reports/
        ├── eval_baseline.json        # Baseline eval run (locked reference)
        ├── history.csv               # Metric trend log across all runs
        └── ...                       # Timestamped eval reports
```

---

> This framework is designed to evolve alongside the ATT&CK dataset and the agent itself. Re-run the full eval suite whenever the ATT&CK version is updated, the embedding model is changed, the prompt templates are modified, or the retrieval top-K is tuned.
