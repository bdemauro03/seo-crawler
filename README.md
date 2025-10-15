# SEO Crawler

A practical, lightweight SEO crawler that audits a website and exports page-level metadata to CSV and JSON.  
It demonstrates end-to-end problem solving across crawling, parsing, heuristics, scoring, and reporting.

## Highlights

- **Purpose-built CLI tool** with clean packaging (`pyproject.toml`) and `src/` layout.
- **Robust parsing** via BeautifulSoup + lxml; **URL normalization** and duplicate avoidance.
- **Heuristic analysis** for low-value pages and a **0–100 SEO score** per URL.
- **Structured outputs** (CSV + JSON) ready for BI tools or data notebooks.
- **Tested + Linted**: lightweight unit test, `ruff` + `black` config via `Makefile`.

## What it does

Crawls starting from a URL (optionally constrained to the same domain).

Extracts core SEO fields:
```bash
    - <title>, <meta name="description">, first <h1>, canonical URL, robots meta
    - HTTP status, load time (ms), word count

Flags low-value pages using simple, explainable rules (see below).

Computes an SEO score (0–100) you can tweak.

Exports to:
    - reports/seo_audit.csv
    - eports/seo_audit.json
```

## Quickstart

```bash
# 1) Create & activate a virtual environment
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows
# .venv\Scripts\activate

# 2) Install dependencies
python -m pip install -r requirements.txt

# 3) Run a crawl (example)
python -m seo_crawler.cli https://example.com --max-pages 200 --same-domain-only --score
```

This will create reports/seo_audit.csv and reports/seo_audit.json.

After packaging (pip install .), you can use the console script:
```bash
seo-crawler https://example.com --max-pages 200 --same-domain-only --score
```

## CLI Usage

```css
seo-crawler START_URL [--max-pages N] [--output PATH] [--exclude P1 P2 ...]
                      [--same-domain-only] [--respect-robots]
                      [--user-agent UA] [--score]
```

### Arguments

START_URL – Starting URL (e.g., https://yoursite.com).

--max-pages – Max pages to crawl (default: 1000).
--output – Output CSV path (default: reports/seo_audit.csv).
--exclude – One or more substrings/regex patterns to skip (e.g., /tag/, /category/, \\?utm_).
--same-domain-only – Restrict crawl to the same eTLD+1 (recommended).
--respect-robots – Obey robots.txt Disallow rules for the selected --user-agent.
--user-agent – Custom UA string (default: seo-crawler-bot/1.0).
--score – Compute a 0–100 SEO score per page.

### Examples

```bash
# Crawl a single domain, skip tag/category pages, compute scores
seo-crawler https://example.com --same-domain-only --exclude "/tag/" "/category/" --score

# Limit to 250 pages, custom UA, export to a custom path
seo-crawler https://docs.example.com --max-pages 250 --user-agent "mybot/0.1" --output out/audit.csv
```

## Output Schema

| Column             | Description                                               |
| ------------------ | --------------------------------------------------------- |
| `url`              | Absolute URL                                              |
| `status`           | HTTP status code                                          |
| `title`            | `<title>` text                                            |
| `meta_description` | `<meta name="description">` content                       |
| `h1`               | First `<h1>` text                                         |
| `canonical`        | Canonical URL from `<link rel="canonical">`               |
| `robots_meta`      | Content of `<meta name="robots">` (e.g., `index, follow`) |
| `word_count`       | Approximate word count of visible text                    |
| `load_time_ms`     | Page fetch time (ms)                                      |
| `is_indexable`     | Boolean (derived; reserved for future logic)              |
| `low_value_flag`   | Boolean flag from heuristics below                        |
| `seo_score`        | Integer 0–100 (only when `--score` is used)               |

## Low-Value Page Heuristics

A page is flagged low-value if any of the following indicate thin or non-indexable content:

- Missing or long <title> (> 70 chars)
- Missing or long meta description (> 160 chars)
- Missing <h1>
- Missing canonical URL
- Word count < 150
- Robots meta contains noindex
- Non-200 HTTP status

You can adjust thresholds and logic in src/seo_crawler/runner.py:

- compute_low_value_flag(pr: PageResult) -> bool
- compute_seo_score(pr: PageResult) -> int

## Project Structure

```bash
seo-crawler/
├── src/
│   └── seo_crawler/
│       ├── __init__.py
│       ├── cli.py          # CLI entrypoint / argument parsing
│       └── runner.py       # Crawl, parse, heuristics, scoring, I/O
├── tests/
│   └── test_import.py      # Minimal smoke test
├── reports/                # Output folder (created on first run)
├── pyproject.toml          # Build metadata & console script
├── requirements.txt        # Dependencies
├── Makefile                # Dev helpers (lint, format, run)
├── .gitignore
└── README.md
```

## Development

```bash
# Install runtime deps
make install

# Format & lint
make format
make lint

# Run smoke tests
make test

# Example run
make run
```

## Packaging

```bash
python -m pip install build
python -m build
python -m pip install dist/seo_crawler-0.1.0-py3-none-any.whl
seo-crawler https://example.com --score
```

## Responsible Use

Only crawl properties you’re authorized to audit. Respect site terms, robots.txt, and rate limits.

## License

MIT © bdemauro03