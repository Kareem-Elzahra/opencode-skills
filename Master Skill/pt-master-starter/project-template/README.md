# Project workspace template

Copy this directory under an approved target-specific project path, then fill in `target.yaml` without credentials. `state.json` and `test-history.jsonl` are updated by the workflow coordinator. Each skill keeps one running report at `reports/<skill-id>.md`; focused runs append new dated sections to the original report. Evidence must be redacted and kept in `evidence/`.

The starter implements workflow *instructions*, not an enforced deduplication database or executable scanning integration. Verify that the agent's actual writes and commands comply before allowing real target testing. For guaranteed deduplication and atomic journal writes, add a deterministic helper script in a later iteration.
