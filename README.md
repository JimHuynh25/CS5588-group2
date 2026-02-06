# CS5588-group2

# CS 5588 Week 3 — Multimodal RAG Module (UMKC Campus Information)

## Project Overview

This project implements a **stakeholder-facing multimodal Retrieval-Augmented Generation (RAG) module** for answering campus-related questions at the University of Missouri–Kansas City (UMKC). The system integrates **PDF documents, tables, and images**, applies OCR to visual content, and generates **grounded answers with citations**, while explicitly refusing to answer when evidence is insufficient.

The goal of this sprint is not only to improve answer accuracy, but also to demonstrate **trust-aware behavior**, **multimodal grounding**, and **deployment readiness**.

---

## Product Brief

### Target Users
- UMKC students seeking information about **parking**, **shuttle transportation**, and **campus navigation**
- Visitors and staff who need quick, evidence-backed answers from official campus documents

### User Value
- Faster access to official information without manually searching long PDFs
- Increased trust through transparent citations
- Reduced risk of misinformation via explicit refusal behavior

---

## System Architecture

The system follows a modular multimodal RAG pipeline:

1. **Ingestion**
   - PDF text extraction using PyMuPDF
   - Image ingestion (maps, tables, figures)
   - OCR applied to images to convert visual content into searchable text

2. **Indexing**
   - Page-level chunking
   - Metadata tracking (document name, page number, modality)

3. **Retrieval**
   - Sparse retrieval (BM25)
   - Dense retrieval (embeddings)
   - Hybrid retrieval (dense + sparse fusion)
   - Optional reranking

4. **Grounded Generation**
   - Evidence-pack construction
   - LLM-based answer generation with enforced citations
   - Explicit refusal when evidence is insufficient

---


## Team Roles

| Team Member | Role | Responsibilities |
|------------|------|------------------|
| **Lyza Iamrache** | **Product Lead** | Defines the stakeholder persona(s), product value, success metrics (trust, usability, time-to-answer), and writes the “If we shipped this” deployment plan. |
| **Ailing Nan** | **Systems Lead** | Builds the multimodal pipeline (PDF ingestion + OCR), implements retrieval (BM25 + FAISS + hybrid fusion + reranking), grounding with citations, and runs ablation studies. |
| **Gia Huynh** | **Evaluation & Risk Lead** | Designs evaluation metrics (P@5, R@10, faithfulness), compares retrieval methods, documents failure cases, and writes governance + mitigation strategies. |

---

## Data Inventory


