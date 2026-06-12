# EXAMPLE DECISIONS

Three runs showing the rules producing each major outcome. Every score below is reproducible from the tables in `rules.md` — no number appears without a matching table line.

---

## CASE 1: CLEAN AUTHORIZE

### Intent file (excerpt)
- Intent: "Ship v2 export endpoint for the reporting module."
- Scope paths: `products/signal_v2/core/`
- Assets: `API-4410`, `AST-9912`
- Open objectives: `OBJ-12: export endpoint`
- Authorized actors: `@engineering_lead`, `@agent_4`

### Change payload
```
commit c71a3 | author @agent_4 | 2026-06-11T22:15:00Z
msg: "OBJ-12: add CSV export route to API-4410"
files: products/signal_v2/core/api.md [Approved] sig:@engineering_lead
```

### Evaluation
- Rule 1: `API-4410` indexed, one canonical path. PASS.
- Escalations: none.
- CCI: 1.00 — no deduction lines match.
- CDS: 0.00 — references OBJ-12, maps to open work item, file in scope, no new deps, status Approved, author authorized.
- Rule 4: nothing fires above AUTHORIZE.

### Ledger directive
`AUTHORIZE EXECUTION` — CCI 1.00, CDS 0.00, Trustworthiness High.

---

## CASE 2: EXECUTION FREEZE (stacked deductions)

### Change payload
```
commit 9fe21 | author @agent_2 | (no timestamp)
msg: "OBJ-12: update export schema doc"
files: products/signal_v2/core/schema.md [Draft] (no signature)
```

### Evaluation
- Rule 1: `schema.md` carries `DOC-2210`, indexed, one path. PASS.
- Escalations: none.
- CCI: 1.00 − 0.30 (Draft without owner signature) − 0.15 (missing timestamp) = **0.55**
- 0.55 < 0.70 hard floor → `EXECUTION FREEZE`.
- CDS still computed and reported: 0.00 (objective referenced, in scope, authorized author).
- Rule 4: FREEZE outranks AUTHORIZE.

### Ledger directive
`EXECUTION FREEZE` — CCI 0.55, CDS 0.00, Trustworthiness Zero. The work may be fine; its provenance is not verifiable, so it cannot enter the execution tier until signed and timestamped.

**Why this matters:** the change *content* is aligned. A drift-only gate would pass it. The CCI catches the missing authority chain — exactly the kind of unverifiable agent output that compounds silently.

---

## CASE 3 (EDGE): THE SILENT DRIFT

This is the case the obvious rules don't cover: everything is verified, signed, and passing — and still wrong.

### Intent file (excerpt)
- Intent: "Ship v2 export endpoint for the reporting module."
- Scope paths: `products/signal_v2/core/`
- Open objectives: `OBJ-12: export endpoint`
- Authorized actors: `@engineering_lead`, `@agent_4`

### Change payload
```
commit a8810 | author @agent_4 | 2026-06-11T23:40:00Z
msg: "Optimized continuous delivery pipeline dependencies"
files:
  products/signal_v2/core/build.md   [Approved] sig:@engineering_lead (ID: DOC-2210)
  infra/pipeline/cache_layer.md      [Approved] sig:@engineering_lead (ID: DOC-3301)
```
(Intent file assets: `API-4410`, `AST-9912`, `DOC-2210` — note `DOC-3301` is well-formed but undeclared.)

### Evaluation
- Rule 1: both IDs well-formed, correctly prefixed, single paths — no integrity violation. `DOC-3301` is undeclared, which routes to drift scoring per Rule 1's undeclared-entity clause, not to HALT.
- Escalations: none.
- CCI: 1.00 − 0.20 (orphan asset: `DOC-3301` path outside declared scope) = **0.80**. Above the 0.70 floor.
- CDS:
  - Message references no declared objective: **+0.15**
  - Message maps to no open work item: **+0.15**
  - `infra/pipeline/cache_layer.md` outside declared scope: **+0.10**
  - `DOC-3301` undeclared dependency: **+0.10**
  - Total: **0.50**
- 0.35 < 0.50 ≤ 0.55 → `MANDATORY LIFECYCLE PIVOT`.

### Ledger directive
`MANDATORY LIFECYCLE PIVOT` — CCI 0.80, CDS 0.50, Trustworthiness Moderate.

**The edge:** nothing here is broken. The agent did competent, signed, verified work — on something nobody asked for, half of it outside scope. This is how AI-assisted projects rot: not through failures, but through valid work accumulating off-axis. The PIVOT directive doesn't reject the work; it freezes forward execution until the intent file is either updated to include the pipeline work (making it in-scope) or the work is re-anchored to OBJ-12. The operator forces that reconciliation instead of letting the silo grow.

---

## CASE 4 (ESCALATION): THE COHERENT DECOY

### Change payload
Three agent commits, seconds apart, identical messages, all writing the same file — and none carrying a stable ID:

```
commit f1102 | @agent_7 | 01:00:00Z | "OBJ-12: export endpoint finalized and verified" | export_final.md [Approved] (no ID)
commit f1103 | @agent_8 | 01:00:02Z | "OBJ-12: export endpoint finalized and verified" | export_final.md [Approved] (no ID)
commit f1104 | @agent_9 | 01:00:04Z | "OBJ-12: export endpoint finalized and verified" | export_final.md [Approved] (no ID)
```

### Evaluation
- Step 2 fires before any scoring: multiple sources in exact agreement, all skipping Rule 1 ID verification → Escalation Trigger 1.
- Directive string, exactly: `CRITICAL ADVERSARIAL HALT: CONFLICTING MULTI-AGENT INJECTION DETECTED`
- Routing field: `ESC-COHERENT-DECOY-CAPTURE-ALARM`. Offending sources (@agent_7, @agent_8, @agent_9) listed in the log. Human review required.

**The edge:** agreement is not verification. Three agents saying the same thing is one claim repeated three times — and unanimity with missing provenance is the signature of uncoordinated consensus trying to force a write-back. The operator treats suspicious harmony as an alarm, not as confidence.

---

## CASE 5 (ESCALATION): THE VERIFIED CONTRADICTION

### Change payload
```
Entity DEP-7001 (v2.3, sig:@engineering_lead): relationship_log states "DEP-7002 is non-functional, do not route through it"
Entity DEP-7002 (v2.3, sig:@platform_lead):    relationship_log states "DEP-7001 routes through this component, fully functional"
```

### Evaluation
- Both entities carry valid signatures and identical active versions — neither can be discounted on provenance.
- Their declared relationships contradict. Escalation Trigger 2 fires.
- Status code, exactly: `CONTEXTUAL AMBIGUITY EXECUTION REJECTION`. Both IDs and both claims written to the log. All state transitions locked pending an administrative override line in the next payload.

**The edge:** the operator is explicitly forbidden from picking a side, however plausible one looks. Two verified sources in contradiction means the *context layer itself* is broken — resolving it by preference would paper over the break. The only honest output is a locked gate and the exact coordinates of the conflict.
