SaaS Data Cleaner

SaaS Data Cleaner is an auditable Python command-line workflow for cleaning customer,
subscription, invoice, and product-usage CSV files. It produces accepted datasets, rejected
records with explicit reason codes, and a processing control total suitable for finance and
revenue-analytics practice.

The project is the first component of a wider SaaS Revenue Intelligence portfolio. It focuses on
the reliability layer that must exist before ARR, NRR, churn, LTV, propensity, or forecasting
models can be trusted.

Implemented capabilities

Normalises source column names to lower snake case.

Trims text and standardises identifiers, currencies, statuses, and billing frequencies.

Parses dates, integers, numeric usage, and money fields.

Uses decimal arithmetic for monetary precision.

Validates required columns and allowed business values.

Rejects every row in a duplicate primary-key group instead of silently selecting one.

Checks customer and subscription references across tables.

Preserves the original CSV row number for investigation.

Writes separate clean and rejected datasets.

Produces deterministic JSON control totals and rejection-reason counts.

Includes linting, static type checking, tests, coverage, package builds, and GitHub CI.

Data flow

flowchart LR
    A["Source CSV files"] --> B["Schema and value checks"]
    B --> C["Cross-table reference checks"]
    C --> D["Clean data"]
    C --> E["Rejected data and reason codes"]
    D --> F["Processing summary"]
    E --> F

The cleaner never converts currencies, repairs broken references, or deletes duplicate records
automatically. Those decisions require an authorised review process.

Required input files

Place four UTF-8 CSV files in one directory:

File

Primary key

Purpose

customers.csv

customer_id

Customer identity and commercial segment

subscriptions.csv

subscription_id

Plan, seats, pricing, currency, and lifecycle

invoices.csv

invoice_id

Billing amount and payment status

product_usage.csv

customer_id, usage_date

Daily adoption and consumption measures

The complete field contract is documented in docs/data-contract.md.
Synthetic input examples containing both valid and invalid cases are available in
data/sample/input.

Quick start with uv

Prerequisites:

Python 3.12

uv

git clone https://github.com/SHIVASAI234/saas-data-cleaner-python.git
cd saas-data-cleaner-python
uv sync --locked
uv run saas-data-cleaner \
  --input-dir data/sample/input \
  --output-dir data/output

Expected log message for the included sample:

INFO Cleaning complete: 9 accepted, 15 rejected from 24 rows

Quick start with pip

python -m venv .venv

Activate the environment on Windows PowerShell:

.venv\Scripts\Activate.ps1

Then install and run:

python -m pip install --upgrade pip
python -m pip install -e .
saas-data-cleaner --input-dir data/sample/input --output-dir data/output

Output contract

For each source table the pipeline writes:

<table>_clean.csv: accepted records with source_row_number.

<table>_rejected.csv: rejected records with source_row_number and error_codes.

It also writes processing_summary.json:

{
  "schema_version": "1.0",
  "tables": {
    "customers": {
      "input_rows": 6,
      "accepted_rows": 3,
      "rejected_rows": 3,
      "rejection_reasons": {
        "DUPLICATE_PRIMARY_KEY": 2,
        "MISSING_SEGMENT": 1
      }
    }
  },
  "totals": {
    "input_rows": 24,
    "accepted_rows": 9,
    "rejected_rows": 15
  }
}

A row can have more than one semicolon-separated error code. Rejection-reason counts can therefore
be greater than the number of rejected rows.

Quality commands

uv run ruff format --check .
uv run ruff check .
uv run mypy src
uv run pytest --cov=saas_data_cleaner --cov-report=term-missing
uv build

CI runs these commands for pull requests and pushes to main. This repository contains continuous
integration only; no deployment workflow or production environment is configured.

Design choices

Evidence preservation: duplicate-key groups are rejected in full and retain source row
numbers.

Exact money representation: monetary values are parsed as Decimal and exported with two
decimal places.

No inferred corrections: the cleaner identifies invalid records but does not invent missing
data or remap unsupported currencies.

Deterministic results: the summary excludes execution timestamps so identical inputs create
comparable control totals.

Small dependency surface: pandas is the only runtime dependency.

Known limitations

Input schemas are currently defined in Python and are not externally configurable.

Only CSV input is supported.

Supported currencies are AUD, CAD, CHF, EUR, GBP, and USD.

Amount fields assume two decimal places; currency-specific minor units are not yet modelled.

Cross-file checks operate in memory and are not designed for warehouse-scale volumes.

The pipeline reports quality failures but does not implement a human remediation workflow.

Roadmap

Add configurable schemas and currency minor-unit rules.

Add Parquet input and output.

Add chunked processing for larger files.

Publish data-quality trend metrics.

Feed accepted datasets into the Revenue Transformation Pipeline project.

Contributing and security

See CONTRIBUTING.md before proposing a change. Report vulnerabilities according
to SECURITY.md.

This project is available under the MIT License.
