# TakeMeter — JRE Discourse Classifier
## AI201 · Project 3

A fine-tuned text classifier that evaluates discourse quality in Joe Rogan Experience (JRE) YouTube comment sections, distinguishing between analytical comments, hot takes, and emotional reactions.

---
## Walkthrough video
https://www.loom.com/share/36b858ec28bc48069d288600c826cd7e

## Community Choice

I chose the **Joe Rogan Experience (JRE)** YouTube comment sections. JRE attracts one of the most opinionated and text-heavy comment communities on YouTube. Viewers debate guests' claims, react emotionally to controversial moments, and occasionally offer structured arguments — creating a natural spectrum from pure reaction to genuine analysis. Three episodes were selected for topical diversity:

- **JRE #1169 — Elon Musk** (tech, AI, societal commentary)
- **JRE #1555 — Alex Jones & Tim Dillon** (conspiracy, politics, media criticism)
- **JRE #1805 — Mike Tyson** (boxing, philosophy, personal transformation)

---

## Label Taxonomy

| Label | Definition |
|---|---|
| `analysis` | Makes a structured point backed by evidence, facts, comparisons, or logical reasoning. The claim is specific and verifiable. |
| `hot_take` | A bold, confident opinion stated without supporting evidence. Assertive or provocative tone, no argument behind it. |
| `reaction` | An immediate emotional or surface-level response — hype, agreement, humor, or feelings. No argument, no real claim. |

**Example per label:**

- **analysis:** *"JR thought he was the voice of reason on this. But way more people have seen the way the machine and its representatives work. So he just looks like an apologist."*
- **hot_take:** *"The first 5 minutes hits different in 2026 — this is possibly Rogan's best episode ever."*
- **reaction:** *"Watching in 2026. I want this Elon back."*

---

## Data Collection

**Source:** YouTube comment sections (public, top-sorted comments)
**Collection method:** Manual copy-paste across three JRE episodes, 70 comments each
**Pre-labeling:** ChatGPT (GPT-4) was used to pre-label all 210 comments using the label definitions. Every label was manually reviewed and corrected — approximately 30+ labels were changed across multiple review passes.

**Final label distribution:**
| Label | Count | % |
|---|---|---|
| reaction | 141 | 67.1% |
| hot_take | 37 | 17.6% |
| analysis | 32 | 15.2% |
| **Total** | **210** | |

**Dataset splits (stratified):**
| Split | Size |
|---|---|
| Train | 147 |
| Validation | 31 |
| Test | 32 |

**3 difficult-to-label examples:**

1. *"I have a whole different view of Elon after this interview. He is an incredibly empathetic person. He's so sad… anxiety-ridden… and all his worries are legitimate."*
   — Initially labeled `analysis`, corrected to `reaction`. Describes a feeling about Elon, not a structured argument.

2. *"The difference in Joe's attitude between hearing Randall Carson talking about climate change as a natural process proven by ice core samples, and hearing Alex Jones saying the exact same thing."*
   — Labeled `analysis`. Makes a specific comparative observation about Joe's behavior with verifiable context.

3. *"EDIT: CONSPIRACY! Elon Musk took it upon himself to try out technology similar to Neuralink and that's why he is the way he is currently."*
   — Labeled `hot_take`. Speculative and bold but not supported by evidence — it asserts rather than argues.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` (HuggingFace)
**Framework:** HuggingFace `transformers` + `Trainer` API
**Hardware:** Google Colab T4 GPU
**Training time:** ~25 seconds (3 epochs on 147 examples)

**Hyperparameters:**
| Parameter | Value | Reasoning |
|---|---|---|
| Epochs | 3 | Standard for small datasets; more risks overfitting |
| Learning rate | 2e-5 | Default for BERT-family fine-tuning, stable |
| Batch size | 16 | Fits T4 GPU comfortably |
| Weight decay | 0.01 | Light regularization for small dataset |

**Key hyperparameter decision:** Learning rate was kept at the default `2e-5`. Given only 147 training examples, a higher learning rate risked overfitting quickly while a lower one may not converge in 3 epochs. The validation accuracy progression (19% → 64% → 67%) confirmed the default was appropriate.

---

## Baseline Description

**Model:** `llama-3.3-70b-versatile` via Groq API (zero-shot, no fine-tuning)

**Prompt used:**
```
You are classifying YouTube comments from the Joe Rogan Experience (JRE) podcast.
Assign each comment to exactly one of the following categories.

