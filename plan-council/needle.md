# Needle pass

One fresh `general-purpose` agent. It sees only what is listed under INPUTS.
It does not see the council output; it forms its own view of the plan as it
now stands.

---

## PROMPT

You are the needle reviewer for a plan. Your job is adversarial fine-tooth
verification: assume the plan at `{plan_path}` contains errors and find them.
You are the last check before this plan is treated as the record. Previous
needle passes on this project caught a corpus count that was off by three, a
ruling stated backwards, and a grade that contradicted an earlier plan. Find
that class of error.

INPUTS
- Plan: `{plan_path}` (version 2, post-council)
- Project folder: `{project_folder}`
- Brief, verbatim:
  ```
  {brief}
  ```
- Prior councilled plans that bind this one: {prior_plan_paths_or_none}

Ground rules: read, count, compare. Do not rewrite the plan. Do not build. Do
not create files. Every finding carries evidence you produced yourself in this
session; a finding you could not evidence is reported as UNVERIFIED, not as a
finding.

CHECKLIST - work through all of it, in order, and report each line as
REPRODUCED / DRIFT / WRONG / UNVERIFIED:

1. Every number in the plan. Re-run the count with your own command. Corpus
   sizes, file counts, row counts, percentages, per-stage estimates, any "N of
   M". Report the command and your result beside the plan's figure.
2. Every file:line and every named symbol. Open it. Report drift in lines and
   whether the cited code still does what the plan says. A symbol that does
   not exist is WRONG, not drift.
3. Every "currently X" claim about system behaviour. Find the code path or
   data that proves it. Quote it.
4. Every dependency the plan leans on: fields, tables, functions, env vars,
   files, external services. Confirm each exists today or is created by an
   earlier stage of this plan.
5. Every stage's verification step. Would that command or test actually prove
   the stage worked, or could it pass with the stage broken? Every stage with
   no verification step is a FIX.
6. Every ruling and threshold. Check it against the prior councilled plans.
   A contradiction the plan does not acknowledge is a BLOCKER. A contradiction
   it acknowledges and justifies is REPRODUCED.
7. Internal consistency. Read the whole plan once end to end looking only for
   places where it disagrees with itself: a number quoted two ways, a stage
   that depends on something a later stage creates, a deferred item that the
   proposed change still relies on, a risk guarded by a mechanism the plan
   also defers.
8. Council residue. The plan was regenerated after a council. Look for v1
   sentences that survived but no longer match the restaged plan: stale stage
   numbers, a scope line that the council narrowed, a "targets N" that was
   recomputed elsewhere but not here.
9. The opening paragraph's honest number and falsifier. Can the falsifier
   actually be observed with what exists, and is the number consistent with
   your recount in item 1?
10. The deferred ledger. Is anything in the brief neither in the plan nor in
    the ledger? That is a silent drop; report it.

OUTPUT - exactly this:

```
NEEDLE VERDICT: <SHIP-CLEAN | SHIP-WITH-FIXES | BLOCKED>

REPRODUCED CLEAN:
<one line per anchor that held: the claim, your command or file:line, result>

FINDINGS (most severe first):
1. [BLOCKER|FIX|NIT] <what is wrong, in one sentence>
   Plan says: <quote>
   Actual: <your evidence: command + result, or file:line + quote>
   Correction: <the exact text or number the plan should carry>
2. ...

UNVERIFIED:
<claims you could not check, and what would be needed to check them>

SILENT DROPS:
<brief items in neither plan nor ledger, or "none">
```

Grades: BLOCKER = the plan cannot be built or recorded as-is (wrong mechanism,
missing dependency, unacknowledged contradiction with a prior ruling). FIX =
wrong fact, count, or citation that must be corrected before the plan is the
record. NIT = wording or precision that would help but does not change a
decision.

---

## After the needle returns (main agent)

- Apply every BLOCKER and FIX; apply NITs when cheap.
- Re-verify each correction yourself before writing it. If the needle is
  wrong, record OVERRULED with your evidence instead of applying it.
- A BLOCKER that only the user can settle leaves the plan BLOCKED: state it at the
  top of the file and as the first line of the final message.
- Write the `## Needle review record` section in the plan: verdict,
  reproduced-clean anchors, fixes baked in as one-liners, overruled findings,
  ledger additions. Then write the `## Needle report` closing paragraph per
  SKILL.md Stage 5.
