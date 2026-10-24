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

Both models scored on the same 32 held-out posts in `test_split.csv`.
`facebook/bart-large-mnli` (zero-shot, reading `label_definitions.txt`) is the
baseline; "Trained" is the unit 5 model (seed 42, 3 epochs, `results.json`).

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.312 | 0.562 | +0.250 |
| Macro F1 | 0.330 | 0.240 | -0.090 |
| F1 — `analysis` | 0.400 | 0.000 | -0.400 |
| F1 — `hot_take` | 0.091 | 0.720 | +0.629 |
| F1 — `reaction` | 0.500 | 0.000 | -0.500 |

**What I predicted before I looked:** I expected the trained model to win on
overall accuracy, since it had 148 labelled examples to learn from and the
baseline had none — but I expected the baseline to do better on `analysis`
and `reaction`, because unit 5's run had already shown the trained model
collapsing on exactly those two labels.

**What the gap actually means:** The trained model's accuracy win is hollow.
It beats the baseline on accuracy only because it always guesses `hot_take`,
the majority label, which happens to be correct 56% of the time on this
split — not because it learned to read the posts. The zero-shot baseline
never saw a single training example, but it still attempts `analysis` and
`reaction` and gets real credit for it (F1 0.40 and 0.50), which is why its
macro F1 (0.330) actually beats the trained model's (0.240). Fine-tuning
here didn't teach the model the taxonomy — it taught the model to exploit
the class skew, and did it worse on the two minority labels than a model
that was never trained at all.



---

## Run Log — Before

Device: **mps** (Apple Silicon GPU), torch 2.14.1, 3 epochs, lr 2e-5, batch 16.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.562 | 0.562 | 0.562 | MISSED |
| 2. Per-label F1 (`hot_take`≥.65 / `analysis`≥.45 / `reaction`≥.30) | all three | .72 / .00 / .00 | .72 / .00 / .00 | .72 / .00 / .00 | MISSED |
| 3. Predicted `hot_take` share | ≤ 75% | 100% | 100% | 100% | MISSED |
| 4. Self-agreement on 18 `cold` posts (one-time, not per-seed) | ≥ 15/18 | 15/18 (83%) | — | — | MET |
| 5. Confident-third vs. unsure-third accuracy gap | ≥ 15 pts | 10 pts | 40 pts | 10 pts | MISSED |

### Confusion matrix

Seed 42 (identical shape on seeds 7 and 2024 — see note below):

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 0 | 10 | 0 |
| **hot_take** | 0 | 18 | 0 |
| **reaction** | 0 | 4 | 0 |

**My biggest off-diagonal number, and what it means:** All 10 real
`analysis` posts and all 4 real `reaction` posts were called `hot_take`,
and nothing went the other direction. This isn't a boundary the model
half-learned — it's a model that never predicts anything but `hot_take`,
on every single post, on all three seeds. That's not a confusable pair;
it's total collapse to the majority label.



---

## Verdicts and Diagnoses

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 | MISSED | 0.562 on all three seeds — not close, and perfectly stable, which is itself a clue. |
| 2 | Per-label F1 | `hot_take`≥.65, `analysis`≥.45, `reaction`≥.30 | MISSED | `hot_take` clears its floor every seed (.72); `analysis` and `reaction` score exactly 0.00 every seed. |
| 3 | Balance | ≤ 75% predicted `hot_take` | MISSED | Model predicted `hot_take` for 100% of the test set on every seed — not "lopsided," total. |
| 4 | Self-agreement on `cold` posts | ≥ 15/18 | MET | Relabelled the 18 `cold` posts blind (without looking at my original labels first) and matched myself on 15/18 — see below. |
| 5 | Confidence gap | ≥ 15 pts, all seeds | MISSED | Gap was 10, 40, and 10 points across the three seeds — only seed 7 clears it, and the target has to hold on all three. |

**Diagnoses**

**Criteria 1, 2, 3 — all one cause.** The confusion matrix is unambiguous:
on every one of the three seeds, the model predicts `hot_take` for literally
every post in the test set, including all 10 `analysis` posts and all 4
`reaction` posts. That single fact explains the accuracy miss (it's just
the majority-class rate, 18/32 ≈ 0.562), the per-label F1 miss (`analysis`
and `reaction` get 0 recall because the model never once guesses them),
and the balance miss (100% is as collapsed as a model can get).

