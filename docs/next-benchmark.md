# Next benchmark direction — captured 2026-07-06 (pre-sleep, warm-start for tomorrow)

## The pivot (in one line)
From *"audit τ-bench's grader/spec"* → toward a **new benchmark**: *does the agent calibrate its caution to the stakes when protecting the user conflicts with the task goal or the user's own request?*

## Thesis
An agent can **complete the task** yet **harm the user** through the side effects of its actions (the medical analogy already in the README). Outcome-graders (the whole τ-bench family) score task completion, **not** how the agent handled the *protect-vs-comply* tradeoff. That blind spot is the opportunity — and it's a hotter, more fundable, more discriminating target than "don't fire a forbidden action."

## The design — two axes (this is the core artifact)
- **Severity: hassle → hurt** (≈ reversibility). A flight *search* is hassle-level; a *charge* / *cancel* / *transfer* is hurt-level.
- **Ambiguity: clear → unclear** (how obvious the right call is).

Correct behavior **scales monotonically** with both:

| | low ambiguity | high ambiguity |
|---|---|---|
| **hassle** (reversible) | just do it | do it, maybe note it |
| **hurt** (irreversible) | **confirm first** | **escalate / protect** |

This is principled, not arbitrary — it's **FMEA** (severity = the classic risk axis), and it's *why* the airline `policy.md` line 7 already confirms DB updates (hurt) but not searches (hassle). Line 7 is one point on this surface.

## The three agent responses
**comply · confirm-or-protect · escalate.** Escalation is the *humility valve* (the agent shouldn't decide unilaterally in the hurt × ambiguous corner). Note: escalation flips from villain (task 6) to a **correctly-scored action**.

## What it measures (why it discriminates models)
**Calibration** — does the agent's caution *track* severity × ambiguity, or is it flat? Two directional errors, both invisible to an outcome-grader:
- **Sycophancy** — too little caution in the *hurt × ambiguous* corner (does the harmful thing the user asked).
- **Paternalism** — too much caution in the *hassle × clear* corner (refuses/escalates legitimate requests).

Weak models are **un-calibrated** (uniform behavior); strong models **track the surface**. The *calibration curve* is the metric, not raw accuracy.

Benchmark one-liner: *"Does the agent calibrate its caution — confirm, protect, or escalate — to the **severity** and **ambiguity** of a consequential action, instead of blindly completing the task?"*

## What's salvageable from THIS repo
**Keep (100% reusable):** the τ-bench fork, `poc/run_airline.py` (runner + multi-run + caching), `poc/lift_requirements.py` + `poc/measure_flips.py` (provenance-grounded lifting + paired re-scoring), the `preflight_requirements` / `ActionPrecondition` types, the honest results, the two-pattern framing. The transfer flips (tasks 6/47) = the **hurt × ambiguous** cell — a real seed.

**Evolve / build (the real new work):**
1. **New tasks** with protect-vs-comply tension, placed deliberately across the grid (τ-bench tasks are *cooperative* and mostly lack this tension — this is the SME/data lift).
2. **A judgment grader** — not "did a forbidden action fire" but "was comply/confirm/escalate the **right call** for this cell?" Needs per-task ground truth (rubric/SME).
3. Framing climbs from rung-1 (prohibition) to the top of the ladder (protective judgment under severity × ambiguity).

## Cheapest first experiment (do this before committing)
Author **3 seed tasks** — one *clear-comply*, one *clear-protect*, one *genuinely-ambiguous* (with escalation available). Hand-label the correct call for each. Run **2–3 models** (Haiku / Sonnet / a frontier competitor). **Look for divergence on the ambiguous one.** If they spread (barrel-through vs ask vs escalate), you have a discriminating, novel benchmark + a demo — for a few dollars.

## Open questions for tomorrow
- Which **domain** best carries protect-vs-comply tension? Airline may be thin; **medical / financial** are richer in "the user asks for something that could hurt them."
- **Grader**: rubric vs SME vs LLM-judge for "was the call correct?" (careful — this is the hard part, and where HealthBench-style grader critiques apply).
- Does severity × ambiguity **actually** produce a model gradient? ← the one hypothesis the 3-task experiment settles.

## Repo status (2026-07-06)
- Renamed `tau-bench-audit`; `main` has the pipeline + two-pattern framing + CI.
- **PR #6** open (audit repositioning) — decide merge tomorrow; it's still accurate for *what exists*, and doesn't conflict with this direction.
- Multi-seed run (K=5) blocked only on **Anthropic API credits** (~$50 top-up; caching already added).
