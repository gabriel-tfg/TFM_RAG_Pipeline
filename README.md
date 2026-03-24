# RAG Pipeline for Biomedical Literature (TFM)

This repository contains the implementation of a Retrieval-Augmented Generation (RAG) pipeline designed for domain-specific question answering in biomedical contexts.

The project focuses on improving how large language models access and use scientific evidence, specifically targeting menopause-related insomnia and cognitive behavioral therapy for insomnia (CBT-I).

---

## Objective

The goal of this work is to build a **sustainable and domain-adapted RAG pipeline** that:

- Retrieves relevant biomedical literature from PubMed / PMC
- Constructs a structured corpus from scientific articles
- Improves answer quality by grounding responses in real evidence
- Reduces hallucinations compared to standalone language models

A key contribution of this project is the **refinement of corpus construction**, moving from naive citation-based retrieval to an approximation of **studies actually included in meta-analyses**.

---

## Methodology Overview

The pipeline consists of the following stages:

### 1. Data Retrieval (PubMed API)

- Query PubMed using E-utilities (`esearch`, `efetch`)
- Retrieve:
  - Meta-analyses (seed documents)
  - Metadata: PMID, title, abstract

---

### 2. PMC Full-Text Processing

- Convert PMID → PMCID
- Retrieve full-text XML using the PMC OAI-PMH API
- Parse JATS XML structure

---

### 3. Reference Extraction

From each meta-analysis:

- Extract all references (`<ref>` tags)
- Parse:
  - PMID
  - DOI
  - Year
  - First author
  - Full reference text

---

### 4. Included Study Detection (Heuristic)

A key improvement of this pipeline:

Instead of using all cited papers, we approximate **studies actually included in the meta-analysis**.

This is done by:

- Extracting:
  - Sections (`<sec>`)
  - Tables (`<table-wrap>`)
- Detecting relevant sections:
  - "Included studies"
  - "Study characteristics"
  - "Results"
- Matching references using:
  - Author + year
  - Presence in tables
  - PMID availability

Each reference is scored and filtered based on a threshold.

---

### 5. Corpus Construction

Final corpus consists of:

- Meta-analyses (seed documents)
- Included-study candidates (filtered references)

Duplicates are removed using PMID.

---

### 6. Chunking

- Sentence-based chunking
- Overlapping windows for context preservation
- Metadata retained per chunk:
  - PMID
  - document type
  - title
  - source ID

---

### 7. Embeddings & Retrieval

- Model: `sentence-transformers/all-MiniLM-L6-v2`
- FAISS index with cosine similarity (inner product)
- Retrieval improvements:
  - Initial candidate expansion
  - Per-document chunk limiting
  - Source-type prioritization (meta-analysis vs study)

---

### 8. Generation (RAG)

- Model: `TinyLlama-1.1B-Chat`
- Deterministic generation (`do_sample=False`)
- Prompt design:
  - Strict grounding in context
  - No hallucinations allowed
  - Fallback: `"Not enough information"`

---

## 📊 Results (Preliminary)

Comparison between baseline LLM and RAG:

| Aspect | Baseline | RAG |
|------|--------|-----|
| Specificity | Low | High |
| Evidence grounding | ❌ | ✅ |
| Hallucinations | Frequent | Reduced |
| Clinical relevance | Limited | Improved |

Observations:

- RAG performs better on **core domain questions (CBT-I & insomnia)**
- Performance decreases for **secondary topics (e.g., vasomotor symptoms)**
- Trade-off observed between:
  - Precision (filtered corpus)
  - Coverage (broader citation-based corpus)

---

## Limitations

- Heuristic detection of included studies is approximate
- Some irrelevant studies may still be included
- Small generative model (TinyLlama) limits reasoning quality
- No formal evaluation metric (BLEU, ROUGE, etc.) yet
- Dependence on XML structure consistency in PMC

---

## Future Work

- Improve included-study detection:
  - Better table parsing
  - Semantic filtering
- Compare:
  - citation-based vs inclusion-based corpus
- Upgrade generative model (e.g., Mistral, Llama)
- Add evaluation benchmarks
- Integrate domain-specific embeddings

---

## 🛠️ Requirements

- Python 3.10+
- Libraries:
  - `requests`
  - `xml`
  - `sentence-transformers`
  - `faiss`
  - `transformers`
  - `torch`

---

## 📁 Repository Structure
