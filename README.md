# TakeMeter: r/soccer Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in r/soccer by categorizing posts into four labels: `analysis`, `hot_take`, `reaction`, and `news`. Built during the 2026 FIFA World Cup group stage, when the subreddit was producing a rich mix of all four post types simultaneously.

---

## Community Choice

**r/soccer** is one of the largest sports communities on Reddit with over 4 million regular participants. It was chosen because its discourse is unusually varied within a single shared context: the same match or transfer story will generate factual news reports, emotional reactions, bold opinions, and structured tactical arguments all within hours of each other.

This makes it an ideal fit for a classification task. A regular participant in this community would immediately recognize the difference between someone posting a verified transfer update and someone asserting that a manager "has to go" with no supporting evidence. The 2026 FIFA World Cup group stage timing was particularly useful — high traffic, high emotion, and a concentration of all four post types in real time.

---

## Label Taxonomy

### `analysis`
A post that constructs a structured argument using specific, verifiable evidence — statistics, historical comparisons, tactical breakdowns, or scenario modeling — where the claim would hold up independently of emotional framing.

**Example 1:** "Portugal's 3-4-2-1 was defensively solid but their wing-backs were unsure whether to press or track, giving Nuno Mendes and Cancelo immense freedom. Congo completed 19 total tackles to Portugal's 11, made 28 clearances to Portugal's 10. Their defensive shape made it incredibly difficult for Portugal to find space in the final third."

**Example 2:** "I really hope FIFA switches back to goal differential as the tiebreaker. Under the current head-to-head format, 9/48 teams have nothing to play for on matchday 3. Under goal differential, all 48 teams would still have something to play for. This also creates unfair scheduling situations like Group A where Czechia faces a rotated Mexico squad."

---

### `hot_take`
A post that states a bold opinion or prediction confidently, with little or no supporting evidence — any evidence cited is decorative rather than load-bearing, and the post asserts rather than reasons.

**Example 1:** "10 men and a statue: Portugal are sacrificing another World Cup for Cristiano Ronaldo's ego."

**Example 2:** "Leaving Cole Palmer at home is a call Thomas Tuchel will come to regret. Tuchel's squad is made up of players all of a similar ilk — every change from the bench last night was like for like."

---

### `reaction`
A post expressing an immediate emotional response to a specific event — a goal, result, transfer, or controversy — with little to no argument, where the post's value is the feeling expressed rather than information or reasoning conveyed.

**Example 1:** "Cristiano Ronaldo shouts at camera 'IM BACK' after post game"

**Example 2:** "Khusanov in tears after 5-0 defeat to Portugal"

---

### `news`
A post that reports a verifiable, factual event or development — transfer confirmations, match results, official records, or statistical milestones — presented without added opinion or emotional framing, where the post's value is the information itself.

**Example 1:** "[Fabrizio Romano] BREAKING: Marco Palestra to Chelsea, here we go! Verbal agreement in place between all parties. Atalanta to receive package over €55m fee plus sell-on clause."

**Example 2:** "Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions (2006, 2010, 2014, 2018, 2022 & 2026)."

---

## Data Collection

**Source:** r/soccer and r/footballtactics post titles and comment sections, collected manually during the 2026 FIFA World Cup group stage (June 17–24, 2026). Additional examples drawn from The Athletic, BBC Sport, ESPN FC, FIFA.com, and Fabrizio Romano's Twitter for `news` examples.

**Labeling process:** Each example was labeled using the definitions above. Post titles were used for `news` examples; full comment text was collected for `analysis`, `hot_take`, and `reaction`. A `notes` column flagged any example requiring non-obvious judgment. An LLM (Claude) was used to pre-label batches of 20–30 examples, with every pre-assigned label reviewed and corrected before inclusion.

**Label distribution:**

| Label | Count | Percentage |
|---|---|---|
| `news` | 75 | 35.4% |
| `reaction` | 60 | 28.3% |
| `hot_take` | 42 | 19.8% |
| `analysis` | 35 | 16.5% |
| **Total** | **212** | **100%** |

