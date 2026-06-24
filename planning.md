# TakeMeter: Planning Document
**Project:** r/soccer Discourse Classifier  
**Community:** r/soccer (Reddit)  
**Model Target:** Fine-tuned DistilBERT for 4-class post classification

---

## 1. Community

I chose r/soccer because it is one of the largest and most active sports communities on Reddit, with over 4 million regular participants and extremely high activity during the 2026 FIFA World Cup. The community produces a wide variety of text-based discourse within a single shared context — the same match or transfer story will generate factual news reports, emotional reactions, bold opinions, and structured tactical arguments all within hours of each other.

This makes r/soccer an ideal fit for a classification task: the discourse is varied enough that the distinctions are meaningful, but grounded enough in a shared domain (soccer) that the labels are stable and interpretable. A regular participant in this community would immediately recognize the difference between someone posting a verified transfer update and someone asserting that a manager "has to go" with no supporting evidence. These distinctions matter to the community because discourse quality is constantly debated — r/soccer users frequently criticize low-effort hot takes and celebrate detailed analytical posts.

---

## 2. Label Taxonomy

### `analysis`
**Definition:** A post that constructs a structured argument using specific, verifiable evidence — statistics, historical comparisons, tactical breakdowns, or scenario modeling — where the claim would hold up independently of any emotional framing.

**Example 1:**
> "I really hope FIFA switches back to goal differential as the tiebreaker. Under the current head-to-head format, 9/48 teams have nothing to play for on matchday 3. Under goal differential, all 48 teams would still have something to play for. This also creates unfair scheduling situations like Group A where Czechia faces a rotated Mexico squad while South Africa plays full-strength South Korea."

This is `analysis` because it presents a structured argument comparing two formats with specific counts (9/48 teams), historical precedent, and a concrete consequence (unfair scheduling asymmetry).

**Example 2:**
> "England v Ghana is the first match at the 2026 FIFA World Cup not to have a single shot on target in the first half. [OptaJoe]"

This crosses into `analysis` territory when paired with context — a verifiable, specific statistical observation that supports a broader point about England's attacking quality.

---

### `hot_take`
**Definition:** A post that states a bold opinion or prediction confidently, with little or no supporting evidence — any evidence cited is decorative rather than load-bearing, and the post asserts rather than reasons.

**Example 1:**
> "10 men and a statue: Portugal are sacrificing another World Cup for Cristiano Ronaldo's ego."

This is `hot_take` because the claim ("sacrificing the World Cup") is a strong judgment stated without statistical support or tactical breakdown — the framing is designed to provoke, not argue.

**Example 2:**
> "Leaving Cole Palmer at home is a call Thomas Tuchel will come to regret. Tuchel's squad is made up of players all of a similar ilk — every change from the bench last night was like for like."

This is `hot_take` because while it contains a tactical observation (like-for-like subs), the core move is a pundit prediction with no verifiable evidence. The observation supports a foregone conclusion rather than leading to one.

---

### `reaction`
**Definition:** A post expressing an immediate emotional response to a specific event — a goal, result, transfer, or controversy — with little to no argument, where the post's value is the feeling expressed rather than information or reasoning conveyed.

**Example 1:**
> "Cristiano Ronaldo shouts at camera 'IM BACK' after post game"

This is `reaction` because it captures an emotional moment in real time. There is no argument or factual report — just the expression of feeling.

**Example 2:**
> "Khusanov in tears after 5-0 defeat to Portugal"

This is `reaction` because it documents an emotional response to a result, with no added analysis or opinion from the poster.

---

### `news`
**Definition:** A post that reports a verifiable, factual event or development — transfer confirmations, match results, official records, or statistical milestones — presented without added opinion or emotional framing, where the post's value is the information itself.

**Example 1:**
> "[Fabrizio Romano] BREAKING: Marco Palestra to Chelsea, here we go! Verbal agreement in place between all parties. Atalanta to receive package over €55m fee plus sell-on clause."

