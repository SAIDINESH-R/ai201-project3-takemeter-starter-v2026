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

TakeMeter sorts text from **r/personalfinance**, a public Reddit community where people describe their money situations and others reply with advice. Each post or top-level comment goes into one of three labels: `question` (someone asking for help with their own money), `reasoned_advice` (advice backed by a fact, number, real rule, or explanation of how something works), or `bare_advice` (advice with no real backing). The interesting line is the second one: plenty of advice on the sub is confident and specific but never says *why*, and this classifier tries to tell that apart from advice that does.

---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `question`

**Definition:** A post where someone describes their own money situation and asks for help with it.

**Example 1:**
> Previous employer took 401k from paychecks, now found out there is no 401k account, what do I do? — Hello! My previous employer had taken deductions from my paychecks for a 401k. I worked there from 2023-2025, where I was under the impression there was a 401k account with fidelity that my retirement would be going to. Fast forward to my new employer offers a 401k plan and I contact fidelity to see if these can be connected/merged somehow. Lo and behold fidelity has nothing for me on file. […] What should I do?

**Example 2:**
> Does anyone in their 20's feel like they're behind everyone else?

### `reasoned_advice`

**Definition:** Advice backed by an actual justification — a fact, a number, a real rule (a law, tax rule, or policy), or an explanation of how something works.

**Example 1:**
> Typically, ADP won't have anything to do with your 401k except processing the deduction. This is a question your old employer needs to answer. If it was a minimal balance, it's entirely possible they cashed you out and sent you a check. You can also check unclaimed funds. If a company is "advanced" enough to use ADP, it's highly unlikely they're pocketing your funds. Not saying it didn't happen, but there are other possibilites.

**Example 2:**
> What you are describing is federally illegal and likely illegal in your state (regardless of your age at the time). […] Report 401k theft in ALL OF THE FOLLOWING WAYS: To the U.S. Department of Labor's Employee Benefits Security Administration (EBSA) […] I've been involved with legal matters and law firms over the years in and outside of the ERISA 401K space and can not over emphasize how important it is to have clear notes in a folder as I've described. […] This is your money and it compounds, so it is far more than $9k and will be sizable as the years pass.

### `bare_advice`

**Definition:** Advice that says what to do without real backing — no reason at all, or only a common opinion.

**Example 1:**
> Do not pay on the house if it isn't in your name.

**Example 2:**
> If the so-called financial advisor tries to sell you an annuity, run away. The only person who makes money from an annuity is the shark selling it to you

### The hardest boundary

**Which two labels:** `reasoned_advice` vs. `bare_advice`

**The decision rule I used every time:** A comment is `reasoned_advice` only if it backs the advice with an actual justification — a fact, a number, a real rule (a law, tax rule, or policy), or how something works. A "reason" that is just a common perception or an insult ("the advisor is a shark") is not proper backing, so the comment is `bare_advice`. The commenter's own personal situation ("my wife and I combined don't make that") is also not backing — it's their situation, not a fact — so it doesn't make a comment `reasoned_advice` either. A rule of thumb ("never co-sign a loan") is popular wisdom, not a real rule, so it doesn't count either. Length doesn't decide it: a short comment can be reasoned, and a long one can be bare.

### What I left out

Replies that neither ask for help nor give advice — commenters doubting the poster's story ("Something isn't aligning"), side questions to the poster ("Don't 401K providers send statements?"), and the poster's own follow-up replies. They don't fit any of the three labels, so I didn't collect them, rather than force them into a label or add a fuzzy fourth one.


---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** r/personalfinance, through the public Arctic Shift Reddit archive (reddit.com itself blocked scripted requests). Original posts (title + body) from roughly September 2026, plus up to four of the highest-scored top-level comments from each thread. Deleted/removed text, AutoModerator, and anything under 80 characters were dropped. I read threads on the site myself first (Milestone 1, notes in `notes.md`).

