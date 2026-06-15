# Relation Extraction — Clinical Oncology Knowledge Graph

This notebook implements the relation extraction (RE) pipeline for building a knowledge graph from NCCN clinical oncology guidelines. It is part of a larger pipeline described in the accompanying master's thesis.

## Overview

Structured clinical triples are extracted from pre-processed guideline text using two complementary LLM passes:

| Pass | Model | Predicates |
|------|-------|------------|
| Core | DeepSeek-V3 | TREATS, CONTRAINDICATED_IN, REQUIRES_BIOMARKER, COMBINED_WITH, HAS_DOSE, HAS_FREQUENCY, HAS_ROUTE, HAS_EVIDENCE_LEVEL, HAS_STRENGTH, ALTERNATIVE_TO |
| Context | GPT-4.1 | INDICATED_FOR_STAGE, USED_IN_POPULATION, ACHIEVES_OUTCOME, HAS_ELIGIBILITY |

Each pass targets a distinct layer of clinical knowledge. The Core pass extracts direct pharmacological and clinical relations; the Context pass captures population-level and outcome-oriented attributes that require broader contextual reasoning.

## Pipeline Position

```
Stage 1: PDF Extraction
Stage 2: Text Cleaning
Stage 3: NER (GLiNER)               # normalized_entities.json
Stage 4: Entity Linking (BEL)       # linked_entities.json
Stage 5: Relation Extraction (KG)   # this notebook
Stage 6: Neo4j Import
```

## Input Files

| File | Description |
|------|-------------|
| `data/chunk_index.json` | Chunked guideline text, keyed by `doc_id → chunk_num → {text, ...}` |
| `data/normalized_entities.json` | GLiNER entities per chunk, keyed by `doc_id → chunk_num → [{text, label, ...}]` |

## Output Files

| File | Description |
|------|-------------|
| `results/re_deepseek_final.json` | Deduplicated triples from DeepSeek-V3 Core pass |
| `results/re_gpt41_final.json` | Deduplicated triples from GPT-4.1 Context pass |
| `results/re_merged_final.json` | Final merged triple store (ready for Neo4j import) |

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

`confidence` is set to `1.0` for explicitly stated relations (`HIGH`) and `0.5` for implied ones (`LOW`).

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

Run cells in order:

1. **Cell 1** — Install dependencies
2. **Cell 2** — Set paths and configuration
3. **Cell 3** — Load chunk index and entity index
4. **Cell 4** — Define prompts and normalization functions
5. **Cell 5** — Initialize API clients
6. **Cell 6** — Define extraction functions
7. **Cell 7** — Define checkpoint helpers
8. **Cell 8** — Run DeepSeek-V3 Core pass (checkpoints every 100 chunks)
9. **Cell 9** — Run GPT-4.1 Context pass (requires `SYSTEM_PROMPT_CONTEXT` to be defined)
10. **Cell 10** — Merge both passes and compute statistics

Cells 8 and 9 can be run in parallel in separate sessions. Both support resumption from checkpoint.

## Notes

- The context-pass prompt (`SYSTEM_PROMPT_CONTEXT`) is not included in this public version. Define it in Cell 4 following the same structure as `SYSTEM_PROMPT_CORE`.
- The `normalize_object()` function contains a domain-specific normalization map that is not included. Implement it based on your target ontology.
- Checkpoint files are saved to `results/` every 100 chunks to support long-running jobs.
- All entity names are stored in lowercase; use `toLower()` when querying in Neo4j.

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

## Related Notebooks

- `NER_BEL.ipynb` — Named Entity Recognition + Entity Linking

