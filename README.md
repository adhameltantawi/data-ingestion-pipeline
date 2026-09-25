# 🚀 Data Ingestion Pipeline — GitHub Events ETL

A hands-on Python project that extracts real-time data from the **GitHub REST API**, transforms it, and loads it into a **PostgreSQL** database using **dlt** (Data Load Tool).

---

## 📌 Project Overview

This project demonstrates a complete **Extract → Transform → Load (ETL)** pipeline built from scratch. Data is sourced from the GitHub Events API, paginated across multiple pages, and efficiently streamed into a relational database using the `dlt` library.

---

## 🛠️ Tech Stack

| Tool          | Purpose                                      |
|---------------|----------------------------------------------|
| Python 3      | Core language                                |
| `requests`    | HTTP client for REST API calls               |
| `dlt`         | Data load tool for schema inference & loading |
| PostgreSQL    | Target database                              |
| Git & GitHub  | Version control + data source (GitHub Events API) |

---

## 📂 Project Structure

```
data-ingestion-pipeline/
├── freeCodeCamp_etl.ipynb   # Step-by-step ETL notebook (main implementation)
└── README.md                # Project documentation
```

---

## 🔄 Pipeline Architecture

The pipeline is built progressively in the notebook across the following stages:

### 1. 📡 Extraction — Fetch GitHub Events

Connect to the public GitHub Events API and pull raw event data (watches, forks, pushes):

```python
import requests

url = "https://api.github.com/repos/DataTalksClub/data-engineering-zoomcamp/events"
response = requests.get(url)
data = response.json()
```

### 2. 🔑 Authentication

Use a personal access token to increase the API rate limit from 60 → 5000 requests/hour:

```python
from google.colab import userdata

API_TOKEN = userdata.get('github-token')
headers = {'Authorization': f'Bearer {API_TOKEN}'}
response = requests.get(url, headers=headers)
```

> **Note:** Store your token securely using environment variables or a secrets manager — never hardcode it.

### 3. ⏱️ Rate Limit Handling

Check remaining API quota and pause execution when exhausted:

```python
remaining = requests.get("https://api.github.com/rate_limit").json()['rate']['remaining']

if remaining == 0:
    time.sleep(60)  # Wait for quota to reset before retrying
```

### 4. 📄 Pagination — Traverse All Pages

The GitHub API uses **Link headers** to signal the next page URL. We follow `rel="next"` until all pages are consumed:

```python
url = "https://api.github.com/repos/DataTalksClub/data-engineering-zoomcamp/events"

while True:
    response = requests.get(url)
    data = response.json()
    print(len(data))
    if 'next' not in response.links:
        break
    url = response.links['next']['url']
```

### 5. ⚡ Generator Pattern — Memory-Efficient Streaming

Instead of collecting all pages in memory, we use a **Python generator** to yield one page at a time:

```python
def events_getter():
    """Generator that yields one page of GitHub events at a time."""
    url = "https://api.github.com/repos/DataTalksClub/data-engineering-zoomcamp/events"
    while True:
        response = requests.get(url)
        yield response.json()
        if 'next' not in response.links:
            break
        url = response.links['next']['url']
```

Usage:
```python
for page in events_getter():
    print(page)  # Process each page without loading all data into memory
```

### 6. 🗄️ Loading — dlt into PostgreSQL

The generator is passed directly to `dlt`, which infers the schema and loads the data:

```python
import dlt

pipeline = dlt.pipeline(
    pipeline_name="github_events",
    destination="postgres",
    dataset_name="github_data"
)

load_info = pipeline.run(events_getter(), table_name="events")
print(load_info)
```

---

## ⚙️ Setup & Usage

### Prerequisites

```bash
pip install requests dlt[postgres]
```

### Environment Variables

Set your GitHub personal access token:

```bash
# Linux / macOS
export GITHUB_TOKEN="your_token_here"

# Windows (PowerShell)
$env:GITHUB_TOKEN = "your_token_here"
```

### Running the Notebook

1. Open `freeCodeCamp_etl.ipynb` in Jupyter or Google Colab.
2. Follow cells sequentially — each section is annotated with explanations.
3. Configure your PostgreSQL connection string in the `dlt` pipeline cell.

---

## 📚 Key Concepts Covered

- **REST API consumption** with `requests`
- **Bearer token authentication**
- **Pagination** using `Link` response headers
- **Generator functions** for memory-efficient data streaming
- **Schema inference** and automated loading with `dlt`
- **Rate limit management** with exponential backoff

---

## 🔗 References

- [GitHub REST API Docs](https://docs.github.com/en/rest)
- [dlt Documentation](https://dlthub.com/docs)
- [FreeCodeCamp Data Engineering Course](https://www.freecodecamp.org/)
