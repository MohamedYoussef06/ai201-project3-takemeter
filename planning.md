# Project 3: TakeMeter — Planning Document

## Community

**r/leagueoflegends** — a subreddit with millions of active members discussing Riot Games' League of Legends. The community produces a high volume of text-heavy posts daily, ranging from detailed mechanical guides to excited patch-day reactions to frustrated rants about balance decisions. This is an ideal classification target because the discourse quality varies enormously and in recognizable ways: experienced players can immediately distinguish a post that teaches something from one that just vents. The community itself has informal norms around these distinctions ("this is just a rant," "actually useful tips"), making the labels meaningful to real users and not just an academic exercise.

---

## Labels

Three labels, mutually exclusive:

### `strategy`
A post that provides actionable, specific guidance — champion builds, itemization decisions, macro plays, mechanics explanations, or patch-adapted advice — where the primary purpose is to help the reader improve or make better in-game decisions. The post must do more than state that something is good or bad; it must explain *why* or *how*.

**Clear examples:**
1. "With the current jungle meta favoring early-game pressure, you should take Smite + Flash and prioritize scuttle at 3:15 rather than full clearing. Here's the path: red → blue → gromp → scuttle. This forces the enemy jungler to react rather than farm freely."
2. "Kraken Slayer is being over-built on ADCs right now. The passive only procs every 3 hits, and against most mid-lane mages you're better off with Galeforce for the dash. Save Kraken for compositions with two or more tanks."

**Uncertain post:** "You should definitely be playing Jinx into this meta, she's just too strong right now." — Reads like strategy advice (champion recommendation) but gives no reasoning. → This is `hype` not `strategy` (see decision rule below).

---

### `hype`
A post expressing excitement, enthusiasm, or strong positive sentiment about a champion, patch, skin, pro play moment, or the game itself — without structured reasoning or actionable content. The post is celebrating or reacting positively, not arguing or teaching.

**Clear examples:**
1. "HOLY MOLY Faker just hit Challenger on every server simultaneously. This man is not human."
2. "Patch 14.10 looks insane. Mobility creep is finally getting addressed and I'm so ready. GG Riot."

**Uncertain post:** "You should definitely be playing Jinx into this meta, she's just too strong right now." — Champion recommendation framing, but no evidence or reasoning. → `hype` (see decision rule below).

---

### `complaint`
A post expressing frustration, criticism, or negative sentiment toward the game, Riot Games, specific champions, balance decisions, matchmaking, or community behavior. The post is venting or criticizing, not proposing solutions or explaining reasoning.

**Clear examples:**
1. "I'm so tired of assassins one-shotting everyone from fog of war with zero counterplay. This game rewards no-brain burst damage and punishes anyone who actually tries to play the game properly."
2. "Riot has buffed Zed four patches in a row while leaving Senna untouched. Their balance team clearly doesn't play the game."

**Uncertain post:** "Why does Riot keep making every new champion a 400-damage skill-shot champion? It's like they hate tanks." — Framed as a question, but it's clearly frustration with no genuine inquiry. → `complaint`.

---

## Decision Rules for Hard Edge Cases

### Primary hard case: strategy-sounding hype
A post that recommends a champion or build but provides no real explanation is `hype`, not `strategy`.

**Decision rule:** If you could remove the conclusion ("play Jinx," "build Kraken") and the remaining content would still teach the reader something, it's `strategy`. If removing the conclusion leaves nothing substantive, it's `hype`. A genuine strategy post contains reasoning that stands on its own.

**Example application:** "Jinx is S-tier right now, you should spam her to climb." → Remove the recommendation → nothing left → `hype`. Compare: "Jinx's late-game hyperscaling means in low-elo games that go 35+ minutes she wins almost every fight — play her if your ranked games are going long." → Remove the recommendation → the observation about late-game scaling still stands → borderline, but the reasoning is real → `strategy`.

### Secondary hard case: complaint disguised as a question
Posts phrased as rhetorical questions ("why does Riot keep doing X?") with no genuine inquiry are `complaint`. If the post would accept an actual answer and engage with it, it's a `discussion` (but we don't have a discussion label, so these go to `complaint` if the frustration tone dominates, or `hype` if the sentiment is positive).

---

## Data Collection Plan

**Source:** r/leagueoflegends — public posts and top-level comments. No private content, no locked threads.

**Target distribution:** ~70 examples per label (210 total), aiming for balance within ±15 examples across classes. This prevents a dominant majority class from producing a trivially-high-accuracy but useless classifier.

