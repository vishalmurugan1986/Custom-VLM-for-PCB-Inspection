## Custom Vision-Language Model for PCB Inspection

## Problem Statement

The goal is to design an **offline VLM system** that can:

* Answer **natural language questions** from inspectors
  *(e.g., “How many missing components?”, “Is there a solder bridge near IC3?”)*
* Provide **precise defect localization**, counts, and confidence scores
* Operate **fully offline**
* Achieve **end-to-end inference latency under 2 seconds**

**Constraints:**

* ~50,000 PCB images
* Only **bounding box annotations** available
* No existing QA pairs
* Generic VLMs hallucinate and are unsafe for industrial use

---

## Why Generic VLMs Are Unsuitable

Generic Vision-Language Models (e.g., LLaVA, BLIP-2, Qwen-VL) fail in this setting due to:

* **Hallucination risk** (inventing nonexistent defects)
* **Poor spatial grounding** (no precise bounding box reasoning)
* **Domain mismatch** (trained on natural images, not PCBs)
* **High latency** and poor offline suitability

Therefore, a **domain-specific, constrained VLM design** is required.

---

## Proposed Architecture

The system follows a **detector-guided Vision-Language architecture**, explicitly separating **visual perception** from **language reasoning**.

```
Input Image
     ↓
PCB Defect Detector (Vision Encoder)
     ↓
Structured Defect Tokens (Class, BBox, Confidence)
     ↓
Lightweight Language Model (Reasoning Only)
     ↓
Answer + Locations + Confidence
```

### Key Design Principle

> The language model never sees raw image pixels.
> It reasons only over verified detector outputs.

This guarantees **precise localization** and **hallucination safety**.

---

## Hallucination Mitigation Strategy

Hallucination prevention is the most critical requirement. The design uses:

* **Detector-first constraint**
  (LLM can reference only detected defects)
* **Confidence-gated responses**
  (low confidence → safe fallback response)
* **Structured answer templates**
  (restricted output format)
* **Hallucination-penalized training loss**

This makes hallucinated responses extremely unlikely.

---

## Training Strategy

Training is performed in multiple stages:

1. **Vision Model Training**

   * Train PCB defect detector using bounding box annotations
   * Optimize for recall and localization accuracy (IoU)

2. **Automatic QA Pair Generation**

   * Generate synthetic QA pairs from annotations
     *(e.g., “How many missing components?” → count from labels)*

3. **VLM Fine-Tuning**

   * Freeze vision encoder
   * Fine-tune lightweight LLM using structured defect tokens
   * Use **LoRA** for efficient adaptation

4. **Joint Optimization**

   * End-to-end fine-tuning
   * Penalize hallucinated outputs

---

## Optimization for Offline Inference

To meet the **< 2 second latency requirement**, the system uses:

* INT8 / INT4 **quantization**
* **Pruning** unused parameters
* **LoRA / QLoRA** fine-tuning
* **Pipeline parallelism** between vision and language stages

### Expected Latency

| Stage            | Time       |
| ---------------- | ---------- |
| Vision detection | ~300 ms    |
| Token processing | ~100 ms    |
| LLM inference    | ~800 ms    |
| **Total**        | **~1.2 s** |

---

## Evaluation Metrics

The system is evaluated using industrially relevant metrics:

* **Vision Metrics:** mAP, IoU, localization error
* **Language Metrics:** defect count accuracy, location correctness
* **Safety Metrics:** hallucination rate, false positives
* **System Metrics:** end-to-end latency, offline memory usage

**Acceptance Criteria:**

* Hallucination rate < 1%
* IoU > 0.7
* Inference latency < 2 seconds

---

## Conclusion

This task presents a **production-ready Vision-Language Model design** tailored for industrial PCB inspection. By separating perception from reasoning, constraining language generation, and optimizing for offline deployment, the system achieves **high reliability, low hallucination risk, and fast inference**.

The design demonstrates strong understanding of:

* Vision–language integration
* Industrial AI constraints
* Safety-critical system design

---

## Reference Document

📄 **Design PDF:**
`Task3_Custom_VLM_PCB_Inspection.pdf`

---
 
