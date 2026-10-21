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

## 1. Overall accuracy is at least 0.70 on the held-out test set.

**Why this target:** With three labels, guessing gets about 0.33. I need the
model to get 7 out of 10 posts right before I'd trust it to sort
r/personalfinance comments without checking each one myself.

---

## 2. The weaker of `reasoned_advice` and `bare_advice` has an F1 of at least 0.60.

**Why this target:** This is my hardest boundary — the only difference is
whether the advice is backed by a fact, number, or real rule, and short
comments can go either way. I'm allowing it to be lower than the overall
target, but it still has to be clearly better than a guess.

---

## 3. `question` has an F1 of at least 0.85.

**Why this target:** Question posts look very different from advice — they
describe the poster's own situation and end in "what should I do?". If the
model can't separate those from replies, it hasn't learned anything useful, so
this label should be close to perfect.

---

## 4. My smallest label is at least 25% of labels.csv.

**Why this target:** Most r/personalfinance comments are advice, so `question`
posts are likely to be my thinnest label. At 25% of 200 that's 50 posts, which
puts about 8 in the test split — the point where a per-label F1 stops jumping
around between seeds.

---

## 5. The model's most-confident third of test predictions is at least 20 percentage points more accurate than its least-confident third.

**Why this target:** If the model is sure about a comment, that should mean
something — I'd want to trust its confident calls and only check the unsure
ones by hand. With about 30 test posts, each third is only about 10 posts, so a
small gap could be noise; 20 points is a gap I'd believe.

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
