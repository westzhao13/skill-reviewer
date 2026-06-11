---
name: csv-deduplicator
description: >
  Remove duplicate rows from CSV files using configurable column matching.
  Use when the user asks to deduplicate a CSV, remove repeated rows from a
  spreadsheet, or clean up tabular data — triggers include "dedup my CSV",
  "remove duplicates from users.csv", "find unique rows", "去重 CSV",
  or the /csv-dedup command. Supports exact matching, case-insensitive
  matching, and matching on a subset of columns. Outputs a cleaned CSV
  plus a removal report. NOT for JSON, Parquet, fuzzy / semantic
  deduplication, or general data transformation.
---

# CSV Deduplicator

Removes duplicate rows from CSV files using deterministic column-based
matching. Produces a cleaned CSV and a removal report.

## When to use

- "Dedup `users.csv`"
- "Remove duplicate rows from this export"
- "Find unique rows ignoring case"
- "Dedup based on the email column only"

## Inputs

| Input | Required | Default | Notes |
|-------|----------|---------|-------|
| Input CSV path | yes | — | Absolute path; must be readable |
| Match columns | no | all columns | Comma-separated column names |
| Case-insensitive | no | true | Set `--case-sensitive` to disable |
| Output path | no | `<input>-dedup.csv` | Overwritten if exists |

## Workflow

1. Validate the input CSV exists and is readable.
2. Parse the header; verify every requested match column exists.
   If a column is missing, fail fast with the available column list.
3. Stream rows through `scripts/dedup.py`, keeping the first occurrence
   of each match-key.
4. Write the cleaned CSV to the output path.
5. Write `<output>-report.md` summarizing rows kept, rows removed, and
   the first 20 removed keys.
6. Verify the output file: row count = kept rows + 1 header.

## Example

Input `users.csv` has 1000 rows; 230 share an email with an earlier row.

```bash
python scripts/dedup.py users.csv --columns email --case-insensitive
```

Output:

- `users-dedup.csv` — 770 data rows + 1 header
- `users-dedup-report.md` — lists the 230 removed duplicates with their
  original row numbers and the email that collided

## Dependencies

- Python 3.10+
- `pandas >= 2.0` (verified at script startup; clear install hint on
  failure)

## Scripts

See `scripts/dedup.py --help` for full flags. Each script opens with a
docstring covering purpose, parameters, return codes, and one usage
example.

## Error handling

All script errors follow the shape:

```
Error: <what failed>
  Cause: <why>
  Fix: <concrete action>
```

Example: `Error: column 'email' not found. Cause: header row contains
only [id, name, signup_date]. Fix: pass --columns <one-of-those> or
add the column to the CSV.`

## Evaluation

`evals/` contains three fixtures: exact-match dedup, case-insensitive
dedup, and subset-column dedup. Tested on Haiku 4.5 and Sonnet 4.6.

## References

- [references/csv-edge-cases.md](references/csv-edge-cases.md) — quoting,
  BOM, CRLF, and mixed-encoding handling.
