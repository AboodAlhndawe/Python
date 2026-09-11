# Capstone Project — Ethical Web Scraper & Security-Style Analysis Tool

**Python Workshop Capstone** — combines the Git discipline, environment
hygiene, log/text parsing, HTTP requests, and file-forensics techniques
practiced across the 3-day intensive into one end-to-end tool.

## What it does

This tool has two modes that share the same data-persistence and
error-handling backbone:

1. **Web Scraper** — given one or more URLs, it ethically fetches the
   page (identifies itself with a User-Agent, retries on timeouts,
   backs off on HTTP 429 rate limits), then extracts:
   - every unique link (`<a href>`)
   - the top words on the page (word-frequency `Counter`)
   - basic page size stats

2. **Log Analyzer** — reuses the workshop's log-parsing techniques
   (line filtering, `re` for IP extraction, `Counter` for tallies) to
   turn a plain-text log (e.g. an SSH auth log) into a short report:
   failed-login count, per-level tally, and the top offending IPs —
   the same "DFIR triage" style script built on Day 3.

Every run saves its results to `data/` as both **CSV** and **JSON**,
so results can be reopened, spreadsheet-analyzed, or fed into another
tool.

## Project structure

```
capstone_project/
├── data/                   # Sample and generated datasets
│   ├── sample_data.csv       # Example of a saved scrape report
│   └── sample_auth.log       # Example log file for analyze-log
├── src/
│   ├── __init__.py
│   ├── main.py               # CLI entry point (argparse + interactive menu)
│   ├── logic.py               # WebScraper, LogAnalyzer, DataStore (OOP core)
│   └── utils.py               # Validation, safe HTTP requests, hashing
├── tests/
│   └── test_logic.py          # Unit tests (pytest) — no network required
├── .gitignore
├── requirements.txt
└── README.md
```

## Requirements

- Python 3.10+
- Packages listed in `requirements.txt` (`requests`, `beautifulsoup4`,
  `rich`, `pytest`)

## Installation

```bash
git clone <your-repo-url>
cd capstone_project
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

## Usage

**Scrape one or more URLs:**
```bash
python -m src.main scrape https://example.com https://httpbin.org
```

**Analyze a log file:**
```bash
python -m src.main analyze-log data/sample_auth.log
```

**Interactive menu (no arguments):**
```bash
python -m src.main
```

### Example output

```
Log Analysis: data/sample_auth.log
┏━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Metric                ┃ Value                       ┃
┡━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Failed password lines │ 5                           │
│ Top IPs               │ [('45.33.12.9', 3), ...]    │
└───────────────────────┴─────────────────────────────┘
Saved report to data/log_report.json
```

## Running the tests

```bash
pip install -r requirements.txt
pytest tests/ -v
```

All 12 unit tests run against local sample data and fixed HTML strings,
so they pass without any network access.

## Technical components checklist

| Requirement                          | Where it lives |
|---------------------------------------|----------------|
| Modular architecture                  | `src/main.py`, `src/logic.py`, `src/utils.py` |
| OOP design                            | `WebScraper`, `LogAnalyzer`, `DataStore` classes in `logic.py` |
| Data persistence                      | CSV + JSON read/write in `DataStore` |
| Error handling & validation           | `utils.validate_url`, `utils.safe_request`, try/except in `main.py` |
| External package integration          | `requests`, `beautifulsoup4`, `rich`, `pytest` |
| Clean, documented code                | PEP 8 naming, docstrings on every class/function, inline comments |

## Known limitations

- The scraper only reads publicly served HTML — it does not execute
  JavaScript, so content rendered client-side won't be captured.
- `LogAnalyzer`'s IP/keyword patterns are tuned for the sample
  auth-log format used in the workshop; adjust the regex in
  `logic.py` for other log formats.
- Always scrape only sites you own or are authorized to test, and
  respect each site's `robots.txt` and terms of service.