| Data Item | Source (Website) | PDF / Image | Modality | Role in Product |
|----------|------------------|-------------|----------|-----------------|
| Student Handbook | [UMKC Student Handbook](https://www.umkc.edu/student-affairs/student-handbook/) | umkc-student-handbook.pdf | PDF / text | Policies, student services, conduct → text evidence |
| Parking Permits | [UMKC Parking Operations](https://parking.umkc.edu/) | Student Permits - Parking Options - Parking _ University of Missouri-Kansas City.pdf | PDF / text | Parking rules, permits, zones → text + table evidence |
| Campus Crime Report (Clery) | [UMKC Police – Campus Crime Information](https://www.umkc.edu/police/campus-crime-information/) | 2025ccfsrumkcpolice.pdf | PDF / table  Crime statistics, trends → numeric + figure evidence |
| Visual Identity Guidelines | [UMKC Brand Guidelines](https://www.umkc.edu/marketing/brand-guidelines/) | visual-identity-guidelines.pdf | PDF / image | Logo, color, design → optional for UI/branding |
| Health Sciences Campus Map | [UMKC Maps](https://www.umkc.edu/maps/) | umkc-health-sciences-campus-map.pdf | PDF / image | Campus map → location-based tasks |
| Volker Campus Map | [UMKC Maps](https://www.umkc.edu/maps/) | umkc-volker-campus-map.pdf | PDF / image | Campus map → directions, shuttle stops |
| Shuttle Schedule | [UMKC Shuttle / Transportation](https://www.umkc.edu/parking/shuttle/) | 2026-spring-shuttle-schedule.pdf | PDF / table | Shuttle routes & times → text + table evidence |
| University Catalog | [UMKC Consumer Information (Catalog)](https://www.umkc.edu/registrar/forms-resources/student-resources/consumer-information.html) | 2025-2026 University Catalog_Archived 9-10-25.pdf | PDF / table / text | Program/course requirements → applicant questions |


---

## User Tasks & Demo Flow

This section demonstrates how the system behaves end-to-end for each stakeholder task, including retrieved evidence and final responses.

---

### Task 1 — Shuttle Transportation

**Question:**  
Which shuttle bus station is closest to Plaster Hall according to the campus map and shuttle route information?

**Top Evidence Citations:**  
- [umkc-volker-campus-map.pdf p1]  
- [umkc-volker-campus-map.pdf p2]  
- [umkc-health-sciences-campus-map.pdf p2]  
- [umkc-health-sciences-campus-map.pdf p1]

**System Answer:**  
According to the provided campus maps and shuttle route information, the closest shuttle bus station to Plaster Hall is the one located at 5115 Oak St., which serves the Administrative Center building. This location is approximately 1 mile away from Plaster Hall based on the distance between these two points on the campus map.  
*[umkc-volker-campus-map.pdf p2]*

**Observation:**  
While the system retrieves relevant campus map pages and provides a plausible spatial inference, shuttle route or schedule documentation is not retrieved. As a result, the answer is only **partially grounded**.

---

### Task 2 — Parking & Pricing

**Question:**  
Which parking lot is closest to Haag Hall, and how much does a student pay for a semester parking permit for that lot?

**Top Evidence Citations:**  
- [Student Permits - Parking Options - Parking _ University of Missouri-Kansas City.pdf p2]  
- [Student Permits - Parking Options - Parking _ University of Missouri-Kansas City.pdf p3]  
- [Student Permits - Parking Options - Parking _ University of Missouri-Kansas City.pdf p1]  
- [Student Permits - Parking Options - Parking _ University of Missouri-Kansas City.pdf p4]

**System Answer:**  
The closest parking lot to Haag Hall is Area 32S, located at 52nd and Rockhill Road. A single-semester all-day accessible permit for this lot costs $135. Multi-semester permits vary by duration; for example, a Fall & Spring permit costs $270, while a Fall, Spring, & Summer permit costs $338. These prices are valid until May 31, 2026.

**Observation:**  
The system correctly retrieves parking permit pricing tables and reports accurate numeric values. However, campus map evidence is not retrieved to explicitly justify the “closest” parking lot determination. The answer is therefore **correct but incompletely grounded**.

---

### Task 3 — Incomplete Evidence (Refusal)

**Question:**  
Will UMKC’s student enrollment decrease in 2026 compared to previous years?

**System Response:**  
Not enough evidence in the retrieved context.

**Observation:**  
The system correctly identifies that the provided documents do not contain enrollment projections and explicitly refuses to answer, demonstrating effective hallucination avoidance.


---

## Ablation & Tradeoffs

We qualitatively compare different retrieval strategies:

- **Dense retrieval** captures semantic similarity but may miss structured tables.
- **Sparse retrieval** reliably retrieves keyword-matching policy documents.
- **Hybrid retrieval** improves coverage across text, tables, and OCR-derived content.

Hybrid retrieval is selected for this sprint due to its **better evidence coverage**, despite slightly lower precision at top ranks.

---

## Failure & Risk Analysis

### Observed Failure
For shuttle-related questions, the system may infer plausible answers using campus maps alone, even when **shuttle route documentation is missing**.

### Risk
This can lead to **under-grounded answers** that appear correct but are not fully supported by evidence.

### Mitigation
- Enforce modality-aware gating (e.g., require shuttle schedule evidence for shuttle questions)
- Trigger partial or full refusal when required modalities are missing
- Surface evidence-pack transparency to users

---

## How to Run

1. Open the provided Jupyter Notebook
2. Install dependencies listed at the top of the notebook
3. Place PDFs and images in the expected data directories
4. Run all cells to reproduce ingestion, retrieval, and demo outputs

---

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
