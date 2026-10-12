# plan-council

A Claude Code skill that turns a brief into a plan you can trust before you build it.

LLM-written plans fail in two ways. They pick the wrong lever because something feels important, and they carry factual errors (a wrong count, a misread file, a contradicted earlier decision) that nobody catches until build time. This skill attacks both.

```
brief → plan v1 → council (4 lenses in parallel) → plan v2 → needle (adversarial fact-check) → plan v3 + needle report
```

1. **Orient.** It reads the code and any earlier plans, runs every count it intends to cite, and names the metric the plan is meant to move.
2. **Plan v1.** Every claim about the current state carries a `file:line` or a reproducible command.
3. **Council.** Four reviewer subagents run in parallel: **ROI** (would this have changed any real observed case?), **Risk** (blast radius, regressions, rollback), **Engineering** (is it buildable as written, and does each verification step prove anything?) and **Product** (is it what was asked for?). One optional domain lens joins when the plan hinges on a specialty. Each returns GO / GO-WITH-CHANGES / DEFER / DOWN.
4. **Reconcile.** Every required change is APPLIED or REJECTED with a reason, with no silent drops. If any lens says DEFER or DOWN, the plan is restaged so that the cheapest evidence-gathering step comes first and the build is gated on it.
5. **Needle.** A fresh agent assumes the plan is wrong. It re-runs every count, opens every `file:line` and checks every ruling against earlier plans. Findings are graded BLOCKER / FIX / NIT, and the main agent re-verifies each fix before applying it.
6. **Needle report.** The plan ends by saying honestly whether it would *move* the metric or only *prepare* for moving it.

It runs autonomously: open questions become stated assumptions instead of interruptions. It uses at most six subagents and writes one plan file, overwritten in place from v1 to v3.

## Install

```bash
git clone https://github.com/emadkhan8371/plan-council
cp -r plan-council/plan-council ~/.claude/skills/
```

Then in Claude Code: `/plan-council <your brief>` or "plan this: …".

## Files

- `plan-council/SKILL.md`: the process and its hard rules
- `plan-council/council.md`: the reviewer prompts (shared preamble, lens focus blocks, output schema)
- `plan-council/needle.md`: the adversarial fact-check prompt

MIT licensed.
