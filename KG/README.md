# Relation Extraction — Clinical Oncology Knowledge Graph

This module implements the relation extraction (RE) pipeline for building a knowledge graph from NCCN clinical oncology guidelines. It represents Stage 3 of the overall pipeline, taking normalized entities from the BEL module and extracting semantic triples.

## Overview
Structured clinical triples are extracted from pre-processed guideline text using two complementary LLM passes based on the novel **Predicate Partitioning Strategy**:

| Pass | Model | Predicates |
|------|-------|------------|
| **Core** | DeepSeek-V3 | `TREATS`, `CONTRAINDICATED_IN`, `REQUIRES_BIOMARKER`, `COMBINED_WITH`, `HAS_DOSE`, `HAS_FREQUENCY`, `HAS_ROUTE`, `HAS_EVIDENCE_LEVEL`, `HAS_STRENGTH`, `ALTERNATIVE_TO` |
| **Context** | GPT-4.1 | `INDICATED_FOR_STAGE`, `USED_IN_POPULATION`, `ACHIEVES_OUTCOME`, `HAS_ELIGIBILITY` |

Each pass targets a distinct layer of clinical knowledge. The Core pass extracts direct pharmacological and clinical relations; the Context pass captures population-level and outcome-oriented attributes that require broader contextual reasoning.

## Input Files
| File | Description |
|------|-------------|
| `chunk_index.json` | Chunked guideline text, keyed by `doc_id → chunk_num → {text, ...}` |
| `entities_consolidated_final.json` | Fully normalized entities per chunk from the BEL module |

## Output Files
| File | Description |
|------|-------------|
| `re_deepseek_final.json` | Deduplicated triples from DeepSeek-V3 Core pass |
| `re_gpt41_final.json` | Deduplicated triples from GPT-4.1 Context pass |
| `re_merged_final.json` | Final merged triple store (ready for Neo4j import) |

### Triple Schema
Each triple in the output has the following fields:
```json
{
  "subject":         "bevacizumab",
  "predicate":       "TREATS",
  "object":          "metastatic colorectal cancer",
  "object_original": "metastatic colorectal cancer",
  "confidence":      1.0,
  "evidence":        "Bevacizumab is recommended for first-line treatment of metastatic CRC.",
  "method":          "deepseek_v3_core",
  "doc_id":          "colon_v2_2025",
  "chunk_id":        14
}
```

## Requirements
```bash
pip install openai json-repair tqdm
```
API keys required (set as environment variables):
```bash
export DEEPSEEK_API_KEY="your-deepseek-api-key"
export OPENAI_API_KEY="your-openai-api-key"
```

## Usage
Run cells in order in the `KG.ipynb` notebook. The extraction supports checkpointing every 100 chunks to facilitate long-running API calls.

## Evaluation: Silver Standard & Partial Match
This module also includes the `silver_standard.tsv` dataset, containing 198 manually verified semantic triples. 

We evaluate LLM extraction using both Exact Match and a custom Partial Match metric (`token_sort_ratio` > 80) to account for lexical variability in clinical text.

## Results (NCCN Corpus, 87 documents)
| Metric | Value |
|--------|-------|
| Total chunks processed | ~12,400 |
| Core triples (DeepSeek-V3) | 39,021 |
| Context triples (GPT-4.1) | 104,896 |
| Merged triples (after dedup) | 143,917 |
| Graph nodes (Neo4j) | 105,767 |
| Graph relationships | 143,917 |
| Relationship types | 14 |
