---
name: plan-council
description: A planning process, end to end and fully autonomous - write the plan, put it through a 4-lens LLM council (ROI / Risk / Engineering / Product, plus one domain lens when the plan hinges on a specialty), regenerate the plan from the verdicts, then run an adversarial NEEDLE pass that re-verifies every number, file:line and ruling against the real code, fold the fixes in, and close with the moved/prepared needle report. Brief in, one councilled plan file out. TRIGGER when the user says "plan this", "make a plan for", "council this", "plan + council", "run it through the council", or /plan-council. DO NOT TRIGGER for a quick "how would you approach X" answer, for reviewing an existing plan without regenerating it (run Stage 2 alone if asked), or for building - this skill ends when the plan file is written.
---

# Plan -> Council -> Regenerate -> Needle

Why this exists: plans get justified by salience rather than measured impact, and
plans carry factual errors that survive until build time. The council catches the
framing errors (wrong lever, wrong stage order, scope that should be deferred).
The needle catches the factual ones - it has caught a 79-vs-76 corpus count, an
inverted ruling, and a grade pin that contradicted an earlier plan. The closing
needle report keeps the plan accountable to the metric it claims to serve.

## Hard rules

- **No pauses.** The user is not watching and answers nothing mid-run. Every open
  question becomes a stated assumption in the plan's "Assumptions & open
  questions" section and is surfaced in the final message. Do not use
  AskUserQuestion inside this skill.
- **Evidence over prose.** Every claim about current state carries a file:line
  or a reproducible count. If you did not open the file or run the count in
  this session, you may not cite it. Estimates are labelled estimates.
- **Prior rulings bind.** Read the latest councilled plans in the same project
  folder first. A new plan that contradicts an earlier ruling must say so
  explicitly and justify it; silent contradiction is a needle BLOCKER.
- **Plans end at the plan.** Do not build. Do not prototype "just to check"
  beyond the reads and counts needed to cite evidence.
- **Agent budget:** 4 fixed lenses + at most 1 domain lens + 1 needle agent =
  6 subagents maximum. Council reviewers launch in parallel in ONE message.
- **One file, overwritten in place** through v1 -> v2 -> v3. The header status
  line tracks the stage. No scratchpad copy of the final.

## Where the file goes

`plan - <short name>.md` in the project folder the brief belongs to, beside any
existing `plan - *.md` files (or in `docs/` if the project keeps plans there). If a plan with that name already exists, leave it
untouched and name the new one `plan - <short name> v<N+1>.md`, where N is the
highest existing version. If the project folder is genuinely unclear, use the
project root and say so in the final message.

Date format in headers: `2 Sep` (day, short month). Get it from `date +"%-d %b"`.

## Stage 0 - Orient (nothing shown to the user)

1. Read the brief. Read the project's memory files and its latest councilled
   plans. Note every prior ruling that touches this plan.
2. Open the code paths the plan will touch. Run every count you intend to cite
   (grep, sqlite, ls, wc) and keep the commands - the needle will re-run them.
3. Name the metric the plan serves: the number the work is actually judged
   against (for client work, the metric the engagement is paid against). If the brief names none, pick the most plausible
   and mark it as an assumption.
4. Decide on a domain lens. Rule: add one only when success hinges on a
   specialty the four fixed lenses cannot judge - three.js / rendering, prompt
   and LLM behaviour, sales or landing copy, video/audio processing, UX motion,
   infra and ops. Name it as a role ("three.js lead", "prompt engineer",
   "content strategist"). Zero domain lenses is the default.

## Stage 1 - Plan v1

Write the file with this skeleton, in this order. Match the project's existing plans in
register: dense, rulings stated as rulings, counts inline, no motivational prose.

```
# Plan - <NAME> v1 (<date>, pre-council)

<3-6 lines: the problem, the metric, the one-sentence mechanism by which this
plan moves it, the honest number it targets (e.g. "targets 6-8 of 11 audited
downs"), and what would falsify it.>

## Current state (evidence)
<what exists today, every claim with file:line or a count + the command>

## Proposed change
<the change, then stages. Each stage: what / files touched / verification step
(the exact test or command that proves it) / rollback>

## What this does NOT do (deferred ledger)
<so nothing silently drops>

## Risks
<what breaks, blast radius, false positives, variance, how each is guarded>

## Assumptions & open questions
<every decision made without the user, stated so they can overturn it>
```

## Stage 2 - Council