No single label exceeds 70% of the dataset.

**Three genuinely difficult examples:**

1. *"Thierry Henry in his post-match analysis about Ronaldo: 'The team needs to score, not you need to score'"* — This is a quote post with emotional delivery. The format looks like `news` (reporting what Henry said) but the content is a bold opinion without evidence. Labeled `hot_take` because the decision rule is to label by content, not format.

2. *"Mbappe is overrated — he only scores against weak opposition. His xG against top-8 sides is 0.3 per game."* — Looks like `analysis` because it cites a stat, but the stat is cherry-picked to support a predetermined conclusion. Labeled `hot_take` because the evidence is decorative, not load-bearing.

3. *"German legend Franz Beckenbauer has passed away today aged 78. RIP Franz. — Fabrizio Romano"* — The "RIP Franz" tribute language gives it emotional tone, but the core content is a factual death announcement. Labeled `news` because the primary value is the information itself, not the feeling expressed.

---

## Fine-Tuning Pipeline

**Base model:** `distilbert-base-uncased` from HuggingFace  
**Training platform:** Google Colab (T4 GPU)  
**Training libraries:** `transformers`, `datasets`, `scikit-learn`

**Key hyperparameter decision:** The default setting of 3 epochs produced near-random accuracy (46.9%) on the validation set — the model was still largely guessing after 3 passes through 148 training examples. I increased `num_train_epochs` from 3 to **10** because with only ~37 examples per class on average, the model needed more passes to learn the subtle linguistic boundaries between labels, particularly `analysis` vs `hot_take`. The training loss dropped steadily from 1.40 at epoch 1 to 0.31 at epoch 10, confirming the model was genuinely learning rather than overfitting (validation accuracy plateaued at 87.5% from epochs 7–10 rather than declining).

**Other settings kept at defaults:** learning rate 2e-5, batch size 16, weight decay 0.01, warmup steps 50.

---

## Baseline Description

**Approach:** Zero-shot classification using `llama-3.3-70b-versatile` via Groq API. The model was given label definitions and one example per label, then asked to output only the label name with no explanation.

**Prompt used:**

```
You are classifying posts from r/soccer, a large football discussion community on Reddit.
Assign each post to exactly one of the following categories:

analysis: The post constructs a structured argument using specific, verifiable evidence — statistics, historical comparisons, tactical breakdowns, or scenario modeling — where the claim would hold up independently of emotional framing.
Example: "Portugal's 3-4-2-1 was defensively solid but their wing-backs were unsure whether to press or track, giving Cancelo and Mendes immense freedom. Congo completed 19 tackles to Portugal's 11."

hot_take: A post that states a bold opinion or prediction confidently, with little or no supporting evidence — any evidence cited is decorative rather than load-bearing, and the post asserts rather than reasons.
Example: "Tuchel has no idea what he's doing. England look exactly the same as under Southgate — boring and clueless. He needs to be sacked after the group stage."

reaction: A post expressing an immediate emotional response to a specific event — a goal, result, transfer, or controversy — with little to no argument.
Example: "Cristiano Ronaldo shouts at camera IM BACK after post game"

news: A post that reports a verifiable, factual event or development — transfer confirmations, match results, official records, or statistical milestones — presented without added opinion or emotional framing.
Example: "Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions (2006, 2010, 2014, 2018, 2022 & 2026)."

Respond with ONLY the label name. Do not explain your reasoning. Do not add punctuation.
Valid labels: analysis, hot_take, reaction, news
```

**How results were collected:** The prompt was run against all 32 test set examples with `temperature=0`. All 32 responses were parseable (0 unparseable).

---

## Evaluation Report

### Overall Results

