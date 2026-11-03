# HealthStream ETL

[![CI](https://github.com/vankudothuadhinaik/HealthStream-ETL/actions/workflows/ci.yml/badge.svg)](https://github.com/vankudothuadhinaik/HealthStream-ETL/actions/workflows/ci.yml)
![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Data](https://img.shields.io/badge/Data-Synthetic%20only-64748b)

A small, testable Python ETL pipeline for **synthetic healthcare encounter events**. It demonstrates ingestion, validation, normalization, duplicate handling, deterministic aggregation, data-quality reporting, and automated testing without using real patient information.

> **Data safety:** every row in `data/sample_encounters.csv` is fabricated. Do not use real patient data in this demo. This is an educational data-engineering project, not a clinical, billing, or production healthcare system.

## At a glance

| Area | Implementation |
|---|---|
| Input | CSV encounter records |
| Processing | Schema validation, normalization, deduplication |
| Outputs | Cleaned CSV, daily summary CSV, JSON quality report |
| Runtime | Python 3.10+ |
| Dependencies | Python standard library |
| Tests | `unittest` |
| CI | GitHub Actions |

## Pipeline

```text
CSV source
  -> schema and value validation
  -> normalize dates, categories and amounts
  -> deduplicate by event_id
  -> cleaned_events.csv
  -> daily_summary.csv + quality_report.json
```

## Quick start

**Requirements:** Python 3.10 or newer.

```bash
git clone https://github.com/vankudothuadhinaik/HealthStream-ETL.git
cd HealthStream-ETL
python healthstream_etl.py --input data/sample_encounters.csv --output-dir output
python -m unittest discover -s tests -v
```

## Output artifacts

- `cleaned_events.csv` — normalized, valid, unique events.
- `daily_summary.csv` — encounter count and total amount grouped by date and encounter type.
- `quality_report.json` — input, accepted, rejected, duplicate counts, and row-level rejection reasons.

## Validation and data-quality rules

- Required columns: `event_id`, `patient_key`, `event_date`, `encounter_type`, `diagnosis_code`, `amount_usd`.
- Event IDs and patient keys must be non-empty.
- Dates must use ISO `YYYY-MM-DD`.
- Amounts must be finite and non-negative; values are rounded to two decimal places using Python Decimal's default `ROUND_HALF_EVEN` rule.
- Repeated event IDs are counted as duplicates and only the first valid occurrence is retained.
- Invalid records are excluded and reported; invalid values are not silently coerced.
- Duplicate header names fail fast. Rows with surplus fields are rejected with their source row number, and malformed CSV quoting fails rather than being silently accepted.

## Design choices

- Deterministic output ordering makes repeated runs reproducible.
- Decimal arithmetic avoids binary floating-point drift in monetary aggregation.
- The standard-library implementation keeps installation lightweight.
- Synthetic pseudonymous keys are used; no names, addresses, dates of birth, or other direct identifiers are included.
- Rejected row contents are not copied into the quality report; only source-row diagnostics and reasons are recorded.

## Project structure

```text
HealthStream-ETL/
├── .github/workflows/ci.yml
├── data/sample_encounters.csv
├── healthstream_etl.py
├── tests/test_healthstream_etl.py
└── README.md
```

## CI and verification

GitHub Actions runs the unit tests and a sample end-to-end pipeline check on pushes and pull requests. Regression coverage includes Decimal rounding, empty datasets, malformed CSV input, duplicate headers, surplus fields, invalid dates, and source-row diagnostics.

See [workflow runs](https://github.com/vankudothuadhinaik/HealthStream-ETL/actions) for the recorded CI history.

## Scope and limitations

This is a portfolio demonstration. It does not claim HIPAA compliance, production-grade security, real EHR integration, clinical validity, or suitability for real patient data.

## Maintainer

[Adhi Naik](https://github.com/vankudothuadhinaik)