analysis: The comment makes a structured point backed by evidence, facts, comparisons,
or logical reasoning. The claim is specific and could be debated with evidence.
Example: "Your big cat is chuffing. He wants to know if you're a threat..."

hot_take: A bold, confident opinion stated without real supporting evidence.
Example: "The first 5 minutes hits different in 2026, this is possibly Rogan's best episode ever."

reaction: An immediate emotional or surface-level response — hype, agreement, humor, or feelings.
Example: "This episode gave me chills omg I can't stop watching."

Respond with ONLY the label name. Do not explain your reasoning.
Valid labels: analysis / hot_take / reaction
```

All 32 test examples were parsed successfully (0 unparseable responses).

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot Groq baseline | 53.1% |
| Fine-tuned DistilBERT | **65.6%** |
| Improvement | **+12.5%** |

---

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.00 | 0.00 | 0.00 | 5 |
| hot_take | 0.00 | 0.00 | 0.00 | 6 |
| reaction | 0.66 | 1.00 | 0.79 | 21 |
| **accuracy** | | | **0.66** | 32 |
| macro avg | 0.22 | 0.33 | 0.26 | 32 |
| weighted avg | 0.43 | 0.66 | 0.52 | 32 |

---

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.29 | 0.40 | 0.33 | 5 |
| hot_take | 0.22 | 0.33 | 0.27 | 6 |
| reaction | 0.81 | 0.62 | 0.70 | 21 |
| **accuracy** | | | **0.53** | 32 |
| macro avg | 0.44 | 0.45 | 0.43 | 32 |
| weighted avg | 0.62 | 0.53 | 0.56 | 32 |

---

### Confusion Matrix — Fine-Tuned Model

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 0 | 0 | 5 |
| **True: hot_take** | 0 | 0 | 6 |
| **True: reaction** | 0 | 0 | 21 |

The diagonal (correct predictions) shows the model only learned to predict `reaction`. Every `analysis` and `hot_take` example in the test set was misclassified as `reaction`.

---

### 3 Wrong Predictions — Analysis

**Wrong Prediction #1**
> *"JR thought he was the voice of reason on this. But way more people have seen the way the machine and its representatives work. So he just looks like an apologist for people who have claimed to be doing good."*
> **True:** analysis | **Predicted:** reaction (confidence: 0.54)

This is a clear `analysis` comment — it makes a causal argument about Joe's public perception with implicit evidence (the machine's representatives). The model predicted `reaction` because it never learned the `analysis` boundary. With only 22 analysis examples in training and the model biased toward `reaction`, any comment without obvious structural cues was defaulted to the majority class.

**Wrong Prediction #2**
> *"The first 5 minutes off the rip hits different in 2026 and I'm not even joking. This is possibly Rogan's best episode ever."*
> **True:** hot_take | **Predicted:** reaction (confidence: 0.51)

This is a borderline case — the comment expresses a strong opinion ("best episode ever") which qualifies as `hot_take`, but the emotional framing ("hits different," "I'm not even joking") reads like `reaction`. The model's confidence was only 51%, showing genuine uncertainty. This reflects the inherent ambiguity in the `hot_take` vs `reaction` boundary for short, casual comments.

**Wrong Prediction #3**
> *"I'm watching this 5 years since its posting... Alex is right about everything. Joe is programmed."*
> **True:** analysis | **Predicted:** reaction (confidence: 0.57)

This was labeled `analysis` because it makes a comparative judgment over time ("5 years later, Alex was right"). However, the model predicted `reaction` — and arguably the model is not entirely wrong. The comment lacks explicit evidence and reads more like an emotional conclusion than structured reasoning. This is a genuine annotation edge case that reveals the `analysis` definition may have been applied too loosely in the dataset.

---

### Sample Classifications

| Comment | Predicted Label | Confidence | Notes |
|---|---|---|---|
| "Watching in 2026. I want this Elon back." | reaction | 0.89 | ✅ Correct — pure nostalgia, no argument |
| "Mike is so bright." | reaction | 0.91 | ✅ Correct — surface-level praise |
| "Joe Rogan gets better and better" | reaction | 0.87 | ✅ Correct — emotional endorsement |
| "Well well well. I'm 5 minutes in and it seems just as relevant as 5 years ago" | reaction | 0.53 | ❌ True: hot_take — model uncertain, defaulted to majority class |
| "Mike and Joe have so much in common in terms of interests it's crazy" | reaction | 0.57 | ❌ True: hot_take — low confidence, boundary ambiguity |

The correctly predicted examples share a clear characteristic: they are short, emotional, and contain no claim. The model learned this pattern reliably. Where it fails is on any comment that contains a claim — regardless of whether it is `hot_take` or `analysis`, the model defaults to `reaction`.

---

### What the Model Learned vs. What I Intended

**What I intended:** A classifier that distinguishes structured reasoning (analysis) from unsupported bold claims (hot_take) from emotional responses (reaction).

**What the model actually learned:** A classifier that identifies `reaction` reliably and predicts `reaction` for everything else.

The gap is entirely explained by dataset imbalance. With 67% of training examples labeled `reaction`, the model found that predicting `reaction` constantly minimized its training loss more effectively than learning the minority class boundaries. The `analysis` and `hot_take` classes had only 22 and 26 training examples respectively — not enough signal for a 66M parameter model to learn meaningful decision boundaries.

The Groq baseline actually outperformed the fine-tuned model on minority classes (analysis F1: 0.33 vs 0.00; hot_take F1: 0.27 vs 0.00) because a large language model with no fine-tuning can apply general reasoning about what constitutes an argument — something our small imbalanced dataset could not teach DistilBERT.

**What would fix this:** Collecting 80–100 examples per class (240–300 total) with balanced representation would likely produce a model that learns all three boundaries. The label definitions themselves are sound — the problem is data quantity, not label quality.

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on defining a hard edge case *before* annotating 200 examples was the most valuable instruction in the project. Forcing myself to write the decision rule for `analysis` vs `reaction` before labeling revealed that the boundary was much harder than it initially seemed — and prevented hundreds of inconsistent labels.

**One way implementation diverged:** The spec recommends aiming for at least 20% per label. My final distribution ended up at 15% analysis and 17% hot_take after thorough relabeling. Rather than artificially adding examples to hit the threshold, I accepted the natural distribution that emerged from honest annotation and documented it. The divergence produced a more honest dataset at the cost of model performance on minority classes — a trade-off I would make the same way again.

---

## AI Usage

**Instance 1 — Pre-labeling 210 comments:**
I directed ChatGPT (GPT-4) to label all 210 comments using the label definitions from planning.md. It produced a complete labeled CSV. I then reviewed every single label manually across multiple passes and corrected approximately 30+ labels. The most common error was ChatGPT over-labeling emotional comments as `analysis` when they contained no actual reasoning — for example, labeling "I have a whole different view of Elon after this interview" as `analysis` rather than `reaction`.

**Instance 2 — Failure pattern analysis:**
After each Colab training run, I pasted the wrong predictions into Claude and asked it to identify systematic patterns. Claude identified that every wrong prediction was classified as `reaction`, pointing to dataset imbalance as the root cause rather than model architecture or prompt issues. I verified this independently by examining the confusion matrix, which confirmed 0 correct predictions for `analysis` and `hot_take`. Claude's pattern identification matched my own reading of the data.

**Instance 3 — planning.md and README drafting:**
Claude helped draft the planning.md and README based on the actual results, label definitions, and wrong prediction analysis developed throughout the project. All factual claims (accuracy numbers, label counts, example quotes) came from the actual dataset and Colab outputs.