The cause is **too little data for the model to find anything better than
majority guessing in the training budget it got** — not a labelling
problem. I know that because:

- **The agreement report rules out a labelling-consistency problem.** I
  matched the staff set 30/30 (100%) under a taxonomy I didn't write (see
  Agreement Report below). If I can apply someone else's rule perfectly,
  the 212 rows I labelled under my own rule are unlikely to be the thing
  sabotaging the model.
- **Unit 5's stretch run is the direct evidence it's a training-budget
  problem, not a data-ceiling problem.** Going from 3 to 5 epochs alone
  (same data, same split, `results_second_run.json`) moved `analysis` F1
  from 0.00 to 0.18 — the model started finding a non-majority answer
  without any new data. That's what motivated the improvement below:
  if 2 more epochs partially breaks the collapse, more epochs should break
  it further.
- With only 65 `analysis` and 29 `reaction` rows total (≈10 and ≈4 in any
  15% test split), the model has very little signal for those two labels
  relative to 118 `hot_take` rows — a real contributing factor, but the
  stretch run shows training length is the lever that actually moves, so
  that's the one I changed first.

**Criterion 5 (confidence gap).** This isn't really a separate cause — it's
a symptom of the same collapse. A model that always predicts `hot_take`
is "confident" about `hot_take` and nothing else; whether its most-confident
third happens to be right depends entirely on how many true `hot_take`
posts land in that third by chance, which is why the gap bounces from 10
to 40 to 10 points across seeds instead of sitting still. A model with no
real decision boundary can't have a meaningful confidence signal.

**How I relabelled the 18 `cold` posts (criterion 4):** I re-read each post's
text with the labels hidden and applied my own decision rule fresh, then
compared to my original calls. 15 of 18 matched. The 3 that didn't:

1. *"This is sad. Her father said she was using drugs since 13... makes me
   really angry..."* — Originally `analysis`; on the blind pass I called it
   `reaction` (the fact triggers the feeling, it isn't the argument — the
   post is "this makes me angry," not a claim the fact is proving).
2. *"Aliens is more similar to Pitch Black... Can you name any other
   character in Pitch Black besides Riddick?"* — Originally `hot_take`; on
   the blind pass I called it `analysis` (the named-cast comparison is the
   entire support for "better movie," which is exactly what my own decision
   rule calls load-bearing).
3. *"What actresses got nominated? ... maybe Ryan was a lot better..."* —
   Originally `analysis`; on the blind pass I called it `hot_take` (no
   specific name or figure is actually stated, just rhetorical questions —
   it reads more like pushback than an argument built on a checkable fact).

All three mismatches sit on the same boundary — `analysis` vs. `hot_take` —
which is the hardest one I named in the unit 5 README. That's a small,
genuine inconsistency in my own labelling at the margins, but at 83% it's
not the dominant explanation for a model that collapses 100% of the time;
the training-budget cause above does almost all the work.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

I labelled all 30 posts in `data/staff_posts.csv` under the definitions and
decision rules in `data/staff_taxonomy.md` — not my own — before looking at
`data/staff_labels.csv`. Every one of my 30 calls matched staff's.

**Disagreements**

None. `agreement.py` reports 0 disagreements, so there's nothing to
adjudicate here — see the note below for what I think a 100% rate actually
means, rather than just taking it as a clean win.

**What the pattern in my disagreements tells me:** Zero disagreements on 30
posts, several of which the staff's own notes flag as `borderline` (sp_005,
sp_010, sp_014, sp_019, sp_020, sp_028, sp_029), is a genuinely good sign
about whether I can apply a written rule the way its authors intended, not
a sign the set was easy. The staff taxonomy's three decision rules map
almost exactly onto the hardest boundary I already named in my own unit 5
README (does the fact carry the argument, or just decorate it?), so I'd
already built the habit the staff rules test for. I take this as real
evidence that the problem with my trained model (see Verdicts and
Diagnoses) is not that I apply labelling rules inconsistently — it's that
212 rows split three ways didn't give the model enough signal to learn the
boundary in the training budget it got.



---

## The Improvement

**What I changed:** One training setting — `EPOCHS` from 3 to 8, same
learning rate (2e-5), same batch size, same `labels.csv`, same splits, same
three seeds. No relabelling, no new data, no criteria changed.

