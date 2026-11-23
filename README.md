# ORIE 5254 Final Project

[One-sentence summary of what this project does and why it exists.]

## Quickstart

From zero to running:

```bash
# 1. Clone
git clone <repo-url> && cd "ORIE 5254 final project"

# 2. Install dependencies
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"

# 3. Run tests
pytest
```

## Usage

```python
# Basic usage example
from src.data_loader import load_data

data = load_data("data/raw/example.csv")
```

## Architecture

[High-level overview: how the major components (data_loader, data_processing, feature_registry, transformations) connect. Include a diagram or flow description.]

## Project Structure

```
ORIE 5254 final project/
├── README.md
├── TODO.md
├── CLAUDE.md
├── pyproject.toml
├── .env.example
├── .gitignore
├── data/
│   ├── raw/              # Immutable source data — never modify
│   ├── interim/          # Intermediate transforms, cleaning outputs
│   └── processed/        # Final, analysis-ready datasets
├── src/
│   ├── config/           # Settings: env vars, paths, constants
│   ├── models/           # Data models, types, validation schemas
│   ├── data_loader/      # Ingestion from files, APIs, databases
│   ├── data_processing/  # Cleaning, validation, normalization
│   ├── feature_registry/ # Feature definitions, metadata, versioning
│   ├── transformations/  # Reusable transform functions
│   └── utils/            # Logging, timing, common helpers
├── tests/                # Mirrors src/
├── scripts/              # Standalone utility/automation scripts
├── notebooks/            # Exploration and prototyping only
├── research/             # Notes, literature, exploration findings
└── reports/              # Generated reports
```
