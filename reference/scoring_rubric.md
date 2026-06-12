# SCORING RUBRIC (QUICK REFERENCE)

All numbers in one place. Full conditions live in `rules.md` — this card is for fast lookup during a run. If this card and rules.md ever disagree, rules.md wins.

## Context Confidence Interval (CCI) — start 1.00, subtract

| Finding | Deduction |
|---|---|
| Missing relationship linkage (only for an entity isolated from all others; same-commit co-occurrence counts as linked; never fires on single-entity payloads or same-commit file sets) | −0.25 |
| `[Draft]` without owner signature (`sig:` absent on file) | −0.30 |
| Missing author identity entirely (no commit author AND no `sig:` anywhere; mutually exclusive with the Draft deduction — never both for one missing signature) | −0.30 |
| Orphan asset (path not in declared scope) | −0.20 |
| Missing timestamp | −0.15 |

Hard floor: **CCI < 0.70 → EXECUTION FREEZE**

## Context-Drift Score (CDS) — start 0.00, add, cap 1.00

| Finding | Addition |
|---|---|
| No declared objective referenced | +0.15 |
| No open work-item mapping | +0.15 |
| Each out-of-scope file touched | +0.10 |
| Each undeclared new dependency | +0.10 |
| Modifies `[Archived]` entity | +0.20 |
| Unauthorized author | +0.15 |

## Directive routing

Evaluate top-down; first matching row wins (mirrors rules.md Rule 4 / system_prompt Step 5):

| Order | Condition | Directive | Trustworthiness |
|---|---|---|---|
| 1 | Rule 1 violation or escalation trigger | INTEGRATION HALT | Zero |
| 2 | CDS > 0.55 | INTEGRATION HALT | Zero |
| 3 | CCI < 0.70 | EXECUTION FREEZE | Zero |
| 4 | 0.35 < CDS ≤ 0.55 | MANDATORY LIFECYCLE PIVOT | Moderate |
| 5 | (all above clear) | AUTHORIZE EXECUTION | CCI ≥ 0.85 → High, else Moderate |

Precedence when multiple fire: HALT > FREEZE > PIVOT > AUTHORIZE.
