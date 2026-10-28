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

**What I predicted before I looked:** I predict the baseline wins or ties: a big general model might understand "backed by a fact" better than a small model trained on ~200 rows.
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
