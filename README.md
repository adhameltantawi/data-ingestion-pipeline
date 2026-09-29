# GitHub Events ETL Pipeline

Extracts real-time activity data from the **GitHub REST API**, transforms it, and loads it into a local **DuckDB** database — demonstrating a production-grade ETL pattern built from scratch in Python.

---

## Overview

| Stage | Description |
|---|---|
| **Extract** | Paginated fetch of GitHub repository events with rate-limit handling and Bearer token auth |
| **Transform** | Flattens nested JSON, parses ISO timestamps, and filters malformed API responses |
| **Load** | Bulk upsert into DuckDB with automatic schema evolution (new columns added at runtime) |

---

## Tech Stack

| Tool | Role |
|---|---|
| `Python 3.10+` | Core language |
| `requests` | HTTP client for REST API calls |
| `duckdb` | Embedded analytical database |
| `datetime` | ISO timestamp parsing |

---

## Project Structure

```
data-ingestion-pipeline/
├── github_events_etl.ipynb   # Main ETL notebook (run this)
└── README.md
```

---

## Pipeline Architecture

```
GitHub REST API
      │
      ▼  (paginated, authenticated)
 fetch_events()          ← generator, one page at a time
      │
      ▼
 process_event()         ← flatten, parse timestamps, skip bad records
      │
      ▼
 DuckDB (github_events)  ← schema evolution + ON CONFLICT DO NOTHING
```

### Key Implementation Details

**Rate Limit Handling**
The API allows 60 unauthenticated requests/hour (5,000 with a token). The pipeline checks remaining quota before processing and sleeps 60 s when exhausted.

**Generator Pattern**
`fetch_events()` yields one page at a time instead of loading all pages into a list. This keeps memory usage constant regardless of total event volume.

**Schema Evolution**
New fields in incoming data trigger an `ALTER TABLE` automatically — the pipeline never fails on unexpected schema changes.

**Idempotency**
`ON CONFLICT DO NOTHING` on the `id` primary key ensures the pipeline is safe to re-run without creating duplicate records.

---

## Setup

### Prerequisites

```bash
pip install requests duckdb
```

### Authentication (Recommended)

A GitHub personal access token raises the rate limit from **60 → 5,000 req/hr**.

```bash
# Linux / macOS
export GITHUB_TOKEN="ghp_your_token_here"

# Windows (PowerShell)
$env:GITHUB_TOKEN = "ghp_your_token_here"
```

> **Security note:** Never hardcode tokens in notebooks. Always use environment variables or a secrets manager.

### Running the Notebook

1. Open `github_events_etl.ipynb` in Jupyter Lab / Notebook or VS Code.
2. Run all cells top-to-bottom — each section is self-contained and annotated.
3. The resulting `github_events.db` file can be queried with any DuckDB client.

---

## Concepts Demonstrated

- **REST API consumption** with `requests` and pagination via `Link` headers
- **Bearer token authentication** and rate limit management
- **Python generators** for memory-efficient streaming
- **Dynamic schema inference** and SQL `ALTER TABLE` at runtime
- **Idempotent loading** with primary-key conflict handling in DuckDB

---

## References

- [GitHub Events API](https://docs.github.com/en/rest/activity/events)
- [DuckDB Documentation](https://duckdb.org/docs/)
- [FreeCodeCamp Data Engineering Course](https://www.freecodecamp.org/)
