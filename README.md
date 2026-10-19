# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

**TakeMeter** sorts comments from **lemmy.world's `!movies` community** — a
general film-discussion forum, not built around any single release — into
three categories: `analysis`, `hot_take`, and `reaction`. The goal is to
separate comments that argue something from a checkable fact from comments
that are just confident opinion, and both of those from comments that are
really just someone reporting how a movie made them feel in the moment. All
212 posts in `labels.csv` are real comments pulled from that community's
public API (no login required), collected and hand-labelled by me for this
project.

**Stretch feature declared:** a second training run with one hyperparameter
changed (epochs: 3 → 5), with the before/after numbers and what moved — see
**The Training Run** below.

---

## Label Taxonomy

### `analysis`

**Definition:** Makes an argument about a movie, show, or performance that is
*carried* by a specific, checkable fact — a score, a dollar figure, a date,
a named comparison, a quoted source — not just decorated by one.

**Example 1:**
> "Even adjusting for inflation, Pluto Nash still wins. It opened to $3.5M in
> today's money. I feel like there hasn't been much marketing for
> Megalopolis. Could be a factor."

**Example 2:**
> "1) Greta did get a nomination, just not for director. 2) Margot didn't,
> but America did. 3) Ryan Gosling was the best part of Barbie, partly
> because of the writing which, again, WAS nominated."

### `hot_take`

**Definition:** A confident claim about a movie, show, or actor's quality or
merit, offered with no checkable fact doing the actual work — even when the
tone is certain.

**Example 1:**
> "He's not a good actor. Not interesting. Boring to watch. He has zero
> talent in the thespian realm. In short, I generally don't like the films he
> is in. Just my opinion."

**Example 2:**
> "Crazy how both of those washed up losers stopped making good movies. Like
> they were good once upon a time, now they play the same boring af
> predictable characters in every shitty B-tier action movie they jump on."

### `reaction`

**Definition:** An immediate, personal, feeling-first response to a movie
just watched or news just heard — reporting the writer's own experience
rather than arguing a general claim about the work.

**Example 1:**
> "Shit! I'm halfway through watching Brooklyn Nine Nine for the first time.
> He's really fun to watch and absolutely nails the role. Rest in peace you
> badass mofo."

**Example 2:**
> "Took my kid to see Moana 2 yesterday. Medium popcorn was $11.50. WTF..."

### The hardest boundary

**Which two labels:** `analysis` vs. `hot_take`.

**The decision rule I used every time:** If a post's conclusion would survive
having its one specific fact deleted, the fact is decoration and the post is
`hot_take`. If deleting the fact would leave the conclusion with nothing
holding it up, the fact is load-bearing and the post is `analysis` — even if
the tone is casual or the post is also opinionated elsewhere. Example: "90%
of that Hollywood shit is hot garbage" uses a number, but the "90%" isn't a
real, checkable figure doing work — delete it and the sentence says the exact
same thing, so it's `hot_take`. "Even adjusting for inflation, Pluto Nash
still wins — it opened to $3.5M" is `analysis`, because the specific box
office figure *is* the argument; without it there's no claim left to make.

---
---

## The Dataset

**Where the posts came from:** `lemmy.world`'s `!movies@lemmy.world`
community (14k+ subscribers, ~17.5k comments at the time of collection),
pulled through Lemmy's public, unauthenticated `/api/v3` endpoints — no login
wall, no scraper fighting bot detection. I pulled comments across `Top`,
`Hot`, and `New` listings for general variety, plus a second and third
targeted pass searching post titles for obituary/"just watched"/nostalgia
language (`"dies at"`, `"just watched"`, `"RIP"`, `"nostalgia"`, etc.)
specifically to find more `reaction`-shaped posts, since that style of
comment turned out to be rare in the general feed.

