# Biomedical NER & Entity Linking Pipeline

A hybrid biomedical information extraction pipeline for oncology clinical guidelines combining:
- GLiNER-based Named Entity Recognition (NER)
- Rule-based normalization
- LLM-assisted post-processing
- Ontology-based entity linking using SapBERT + UMLS

---

## Overview
The pipeline processes oncology guideline documents converted from PDF to Markdown and transforms unstructured clinical text into structured biomedical entities for knowledge graph construction and clinical NLP applications. This module represents Stages 1 and 2 of the overall Knowledge Graph construction pipeline.

---

## Stage 1 — Biomedical NER (GLiNER)
Named Entity Recognition is performed using `Ihor/gliner-biomed-large-v1.0` in zero-shot mode, requiring no domain-specific fine-tuning.

### Extracted Entity Categories
| Category | Entity Types |
|---|---|
| Clinical | `condition`, `anatomy`, `histology_or_subtype`, `stage_or_risk` |
| Diagnostic | `diagnostic_test`, `biomarker_or_gene` |
| Treatment | `drug`, `regimen`, `dose`, `route`, `frequency`, `duration_or_timing` |
| Patient Context | `population`, `eligibility_criterion` |
| Safety | `contraindication_or_caution`, `adverse_event` |
| Guideline Metadata | `recommendation_strength`, `evidence_or_category`, `outcome_or_goal` |

---

## Stage 2 — Entity Consolidation (4-Group Strategy)

To ensure high-quality graph nodes, raw NER spans are normalized using a four-group strategy based on entity semantics:

### Group A: Ontological Linking (SapBERT + UMLS)
Semantic normalization for ontology-dependent entities using SapBERT biomedical embeddings (`cambridgeltl/SapBERT-from-PubMedBERT-fulltext`), dense retrieval via FAISS, and UMLS concept dictionaries (2024AB).
- **Target entities:** `drug`, `condition`, `biomarker_or_gene`, `diagnostic_test`, `anatomy`, `histology_or_subtype`, `adverse_event`.

### Group B: Parametric Normalization (Rule-Based)
Regex and rule-based standardization mapped to international standards.
- **Dose units:** Mapped to UCUM.
- **Frequency codes:** Mapped to HL7 FHIR GTSAbbreviation.
- **Route of administration:** Mapped to SNOMED CT / FDA standard terms.

### Group C: Categorical Normalization (Lookup Tables)
Mapping NCCN-specific acronyms and categories to controlled vocabularies.
- **Target entities:** `evidence_or_category`, `recommendation_strength`, `stage_or_risk`, `regimen`.

### Group D: Contextual Normalization (LLM-Assisted)
Complex, multi-word contextual entities processed via LLM-based classification (DeepSeek-V3) to extract the core semantic meaning.
- **Target entities:** `population`, `outcome_or_goal`, `contraindication_or_caution`, `eligibility_criterion`, `intervention`.

---

## Dataset Statistics
| Property | Value |
|---|---|
| Total documents | 87 |
| Source format | PDF → Markdown |
| Domain | Oncology (NCCN) |
| Total extracted entities | 75,000+ |
| Number of entity types | 20 |

---

## Usage
1. Install dependencies: `pip install -r requirements.txt`
2. Ensure you have the preprocessed chunked Markdown files.
3. Run the `NER_BEL.ipynb` notebook sequentially.
