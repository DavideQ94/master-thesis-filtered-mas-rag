# Dynamic Filtered Multi-Agent with RAG

This repository contains the implementation used for the thesis experiments comparing four LLM-based architectures: a Single-Agent System (SAS), SAS with RAG, Multi-Agent with RAG, and Dynamic Filtered Multi-Agent with RAG.

## Main Entry Point

The main notebook of the repository is `00_main`.

`00_main` acts as the central orchestrator of the experiment. It loads the configuration, selects the architecture to execute, runs the required notebooks, processes the evaluation questions, and stores the resulting outputs.

The experiments should therefore normally be started from `00_main` rather than by executing the architecture notebooks independently.

## Repository Structure

```text
00_main
    Central experiment orchestrator

SAS
    Single-agent baseline

SAS_RAG
    Single-agent architecture with RAG

MAS_RAG
    Static multi-agent architecture with RAG

Dynamic_Filtered_MAS_RAG
    Dynamic Filtered Multi-Agent architecture with RAG
```

The architecture-specific notebooks contain the implementation of each experimental configuration, while their execution is coordinated through `00_main`.

## Experiment Flow

```text
00_main
   ↓
Experiment configuration
   ↓
Evaluation questions
   ↓
Selected architecture
   ↓
SQL execution and answer generation
   ↓
Results and execution logs
```

All architectures process the same evaluation questions and use the same underlying data and LLM environment, while differing in how retrieval, semantic context, and agent coordination are handled.
