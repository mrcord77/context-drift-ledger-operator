# Sample run

A pre-populated workspace showing what the operator leaves behind over a few days of gating: `ledger_log.jsonl` (one line per verdict, append-only) and `drift_dashboard.html` (open it in a browser — drift trend against the PIVOT/HALT lines, full verdict history). Note the story the trend tells: clean → a provenance freeze → scores climbing → pivot → adversarial halt. Regenerate the dashboard any time with `python scripts/build_drift_dashboard.py --log sample_run/ledger_log.jsonl --out sample_run/drift_dashboard.html`.
