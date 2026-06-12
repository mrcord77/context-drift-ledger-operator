# CDLO — the Context-Drift Ledger Operator

**Technical report (citable):** [doi.org/10.5281/zenodo.20673314](https://doi.org/10.5281/zenodo.20673314)

**When you hand work to AI agents, who checks that the work stayed on mission?**

Not "is the code broken" — tests catch that. Not "is it well-written" — review catches that. This: you asked for one thing, the agent built something adjacent, and every checker said yes because nothing was *wrong* — it just wasn't what the project declared it was doing. Multiply by a fleet of agents and a repo quietly becomes a different project.

CDLO is a gate for that failure. You declare your project's intent in one small file — what you're building, which files are in play, who's allowed to act. Every incoming change gets measured against it and receives exactly one of four rulings, from "proceed" to "stop and re-anchor to the mission." The rules are fixed, published, and arithmetic — no AI judges anything; a plain script can recheck every score. It was tested by trying to break it, four real defects were found and fixed, and the whole saga is documented here because a gate that audits other people's work should publish its own audit.

This is the original instantiation of a pattern that's now been aimed twice — the second, [PR Warden](https://github.com/mrcord77/pr-warden), gates pull requests against tickets and includes a browser demo you can click right now. The worksheet for aiming it at *your* workflow is **`AIMING.md`**; the full story is **`PAPER.md`**.

---

# Under the hood

*Everything below is the engineering layer. You don't need it to use the gate — it's here because trustworthy means checkable.*

*Packaging note: this operator folder follows Van Clief's Interpretable Context Methodology (ICM, arXiv:2603.16021) — ICM is the folder-as-architecture convention; the governance pattern inside it is this project's contribution.*

## The pattern (and how to aim it at YOUR workflow)

**The full story in one read:** `PAPER.md` — the pattern, both instantiations, the five-defect testing record, and the measured case for boolean-test rules. This repo is the pattern's first aiming; the second (PR Warden — pull requests vs. tickets, with a browser demo and a no-API-key CI Action) is at github.com/mrcord77/pr-warden.

CDLO is the first instantiation of a reusable pattern: **declared baseline + incoming change + boolean-test scoring + four verdicts + a model-free referee.** The second instantiation — PR Warden, gating pull requests against tickets — was translated from this gate in about a day, with roughly 70% transferring untouched, and converged with zero live-testing defects against this gate's four (the measured payoff of writing conditions as boolean tests from day one). The 30% that doesn't transfer is *aiming*, documented as a ten-step worksheet in **`AIMING.md`**, including a filled worksheet for a third, unbuilt domain. If your workflow has a baseline document and a stream of changes that should serve it, this pattern can gate it.

## Tested before you got here

This folder was adversarially tested in a live Claude Code session before submission — and the test caught a real defect: the original Rule 1 conflated "undeclared ID" with "invalid ID," which would have made the drift-detection band unreachable. The rule was split, the suite re-run, and the live retest matched the model-free verifier to the digit. The full transcript and the defect story are in `reference/verified_run.md`. An operator that governs other people's drift should survive its own gate; this one did.

## Verify the math without the AI

`python scripts/verify_rules.py` runs the scoring tables from `rules.md` as plain code against all three examples plus six boundary cases (exact thresholds, precedence conflicts). Per ICM, mechanical work doesn't need a model: the deduction/addition math is deterministic, so it's verifiable as a script. If the script and the operator ever disagree on a score, the script is right and the prompt needs tightening.

## Decisions it makes vs. escalates

**Decides:** authorize, freeze, pivot, halt — on every well-formed input, without asking.
**Escalates (locks the gate for a human):** multiple agents agreeing while skipping ID verification (coherent decoy), and two verified sources contradicting each other. The operator never resolves a verified contradiction by picking a side.
