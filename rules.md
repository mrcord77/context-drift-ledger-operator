# RECONCILIATION RULES MATRIX

Deterministic decision logic for the Context-Drift Ledger Operator (CDLO). Every rule here is computable from the pasted input alone. Missing information is a deduction, never an inference. "Good judgment" and case-by-case flexibility are unauthorized.

---

## RULE 1: STABLE IDENTIFIER & CANONICAL PATH CONSTRAINT

**Condition:** Every entity referenced in the payload (asset, document, API, dependency) must have:
1. A unique alphanumeric stable ID prefixed by domain type (`API-`, `DOC-`, `AST-`, `DEP-`)
2. Exactly one canonical path in the intent file's declared scope

**Evaluation:** Scan every reference tag in the payload against the intent file's asset list.

**Integrity violation (HALT):** Any reference with a missing/invalid prefix, no ID at all, or an ID mapped to two or more competing paths → Canonical Knowledge Status = UNVERIFIED, Context Confidence Interval = 0.00, directive = `INTEGRATION HALT`. No partial credit. No further rules are evaluated. (Escalation triggers — Rule 5 — are audited *before* this check in the pipeline, so multi-source decoy patterns are detected even when their payloads would also fail here.)

**Undeclared entity (NOT a Rule 1 violation):** A well-formed, correctly prefixed ID that is simply absent from the intent file's asset list is a *drift finding*, not an integrity failure. It does not halt. It is scored downstream: −0.20 CCI (orphan asset, Rule 2) and +0.10 CDS (undeclared dependency, Rule 3). Rationale: the operator's purpose is to *measure* work drifting outside declared intent; if every undeclared ID halted, the drift bands could never be reached and the PIVOT directive would be dead code.

---

## RULE 2: CONTEXT CONFIDENCE INTERVAL (CCI)

**A note on the numbers, here and in Rule 3:** the weights are *policy defaults, not measured quantities*. There is no empirical reason −0.30 for a missing signature couldn't be −0.25 or −0.35; what matters is that the values are fixed, written down, applied identically to every payload, and tunable in one place (`config.md`) rather than improvised per decision. Treat the scores as reproducible policy outputs — auditable and consistent — not as objective measurements of project health.

**A note on the name:** "Context Confidence Interval" is a constructed integrity score, not a confidence interval in the statistical sense. It carries no probability semantics — it is a deterministic weighted deduction from 1.00. The name is retained for consistency with the ledger format; read it as "Context Integrity Score."

**Start at 1.00. Apply every deduction that matches. Deductions stack.**

| Finding | Deduction |
|---|---|
| Entity missing relationship linkage. An entity counts as LINKED if it (a) appears in the same commit/change as another entity — co-occurrence in one commit IS implicit linkage — or (b) has an explicit relationship-graph entry connecting it to another payload or intent entity. This deduction fires ONLY for an entity with neither: present in the payload but isolated from every other entity. Consequences: never fires on single-entity payloads (nothing to link to); never fires on multiple files within one commit (the commit links them). | −0.25 |
| Document or asset in `[Draft]` status without an owner signature (`sig:` field absent on the file) | −0.30 |
| Asset path not listed in the intent file's declared scope (orphan) | −0.20 |
| Modification missing a timestamp | −0.15 |
| Modification missing an author identity entirely — fires ONLY when the change carries no commit author AND no `sig:` field anywhere. A present commit author satisfies this test. MUTUAL EXCLUSION: at most ONE signature deduction per entity per run — a `[Draft]` file with a missing `sig:` field takes the Draft deduction above, never both. The same missing signature is never double-counted. | −0.30 |

**Hard floor:** CCI < 0.70 → directive = `EXECUTION FREEZE` regardless of drift score. Rule 3 is still computed and reported, but cannot override the freeze.

Valid lifecycle labels are exactly `[Approved]`, `[Draft]`, `[Archived]`. Any other label string counts as missing status → treat as `[Draft]` without signature (−0.30).

---

## RULE 3: CONTEXT-DRIFT SCORE (CDS)

**Start at 0.00. Apply every addition that matches. Additions stack. Cap at 1.00.**

