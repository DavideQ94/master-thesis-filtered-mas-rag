# Filtered Multi-Agent RAG for Structured Data Querying

**Master Thesis** — Comparative evaluation of LLM-based agent architectures for answering natural-language questions over a relational healthcare database.

## Overview

This repository contains the full experiment code, evaluation framework, and results for a thesis comparing four progressively complex LLM-based architectures on a common benchmark of 25 business questions. All architectures query the same 10 pseudonymised analytical tables through Databricks SQL and are evaluated with a claim-level precision/recall/F1 framework.

The core research question: **Does dynamic multi-agent filtering improve answer quality and groundedness over simpler single-agent or static multi-agent designs?**

## Architectures

| # | Architecture | RAG | Multi-Agent | Semantic Delivery | Notebook |
|---|---|---|---|---|---|
| 1 | **SAS** — Single Agent System | No | No | Direct (full semantic in context) | `01_Single_Agent_System_SAS` |
| 2 | **SAS+RAG** — Single Agent + Retrieval | Yes | No | Retrieved (top-k=5 chunks) | `02_RAG_Single_Agent` |
| 3 | **MAS+RAG** — Multi-Agent + Retrieval | Yes | Yes (static) | Retrieved | `03_MAS_RAG` |
| 4 | **Dynamic Filtered MAS+RAG** | Yes | Yes (adaptive) | Retrieved + filtered | `04_Dynamic_Filtered_MAS_RAG` |

### Multi-Agent Roles (MAS + Dynamic)

| Agent | Role |
|---|---|
| Domain Expert | Healthcare compliance specialist — interprets business questions |
| Data Engineer | Schema navigator — identifies correct tables, joins, alias columns |
| SQL Developer | Generates precise Databricks SQL queries |
| Quantitative Analyst | Interprets SQL results, computes rates, validates numbers |
| Quality Auditor | Validates grounding, catches hallucinations, produces final answer |

## Key Results (Mean ± Std, 3 Repetitions × 25 Questions)

| Architecture | F1 ↑ | Precision ↑ | Recall ↑ | SQL Success | Hallucination ↓ | Groundedness ↑ | Latency | Tokens |
|---|---|---|---|---|---|---|---|---|
| SAS | 0.108 ± 0.014 | 0.184 ± 0.031 | 0.115 ± 0.038 | 35% | 0.565 ± 0.064 | 0.616 ± 0.049 | 6.0s | 17,591 |
| SAS+RAG | 0.100 ± 0.016 | 0.203 ± 0.023 | 0.123 ± 0.030 | 29% | 0.479 ± 0.037 | 0.603 ± 0.009 | 5.9s | 12,062 |
| MAS+RAG | 0.170 ± 0.057 | 0.224 ± 0.067 | 0.194 ± 0.073 | 43% | 0.371 ± 0.026 | 0.396 ± 0.047 | 14.3s | 40,880 |
| **Dynamic Filtered** | **0.263 ± 0.017** | **0.272 ± 0.033** | **0.301 ± 0.012** | 33% | **0.202 ± 0.045** | **0.787 ± 0.032** | 9.3s | 21,926 |

The Dynamic Filtered MAS+RAG achieves the highest F1 (+55% over MAS+RAG), best groundedness (0.787), and lowest hallucination rate (0.202), while using 46% fewer tokens than static MAS+RAG.

## Repository Structure

```
├── configs/
│   └── mt_config                          # Catalog paths, model endpoint, LLM params,
│                                          # agent profiles, approved tables, safety rules
│
├── src/
│   ├── mt_src                             # Shared utility functions
│   ├── mt_logging                         # v3 logging functions and Delta table setup
│   ├── mt_evaluation                      # Claim-level P/R/F1, groundedness scoring
│   ├── mt_evaluation_questions            # 25 benchmark questions + ground truth claims
│   ├── mt_context_builder                 # Builds technical + semantic context from
│   │                                      # metadata tables (architecture-aware)
│   └── mt_retrieval                       # Vector search / retrieval layer
│
├── notebooks/
│   ├── 00_main                            # Original single-run orchestrator
│   ├── 01_Single_Agent_System_SAS         # Architecture 1 — SAS
│   ├── 02_RAG_Single_Agent                # Architecture 2 — SAS+RAG
│   ├── 03_MAS_RAG                         # Architecture 3 — Multi-Agent RAG
│   ├── 04_Dynamic_Filtered_MAS_RAG        # Architecture 4 — Dynamic Filtered
│   ├── mt_experiment_3rep                 # 3-repetition experiment orchestrator
│   ├── mt_experiment_thesis_final         # Final thesis experiment run
│   ├── mt_benchmark                       # Benchmark suite
│   └── mt_restore_results                 # Results recovery utility
│
├── results/
│   ├── mt_results                         # Results summary notebook
│   ├── mt_figures                         # General figures
│   ├── mt_thesis_figures                  # Thesis figures (matplotlib)
│   ├── mt_thesis_pdf_figures              # PDF-exported comparison figures
│   └── thesis_figures/                    # Generated PDFs
│       ├── comp1_sas_vs_sas_rag.pdf
│       ├── comp2_sas_rag_vs_mas_rag.pdf
│       └── comp3_mas_rag_vs_dynamic.pdf
│
├── data/
│   ├── tables/                            # Exported analytical tables (parquet)
│   │   ├── mt_safe_user/
│   │   ├── mt_safe_company/
│   │   ├── mt_safe_assignment/
│   │   ├── mt_safe_assignment_course_progress/
│   │   ├── mt_safe_course/
│   │   ├── mt_safe_certificate/
│   │   ├── mt_safe_exam_result/
│   │   ├── mt_safe_company_contracts/
│   │   ├── mt_safe_company_type/
│   │   └── mt_safe_opportunity/
│   ├── metadata/                          # Schema and semantic layer (parquet)
│   │   ├── mt_information_schema/
│   │   ├── mt_semantic_table_metadata/
│   │   ├── mt_semantic_column_metadata/
│   │   ├── mt_semantic_metric_definitions/
│   │   ├── mt_semantic_join_rules/
│   │   └── mt_semantic_business_rules/
│   └── experiment_3rep_results.parquet    # Full experiment output (300 runs)
│
├── .gitignore
├── README.md
└── requirements.txt
```

