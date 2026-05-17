# pid-data-extraction
P&amp;ID Data Extraction Python

# DigitalSketch.ai - Python Sample Code

Official Python sample code for the [DigitalSketch.ai](https://digitalsketch.ai) P&ID digitization API.

This client uploads an image or multi-page PDF to the DigitalSketch API, runs the extraction pipeline, and saves structured predictions as JSON.

- Website: [digitalsketch.ai](https://digitalsketch.ai)
- API base: [api.digitalsketch.ai](https://api.digitalsketch.ai)
- API docs: [api.digitalsketch.ai/documentation](https://api.digitalsketch.ai/documentation)
- Homepage (in this repo): open [`index.html`](./index.html) in a browser

---

## Features

- Single-image upload (PNG, JPG, JPEG, BMP) via base64
- Multi-page PDF upload via multipart, with per-page processing
- Polls PDF processing, pipeline run-id resolution, and pipeline status
- Saves structured predictions per page as JSON in `output/`
- All polling intervals and timeouts configurable via environment variables

---

## Prerequisites

- Python 3.8 or newer
- A DigitalSketch.ai API key - sign up at [digitalsketch.ai](https://digitalsketch.ai)

---

## Quick Start

### 1. Clone and install

```bash
git clone https://github.com/<your-org>/digitalsketch-python-sample.git
cd digitalsketch-python-sample

python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

### 2. Configure your API key

Copy `.env.example` to `.env` and add your key:

```env
DIGITALSKETCH_API_KEY=your_api_key_here
DIGITALSKETCH_API_BASE=https://api.digitalsketch.ai

# Optional polling / timeout overrides (defaults shown)
PIPELINE_STATUS_POLL_INTERVAL_SECONDS=40
PIPELINE_STATUS_TIMEOUT_SECONDS=7200
RUN_ID_POLL_INTERVAL_SECONDS=15
RUN_ID_TIMEOUT_SECONDS=600
PDF_POLL_INTERVAL_SECONDS=2
PDF_TIMEOUT_SECONDS=900
```

### 3. Run the extractor

Place a P&ID file in the `sample/` folder (PNG, JPG, JPEG, BMP, or PDF). If no filename is given, the first supported file in the folder is used.

```bash
# Process the first supported file in sample/
python extract_predictions.py

# Or specify a file
python extract_predictions.py sample.pdf
python extract_predictions.py multipage.pdf
python extract_predictions.py sample.jpg
```

### 4. Inspect the output

Predictions are written to `output/` as JSON:

```
# Single image
output/<name>_<imageid>_predictions.json

# Multi-page PDF (one file per page)
output/<name>_page1_<imageid>_predictions.json
output/<name>_page2_<imageid>_predictions.json
```

---

## How the Pipeline Works

| Step | Endpoint | Purpose |
|------|----------|---------|
| 1 | `POST /digitalsketch/uploadimage` | Upload a single image (base64) |
| 1 | `POST /digitalsketch/uploadpdf/multipart` | Upload a multi-page PDF |
| 2 | `POST /digitalsketch/uploadpdfstatus` | Poll PDF processing, get per-page `imageid`s |
| 3 | `POST /digitalsketch/{imageid}/imagedetails` | Get metadata for an uploaded image |
| 4 | `POST /digitalsketch/pipeline/start` | Start the digitization pipeline |
| 4 | `POST /digitalsketch/pipeline/id` | Resolve `run_id` if not returned synchronously |
| 5 | `POST /digitalsketch/pipeline/status` | Poll pipeline status by `run_id` |
| 6 | `POST /digitalsketch/diagram/predictions/all` | Retrieve final structured predictions |

Full reference: [api.digitalsketch.ai/documentation](https://api.digitalsketch.ai/documentation)

---

## Repository Layout

```
.
|-- extract_predictions.py    # End-to-end pipeline client
|-- index.html                # Project homepage (GitHub Pages ready)
|-- requirements.txt          # Python dependencies
|-- .env.example              # Template for environment variables
|-- .gitignore
|-- sample/                   # Drop your P&ID images / PDFs here
|   |-- sample.jpg
|   |-- sample.pdf
|   `-- multipage.pdf
`-- output/                   # Predictions written here (gitignored)
```

---

## Troubleshooting

- **`ERROR: DIGITALSKETCH_API_KEY is not set`** - create `.env` from `.env.example` and add your key.
- **HTTP 401 / 403** - verify your API key at [api.digitalsketch.ai/documentation](https://api.digitalsketch.ai/documentation).
- **Pipeline timeout** - large PDFs may need a higher `PIPELINE_STATUS_TIMEOUT_SECONDS`.
- **"Image size too small" warning** - the corresponding PDF page could not be extracted; an empty predictions file is written and processing continues for the remaining pages.

---

## GitHub Pages

The included `index.html` is a standalone homepage that can be served as the project landing page.

1. Push this repo to GitHub.
2. In the repo, go to **Settings -> Pages**.
3. Set **Source** to `Deploy from a branch`, select `main` and `/ (root)`, then save.
4. Your homepage will be available at `https://<your-org>.github.io/<repo-name>/`.

---

## License

(c) 2025 DigitalSketch.ai, Inc. All rights reserved.