This is `news` because it reports a verifiable transaction with specific, attributable details — no opinion or emotional framing added by the poster.

**Example 2:**
> "Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions (2006, 2010, 2014, 2018, 2022 & 2026)."

This is `news` because it reports a verifiable record with specific years cited — the information itself is the point.

---

## 3. Hard Edge Cases

### Edge Case 1: Quote posts from managers or pundits
**The problem:** Many r/soccer posts quote a manager or pundit saying something bold. The post format looks like `news` (it's reporting what someone said), but the *content* of the quote is a `hot_take`.

**Example:**
> "Mourinho: 'There is a silly theory that you can be great without winning. For me, it's totally silly.'"

**Decision rule:** Label by the *content of the quote*, not the format. If the quote itself is a bold claim without evidence → `hot_take`. If the quote reports a factual development ("We've agreed to sign X") → `news`. The Mourinho post above is `hot_take` because the content is an unsupported assertion, not a factual report.

---

### Edge Case 2: Stat-backed hot takes
**The problem:** A post cites one or two statistics to support what is fundamentally a predetermined conclusion. It looks like `analysis` because numbers are present, but the reasoning is decorative.

**Example:**
> "Mbappe is overrated — he only scores against weak opposition. His xG against top-8 sides is 0.3 per game."

**Decision rule:** Ask whether the evidence is *load-bearing* or *decorative*. If removing the stat would collapse the argument → the stat is doing real analytical work → `analysis`. If removing the stat leaves the same bold claim intact → the stat is decorative → `hot_take`. The Mbappe post above is `hot_take` because "he's overrated" is the predetermined conclusion; the xG figure is cherry-picked to sound credible.

---

### Edge Case 3: Celebratory stat posts
**The problem:** Some posts report a record or milestone with clear celebratory framing (ALL CAPS, exclamation points), blurring the line between `news` and `reaction`.

**Example:**
> "[ESPN FC] History for Curaçao as they earn their first-ever World Cup point! What a moment for the smallest country to EVER qualify for the World Cup!"

**Decision rule:** If the post's primary content is a verifiable fact and the emotional framing is editorial color added by the source or poster → `news`. If the post contains no factual claim and is purely expressing excitement → `reaction`. The Curaçao post is `news` because the milestone is the substance; the exclamation points are presentation.

---

## 4. Data Collection Plan

**Source:** r/soccer comment sections and post titles, collected manually during the 2026 FIFA World Cup group stage.

**Target distribution (200 total, 50 per label):**

| Label | Count | Primary source |
|---|---|---|
| `analysis` | 50 | Comment sections of tactical threads, post-match discussions, "why did X lose" debates |
| `hot_take` | 50 | Messi/Ronaldo debate threads, manager criticism posts, controversial result threads |
| `reaction` | 50 | World Cup match thread comments during and immediately after games |
| `news` | 50 | Post titles: transfer news, result posts, official records |

**If a label is underrepresented after 150 examples:**
- For `analysis`: Search r/soccer for threads tagged [Tactical] or post-match analysis posts; look for long comments with multiple paragraphs
- For `hot_take`: Search for threads about controversial managers (Tuchel, Mourinho-era debates) or GOAT debates
- Do not pad underrepresented labels with borderline examples just to hit the number — quality of annotation matters more than exact balance

**Collection method:** Manual copy-paste into a CSV with columns: `text`, `label`, `notes`. The `notes` column will flag any example that required non-obvious label judgment. For `news` labels, post titles are sufficient. For the other three labels, full comment text was collected (not just the title of the parent post).

**Actual final distribution collected:**

| Label | Count | Percentage |
|---|---|---|
| `news` | 75 | 35.4% |
| `reaction` | 60 | 28.3% |
| `hot_take` | 42 | 19.8% |
| `analysis` | 35 | 16.5% |
| **Total** | **212** | **100%** |

`analysis` and `hot_take` were harder to collect than expected — they live primarily in comment sections rather than post titles, requiring more manual effort. `news` was overcollected because post titles are fast to gather. No single label exceeded 70%.

---

## 5. Evaluation Metrics

**Primary metrics:**
- **Per-class F1 score** for all four labels — this is the most important metric because it balances precision and recall for each individual label and penalizes both over-prediction and under-prediction equally
- **Macro-averaged F1** — treats all four classes equally regardless of their frequency in the test set, which is appropriate here because all four labels are equally important to the classifier's usefulness
- **Confusion matrix** — shows which label pairs the model confuses most, which reveals whether the hard edge cases (e.g., `hot_take` vs `analysis`) are being learned correctly

**Why accuracy alone is not enough:**
With four balanced classes (25% each), a model that always predicts `news` would achieve 25% accuracy. Even with slight imbalance, a model could reach 40-50% accuracy by over-predicting the majority class. Per-class F1 and the confusion matrix reveal whether the model has actually learned all four distinctions or is just exploiting distributional shortcuts.

**Secondary metrics (stretch features):**
- **Confidence calibration:** Whether the model's probability scores are meaningful — does a 90% confident prediction get it right more often than a 60% one?
- **Error pattern analysis:** Whether wrong predictions share a systematic pattern rather than being random noise.

---

## 6. Definition of Success

**Minimum threshold for "good enough":**
- Overall accuracy ≥ 70% on the held-out test set
- No single label with F1 < 0.55 — a model that completely fails on one class is not deployable even if overall accuracy looks acceptable
- Fine-tuned model must outperform the zero-shot Groq baseline by at least 10 percentage points in overall accuracy

**Threshold for "genuinely useful in a community tool":**
- Overall accuracy ≥ 78%
- All four per-class F1 scores ≥ 0.65
- The `analysis` vs `hot_take` confusion (the hardest boundary) should account for fewer than 30% of total errors

**Rationale:** These thresholds are set conservatively because the task involves subjective judgment — a human annotator would not achieve 100% agreement on these labels. A classifier at 78%+ accuracy with balanced per-class performance would be genuinely useful for things like filtering low-effort posts, surfacing analytical content, or flagging reaction posts in non-match-thread contexts.

**Actual results vs. thresholds:**
- Overall accuracy: 90.6% ✅ (exceeded both thresholds)
- All per-class F1 ≥ 0.65: ✅ (lowest was `reaction` at 0.84)
- Fine-tuning improvement over baseline: +9.4% (just under the 10pt target; baseline was unusually strong at 81.2%)

---

## 7. AI Tool Plan

### Label stress-testing
I gave Claude my four label definitions and the three edge case descriptions above, and asked it to generate 8–10 posts that sit at the boundary between two specific label pairs: `hot_take` vs `analysis`, and `news` vs `reaction`. Several generated examples revealed that my initial definition of `news` was too broad — celebratory stat posts with excited language were genuinely ambiguous. I added the decision rule "if the primary content is a verifiable fact, label it news regardless of emotional framing" as a result. This was done before any annotation began.

### Annotation assistance
I used Claude to pre-label batches of 20–30 examples at a time by providing my label definitions and a set of raw posts and asking it to assign one label per post with a one-sentence justification. Claude pre-labeled approximately 120 of the 212 examples. I reviewed and corrected every pre-assigned label — roughly 15–20% required correction, mostly on the `hot_take`/`reaction` boundary for quote posts. All pre-labeled examples are flagged in the `notes` column of the CSV.

### Failure analysis
After fine-tuning, I pasted all 3 misclassified examples from the test set into Claude and asked it to identify common patterns. Claude identified that all three errors involved the `reaction` label and that emotional language was the likely confounding feature. I verified this pattern by re-reading the errors myself and confirmed it was accurate. The full pattern analysis is documented in the README evaluation report.

---