## Data

All data lives in Unity Catalog under `dev_forge_default.mt_davide`. The tables are pseudonymised (PII replaced with synthetic IDs) for thesis publication. CSV exports of all tables are included in `data/` for offline import.

### Analytical Tables (10) — Queried by Agents

| Table | Description | Key Columns |
|---|---|---|
| `mt_safe_user` | User-certificate records (32 cols, includes alias columns) | `user_id`, `course_id`, `assignment_id1`, `certificate_type`, `year`, `month` |
| `mt_safe_company` | Company master data | `company_ID`, `company_Status`, `subTypeId` |
| `mt_safe_assignment` | Assignment definitions | `assignment_ID`, `company_ID1`, `deadline_assignment` |
| `mt_safe_assignment_course_progress` | Course progress per user per assignment | `user_id`, `course_id`, `assignment_id1`, `course_type` |
| `mt_safe_course` | Course catalog | `course_id1`, `course_name` |
| `mt_safe_certificate` | Completed certifications | `id`, `userId`, `courseId`, `assignmentId` |
| `mt_safe_exam_result` | Exam attempts and scores | `examId`, `exam_userId`, `exam_assignmentId` |
| `mt_safe_company_contracts` | Company contract details | `id`, `companyId` |
| `mt_safe_company_type` | Company type lookup (MVZ, Care, Baercare) | `type_id`, `type_name` |
| `mt_safe_opportunity` | Sales opportunities | `opportunity_id`, `account_id` |

### Metadata Tables (6) — Used to Build Agent Context

| Table | Purpose |
|---|---|
| `mt_information_schema` | Technical schema (column names, types, join keys, pseudonymisation flags) |
| `mt_semantic_table_metadata` | Business descriptions per table (grain, business key) |
| `mt_semantic_column_metadata` | Business descriptions per column + synonyms |
| `mt_semantic_metric_definitions` | KPI formulas and definitions |
| `mt_semantic_join_rules` | Semantic join rules between tables |
| `mt_semantic_business_rules` | Business rules and logic (e.g., on-time compliance definition) |

## Evaluation Framework

### 25 Benchmark Questions

| Difficulty | Count | Types |
|---|---|---|
| Easy | 5 | Basic aggregation, temporal counts, simple lookups |
| Medium | 5 | Ranking, trend comparison, segmented metrics |
| Hard | 5 | Multi-metric correlation, derived metrics, churn analysis |
| Very Hard | 5 | Temporal pattern detection, cohort analysis, multi-step joins |
| Unanswerable | 5 | Hallucination tests — data does not exist in schema |

### Metrics

- **Answer Quality**: Claim-level Precision, Recall, F1 (RAGChecker-inspired)
- **Groundedness**: Fraction of generated claims traceable to SQL evidence
- **Hallucination Risk**: Fraction of claims not grounded in evidence
- **SQL Success Rate**: Percentage of questions where valid SQL was generated and executed
- **Efficiency**: Latency (seconds) and total tokens per question
- **Abstention**: Correct abstention on unanswerable questions vs. false abstention on answerable ones

## Technology Stack

| Component | Technology |
|---|---|
| Platform | Databricks (Azure) |
| LLM | Meta-Llama-3.1-8B-Instruct (via Databricks Model Serving) |
| Embeddings | databricks-gte-large-en |
| Data Catalog | Unity Catalog (`dev_forge_default.mt_davide`) |
| Retrieval | Databricks Vector Search (top-k=5, threshold=0.3) |
| Storage | Delta Lake |
| Languages | Python, PySpark, Databricks SQL |

## Reproducibility

1. **Import notebooks** into a Databricks workspace preserving the folder structure
2. **Load data**: Import the CSV files from `data/` into Unity Catalog under `dev_forge_default.mt_davide` (or update `MT_CATALOG`/`MT_SCHEMA` in `configs/mt_config`)
3. **Configure model endpoint**: Ensure `databricks-meta-llama-3-1-8b-instruct` is available as a serving endpoint (or update `MT_MODEL_ENDPOINT` in `configs/mt_config`)
4. **Run**: Execute `notebooks/00_MAIN` — it orchestrates 3 repetitions × 4 architectures × 25 questions
5. **Analyse**: Open notebooks in `results/` to reproduce figures and summary tables
6. **Pre-computed results**: `results/experiment_thesis_results.csv` contains the full 300-run output, enabling analysis without re-running the experiment

> **Note**: The experiment uses `EXPERIMENT_SEED = 42` for reproducibility, but LLM outputs are inherently non-deterministic. Results may vary slightly across runs.

## License

This code is part of a master thesis and is provided for academic purposes.
