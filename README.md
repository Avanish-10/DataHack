

# Backstory–Novel Consistency Verifier

## 1. Project Overview

The **Backstory–Novel Consistency Verifier** is an automated reasoning system that determines whether a given **character backstory** is **consistent** or **contradictory** with respect to a **novel**.
The system decomposes backstories into atomic claims, retrieves multi-hop evidence from the novel using a hybrid retrieval strategy, and applies an agentic evaluation framework to reach a **single, definitive label**.

**Final Output (strict constraint):**

```
consistent | contradict
```

No partial, neutral, or probabilistic outputs are allowed.

The system is explicitly designed to **generalize to unseen novels** and does **not depend on any training CSV at inference time**.

---

## 2. High-Level Architecture (Textual Diagram)

```
Input:
 ├── Character Backstory (free text)
 └── Novel (full text)

Pipeline:
 ├── Narrative-Aware Novel Chunking (Pathway RecursiveSplitter)
 ├── Backstory Claim Extraction (spaCy)
 │     ├── Atomic Claims
 │     └── Normalized Claims
 ├── Hybrid Retrieval
 │     ├── Semantic Retrieval (SBERT embeddings)
 │     ├── Lexical Retrieval (BM25)
 │     └── Score Fusion
 ├── Multi-Hop Evidence Retrieval
 ├── Evidence Pruning
 │     ├── Relevance Filtering
 │     └── Contradiction Detection
 ├── Agentic Evaluation
 │     ├── Evidence Agent
 │     ├── Causal Agent
 │     ├── Devil’s Advocate Agent
 │     └── Ensemble Scoring
 └── Backstory-Level Decision

Output:
 └── {consistent | contradict}
```

---

## 3. Detailed Pipeline

### 3.1 Narrative-Aware Novel Chunking

* The novel is segmented using **Pathway RecursiveSplitter**.
* Chunking is **structure-aware**, preserving narrative continuity across:

  * events
  * timelines
  * causal sequences
* Chunk boundaries are optimized for downstream retrieval rather than fixed token sizes.

This avoids evidence fragmentation common in naive sliding-window approaches.

---

### 3.2 Claim Extraction from Backstories

Backstories are decomposed using **spaCy-based linguistic parsing**.

#### Atomic Claims

* Each sentence is split into **minimal factual assertions**.
* Example:

  ```
  "He grew up in poverty and learned medicine from his grandmother."
  ```

  becomes:

  * He grew up in poverty.
  * He learned medicine from his grandmother.

Atomic claims are **evaluation units**.

#### Normalized Claims

Each atomic claim is normalized to improve retrieval:

* pronoun resolution
* tense normalization
* removal of stylistic modifiers
* canonical subject–predicate–object structure

Normalized claims are used **only for retrieval**, not for final judgment.

---

### 3.3 Hybrid Retrieval Strategy

The system uses **hybrid retrieval** to maximize recall.

#### Semantic Retrieval

* Embeddings generated using **SentenceTransformers (SBERT)** by default
* Optional OpenAI embeddings supported
* Captures paraphrased and implicit evidence

#### Lexical Retrieval

* **BM25 (rank-bm25)** for exact and near-exact term matches
* Ensures factual grounding for concrete details (dates, roles, locations)

#### Fusion

* Semantic and lexical scores are combined via weighted fusion
* Top-K candidates form the evidence pool

---

### 3.4 Multi-Hop Evidence Retrieval

* Initial evidence chunks may be incomplete
* The system performs **iterative retrieval hops**:

  * expand from initial evidence
  * follow narrative or causal references
* This enables reasoning over:

  * temporally separated facts
  * implied contradictions

---

### 3.5 Evidence Pruning

Evidence candidates are pruned in two stages:

1. **Relevance Filtering**

   * Semantic alignment with the claim
   * Removal of tangential narrative content

2. **Contradiction Detection**

   * Checks for negation, temporal conflict, role mismatch, or causal impossibility
   * Evidence explicitly opposing a claim is retained and flagged

---

## 4. Agentic Evaluation Framework

Each atomic claim is evaluated by **multiple specialized agents**.

### 4.1 Evidence Agent

* Verifies whether retrieved evidence **supports or contradicts** the claim
* Focuses on explicit textual grounding

### 4.2 Causal Agent

* Evaluates **temporal and causal consistency**
* Flags contradictions such as:

  * impossible timelines
  * mutually exclusive life events

### 4.3 Devil’s Advocate Agent

* Actively searches for **counter-evidence**
* Assumes the claim is false unless proven otherwise
* Reduces false positives for consistency

---

### 4.4 Ensemble Scoring

* Each agent produces a structured judgment
* Judgments are combined using **ensemble logic**
* A single **claim-level verdict** is produced

---

## 5. Final Decision Logic

* All atomic claims are evaluated independently
* **Backstory-level rule**:

```
If ANY atomic claim is contradicted → contradict
Else → consistent
```

This conservative rule ensures:

* no partial acceptance
* strict logical consistency

---

## 6. How to Run the Project

### 6.1 Environment Setup

```bash
pip install spacy sentence-transformers rank-bm25 pathway numpy scikit-learn
python -m spacy download en_core_web_sm
```

### 6.2 Execution Steps

1. Provide:

   * full novel text
   * character backstory text
2. Run the main pipeline script:

```bash
python run_verifier.py --novel novel.txt --backstory backstory.txt
```

### 6.3 Output

```text
consistent
```

or

```text
contradict
```

No auxiliary outputs are produced at inference time.

---

## 7. Limitations

* Implicit contradictions requiring deep world knowledge may be missed
* Very long novels increase retrieval cost
* Coreference resolution errors can affect claim normalization
* Currently focused on **binary consistency**, not explanation generation

---

## 8. Future Work

* Symbolic–neural hybrid contradiction reasoning
* Cross-chapter temporal graph modeling
* Evidence explanation generation (optional, non-inference mode)
* Scaling to multi-character joint consistency checking

---

## 9. Key Design Principles

* Generalization over memorization
* Strict binary decision boundary
* Retrieval-first, reasoning-second architecture
* Agentic redundancy for robustness

---
