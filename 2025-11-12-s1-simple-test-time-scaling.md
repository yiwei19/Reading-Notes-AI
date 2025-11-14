# s1: Simple Test-Time Scaling — Paper Summary  
*Niklas Muennighoff et al., EMNLP 2025*  
**Link:** https://github.com/simplescaling/s1  
**Tags:** reasoning, test-time-scaling, LLMs, SFT, small-data

---

## Table of Contents
- [1. Overview](#1-overview)
- [2. Problem Statement](#2-problem-statement)
- [3. Why It Matters](#3-why-it-matters)
- [4. Key Contributions](#4-key-contributions)
- [5. Method Details](#5-method-details)
  - [5.1 Constructing s1K (1000-sample dataset)](#51-constructing-s1k-1000-sample-dataset)
  - [5.2 Budget Forcing for Test-Time Scaling](#52-budget-forcing-for-test-time-scaling)
- [6. Experiments & Results](#6-experiments--results)
- [7. Limitations](#7-limitations)
- [8. My Reflections](#8-my-reflections)

---

# 1. Overview

This paper proposes **the simplest method to achieve test-time scaling for LLM reasoning**.  
Instead of expensive RL (as used by OpenAI’s o1), the authors show that:

- **1000 carefully selected reasoning samples**  
- + **supervised fine-tuning (SFT) on Qwen2.5-32B**  
- + **a simple decoding trick called Budget Forcing**  

→ can produce a model (**s1-32B**) that *outperforms* o1-preview and nearly matches Gemini 2.0 Thinking on competition math tasks (AIME24 / MATH500).

---

# 2. Problem Statement

Test-time scaling is a new paradigm where **increasing inference-time compute improves reasoning accuracy**.  
However:

- closed models (o1, o3) do not disclose their method  
- open-source replicas rely on costly RL pipelines  
- no previous method *cleanly reproduces* the test-time scaling phenomenon  

**Goal:**  
Find the *simplest possible* open-source recipe for test-time scaling that works.

---

# 3. Why It Matters

Reasoning is becoming the critical bottleneck for LLM performance.

Traditional scaling relies on massive pretraining.  
But test-time scaling offers a different axis:

> Let the model “think longer” and get better answers — without retraining.

If small-data SFT + simple inference tricks work, this dramatically lowers the barrier to strong reasoning models.

---

# 4. Key Contributions

### ✔ **(1) A tiny but powerful reasoning dataset — s1K (1000 samples)**  
Selected from an initial **59,029 question pool** using 3 principles:

- **difficulty**  
- **diversity**  
- **quality**

Surprisingly, training on all 59k *does not* improve performance over the 1k set.


### ✔ **(2) Budget Forcing — a simple decoding-time control method**

Two operations:

#### **Force the model to stop thinking**  
Append end-of-thinking delimiter when token budget is reached.

#### **Force the model to think more**  
Suppress stop-token and append `"Wait"` to encourage deeper reasoning.

This can lead to **self-correction**, e.g.:

> “The answer is 2.” → Wait → “Actually it's 3.”


### ✔ **(3) A strong open-source reasoning model — s1-32B**

- Based on Qwen2.5-32B-Instruct  
- Finetuned for **only 26 minutes on 16×H100**  
- Outperforms o1-preview on MATH500 + AIME24  
- Shows *clear test-time scaling* behavior  

---

# 5. Method Details

## 5.1 Constructing s1K (1000-sample dataset)

From 59k initial problems (math Olympiad, standardized tests, sciences, teasers), they filter down using:

- difficulty ranking  
- reasoning diversity  
- sample clarity and quality  
- reasoning-trace usefulness  

Reasoning traces and answers are distilled from Gemini Thinking and DeepSeek R1.

**Key insight:**  
Quality > Quantity.  
Good reasoning traces matter more than correctness (≈50% traces contain wrong answers).

---

## 5.2 Budget Forcing for Test-Time Scaling

A lightweight, inference-only technique to control thinking time.

### **To enforce a maximum**
Append:

```text
<end-of-thinking>
Final Answer:
```


### To enforce a minimum

Suppress <end-of-thinking> and append:

```text
Wait
```

Sequential scaling (longer reasoning continuation) is shown superior to parallel scaling (majority voting).



# 6. Experiments & Results

### Model: s1-32B (Qwen2.5-32B + SFT on s1K + Budget Forcing)

### Benchmarks
-	AIME24 (competition math)
-	MATH500
-	GPQA Diamond (PhD-level science)

### Key findings

✔ Clear test-time scaling curve
Accuracy increases as thinking-tokens increase.

✔ Beats o1-preview
Especially on AIME24 and MATH500.

✔ Nearly matches Gemini 2.0 Thinking
Despite using:
- 26 min of SFT
- only 1000 samples
- no RL
- simple inference control

✔ More sample-efficient than all other open models

---

# 7. Limitations
- Budget forcing eventually plateaus
- Sequential scaling limited by context length
- Distillation depends on access to strong teacher models
- Not evaluated on creative or abstract reasoning tasks

---

# 8. My Reflections

This work is fascinating because it suggests:
- Reasoning quality is not primarily about dataset size
Good traces > big datasets.
- “Wait” tokens as a way to prompt self-reflection
Super simple yet powerful.

For my future projects (robotics planning, vision-language agents, NeRF reasoning):
- test-time scaling could improve long-horizon planning
- budget forcing could help enforce multi-step verification
- small SFT datasets might be enough for robust reasoning

I want to experiment with applying budget forcing to:
- multi-step robotics tasks
- NeRF inversion and geometry reasoning
- my future embodied agent projects

---

Created by Ivy Wei — Daily Paper Reading Log.