**How I labelled them:** I used a keyword-pattern script as a first-pass,
disclosed "AI" pre-labeller (score each comment against word/phrase lists
for each of the three labels, assign the top-scoring label), then I read and
corrected every single one of the 212 rows by hand — including re-reading
whole batches of 50-160 at a time and overruling the heuristic wherever its
guess didn't hold up against my decision rule. 18 rows are marked `cold` in
the `note` column: posts where I deliberately labelled against or ahead of
the heuristic's suggestion, as the honesty check Milestone 4 asks for. Every
other row's `note` says `pre-labelled (keyword heuristic), read and
corrected`.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `hot_take` | 118 | 55.7% |
| `analysis` | 65 | 30.7% |
| `reaction` | 29 | 13.7% |
| **Total** | 212 | 100% |

No label exceeds the 70% cap, but `reaction` is genuinely thin — it was the
hardest label to find in the wild. Most comments on a movie are either
flat opinion or argued critique; comments that read as an immediate,
in-the-moment feeling turned out to need targeted searching (obituary
threads, "just watched" threads) rather than just showing up in the general
feed. I call this out explicitly in `criteria.md`'s per-label and consistency
targets rather than papering over it.

**Three hard cases**

**1.**
> *The post:* "I really liked it. Bikes and cars go vroom, guns go bang,
> flamethrowers go woosh. 10/10. I'm contemplating going again next weekend.
> My roommate was bored. Didn't like the writing, didn't like Taylor-Joy's
> performance. He rated it 'not good.' I didn't really disagree with any of
> his specific complaints, I just didn't care about them."
>
> *Could have been:* `analysis` (it names specific critique points — writing,
> a named performance) or `reaction` (it's reporting a personal, just-watched
> experience with no real argument being made).
>
> *I chose:* `reaction`. The named critique points belong to the roommate,
> not the writer — the writer's own contribution is pure immediate feeling
> ("I really liked it," "I'm contemplating going again"), with zero interest
> in arguing a conclusion. Naming someone else's specific complaints isn't
> the same as building an argument from them.

**2.**
> *The post:* "There was suppose to be five. Every three-episode arc of
> season two was suppose to be a full season. But Disney said the show was
> way too expensive so Tony Gilroy negotiated one final stupid-expensive
> season and rolled everything he has outlined into it."
>
> *Could have been:* `hot_take` (my keyword script flagged it that way — no
> explicit score or percentage) or `analysis`.
>
> *I chose:* `analysis`. It names a specific, checkable production fact
> (Disney's budget decision, the showrunner's renegotiation) that is the
> entire argument for *why* the season structure changed. This is exactly
> the case my decision rule is for: the heuristic missed it because it has
> no number or date, but the fact is still load-bearing.

**3.**
> *The post:* "Rip A24, it will be remembered for the good movies they had."
>
> *Could have been:* `reaction` (my keyword script flagged "RIP" as
> reaction-shaped) or `hot_take`.
>
> *I chose:* `hot_take`. "RIP" here isn't grief about something that just
> happened to the writer — it's a flat, general claim about a studio's
> legacy with nothing checkable behind it. This is the kind of case where a
> surface keyword (`RIP`) pointed the wrong way and reading the actual
> content mattered more than the trigger word.

---

## The Training Run

**Base model:** `distilbert-base-uncased`.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42 (the notebook's defaults — unchanged for the primary run).

**Anything I changed from the defaults, and why:** Nothing for the primary
run (`results.json`). Device: Apple Silicon GPU (MPS), recorded directly in
`results.json`.

**Split sizes:** 70/15/15 stratified split of 212 rows → **train 148 · val
32 · test 32**. Per-label counts in the 32-row test split: `hot_take` 18,
`analysis` 10, `reaction` 4. `reaction`'s test count is well under 8 — per
the notebook's own warning, its F1 on this split is going to jump around a
lot between seeds, which is exactly why `criteria.md`'s target for it is the
loosest of the three.

**Primary run result (seed 42, `results.json`):** accuracy **0.563**, macro
F1 **0.240**. The model collapsed to predicting `hot_take` for every single
test example — precision/recall/F1 of 0 for both `analysis` and `reaction`.
This is the class-imbalance failure mode the unit warns about, happening in
practice: with `hot_take` at 55.7% of only 148 training rows and 3 epochs,
the model found "always guess the majority class" before it found anything
about the actual boundaries. This is a real, legitimate result for this
unit — the brief is explicit that how well the model scores isn't graded
here, only whether the run happened and the metrics were kept honestly.

**Stretch — second training run, one hyperparameter changed:** I reran
training with everything else identical except **epochs: 3 → 5**
(`results_second_run.json`). Accuracy moved **0.563 → 0.594**, macro F1 moved
**0.240 → 0.306**. More importantly, the model stopped fully collapsing:
`analysis` went from F1 0.000 → **0.182** and `hot_take` improved to
**0.735**; `reaction` stayed at 0.000 (its test split is only 4 examples, and
2 more epochs on 148 rows isn't enough signal to pull a 29-example label out
of the majority-class local minimum). What moved: two extra epochs gave the
model just enough additional gradient steps to start separating `analysis`
from `hot_take` on some examples, but did nothing for the thinnest label —
which is itself a useful, specific finding about where this dataset's limit
is, not just a parameter-tuning footnote.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I wrote a keyword/regex heuristic (not a hosted LLM) to
  pre-label all ~550-1100 candidate comments across the three labels, scoring
  each comment against word lists for `analysis` (box office/score/date
  patterns), `reaction` (immediate first-person/grief/just-watched language),
  and `hot_take` (best/worst/overrated-style opinion words), defaulting
  unmatched comments to `hot_take`.
- *What came back:* A heavily skewed first pass — out of ~1100 cleaned
  candidates, the heuristic found only ~50 plausible `reaction` examples and
  ~150 `analysis` examples against ~900 `hot_take`-flagged ones. It also
  mis-flagged plenty of individual comments: anything containing "RIP" got
  flagged `reaction` even when the post was a flat opinion ("Rip A24, it will
  be remembered...") rather than grief, and anything with a dollar figure or
  year got flagged `analysis` even when the number was decorative rather than
  load-bearing.
- *What I changed:* I read every heuristic-flagged `reaction` and `analysis`
  candidate myself and reclassified anything the heuristic got wrong based on
  my decision rule (see **Label Taxonomy**) — about 12 of 51 `reaction`-
  flagged comments turned out to be `hot_take` on close reading, and a few
  `hot_take`-flagged comments (like the Disney/Andor budget one in **Three
  hard cases**) turned out to be `analysis` the keyword list had no pattern
  for. I also ran three extra targeted search passes against Lemmy's search
  API for obituary/nostalgia/"just watched" language specifically because
  the first pass surfaced so few genuine `reaction` examples.

**Moment 2**

- *What I asked for:* Nothing from a hosted AI model for the taxonomy itself
  — the three label definitions, the decision rule, and the criteria in
  `criteria.md` are mine, written from actually reading the comments pulled
  above. I used this session's own agentic coding assistant (not a prompted
  "write me a taxonomy" request) to write and run the Python collection,
  cleaning, and heuristic-labelling scripts, and to drive the training
  notebook's execution end to end.
- *What came back:* Working scrapers against the wrong API shape twice before
  I found one that worked — Reddit's JSON endpoints now require a login
  (HTTP 302 to `/login`), so I switched to Lemmy, whose `/api/v3` is public.
- *What I changed:* Nothing about the taxonomy or the labels — the only
  thing the assistant's output changed was *which public forum* the data
  came from, which is an infrastructure decision, not a labelling one.

**Pre-labelling disclosure:** Every row in `labels.csv` was pre-labelled by
the keyword heuristic described above, then read and corrected by hand.
18 rows are marked `cold` in the `note` column — these are posts I
deliberately labelled by my own judgment first, overriding or independent of
the heuristic's suggestion, as the Milestone 4 honesty check. The remaining
194 rows are marked `pre-labelled (keyword heuristic), read and corrected`.
No row's label is the heuristic's raw, unreviewed output.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
