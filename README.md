# HackerRank Orchestrate October 2026 — Ghost Data Still Exists

This repository contains the participant submission for the HackerRank Orchestrate October 2026 challenge: "Ghost Data Still Exists."

## Challenge Overview

Build an AI agent that audits whether a user's personal data was actually deleted. A deletion workflow may report **Complete** while data still exists in databases, logs, support attachments, exports, search indexes, archives, or backups. Sync and restore processes may also bring deleted data back.

For each deletion request, the agent must:
1. Determine whether deletion was actually authorized
2. Return `COMPLETE`, `INCOMPLETE`, or `NOT_APPLICABLE`
3. Report any remaining data or recreation risk
4. Produce the ordered remediation and verification steps required to finish an authorized deletion safely

## Repository Structure

```
.
├── README.md                  # This file
├── code/
│   ├── main.py               # Main pipeline entry point
│   ├── evaluation/
│   │   ├── evaluation_report.md  # Case/row/field/sequence metrics, custom test cases
│   │   ├── usage_report.md       # Final full-dataset run counts, token/cost totals
│   │   ├── main.py               # Scorer/evaluation script
│   │   └── scorer.py
│   └── tests/                # Unit and regression tests
├── output.csv                 # Generated: 518 rows, case_id,deletion_status,steps_json
├── code.zip                   # Packaged submission: source, tests, README, evaluation reports
├── log.txt                   # Agent turn transcript (AGENTS.md format)
└── dataset/                  # Participant-facing inputs (not included in code.zip)
```

## Quick Start

### Prerequisites

- Python 3.10 or later
- `pdftotext` (for PDF evidence) — optional; falls back to `pypdf`
- `tesseract` (for OCR of image evidence) — optional; VLM model `GHOSTDATA_VLM_MODEL` enables local vision fallback
- SQLite3 (standard library, included)

### Environment Setup

```bash
# Create and activate virtual environment
python -m venv .venv

# Windows PowerShell:
.\.venv\Scripts\Activate.ps1

# macOS / Linux:
source .venv/bin/activate

# Install optional dependencies
pip install pypdf  # if PDF evidence parsing is needed
```

### Run the Pipeline

From the repository root, generate the full prediction file:

```bash
python code/main.py
```

This reads participant-facing inputs under `dataset/` and writes `output.csv` in the repository root.

To score a prediction file against the public sample answers:

```bash
python code/evaluation/main.py output.csv
```

The evaluator defaults to `dataset/sample_output.csv` as its reference.

### Run a Subset

```bash
python code/main.py --requests qa_empty_subset_requests.csv --output output_subset.csv
```

### OCR Availability

The pipeline uses Tesseract TSV OCR when the executable is available (`PATH`, or the standard Windows Program Files install location). Without an OCR executable, image evidence IDs are reported as skipped unless `GHOSTDATA_VLM_MODEL` enables the optional local vision transcription fallback.

OCR text is cached in SQLite by source-file SHA-256 and extractor version. Set `GHOSTDATA_LOCAL_TEXT_CACHE` to choose the cache file.

The CLI reports OCR availability, image/cache counts, extracted text evidence IDs, and skipped IDs with reasons.

## Output

Create `output.csv` with these columns in this exact order:

```text
case_id,deletion_status,steps_json
```

Produce exactly one row per request. `deletion_status` is `COMPLETE`, `INCOMPLETE`, or `NOT_APPLICABLE`. `steps_json` is an ordered JSON array containing findings, remediation actions, and verification checks. Use an empty array (`[]`) for `NOT_APPLICABLE`. Evidence references may be a `file_id`, `activity_id`, or `audit_id`.

The final file must contain 518 data rows sorted by `case_id` (plus the header row).

## Evaluation

`output.csv` will be compared with hidden canonical answers. Scoring considers:

- deletion assessment accuracy
- current and historical identity resolution
- discovery of remaining data and recreation risks
- correct policy application and evidence references
- completeness and ordering of remediation and verification steps

## Code Packaging

### Package code.zip

**Windows (PowerShell):**

```powershell
Compress-Archive -Path README.md, code -DestinationPath code.zip -Force
```

Keep evaluation reports in `code/evaluation/` so they are bundled automatically with `code`. Do not include `dataset/`, virtual environments (`.venv`, `venv`, `node_modules`), or build caches (`__pycache__`). On Windows, keep `.venv` at the repository root (outside `code/`) and remove any `code/__pycache__` folders before running `Compress-Archive`.

Do not include nested archives (`.zip`, `.tar`, `.gz`, `.tgz`, `.7z`, `.rar`) inside `code.zip`.

### Submission Items

- `output.csv` — 518 data rows with exact header `case_id,deletion_status,steps_json`
- `code.zip` — runnable solution, prompts, configuration, `README.md`, and `evaluation/` reports
- `log.txt` — root session transcript generated by the AI coding agent, uploaded to `chat_transcript` slot
- No API keys, credentials, real personal data, or sensitive configuration values included

## Important Disclosures

### case_55 and case_61 — Evidence-Limited INCOMPLETE Rows

Two INCOMPLETE rows have empty step arrays (`[]`) because admissible pre-cutoff evidence links identity but does not establish resource state at the audit cutoff:

- **case_55**: pre-cutoff backup archive evidence gives no restore-permission proof; activity_4006 is identity-only; activity_3387 is post-cutoff
- **case_61**: identity-link claim (activity_4007 → object_19) exists but does not prove current resource state

No contract-valid findings, remediation, or verification steps can be generated for these cases without inferring state beyond admissible evidence. They are retained as disclosed limitations in the submitted output.

### case_12 Policy / Activity Conflict

The current operation-based treatment emits `DELETE` for `analytics_9914`, matching the public sample answer. However, `system_activity.csv` activity_3435 reports a `DELETE_RECORD` attempt with result `FAILED`, which does not prove successful deletion. The supplied `deletion_rules.csv` rule_07 says an attributable `ANALYTICS_PROFILE` requires `ANONYMIZE`. The public sample's DELETE expectation, failed operation event, and policy rule remain in conflict; this discrepancy is documented but not resolved in the current output.

## Agent Trace (log.txt)

This repository includes `AGENTS.md`, which instructs compatible AI coding tools to append conversation summaries to `log.txt` in the repository root:

- Always launch your AI coding agent from the repository root (or explicitly ask it to read `./AGENTS.md` first) so every turn is logged with the required `User Prompt`, `Agent Response Summary`, `Actions`, and `Context` sections.
- Upload `log.txt` directly as your `chat_transcript` submission on HackerRank.
- Do not include API keys, tokens, credentials, or secrets in prompts, code, configuration, or `log.txt`.

## Chat Transcript

The repository root `log.txt` follows the AGENTS.md turn format. It is included as the `chat_transcript` submission component.

## Policy Authority

`deletion_rules.csv` is authoritative over `sample_output.csv`. When they conflict, rule_07 requires `ANALYTICS_PROFILE` to be anonymized (`ANONYMIZE_RECORD` + `VERIFY_RECORD_ANONYMIZED`). The current code previously emitted `DELETE`; this has been corrected via generic policy-to-operation mapping with regression coverage.

---

**Generated**: 2026-10-25

**Submission**: `VivaanReddy3011/Codex-1`, commit `979f555`