| Finding | Addition |
|---|---|
| Change does not reference any declared objective or milestone in the intent file | +0.15 |
| Commit/change message does not map to any open work item | +0.15 |
| Each file or asset touched that is outside the intent file's declared scope paths | +0.10 |
| Each new dependency introduced that is not in the intent file's asset list | +0.10 |
| Change modifies a `[Archived]` entity | +0.20 |
| Change author is not listed in the intent file's authorized actors | +0.15 |

**Thresholds:**

| CDS | Directive | Meaning |
|---|---|---|
| ≤ 0.35 | `AUTHORIZE EXECUTION` | Locked alignment |
| > 0.35 and ≤ 0.55 | `MANDATORY LIFECYCLE PIVOT` | Work is valid but operating in a silo, drifting from declared intent. Pivot forces re-anchor to the intent file before further execution. |
| > 0.55 | `INTEGRATION HALT` | Decoupled beyond recoverable boundary |

---

## RULE 4: DIRECTIVE PRECEDENCE

When rules conflict, precedence is fixed:

1. `INTEGRATION HALT` (from Rule 1 violation, CDS > 0.55, or any escalation trigger)
2. `EXECUTION FREEZE` (CCI < 0.70)
3. `MANDATORY LIFECYCLE PIVOT` (CDS in 0.35–0.55 band)
4. `AUTHORIZE EXECUTION` (only when nothing above fires)

The ledger always reports both CCI and CDS even when a higher-precedence directive short-circuits the outcome.

---

## RULE 5: ESCALATION TRIGGERS (override everything above)

### TRIGGER 1: COHERENT DECOY (consensus capture)
**Detect:** Two or more agent/script sources submit changes whose outputs agree exactly, but whose logs skip the Rule 1 stable-ID verification.
**Act:** Deny routing. Routing field = `ESC-COHERENT-DECOY-CAPTURE-ALARM`. Directive string, exactly: `CRITICAL ADVERSARIAL HALT: CONFLICTING MULTI-AGENT INJECTION DETECTED`. List the offending source identifiers in the execution log. Human review required.

### TRIGGER 2: VERIFIED CONTRADICTION
**Detect:** Two entities both carry valid signatures and identical active versions, but their declared relationships contradict (A says B is broken, B says A is functional).
**Act:** Never resolve or pick a side. Status code, exactly: `CONTEXTUAL AMBIGUITY EXECUTION REJECTION`. Write the conflicting graph intersection (both entity IDs and both contradictory claims) in the execution log. Lock all transitions pending an explicit administrative override line in the next payload.

---

## REQUIRED INPUTS

The operator evaluates exactly two pasted artifacts per run:
1. **Intent file** — one paragraph of declared build intent, plus: declared scope paths, asset/ID list, open objectives or work items, authorized actors.
2. **Change payload** — commit log, doc update, or dependency change, with whatever IDs, signatures, timestamps, and status labels it carries.

If the intent file is absent → `INTEGRATION HALT`, reason: no baseline to measure drift against. The operator never reconstructs intent from the change itself.

---

## GIT MODE (payloads generated by scripts/extract_payload.py)

A payload whose first line reads `# GIT-MODE PAYLOAD` comes from raw git history, where some CDLO metadata categories do not exist. In git mode only, these mappings apply:

- **Stable identity:** the file path is the stable identifier (git guarantees path uniqueness — competing paths are impossible). A file mapped to a declared asset ID in the intent file is *declared*; a file marked `(undeclared)` is an undeclared entity → drift scoring per Rule 1's undeclared-entity clause (−0.20 CCI orphan if also out-of-scope is not double-counted; apply orphan only when the path is outside declared scope, undeclared_dep when the ID/path is absent from the asset list — both can apply to the same file when both conditions hold).
- **Signature:** the git commit author. Present on every commit; the missing-signature deduction cannot fire in git mode.
- **Timestamp:** the git commit timestamp. Same — always present.
- **Lifecycle status labels:** git has none. Per `config.md` `git_mode_status_deductions: off`, status-label deductions are skipped. Hand-assembled payloads are unaffected by this section.

Everything else — CDS additions, thresholds, precedence, escalation triggers — applies unchanged.
