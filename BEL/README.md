
# Biomedical NER & Entity Linking Pipeline

A hybrid biomedical information extraction pipeline for oncology clinical guidelines combining:

- GLiNER-based Named Entity Recognition (NER)
- Rule-based normalization
- LLM-assisted post-processing
- Ontology-based entity linking using SapBERT + UMLS

---

# Overview

The pipeline processes oncology guideline documents converted from PDF to Markdown and transforms unstructured clinical text into structured biomedical entities for knowledge graph construction and clinical NLP applications.

---

# Stage 1 — Biomedical NER (GLiNER)

Named Entity Recognition is performed using `Ihor/gliner-biomed-large-v1.0`.

## Extracted Entity Categories

| Category | Entity Types |
|---|---|
| Clinical | condition, anatomy, histology_or_subtype, stage_or_risk |
| Diagnostic | diagnostic_test, biomarker_or_gene |
| Treatment | drug, regimen, dose, route, frequency, duration_or_timing |
| Patient Context | population, eligibility_criterion |
| Safety | contraindication_or_caution, adverse_event |
| Guideline Metadata | recommendation_strength, evidence_or_category, outcome_or_goal |

---

# Text Processing

- Markdown cleanup and LaTeX normalization
- Unicode normalization and removal of formatting artifacts
- Paragraph-aware segmentation with token-safe chunking

---

# Policy-Based Calibration

- Confidence filtering and structural validation
- Entity routing and duplicate removal
- Schema normalization

---

# Entity Linking (SapBERT + UMLS)

Semantic normalization for ontology-dependent entities using SapBERT biomedical embeddings (`cambridgeltl/SapBERT-from-PubMedBERT-fulltext`), dense retrieval via FAISS, and UMLS concept dictionaries.

Target entity groups: drug, condition, biomarker_or_gene, diagnostic_test, anatomy, histology_or_subtype, adverse_event.

---

# LLM-Assisted Classification

Contextual entity types processed via LLM-based classification: regimen, intervention, eligibility_criterion, outcome_or_goal.

---

# Dataset

| Property | Value |
|---|---|
| Total documents | 87 |
| Source format | PDF → Markdown |
| Domain | Oncology |
| Total extracted entities | 75,000+ |
| Number of entity types | 20 |

---

# Technologies

| Component | Technology |
|---|---|
| Biomedical NER | GLiNER |
| Entity Linking | SapBERT |
| Ontology Resources | UMLS |
| Vector Search | FAISS |
| LLM Post-processing | DeepSeek V3 |
| Frameworks | PyTorch, Transformers |

---

# Repository Structure

```text
Biomedical_NER_BEL/
│
├── notebooks/
│   └── NER_BEL.ipynb
│
├── outputs/
│   ├── normalized_outputs/group_*.md
│   ├── linked_entities/gliner_combined.json
│   └── entities_consolidated_final.json
│
├── README.md
│
└── requirements.txt
```


---

## Results

### Group A — Dense Retrieval Entity Linking (SapBERT + UMLS)

| Label | Count | Target Ontology | % of Coverage |
|---|---:|---|---:|
| `drug` | 11,785 | RxNorm / ATC / NCI | 87.6% |
| `condition` | 4,311 | NCI / SNOMED CT / MSH | 87.8% |
| `biomarker_or_gene` | 8,394 | NCI / HGNC | 84% |
| `histology_or_subtype` | 5,586 | NCI / SNOMED CT | 73.1% |
| `diagnostic_test` | 9,225 | LOINC / SNOMED CT | 77.2% |
| `anatomy` | 4,297 | SNOMED CT / FMA | 88.2% |
| `adverse_event` | 6,155 | MedDRA / SNOMED CT | 85.3% |

---

### Group B — Rule-Based Normalization

| Label | Total | % of Rule-Based Coverage |
|---|---|---|
| `dose` | 1,516 | 99.8% |
| `frequency_or_interval` | 553 | 98.0% |
| `route` | 872 | 93.6% |
| `duration_or_timing` | 218 | 77.5% |

### Group C — NCCN Controlled Vocabulary

| Label | Total | % of Rule-Based Coverage |
|---|---|---|
| `evidence_or_category` | 512 | 100.0% |
| `recommendation_strength` | 516 | 100.0% |
| `stage_or_risk` | 4,485 | 92.3% |
| `regimen` | 2,714 | 81.7% |

### Group D

| Label | Total | Rule-Based Coverage |
|---|---|---|
| `population` | 837 | 82.6% |
| `outcome_or_goal` | 2,353 | 88.1% |
| `contraindication_or_caution` | 887 | 46.2% |
| `eligibility_criterion` | 521 | 84.8% |
| `intervention` | 9,391 | 86.3% |

---

---

## Standards

| Domain | Standard | Used for |
|---|---|---|
| Dose units | UCUM |
| Frequency codes | HL7 FHIR GTSAbbreviation |
| Route of administration | SNOMED CT / FDA |
| Regimen acronyms | NCI Thesaurus / HemOnc.org |
| Evidence categories | NCCN Guidelines |
| Entity Linking | UMLS (2024AB) |

