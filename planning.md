# TakeMeter — Planning Document
## AI201 · Project 3

---

## 1. Community Choice

I chose the **Joe Rogan Experience (JRE)** YouTube comment sections. JRE is one of the most-watched podcasts in the world, and its comment sections are unusually text-heavy and opinionated compared to typical YouTube content. Viewers regularly debate the guests' claims, react emotionally to controversial moments, and occasionally offer structured arguments about what was said. This variety makes it an ideal community for a discourse quality classifier — the range from pure reaction ("this episode gave me chills") to genuine analysis ("Elon warned about AI regulation for years and nobody listened") is wide enough to be interesting and grounded enough to be learnable.

I collected comments from three episodes to ensure topical diversity:
- **JRE #1169 — Elon Musk**: tech, AI, space, societal commentary
- **JRE #1555 — Alex Jones & Tim Dillon**: conspiracy, politics, media criticism
- **JRE #1805 — Mike Tyson**: boxing, philosophy, personal transformation

---

## 2. Label Taxonomy

### `analysis`
**Definition:** The comment makes a structured point backed by evidence, facts, comparisons, or logical reasoning. The claim is specific and could be debated or verified with evidence.

**Example 1:**
> "Your big cat is chuffing. He wants to know if you're a threat. If you chuff back you're telling him you're not here to cause issues. But if you don't chuff back… it can be a sign that you are challenging."

**Example 2:**
> "JR thought he was the voice of reason on this. But way more people have seen the way the machine and its representatives work. So he just looks like an apologist for people who have claimed to be doing good."

---

### `hot_take`
**Definition:** A bold, confident opinion stated without real supporting evidence. The tone is assertive or provocative, but no argument is provided to back the claim.

**Example 1:**
> "The first 5 minutes off the rip hits different in 2026 and I'm not even joking. This is possibly Rogan's best episode ever."

**Example 2:**
> "Mike and Joe have so much in common in terms of interests it's crazy."

---

### `reaction`
**Definition:** An immediate emotional or surface-level response — hype, agreement, humor, nostalgia, or feelings. No argument, no real claim.

**Example 1:**
> "Watching in 2026. I want this Elon back."

**Example 2:**
> "I love watching Mike laughing, it's great!"

---

## 3. Hard Edge Cases

### The hardest boundary: `analysis` vs `reaction`
Many JRE comments *sound* analytical because they reference something specific said in the episode, but are actually just emotional responses dressed up in descriptive language.

**Example ambiguous post:**
> "I have a whole different view of Elon after this interview. He is an incredibly empathetic person. He's so sad… anxiety-ridden… and all his worries are legitimate."

This was initially labeled `analysis` but is actually `reaction` — it expresses a feeling about Elon rather than making a verifiable argument. The commenter is not reasoning; they are reacting emotionally to watching him.

**Decision rule:** If removing the opinion framing would leave no verifiable claim or structured reasoning behind, it is `reaction`. If a specific fact, comparison, or causal claim survives the removal of emotional language, it is `analysis`.

### The second hardest boundary: `hot_take` vs `reaction`
Short jokes, humorous observations, and casual one-liners were frequently mislabeled as `hot_take` when they are actually `reaction`. A hot_take requires a bold *claim* — a reaction just requires a feeling or observation.

**Decision rule:** Ask "is this person asserting something controversial or bold?" If yes → `hot_take`. If they are just describing what they noticed or how they felt → `reaction`.

---

## 4. Data Collection Plan

**Source:** YouTube comment sections — public, no authentication required.

**Target:** 70 comments per episode × 3 episodes = 210 total comments.

**Collection method:** Manual copy-paste into a Google Sheet with columns: `#`, `source`, `comment`, `label`. Comments were collected from Top Comments sort to ensure quality and engagement variety.

**Pre-labeling:** ChatGPT (GPT-4) was used to pre-label all 210 comments using the label definitions above. Every label was then manually reviewed and corrected across multiple review passes. Approximately 30+ labels were changed from the AI's initial assignments after human review.

**Imbalance handling:** After all corrections, `reaction` represented 67.1% of the dataset — reflecting the genuine distribution of YouTube comments where emotional responses naturally dominate. This imbalance was documented and addressed in the evaluation report rather than artificially rebalanced.

**Final label distribution:**
| Label | Count | % |
|---|---|---|
| reaction | 141 | 67.1% |
| hot_take | 37 | 17.6% |
| analysis | 32 | 15.2% |

---

## 5. Evaluation Metrics

**Primary metric: Overall accuracy** — reported for both the fine-tuned model and the Groq zero-shot baseline to measure the direct benefit of fine-tuning.

**Why accuracy alone is not enough:** With 67% of examples being `reaction`, a model that predicts `reaction` for every input would achieve 67% accuracy. Accuracy alone cannot reveal whether the model has actually learned to distinguish between classes.

**Additional metrics used:**
- **Per-class F1 score** — the harmonic mean of precision and recall for each label. This exposes whether the model has completely failed to learn a class (F1 = 0) while still achieving decent overall accuracy.
- **Confusion matrix** — shows exactly which labels are being confused and in which direction, enabling diagnosis of specific failure modes.
- **Macro-average F1** — averages F1 across all classes equally, penalizing models that ignore minority classes.

---

## 6. Definition of Success

A classifier would be **genuinely useful** as a community moderation or feed-ranking tool if it achieves:
- Overall accuracy ≥ 65% on the test set
- F1 ≥ 0.50 for at least two of the three classes
- Fine-tuned model meaningfully outperforms the zero-shot baseline

**Minimum acceptable threshold:** Overall accuracy above the majority-class baseline (67% = always predicting reaction). Falling below that would mean fine-tuning learned nothing useful.

**Honest assessment:** The final model (65.6% accuracy, reaction F1 = 0.79, analysis F1 = 0.00, hot_take F1 = 0.00) partially meets these criteria. It outperforms the Groq baseline (53.1%) but does not clear the majority-class ceiling. The model learned to identify `reaction` reliably but could not distinguish `analysis` from `reaction` or `hot_take` from `reaction` — a direct consequence of the dataset imbalance.

---

## 7. AI Tool Plan

### Label stress-testing
Before finalizing label definitions, ChatGPT was asked to generate borderline comments that could plausibly belong to two labels. This exercise revealed that the `analysis` vs `reaction` boundary was the most problematic — many emotionally-worded comments about Elon Musk or Mike Tyson superficially resembled analysis but contained no verifiable reasoning. The definitions were tightened as a result.

### Annotation assistance
ChatGPT (GPT-4) was used to pre-label all 210 comments in one pass using the label definitions from this document. The pre-labels were used as a starting point only — every single label was reviewed manually, and approximately 30+ were corrected across multiple review passes. Pre-labeling sped up the process but did not replace human judgment.

### Failure analysis
After each training run, wrong predictions were pasted into Claude and ChatGPT to identify systematic patterns. Both tools consistently flagged that the model was predicting `reaction` for almost all wrong predictions — confirming that the imbalance, not the model architecture or prompt, was the primary failure driver. This analysis directly informed the reflection section of the evaluation report.