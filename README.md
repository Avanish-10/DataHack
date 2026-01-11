

# Backstory–Novel Consistency Verifier

## 1. Overview

This project implements an automated system for verifying whether a **character backstory** is **consistent** or **contradictory** with respect to a **novel**.
Given a novel–backstory pair, the system performs narrative-aware retrieval and multi-agent reasoning to output a **single binary label**.

**Output (strict):**

```
consistent | contradict
```

The system is designed to **generalize to unseen novels** and operates **entirely at inference time**, without reliance on training datasets.

---

## 2. Problem Framing

For each test instance:

* Input:

  * Full novel text
  * Character backstory text
* Output:

  * One label indicating whether the backstory contradicts the novel

If **any atomic claim** in the backstory contradicts the novel, the entire backstory is classified as **contradict**; otherwise, it is **consistent**.

---

## 3. High-Level Architecture

```
Novel Text
   ↓
Narrative-Aware Chunking (Pathway RecursiveSplitter)
   ↓
Hybrid Retrieval Index
   ├── Semantic (SBERT embeddings)
   └── Lexical (BM25)

Backstory Text
   ↓
spaCy-Based Claim Extraction
   ├── Atomic Claims
   └── Normalized Claims
   ↓
Multi-Hop Evidence Retrieval
   ↓
Evidence Pruning
   ├── Relevance Filtering
   └── Contradiction Detection
   ↓
Agentic Evaluation
   ├── Evidence Agent
   ├── Causal Agent
   └── Devil’s Advocate Agent
   ↓
Ensemble Scoring
   ↓
Final Backstory-Level Decision
```

---

## 4. Handling Long Context

* Novels are chunked using **Pathway RecursiveSplitter**
* Chunking preserves:

  * narrative continuity
  * temporal flow
  * causal relationships
* This avoids truncation and evidence fragmentation common in fixed-window methods

---

## 5. Claim Extraction and Normalization

### Atomic Claims

* Backstories are decomposed into minimal factual units using **spaCy dependency parsing**
* Each atomic claim represents a single verifiable assertion

### Normalized Claims

* Claims are rewritten into canonical forms for retrieval:

  * pronoun resolution
  * tense normalization
  * removal of stylistic modifiers
* Normalized claims are **used only for retrieval**, not for judgment

---

## 6. Retrieval Strategy

### Hybrid Retrieval

* **Semantic retrieval** using SBERT embeddings captures paraphrased evidence
* **Lexical retrieval** using BM25 ensures exact factual grounding
* Scores are fused to form a high-recall evidence pool

### Multi-Hop Retrieval

* Evidence is expanded iteratively to capture:

  * temporally distant facts
  * causally linked events
* Enables detection of indirect contradictions

---

## 7. Evidence Pruning

Evidence candidates are filtered using:

1. **Relevance scoring** to remove unrelated narrative text
2. **Contradiction detection**, including:

   * negation
   * temporal conflict
   * role or identity mismatch
   * causal impossibility

---

## 8. Agentic Evaluation

Each atomic claim is evaluated by three agents:

* **Evidence Agent**
  Checks direct textual support or contradiction.

* **Causal Agent**
  Evaluates temporal and causal coherence across events.

* **Devil’s Advocate Agent**
  Actively searches for counter-evidence to prevent false consistency.

Agent outputs are combined via **ensemble scoring** to produce a claim-level verdict.

---

## 9. Final Decision Logic

* Claims are evaluated independently
* **Decision rule:**

```
If any claim is contradicted → contradict
Else → consistent
```

The system always outputs **exactly one label per test example**.

---

## 10. Running the Code (Evaluator Setup)

The code is designed to be executed **end-to-end without manual intervention**.

### Input Assumption

* Evaluators provide a CSV file containing novel–backstory pairs
* The CSV is treated strictly as an inference-time input container

### Execution

```bash
python run_verifier.py --input_csv <provided_input>.csv --output_csv results.csv
```

### Output

* The system generates a file named:

```
results.csv
```

* The file contains **one row per test example** with a single prediction label

---

## 11. Reproducibility

* No training data is required at inference
* No hardcoded file paths or interactive steps
* All predictions are deterministic given the input

---

## 12. Limitations and Failure Cases

* Implicit contradictions requiring deep world knowledge may be missed
* Extremely long novels increase retrieval cost
* Coreference errors can affect claim normalization
* Binary output does not expose explanations by design

---

## 13. Tech Stack

* Python
* spaCy
* SentenceTransformers (SBERT)
* rank-bm25
* Pathway
* NumPy / scikit-learn

---

## 14. Design Principles

* Generalization over memorization
* Retrieval-first, reasoning-second
* Conservative contradiction handling
* Strict binary decision boundary

---