**Collection method:**
1. Browse the subreddit's Hot, Top (past month), and New feeds
2. Copy post title + body text (or top-level comment) into a Google Sheet with columns: `id`, `text`, `label`, `notes`
3. Use Claude as a pre-labeler: provide the label definitions and 10–15 raw posts at a time, ask for a suggested label per post with a one-sentence reason
4. Review every pre-assigned label myself and correct where Claude misapplied the definitions — document corrections in the `notes` column
5. Flag the pre-labeled column separately to disclose which examples were AI-assisted

**If a label is underrepresented:** Specifically target that label type. For `strategy`, look in the weekly "Champion Discussion" and "Help & Support" megathreads. For `hype`, look at patch day threads and highlight/clip posts. For `complaint`, look in post-patch threads and rant-adjacent posts.

**What counts as a unit:** Either a standalone post (title + body, minimum 2 sentences) or a top-level comment (minimum 30 words). Short one-liners like "lol" or "true" are excluded.

---

## Evaluation Metrics

**Primary metric: macro F1-score.** The dataset aims for balance, but even with effort one class may be somewhat over- or underrepresented. Macro F1 averages F1 across all classes without weighting by frequency, so it penalizes a model that ignores a smaller class. Accuracy would reward a model that achieves 60% by predicting the majority class — macro F1 would not.

**Per-class F1 and confusion matrix.** The confusion matrix reveals *which* errors the model is making. A model that confuses `strategy` and `hype` is making a different kind of mistake than one that confuses `hype` and `complaint`. Understanding the confusion pattern is essential for knowing whether the errors are random noise or a systematic label-boundary problem.

**Why accuracy alone is insufficient:** If `complaint` ends up at 45% of the dataset due to collection bias, a model that predicts `complaint` for everything scores 45% accuracy. Macro F1 would give this model ~15% because its F1 on the other two classes is 0. Accuracy would hide this failure.

---

## Definition of Success

**Minimum viable threshold:** Macro F1 ≥ 0.70 on the held-out test set. At this level, the classifier is making meaningful distinctions — not just guessing the majority class — and is reliable enough to use as a first-pass filter in a community tool.

**Deployment-ready threshold:** Macro F1 ≥ 0.80 with no single class below F1 = 0.70. A class with F1 below 0.70 means the model is systematically failing on that type of post, which would produce unfair or misleading results for that segment of the community.

**What "good enough" looks like in practice:** If the classifier were used to tag new posts in a subreddit tool, a user seeing a `strategy` tag should find it correct at least 4 times out of 5. A macro F1 of 0.80 approximates that standard across all three labels.

**Failure condition:** If macro F1 is below 0.65 or any class F1 is below 0.55, the classifier is not distinguishing labels reliably enough to be useful and the label definitions or annotation quality need revisiting.

---

## AI Tool Plan

### 1. Label stress-testing
Before annotating more than 30 examples, I will give Claude the three label definitions and the primary edge case (strategy-sounding hype) and ask it to generate 10 posts that sit at the boundary between `strategy` and `hype`. I will then attempt to classify each generated post using my definitions. If I cannot assign a label confidently to more than 2 out of 10, the boundary needs tightening. This happens before bulk annotation to avoid locking in a flawed taxonomy across 200 examples.

### 2. Annotation assistance
I will use Claude (claude-sonnet-4-6) to pre-label batches of 10–15 posts at a time. The prompt will include: the three label definitions, the two decision rules, and 2 examples per class as a few-shot header. Claude will assign a label and provide a one-sentence reason. I will review every suggested label and mark each row with a `prelabeled_by_ai` flag (boolean column in the CSV). Corrections will be noted in the `notes` column. I will not skip review for any pre-labeled example.

### 3. Failure analysis
After model evaluation, I will extract all test-set examples where the model's prediction differs from the true label. I will give this list to Claude with the label definitions and ask it to identify patterns: Are the errors concentrated in a particular label? Do they share surface features (short posts, question-phrasing, sarcasm)? I will then verify the identified patterns by manually reading the flagged examples — I will not take Claude's pattern summary at face value without checking the actual text.

---

## Hard Edge Cases Encountered During Annotation

*(To be filled in during Milestone 3 annotation — document at least 3 cases here.)*

| Post excerpt | Candidate labels | Decision | Reason |
|---|---|---|---|
| TBD | TBD | TBD | TBD |

---

## AI Usage Disclosure

*(To be completed after annotation and modeling.)*

- Claude (claude-sonnet-4-6) used for: label stress-testing, pre-labeling annotation batches, failure analysis
- All pre-labeled examples reviewed and corrected by the author
- Pre-labeled examples flagged in CSV with `prelabeled_by_ai = True`
- planning.md written with Claude Code assistance