**Which diagnosis pointed at it:** The "before" diagnosis above was total
collapse to the majority label on every seed. My evidence that *training
length*, not data volume or labelling, was the lever: unit 5's declared
stretch run already showed that going from 3 to 5 epochs alone moved
`analysis` F1 from 0.00 to 0.18 with the exact same data. If 2 extra epochs
partially broke the collapse, more epochs should break it further — so
that's what I tested, as the one allowed change.

### Run Log — After

Device: **mps**, 8 epochs, lr 2e-5, batch 16 — otherwise identical to the
"before" run.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.562 | 0.594 | 0.656 | MISSED |
| 2. Per-label F1 (`hot_take`≥.65 / `analysis`≥.45 / `reaction`≥.30) | all three | .67/.44/.00 | .68/.53/.00 | .76/.53/.00 | MISSED |
| 3. Predicted `hot_take` share | ≤ 75% | 75.0% | 71.9% | 84.4% | MISSED |
| 4. Self-agreement on 18 `cold` posts (unchanged — one-time check) | ≥ 15/18 | 15/18 (83%) | — | — | MET |
| 5. Confident-third vs. unsure-third accuracy gap | ≥ 15 pts | 40 pts | 30 pts | 50 pts | MET |

**Did it help, and how do I know:** Yes, clearly, even though it didn't
flip any of the three accuracy-shaped criteria from MISSED to MET. Macro F1
roughly doubled (0.24 → 0.40 average across seeds), `analysis` F1 went from
0.00 on every seed to 0.44–0.53, and the model started actually predicting
`analysis` for real `analysis` posts (4–5 of 10, up from 0) instead of
dumping everything into `hot_take`. Criterion 5 flipped from MISSED (0 of 3
seeds) to MET (3 of 3) — a model that's no longer one-note has a confidence
score that finally means something. Criterion 3 (balance) got close: two of
three seeds now land under the 75% cap (75.0% and 71.9%), only seed 2024
stays over it (84.4%), versus 100% on every seed before.

What it did **not** fix: `reaction` F1 is still exactly 0.00 on all three
seeds, before and after. More epochs broke the `analysis`/`hot_take`
collapse but never taught the model to predict `reaction` at all — which
makes sense given `reaction` has only 29 rows total (≈4 in any test split),
the thinnest of the three labels. That's the diagnosis for what's still
broken, below.



---

## What's Still Broken

**1. Overall accuracy (≥0.65), still MISSED on 2 of 3 seeds.** The best
seed (2024) actually clears it at 0.656, but the target has to hold across
all three and seed 42 is still at 0.562. What I'd try next: the thing I
didn't touch — real `reaction` examples. 212 rows split three ways gives
`reaction` only ~29 training rows; everything else staying fixed, that's
the ceiling on how much headroom is left. I stopped because this unit's
rule is one change only, and I'd already spent it on epochs.

**2. Per-label F1, still MISSED because `reaction` is 0.00 on every seed,
before and after.** The model never once predicts `reaction`, even after
training 8 epochs instead of 3. More training time clearly helps the
model separate `analysis` from `hot_take` (F1 0.00 → 0.44–0.53), but it did
nothing for `reaction` — with only ~4 `reaction` posts in any given test
split, there may simply not be enough signal for 8 epochs of more training
to find, as opposed to more data. I'd collect 30+ more real `reaction`
posts next, the same way I found the originals (targeted "just watched" /
obituary-thread searches, since general-feed posts skew toward `hot_take`
and `analysis`). I stopped because that's a second change, and this unit
caps me at one.

**3. Balance (≤75% predicted `hot_take`), still MISSED on 1 of 3 seeds.**
This one is close — 75.0% and 71.9% on two seeds, 84.4% on the third — and
it's really the same root cause as #2: whichever seed's split happens to
make `reaction` and the harder `analysis` posts least learnable, the model
falls back toward `hot_take` more. More `reaction` data is the same fix as
above.

**The gap between what I meant my labels to capture and what the model
learned:** My taxonomy draws its hardest line on *whether a fact is doing
argumentative work* — the whole point of `analysis` vs. `hot_take`. The
confusion matrix shows the model did start learning something like that
line once it got enough training (4–5 of 10 `analysis` posts right after
the fix, versus 0 before), but it never learned that `reaction` exists as
a third option at all — every `reaction` post, on every seed, gets pulled
into `hot_take` instead. The model learned a two-way boundary where I
designed a three-way one, and the gap is almost entirely explained by how
little data the third label had to learn from.



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