**How I labelled them:**
1. **20 cold**, by me alone with no AI, before seeing any pre-labels. Marked `cold`.
2. **The other 262 were pre-labelled by Claude** using my definitions and decision rule, and Claude flagged 83 as hard (boundary cases).
3. **I reviewed 32 of the 83 hard rows** one by one (kept every pre-label). Marked `pre-labelled by Claude, checked by me`.
4. **The remaining 230 rows were not reviewed by me.** Marked `pre-labelled by Claude, not reviewed`. See the disclosure in How I Used AI.
5. Replies that neither asked for help nor gave advice (doubting the poster, side questions, jokes) were dropped, as decided in my taxonomy — 26 from the pool plus one of my cold rows.
6. 12 extra `question` posts were added afterwards to get `question` above 25% (criterion 4).

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reasoned_advice` | 105 | 37.2% |
| `bare_advice` | 100 | 35.5% |
| `question` | 77 | 27.3% |
| **Total** | 282 | 100% |

**Three hard cases**

**1.**
> *The post:* "My wife and I have been married for more than 40 years and we've always combined all income, all debt, and all property. For anything involving money, we act as one. That approach has been great for us both personally and financially, and that's what I would recommend to most couples."
>
> *Could have been:* `reasoned_advice` or `bare_advice`
>
> *I chose, because:* I labelled it `reasoned_advice` in my cold round. Reading it against my own rule afterwards, the only backing is the commenter's own marriage — personal experience, which my rule says is not backing — so the rule says `bare_advice`. I left the cold label as I made it; it's evidence that personal experience *feels* like a reason in the moment even after I'd written that it isn't one.

**2.**
> *The post:* "The data show that immediate deployment into the market always wins in the long term. Don't try to time the market."
>
> *Could have been:* `reasoned_advice` or `bare_advice`
>
> *I chose, because:* `reasoned_advice` — "the data show" is a factual claim about how investing works. The case for `bare_advice` is that no data is actually given and "don't time the market" is a rule of thumb, which my rule excludes. I kept reasoned, but this is the weakest kind of "fact" my rule lets in.

**3.**
> *The post:* "Did your lease start on the 13th? Perhaps the term end is 12 months exactly from term start"
>
> *Could have been:* `question` or dropped (not advice, not a help request)
>
> *I chose, because:* I labelled it `question` in my cold round because it is literally a question. But my `question` label is for someone asking about *their own* money, and I'd decided to drop clarifying replies to the poster — so by my own taxonomy it shouldn't be in the data at all. Kept as made, since it's a cold row.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128 tokens, seed 42.

**Device:** `mps` (Apple Silicon GPU), torch 2.14.1. Training took about 38 seconds.

**Anything I changed from the defaults, and why:** Nothing. Only `LABELS` was set to my three labels.

**Split sizes:** 196 train / 43 val / 43 test.

| Label | Train | Val | Test |
|---|---|---|---|
| `question` | 53 | 12 | 12 |
| `reasoned_advice` | 73 | 16 | 16 |
| `bare_advice` | 70 | 15 | 15 |

Every label has at least 12 in the test split, above the ~8 where per-label scores get jumpy — but 12–16 is still small, so expect per-label F1 to move between seeds next unit.


---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* A stress test of my labels — eight posts written to sit on my label boundaries, for me to sort with my own rule.
- *What came back:* Eight short comments. I sorted seven cleanly. I couldn't decide "Never co-sign a loan. Ever." because my definition said "a rule" counted as backing, and that sounds like a money rule.
- *What I changed:* I tightened `reasoned_advice` to say a *real* rule (a law, tax rule, or policy). Rules of thumb like "never co-sign" are popular wisdom and count as `bare_advice`.

**Moment 2**

- *What I asked for:* Help collecting posts, then pre-labelling everything except my 20 cold rows using my definitions and decision rule.
- *What came back:* A collection script pulling 296 posts/comments from a public Reddit archive, and pre-labels for all of them with 83 flagged as hard boundary cases, each with a one-line reason. The pre-labels also showed `question` at 24% — just under my criterion 4.
- *What I changed:* I had 12 more question posts collected to get `question` to 27%. I reviewed 32 of the hard rows one at a time and kept all 32 pre-labels.

**Pre-labelling disclosure:** 262 of 282 rows were pre-labelled by Claude. I reviewed 32 of them (all from the flagged-hard set) and agreed with every one. **The other 230 were not reviewed by me** — I stopped the review partway through and accepted the remaining pre-labels as they were. Every row's `note` column says which group it is in (`cold`, `checked by me`, or `not reviewed`). Only the 20 cold rows are labels I made independently. If the model struggles on the `reasoned_advice`/`bare_advice` boundary next unit, the unreviewed rows are one of the first places to look.

**Unit 6 AI use:** Claude ran the baseline, the three-seed runs before and after, and the confusion matrices, and drafted the unit 6 write-up from those results. It spotted the training-loss evidence for under-training that I chose as my improvement. It also labelled sp_013–sp_030 of the staff set at my request (I did sp_001–sp_012). My baseline prediction was written and committed before the baseline ran.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

Both scored on the same 43 held-out posts (`test_split.csv`, seed 42). Baseline: `facebook/bart-large-mnli` zero-shot, given only the three definitions in `label_definitions.txt`. Trained: the unit 5 model (3 epochs).

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.558 | 0.698 | +0.140 |
| Macro F1 | 0.489 | 0.705 | +0.216 |
| F1 — `question` | 0.733 | 0.762 | +0.029 |
| F1 — `reasoned_advice` | 0.615 | 0.727 | +0.112 |
| F1 — `bare_advice` | 0.118 | 0.625 | +0.507 |

**What I predicted before I looked:** I predict the baseline wins or ties: a big general model might understand "backed by a fact" better than a small model trained on ~200 rows.

**What the gap actually means:** My prediction was wrong. The trained model beat the baseline by 14 points of accuracy, and almost all of the gap is one label: `bare_advice` (0.118 → 0.625). The baseline handles `question` nearly as well as the trained model — asking for help looks like asking for help to any language model. But it almost never says `bare_advice`, because "without real backing" only means something under my rules (personal experience and rules of thumb don't count), and a one-line definition can't carry those rules. Fine-tuning is what taught the model where *my* line is.

---

## Run Log — Before

Device: `mps` (Apple Silicon), torch 2.14.1. Defaults: 3 epochs, lr 2e-5, batch 16, max length 128.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 0.698 | 0.791 | 0.651 | MISSED |
| 2. Weaker of `reasoned_advice` / `bare_advice` F1 | ≥ 0.60 | 0.625 | 0.667 | 0.000 | MISSED |
| 3. `question` F1 | ≥ 0.85 | 0.762 | 0.957 | 0.923 | MISSED |
| 4. Smallest label share of labels.csv | ≥ 25% | 27.3% | 27.3% | 27.3% | MET |
| 5. Most-sure third minus least-sure third accuracy | ≥ 0.20 | 0.286 | 0.000 | 0.500 | MISSED |

Accuracy spread across seeds: **0.140** — more than 10 points, so a single run of this model isn't a stable measurement.

### Confusion matrix

Seed 42:

| true \ predicted | question | reasoned_advice | bare_advice |
|---|---|---|---|
| **question** | 8 | 0 | 4 |
| **reasoned_advice** | 1 | 12 | 3 |
| **bare_advice** | 0 | 5 | 10 |

Seed 2024, for comparison (the collapse):

| true \ predicted | question | reasoned_advice | bare_advice |
|---|---|---|---|
| **question** | 12 | 0 | 0 |
| **reasoned_advice** | 0 | 16 | 0 |
| **bare_advice** | 2 | 13 | 0 |

**My biggest off-diagonal number, and what it means:** In seed 42, 5 real `bare_advice` posts were called `reasoned_advice` and only 3 went the other way; across all three seeds that direction totals 25 (5 + 7 + 13) against 4 the other way. The model leans hard toward calling unbacked advice "reasoned" — and in seed 2024 it never predicted `bare_advice` at all.

---

## Verdicts and Diagnoses

Verdicts are for the **before** run, against the unit 5 targets. A target has to hold on all three seeds.

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.70 | MISSED | Seeds 42 (0.698) and 2024 (0.651) are under 0.70. |
| 2 | Weaker of reasoned/bare F1 | ≥ 0.60 | MISSED | `bare_advice` was 0.000 on seed 2024. |
| 3 | `question` F1 | ≥ 0.85 | MISSED | Seed 42 was 0.762. |
| 4 | Smallest label ≥ 25% | ≥ 25% | MET | `question` is 77 / 282 = 27.3%. It's a property of the file, so the same on every seed. |
| 5 | Confidence gap | ≥ 0.20 | MISSED | Seed 7 had no gap at all (0.786 vs 0.786). |

**Diagnoses**

- **Criteria 1, 2 and 5: the model was under-trained.** Training loss ended at 0.93–0.99 on all three seeds. A model that guesses evenly among three labels scores about 1.10, so it had barely moved off guessing. That explains why the seed-2024 model never predicted `bare_advice` (it fell back to the two easier labels), why accuracy swung 14 points between seeds, and why its confidence meant nothing on seed 7. The evidence that this was training and not data: with no change to the data, 8 epochs fixed all three (see The Improvement).
- **Criterion 2: also a genuinely hard pair, possibly labelled inconsistently.** Nearly every error is between `reasoned_advice` and `bare_advice`, which is the boundary I knew was hardest in unit 5. I have two pieces of evidence that the labels themselves are part of the problem. In my 20 cold labels, I broke my own personal-experience rule at least once (hard case 1, labelled `reasoned_advice`), and the boundary rule changed during unit 5 after the stress test. And 230 of the 282 rows are pre-labels I never reviewed. The agreement check below would show whether I apply a written rule consistently, but staff labels aren't available yet, so this part of the diagnosis is still open.
- **Criterion 3: too few test examples for `question`.** Seed 42 had 12 `question` posts in test, and 4 of them were called `bare_advice`. At n = 12, each post is about 8 points of F1, so one draw can sink a strict 0.85 target that the other two seeds cleared (0.957, 0.923).

---

## Agreement Report

**Status: incomplete.** `agreement.py` needs `data/staff_labels.csv`, which comes from the TF and hasn't been received. `my_staff_labels.csv` is committed so the comparison can run as soon as it arrives.

**Who labelled `my_staff_labels.csv`:** I labelled sp_001–sp_012 myself under the staff taxonomy. **sp_013–sp_030 were labelled by Claude**, after I asked it to finish the set, so only the first 12 are evidence about how I apply a written rule.

**Agreement rate:** ___ / 30 = ___% (pending staff labels)

**Disagreements:** pending staff labels.

**What the pattern in my disagreements tells me:** pending.

---

## The Improvement

**What I changed:** `EPOCHS` from 3 to 8. Nothing else: same labels, same seeds, same learning rate, batch size and max length.

**Which diagnosis pointed at it:** Under-training. Training loss was stuck at 0.93–0.99 (chance is about 1.10), and on seed 2024 the model never predicted `bare_advice`. After the change, training loss is about 0.64.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 0.791 | 0.767 | 0.767 | MET |
| 2. Weaker of `reasoned_advice` / `bare_advice` F1 | ≥ 0.60 | 0.750 | 0.690 | 0.667 | MET |
| 3. `question` F1 | ≥ 0.85 | 0.815 | 0.917 | 0.917 | MISSED |
| 4. Smallest label share of labels.csv | ≥ 25% | 27.3% | 27.3% | 27.3% | MET |
| 5. Most-sure third minus least-sure third accuracy | ≥ 0.20 | 0.286 | 0.429 | 0.358 | MET |

Accuracy spread across seeds: **0.023** (was 0.140).

Seed 42 confusion matrix after:

| true \ predicted | question | reasoned_advice | bare_advice |
|---|---|---|---|
| **question** | 11 | 0 | 1 |
| **reasoned_advice** | 1 | 11 | 4 |
| **bare_advice** | 3 | 0 | 12 |

**Did it help, and how do I know:** Yes. Two criteria moved from MISSED to MET (1 and 5), criterion 2 recovered from 0.000 on its worst seed to 0.667, and the accuracy spread dropped from 14 points to 2.3, so these numbers are now stable enough to trust. The direction of the main error flipped, though. Before, the model called `bare_advice` "reasoned" 25 times and the reverse 4 times. After, it calls `reasoned_advice` "bare" 14 times (4 + 6 + 4) and the reverse 6 times (0 + 2 + 4). The pair is still hard; the model is now just erring the other way.

---

## What's Still Broken

- **Criterion 3 (`question` F1 ≥ 0.85)** still misses on seed 42 (0.815). That seed has 12 test questions, and 1 was called `bare_advice`; the other two seeds clear the target easily. What I'd do: add more question posts so the test split holds more of them, which makes the per-label score less jumpy. Why I stopped: the brief allows one improvement per unit, and I used it on under-training.
- **The `reasoned_advice` / `bare_advice` boundary** is still where most errors are. What I'd do: review the 230 unreviewed pre-labels against the written rule, starting with comments that cite personal experience or a rule of thumb. Why I stopped: same one-change rule, and the agreement report I'd use to justify it isn't back yet.

**The gap between what I meant my labels to capture and what the model learned:** I meant `reasoned_advice` vs `bare_advice` to be about whether a reason actually *backs* the advice, but after training the model still trades errors across that pair in both directions, so it hasn't fully learned the line. My guess is it partly learned surface cues like length, numbers and "because" rather than whether the backing is a real fact, which would explain why short reasoned comments now get called bare.


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
