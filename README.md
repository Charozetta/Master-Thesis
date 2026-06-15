# Automated Construction of a Medical Knowledge Graph from Oncological Clinical Guidelines

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Neo4j](https://img.shields.io/badge/Neo4j-AuraDB-018bff.svg)](https://neo4j.com/)

> **Note:** This repository contains the code and methodology for my Master's Thesis in Data Science at the Higher School of Economics (HSE University), Moscow. 

This project implements an end-to-end automated pipeline for constructing a queryable Medical Knowledge Graph (MKG) from highly complex, unstructured clinical guidelines published by the National Comprehensive Cancer Network (NCCN). 

The system leverages a hybrid architecture combining zero-shot Named Entity Recognition (GLiNER), dense-retrieval Entity Linking (SapBERT + FAISS), and LLM-driven Relation Extraction (DeepSeek-V3 / GPT-4.1) to transform 87 PDF guidelines into a structured Neo4j graph with over 105,000 nodes and 143,000 edges.

## Research Problem & Motivation

Oncological clinical guidelines encode critical, evidence-based knowledge regarding treatment regimens, dosing schedules, and biomarker requirements. However, this knowledge is locked in unstructured PDF documents containing complex nested logic (e.g., *"Drug X at Dose Y is indicated for Stage Z patients with Biomarker B"*). 

Existing NLP tools struggle to parse this domain-specific complexity without extensive manual annotation. This project solves this by proposing a **zero-shot, ontology-driven extraction pipeline** that requires no task-specific fine-tuning, making it scalable to rapidly evolving medical literature.

##  Pipeline Architecture

The pipeline consists of four main stages, reflecting the logical sequence of knowledge structuring:

1. **Data Ingestion & Preprocessing:** Conversion of 87 NCCN PDF guidelines to Markdown, with domain-specific cleaning to remove LaTeX artifacts and token-safe chunking.
2. **Biomedical NER (Stage 1):** Zero-shot extraction of 20 clinical entity types (e.g., `drug`, `regimen`, `biomarker`, `stage`) using the `gliner-biomed-large-v1.0` model.
3. **Entity Consolidation (Stage 2):** A novel 4-group normalization strategy:
   - **Group A (Ontological):** SapBERT + FAISS linking against UMLS (2024AB) for real-world entities.
   - **Group B (Parametric):** Rule-based normalization (regex) for doses, frequencies, and routes.
   - **Group C (Categorical):** Lookup tables for NCCN-specific evidence levels.
   - **Group D (Contextual):** LLM-assisted normalization for complex contextual entities.
4. **Relation Extraction & KG Construction (Stage 3):** A hybrid LLM approach utilizing DeepSeek-V3 for "core" predicates (e.g., `TREATS`, `HAS_DOSE`) and GPT-4.1 for complex "contextual" predicates (e.g., `USED_IN_POPULATION`).

## Key Results & Contributions

- **Predicate Partitioning Strategy:** Demonstrated that splitting semantic relations into *core* and *contextual* categories allows for optimal routing between different LLMs, reducing API costs by a factor of 43 while maintaining or improving accuracy.
- **Partial Match Evaluation:** Proposed and validated a `token_sort_ratio`-based partial match metric for evaluating clinical relation extraction, proving that strict exact-match metrics systematically underestimate LLM performance on highly variable medical text.
- **Silver Standard Dataset:** Created a manually verified dataset of 198 semantic triples for evaluating zero-shot clinical RE.
- **Bilingual Graph RAG:** Implemented a LangChain-based natural language interface allowing oncologists to query the Neo4j graph in both English and Russian.

## Repository Structure

```text
.
├── BEL/                        # Biomedical Entity Linking & NER Module
│   ├── NER_BEL.ipynb           # Main notebook for NER and Entity Consolidation
│   └── README.md               # Detailed documentation for the BEL pipeline
|   └── requirements.txt        # Necessary Libraries for EL performing
│
├── KG/                         # Knowledge Graph & Relation Extraction Module
│   ├── KG.ipynb                # Main notebook for LLM-based Relation Extraction
│   ├── silver_standard.tsv     # Manually verified dataset for RE evaluation
│   └── README.md               # Detailed documentation for the RE pipeline
│
└── Data Artifacts (Samples)    # Intermediate JSON outputs demonstrating the data flow
    ├── gliner_combined.json            # Raw NER outputs
    ├── group_a_linked.json             # SapBERT UMLS linking results
    ├── group_b_normalized.json         # Parametric entity normalization
    ├── group_c_normalized_v2.json      # Categorical entity normalization
    ├── group_d_normalized_v2.json      # Contextual entity normalization
    └── entities_consolidated_final.json # Final merged entities ready for RE
```

*Note: The raw NCCN PDF documents and the full Neo4j database dump are not included in this repository due to licensing and size constraints.*

## Getting Started

### Prerequisites
- Python 3.10+
- Access to a Neo4j instance (AuraDB recommended)
- API Keys for OpenAI (GPT-4.1) and DeepSeek (V3)

### Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/Master-Thesis.git
   cd Master-Thesis
   ```
2. Install dependencies for the BEL module:
   ```bash
   cd BEL
   pip install -r requirements.txt
   ```

### Execution Flow
To reproduce the pipeline (assuming you have preprocessed text chunks):
1. Run `BEL/NER_BEL.ipynb` to perform entity extraction and normalization.
2. Run `KG/KG.ipynb` to perform relation extraction and generate the final semantic triples.

## Contact & Citation

If you find this work useful for your research, please consider citing the thesis or reaching out for collaboration.

**Author:** Iuliia Kozina  
**Email:** [juliakozina@gmail.com]
