# CS5588-group2
## Team Roles (Required)

| Team Member | Role | Responsibilities |
|------------|------|------------------|
| **Lyza Iamrache** | **Product Lead** | Defines the stakeholder persona(s), product value, success metrics (trust, usability, time-to-answer), and writes the “If we shipped this” deployment plan. |
| **Ailing Nan** | **Systems Lead** | Builds the multimodal pipeline (PDF ingestion + OCR), implements retrieval (BM25 + FAISS + hybrid fusion + reranking), grounding with citations, and runs ablation studies. |
| **Gia Huynh** | **Evaluation & Risk Lead** | Designs evaluation metrics (P@5, R@10, faithfulness), compares retrieval methods, documents failure cases, and writes governance + mitigation strategies. |

## Evaluation

We evaluate the system with an emphasis on **grounding correctness, modality completeness, and trust-aware behavior**, rather than answer plausibility alone. Given the small number of stakeholder tasks in this sprint, we adopt a **lightweight, manual evaluation protocol** aligned with the course rubric.

### Evaluation Methodology

#### Ground-Truth Definition
For each task, we **manually define the minimum required ground-truth evidence** based on the semantics of the question, rather than on which documents happen to be retrieved:

- Tasks involving **spatial proximity** (e.g., *closest*) require **campus map evidence**.
- Tasks involving **service availability** (e.g., *shuttle bus stations*) additionally require **shuttle route or schedule documentation**.
- Tasks involving **pricing** require **official parking permit tables**.
- For questions where no supporting documentation exists (e.g., enrollment projections), the correct system behavior is **explicit refusal**, and no ground truth is defined.

This definition reflects the evidence that is *necessary* to justify an answer, not merely evidence that makes an answer appear plausible.

---

#### Precision@5 and Recall@10
Using the ranked list of retrieved evidence citations printed by the system, we compute:

- **Precision@5 (P@5):**  
  The proportion of the top 5 retrieved evidence items that belong to the manually defined ground-truth set.

- **Recall@10 (R@10):**  
  The proportion of the ground-truth evidence set that appears within the top 10 retrieved results.

Due to the small number of evaluation queries, these metrics are computed **manually** from the retrieval outputs, which is appropriate for an early-stage product prototype.

---

#### Faithfulness Assessment
Faithfulness is evaluated qualitatively using the following labels:

- **Yes:** The answer’s key claims are fully supported by retrieved evidence and cited correctly.
- **Partial:** The answer is plausible or partially correct, but required evidence modalities or documents are missing.
- **Refusal:** The system correctly refuses to answer due to insufficient evidence.
- **No:** The answer makes unsupported claims not justified by retrieved evidence.

---

### Evaluation Results

| Task | Description | Method | P@5 | R@10 | Faithfulness |
|------|------------|--------|-----|------|--------------|
| Q1 | Plaster Hall shuttle station | Hybrid | 0.40 | 0.67 | Partial |
| Q2 | Haag Hall parking + permit cost | Hybrid | 0.20 | 0.50 | Partial |
| Q3 | 2026 enrollment trend | Hybrid | N/A | N/A | Refusal |

---

### Task-Level Analysis

**Task 1 – Plaster Hall Shuttle Station**  
The system retrieves relevant **campus map pages** and uses them to infer spatial proximity. However, **shuttle route or schedule documentation is not retrieved**, which is required to confirm whether the identified location is an active shuttle stop. As a result, the answer is a plausible but **under-grounded inference** rather than a fully supported claim.

**Task 2 – Haag Hall Parking and Permit Cost**  
The system correctly retrieves **parking permit pricing tables** and reports accurate semester cost values. However, **campus map evidence is not retrieved** to justify the “closest” parking lot determination. The final answer is correct but relies on implicit assumptions for spatial reasoning, leading to incomplete grounding.

**Task 3 – Enrollment Projection (2026)**  
The system correctly identifies that the provided documents do not contain enrollment projections and explicitly refuses to answer. This demonstrates appropriate **hallucination avoidance and trust-aware behavior**.

---

### Summary
Overall, the evaluation shows that the system performs well in **retrieving relevant documents and avoiding unsupported answers**, but also highlights an important limitation: without explicit modality constraints, a multimodal RAG system may produce **plausible but insufficiently grounded answers**. These findings motivate stricter evidence gating and modality-aware refusal logic in future iterations.