Launch the reviewers as parallel `Agent` calls (subagent_type
`general-purpose`) in a single message: ROI, Risk, Engineering, Product, plus
the domain lens if Stage 0 chose one. Build each prompt from `council.md` in
this skill folder: the shared preamble, that lens's focus block, and the output
schema, filled with the plan path, the brief verbatim, the project folder, the
metric, and the prior binding rulings you collected.

Reviewers must read the code themselves and are told the plan may be wrong.
When they return, keep every "required change" verbatim in your working notes.
Do not summarise them away before reconciling.

## Stage 3 - Reconcile and regenerate v2

Reconciliation rules, applied in order:

- **Unanimous DOWN.** Do not write a build plan. Rewrite the file as a
  "councilled DOWN" record: the brief, a summary of v1, each lens's verdict and
  reasoning, and what would have to be true to revive it. Skip Stage 4's
  verification (nothing to build) but still write the Stage 5 needle report
  (what we learned counts as "prepared"). Deliver.
- **Any DOWN or DEFER alongside GO / GO-WITH-CHANGES.** Restage the plan. Stage
  0 becomes the cheapest thing that produces the evidence the dissenting lens
  needed (telemetry, a census, a shadow run, a dark-launch counter); the build
  becomes Stage 1 and is gated on Stage 0's result with a stated threshold.
  Name the lens that forced the staging.
- **All GO / GO-WITH-CHANGES.** Apply the required changes.

Every required change from every reviewer is either APPLIED (say where in the
plan) or REJECTED (one line saying why). No silent drops. A finding raised
independently by two or more lenses is a "convergent core finding" and goes at
the top of the council section.

Rewrite the file in place. Header becomes `v2 (<date>, post-council)`. Add:

```
## Council (<date>)
<one line per lens: LENS - VERDICT - the one finding that drove it>
Convergent core finding: <if any>
Required changes: <table or list, each APPLIED (where) or REJECTED (why)>
```

## Stage 4 - Needle pass

Spawn ONE fresh `general-purpose` agent with the prompt in `needle.md`, giving
it only the v2 plan path, the project folder, the brief, and the list of prior
councilled plans. It is adversarial: it assumes the plan contains errors and
looks for them. It re-runs every count, opens every file:line, checks every
ruling against prior plans, checks internal consistency, and hunts for v1
residue that the council changes should have removed. It returns a findings
table graded BLOCKER / FIX / NIT, each with evidence and a proposed correction.

Then you:

1. Apply every BLOCKER and FIX. Apply NITs when cheap.
2. Re-verify each correction yourself (open the file, re-run the count). The
   needle can be wrong too. If you overrule a finding, record it as OVERRULED
   with the evidence.
3. Header becomes `v3 (<date>, post-council, needle-passed)`. Add a
   `## Needle review record (<date>): <SHIP-CLEAN | SHIP-WITH-FIXES | BLOCKED>`
   section listing what reproduced clean (list
   the anchors), fixes baked in (each one-liner), overruled findings, anything
   added to the deferred ledger as a result.

If the needle returns a BLOCKER you cannot resolve without the user (a contradiction
with a prior ruling that only the user can settle), the verdict is BLOCKED: leave the
plan at v3 with the block stated at the top and make it the first line of the
final message.

## Stage 5 - Needle report (closing section)

For a plan this is prospective. One tight paragraph headed `## Needle report`:

- the metric it serves and the mechanism;
- PREPARES or WOULD MOVE. A plan almost always PREPARES. "Moved" may only be
  claimed on evidence about the whole output a human judges (e.g. the
  client's marks on complete deliverables, or a proxy validated against them).
  Component-level evidence - a detector passing N/N on a corpus, fixture evals,
  false positives removed - is always "prepared", because output quality is a
  conjunction and fixing one failure class reveals the next;
- the honest number, quoted as one draw when it comes from a single eval roll
  (LLM systems have real run-to-run variance);
- what would falsify it, and the first measurement that would show it.

## Stage 6 - Deliver

The final message must stand alone for a reader who saw none of the run:

1. Path of the plan file, and BLOCKED first if the needle blocked.
2. Council verdict table (lens / verdict / driving finding).
3. Convergent core finding, if any.
4. What changed v1 -> v3, three to six bullets.
5. What the needle caught, and anything you overruled.
6. The needle report paragraph.
7. Assumptions the user should overturn if wrong, as a short list.

If the project has a memory file, add a one-line status pointer to it (date,
plan name, verdict). Do not save the plan's content to memory; the file is the
record.
