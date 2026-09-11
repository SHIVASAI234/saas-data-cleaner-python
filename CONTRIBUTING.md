# Contributing

## Development setup

```bash
uv sync --locked --dev
```

## Change workflow

1. Create a focused branch from `main`.
2. Add or update tests with the implementation.
3. Run every quality command locally.
4. Update the data contract and README when behaviour changes.
5. Open a pull request using the repository template.

## Required checks

```bash
uv run ruff format --check .
uv run ruff check .
uv run mypy src
uv run pytest --cov=saas_data_cleaner --cov-report=term-missing
uv build
```

Do not include customer data, credentials, access tokens, or other sensitive information in code,
fixtures, logs, issues, or pull requests.