| Model | Accuracy | Test Set Size |
|---|---|---|
| Zero-shot baseline (Groq llama-3.3-70b) | 81.2% | 32 |
| Fine-tuned DistilBERT (10 epochs) | **90.6%** | 32 |
| Improvement | +9.4% | — |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 1.00 | 1.00 | 1.00 | 5 |
| `hot_take` | 1.00 | 0.86 | 0.92 | 7 |
| `reaction` | 0.80 | 0.89 | 0.84 | 9 |
| `news` | 0.91 | 0.91 | 0.91 | 11 |
| **macro avg** | **0.93** | **0.91** | **0.92** | 32 |

### Per-Class Metrics — Baseline

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| `analysis` | 1.00 | 1.00 | 1.00 | 5 |
| `hot_take` | 0.86 | 0.86 | 0.86 | 7 |
| `reaction` | 0.64 | 0.78 | 0.70 | 9 |
| `news` | 0.89 | 0.73 | 0.80 | 11 |
| **macro avg** | **0.85** | **0.84** | **0.84** | 32 |

### Confusion Matrix — Fine-Tuned Model

|  | Predicted: analysis | Predicted: hot_take | Predicted: reaction | Predicted: news |
|---|---|---|---|---|
| **True: analysis** | 5 | 0 | 0 | 0 |
| **True: hot_take** | 0 | 6 | 1 | 0 |
| **True: reaction** | 0 | 0 | 8 | 1 |
| **True: news** | 0 | 0 | 1 | 10 |

The model made 3 errors total. All errors involve the `reaction` label — either predicting `reaction` when the true label was something else, or predicting another label when `reaction` was correct. The `analysis` label was classified perfectly.

### Error Analysis — 3 Wrong Predictions

**Error #1**
- **Text:** "Thierry Henry in his post-match analysis about Ronaldo after Portugal's game vs DR Congo: The team needs to score, not you need to score"
- **True label:** `hot_take` | **Predicted:** `reaction` (confidence: 0.53)
- **Analysis:** This is a quote post with emotional, in-the-moment delivery ("The team needs to score, not you!"). The model correctly detected the emotional register but failed to recognize that the content is a bold opinion assertion rather than an immediate feeling. The root cause is annotation boundary ambiguity: quote posts from pundits look like reactions on the surface but should be classified by their content. With only a few such examples in the training set, the model learned to weight the emotional framing over the content type.

**Error #2**
- **Text:** "German legend Franz Beckenbauer has passed away today aged 78. RIP Franz. — Fabrizio Romano"
- **True label:** `news` | **Predicted:** `reaction` (confidence: 0.41)
- **Analysis:** The "RIP Franz" tribute language is emotionally charged, which pulled the model toward `reaction`. The low confidence (0.41) shows the model was genuinely uncertain. This is a distributional problem: death announcements are rare in the training set and the emotional language pattern ("RIP") appears more often in reaction examples. The fix would be more death/retirement announcement examples in training to teach the model that emotional language in a factual report doesn't make it a reaction.

**Error #3**
- **Text:** "Kansas City honors Lionel Messi after World Cup hat trick"
- **True label:** `reaction` | **Predicted:** `news` (confidence: 0.53)
- **Analysis:** This post reads like a news headline — it describes a factual event (a city honoring a player) with neutral language. The model correctly detected the headline format and predicted `news`. However, the annotation treats this as a `reaction` because it documents a celebratory community moment rather than reporting a verifiable fact. This reveals an annotation consistency issue: short posts documenting celebratory events sit on the `news`/`reaction` boundary, and my decision rule ("if the primary content is a verifiable fact, it's news") was not applied consistently here. This is the most defensible wrong prediction — a second annotator might label this `news`.

### Sample Classifications

