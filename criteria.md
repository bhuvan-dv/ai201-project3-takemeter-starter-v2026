# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy

The model scores at least **0.65 overall accuracy** on the held-out test split.

**Why this target:** `hot_take` is 55.7% of my 212-row CSV (118/212), so a classifier
that always guesses `hot_take` and nothing else already scores about 0.56 on
accuracy without having learned anything. 0.65 is deliberately set above that
naive-majority baseline — it's the line past which the model has to actually be
reading `analysis` and `reaction` posts correctly some of the time, not just
exploiting the class skew in r/movies-style discussion threads.

---

## 2. Per-label performance

Every label clears its own F1 floor on the test split: `hot_take` ≥ **0.65**,
`analysis` ≥ **0.45**, `reaction` ≥ **0.30**.

**Why this target:** The three labels don't have equal data — 118 `hot_take`,
65 `analysis`, 29 `reaction` — so one shared bar would be unfair to the
smallest class and too easy for the largest. `reaction`'s floor is the
loosest because only ~4 of its examples land in the 15% test split (29 × 0.15
≈ 4.4); an F1 computed on 4 examples moves in quarter-steps, so 0.30 is about
as precise a number as that split size can support. `analysis` sits in the
middle because it's a harder distinction to learn than `hot_take` — it
requires the model to notice *whether* a specific fact is doing argumentative
work, not just spot emotionally charged language.

---

## 3. Balance / collapse into the majority label

On the held-out test split, the model predicts `hot_take` for **no more than
75%** of the posts.

**Why this target:** This is the imbalance risk made concrete and measurable.
Because `hot_take` is already the majority of my training data, the single
most likely failure mode for this model isn't "confused between two labels" —
it's "learned to just say `hot_take` most of the time because that's usually
right." A model that predicts `hot_take` on, say, 95% of the test set could
still clear criterion 1's accuracy bar by accident. Capping the predicted
share at 75% (above the 55.7% true share, but well short of "guesses one
thing almost always") is a specific, checkable way to catch that failure mode
even if overall accuracy looks fine.

---

## 4. Consistency (self-agreement on the cold-labelled posts)

Relabelling the 18 posts I marked `cold` in `labels.csv` — without looking at
my original labels first — I agree with my own earlier labels on at least
**15 of 18 (≈ 83%)**.

**Why this target:** The `cold` rows are the ones I labelled before I'd read
any AI-assisted pre-labels, specifically as an honesty check per Milestone 4.
If I can't reproduce my own calls on my own taxonomy most of the time, the
200+ rows I labelled quickly with AI assistance were built on a taxonomy I
wasn't applying consistently either, and that's a labelling problem, not a
modelling one. 83% (15/18) is high but not 100%, because I already know two
or three of these 18 sit right on the `analysis`/`hot_take` boundary — the
same boundary my decision rule in the README exists to resolve, and even a
written rule doesn't make every edge case mechanical on a second pass.

---

## 5. Confidence means something

The model's predictions in the most-confident third of the test set are
correct at least **15 percentage points more often** than its predictions in
the least-confident third.

**Why this target:** `distilbert-base-uncased` outputs a softmax score
whether or not it actually knows anything, so the only way to tell if that
number is meaningful is to check whether high confidence actually tracks
correctness. A 15-point gap is a real, checkable separation — not "confident
predictions are slightly better," which could happen by chance on a 32-row
test split, but a gap big enough that it's unlikely to be noise given how few
test examples (≈32, per the 70/15/15 split) this model gets scored on.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
