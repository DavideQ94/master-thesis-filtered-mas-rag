# Dynamic Filtered Multi-Agent RAG for Structured Data Querying

**Master Thesis** — Comparative evaluation of LLM-based agent architectures for answering natural-language questions over a relational healthcare database.

> **Main entry point:** `notebooks/00_MAIN`  
> `00_MAIN` is the central experiment orchestrator. It executes the complete thesis experiment across **3 repetitions × 4 architectures × 25 evaluation questions**, resulting in **300 runs**. The architecture notebooks are called and coordinated from this notebook and normally do not need to be executed independently.

## Overview

This repository contains the implementation, evaluation framework, data, and results used to compare four LLM-based architectures for analytical question answering. All architectures operate on the same 10 pseudonymised relational tables through Databricks SQL and are evaluated under the same experimental setup.

The four evaluated architectures are:

1. Single-Agent System (SAS)
2. Single-Agent System with RAG (SAS+RAG)
3. Multi-Agent System with RAG (MAS+RAG)
4. Dynamic Filtered Multi-Agent System with RAG

The central research question is whether dynamic filtering can retain or improve the quality of multi-agent reasoning while reducing its computational overhead.

## Architectures

| # | Architecture | RAG | Multi-Agent | Semantic Delivery | Notebook |
|---|---|---|---|---|---|
| 1 | **SAS** — Single-Agent System | No | No | Complete semantic layer | `01_Single_Agent_System_SAS` |
| 2 | **SAS+RAG** — Single Agent with Retrieval | Yes | No | Question-specific retrieval | `02_RAG_Single_Agent` |
| 3 | **MAS+RAG** — Multi-Agent with Retrieval | Yes | Yes, static | Retrieved context | `03_MAS_RAG` |
| 4 | **Dynamic Filtered MAS+RAG** | Yes | Yes, adaptive | Retrieved and filtered context | `04_Dynamic_Filtered_MAS_RAG` |

### Multi-Agent Roles

| Agent | Role |
|---|---|
| Domain Expert | Interprets domain terminology and business requirements |
| Data Engineer | Identifies relevant tables, columns, and relationships |
| SQL Developer | Generates and executes Databricks SQL queries |
| Quantitative Analyst | Interprets numerical results and performs additional calculations |
| Quality Auditor | Validates evidence and produces the final response |

## Key Results

| Architecture | F1 ↑ | Precision ↑ | Recall ↑ | SQL Success | Hallucination ↓ | Groundedness ↑ | Latency | Tokens |
|---|---|---|---|---|---|---|---|---|
| SAS | 0.108 ± 0.014 | 0.184 ± 0.031 | 0.115 ± 0.038 | 35% | 0.565 ± 0.064 | 0.616 ± 0.049 | 6.0s | 17,591 |
| SAS+RAG | 0.100 ± 0.016 | 0.203 ± 0.023 | 0.123 ± 0.030 | 29% | 0.479 ± 0.037 | 0.603 ± 0.009 | 5.9s | 12,062 |
| MAS+RAG | 0.170 ± 0.057 | 0.224 ± 0.067 | 0.194 ± 0.073 | 43% | 0.371 ± 0.026 | 0.396 ± 0.047 | 14.3s | 40,880 |
| **Dynamic Filtered MAS+RAG** | **0.263 ± 0.017** | **0.272 ± 0.033** | **0.301 ± 0.012** | 33% | **0.202 ± 0.045** | **0.787 ± 0.032** | 9.3s | 21,926 |

Dynamic Filtered MAS+RAG improves the quality--efficiency trade-off relative to static MAS+RAG, increasing F1 and groundedness while reducing hallucination risk, token consumption, and latency.

## Repository Structure

```text
├── configs/
│   └── mt_config                          # Catalog paths, model endpoint, LLM parameters,
│                                          # agent profiles, approved tables, safety rules
│
├── src/
│   ├── mt_src                             # Shared utility functions
│   ├── mt_logging                         # Logging functions and Delta table setup
│   ├── mt_evaluation                      # Claim-level evaluation and groundedness
│   ├── mt_evaluation_questions            # 25 evaluation questions and reference claims
│   ├── mt_context_builder                 # Technical and semantic context construction
│   └── mt_retrieval                       # Vector-search retrieval layer
│
├── notebooks/
│   ├── 00_MAIN                            # MAIN ORCHESTRATOR:
│   │                                      # runs 3 repetitions × 4 architectures
│   │                                      # × 25 questions = 300 experiment runs
│   ├── 01_Single_Agent_System_SAS         # Architecture 1 — SAS
│   ├── 02_RAG_Single_Agent                # Architecture 2 — SAS+RAG
│   ├── 03_MAS_RAG                         # Architecture 3 — MAS+RAG
│   ├── 04_Dynamic_Filtered_MAS_RAG        # Architecture 4 — Dynamic Filtered MAS+RAG
│   ├── mt_experiment_3rep                 # Supporting experiment notebook
│   ├── mt_experiment_thesis_final         # Final thesis experiment utilities
│   ├── mt_benchmark                       # Benchmark utilities
│   └── mt_restore_results                 # Results recovery utility
│
├── results/
│   ├── mt_results                         # Results summary notebook
│   ├── mt_figures                         # General figures
│   ├── mt_thesis_figures                  # Thesis figures
│   ├── mt_thesis_pdf_figures              # PDF comparison figures
│   └── thesis_figures/
│       ├── comp1_sas_vs_sas_rag.pdf
│       ├── comp2_sas_rag_vs_mas_rag.pdf
│       └── comp3_mas_rag_vs_dynamic.pdf
│
├── data/
│   ├── tables/                            # Exported analytical tables
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
│   ├── metadata/                          # Technical schema and semantic layer
│   │   ├── mt_information_schema/
│   │   ├── mt_semantic_table_metadata/
│   │   ├── mt_semantic_column_metadata/
│   │   ├── mt_semantic_metric_definitions/
│   │   ├── mt_semantic_join_rules/
│   │   └── mt_semantic_business_rules/
│   └── experiment_3rep_results.parquet    # Complete 300-run experiment output
│
├── .gitignore
├── README.md
└── requirements.txt