| Post (truncated) | Predicted Label | Confidence | Notes |
|---|---|---|---|
| "Portugal's 3-4-2-1 was defensively solid but their wing-backs were unsure whether to press or track..." | `analysis` | 0.94 | ✅ Correct. Strong tactical argument with specific formations and causal reasoning. |
| "Ronaldo Nazário: It's time for the world to stop hiding and accept the fact that Messi is the greatest of all time." | `hot_take` | 0.89 | ✅ Correct. Bold GOAT claim from a famous figure with no supporting evidence. |
| "Khusanov in tears after 5-0 defeat to Portugal" | `reaction` | 0.91 | ✅ Correct. Pure emotional moment — no argument or factual report. |
| "[Fabrizio Romano] BREAKING: Marco Palestra to Chelsea, here we go! Verbal agreement in place between all parties." | `news` | 0.96 | ✅ Correct. Classic transfer confirmation with specific fee and deal details. |
| "Thierry Henry: The team needs to score, not you need to score" | `reaction` | 0.53 | ❌ Wrong (true: hot_take). Emotional delivery of a pundit opinion confused the model. |

---

## Reflection: What the Model Captured vs. What Was Intended

The fine-tuned model learned the `news` and `analysis` boundaries extremely well — both achieved perfect or near-perfect F1. The more interesting finding is where it struggled: the `reaction`/`hot_take` boundary, specifically for **quote posts from pundits and managers**.

What the model appears to have learned is a **surface-level emotional register detector**: posts with excited or emotional language get classified as `reaction`, posts with calm declarative language get classified as `news` or `hot_take`. This works most of the time because reactions genuinely tend to be emotional and news genuinely tends to be neutral. But it fails on quote posts, where a pundit's emotional delivery of a bold opinion gets misclassified as `reaction`.

The intended distinction was based on *content structure* (assertion without evidence = hot_take, feeling expressed = reaction), but the model learned a *tone* shortcut instead. This is a classic case of a model learning a spurious correlation that works on most training examples but breaks on the hard cases.

To fix this, I would need more quote-post examples explicitly labeled `hot_take` in training — enough that the model learns that emotional tone does not automatically mean `reaction` when the speaker is making a bold claim.

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on defining a hard edge case before annotating any examples was the most valuable guidance in the project. Writing the decision rule for quote posts (label by content, not format) before collecting data forced me to think through the exact boundary that the model later struggled with. Without that explicit rule, my annotations would have been inconsistent and the model would have learned an even noisier signal.

**One way implementation diverged:** The spec suggested 3 epochs as the default and recommended caution about increasing them. In practice, 3 epochs was completely insufficient for a 4-class task with only 148 training examples — the model achieved only 46.9% accuracy. I increased to 10 epochs, which is higher than the spec's cautious guidance, but justified by the small dataset size and the clear learning signal in the training loss curve. The spec's caution about overfitting didn't apply here because the validation accuracy plateaued rather than declining.

---

## AI Usage

**Instance 1: Pre-labeling annotation batches.** I provided Claude with my four label definitions and batches of 20–30 unlabeled posts and asked it to assign one label per post with a one-sentence justification. Claude pre-labeled approximately 120 of the 212 examples. I reviewed and corrected every pre-assigned label — roughly 15–20% required correction, mostly on the `hot_take`/`reaction` boundary for quote posts. All pre-labeled examples are flagged in the `notes` column of the CSV.

**Instance 2: Failure pattern analysis.** After fine-tuning, I pasted the 3 wrong predictions into Claude and asked it to identify common themes. Claude identified that all three errors involved the `reaction` label and that emotional language was the likely confounding feature — posts with emotional tone were being pulled toward `reaction` regardless of their content structure. I verified this pattern by re-reading the errors myself and confirmed it was accurate. This pattern is documented in the error analysis section above.

**Instance 3: Label stress-testing.** Before annotating any examples, I gave Claude my four label definitions and asked it to generate 8–10 boundary posts that sat between two specific label pairs (`hot_take` vs `analysis`, and `news` vs `reaction`). Several of the generated examples revealed that my initial definition of `news` was too broad — celebratory stat posts with excited language were genuinely ambiguous. I added the decision rule "if the primary content is a verifiable fact, label it news regardless of emotional framing" as a result.
