# SmartDocs Q&A — Intelligent Document Assistant

RAG-powered document Q&A system using Claude API + FAISS + FastAPI.

---

## Prerequisites

- Python 3.10 or 3.11
- pip
- API key: **Anthropic only** — embeddings run locally via `sentence-transformers` (no OpenAI needed)

---

## Quick Start (5 minutes)

### 1. Extract the zip and enter the folder

```bash
cd smartdocs
```

### 2. Create a virtual environment

```bash
python -m venv venv

# Mac/Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Set your API keys

```bash
cp .env.example .env
```

Open `.env` and fill in:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

### 5. Start the server

```bash
uvicorn app.main:app --reload --port 8000
```

### 6. Open the UI

Go to: **http://localhost:8000**

---

## Using the App

### Option A — Web UI (recommended)

1. Open http://localhost:8000
2. Drag-and-drop your PDF/DOCX/HTML/TXT files into the upload zone
3. Click **Upload & Index** — wait for "Done — N chunks indexed"
4. Type your question and press Enter

### Option B — API (Swagger)

Open http://localhost:8000/docs for the full interactive API.

### Option C — Pre-load sample documents

Two sample documents are included in `data/docs/`:
- `acme_policy_2024.txt` — Employee policy handbook
- `cloudstore_faq.txt` — Product FAQ

Ingest them from the terminal:

```bash
python scripts/ingest_local.py --path data/docs
```

Then open the UI and start asking questions like:
- *"How many days of annual leave do employees get?"*
- *"What is the refund policy for CloudStore Pro?"*
- *"What is the hotel reimbursement rate internationally?"*

---

## Running Evaluation

Make sure the server is running and documents are indexed, then:

```bash
python scripts/eval_simple.py
```

This runs 15 golden Q&A pairs and outputs:
- `eval/report/eval_results.csv` — per-golden scores
- `eval/report/metrics_chart.png` — bar chart + per-golden plot
- `eval/report/eval_report.md` — full markdown report

---

## Project Structure

```
smartdocs/
├── app/
│   ├── main.py          ← FastAPI app + all endpoints
│   ├── loader.py        ← PDF/DOCX/HTML/TXT loader
│   ├── chunker.py       ← Token-based text chunker
│   ├── embedder.py      ← OpenAI embeddings
│   ├── index.py         ← FAISS index build/load
│   ├── retriever.py     ← FAISS similarity search
│   ├── guardrails.py    ← Unsafe query detection
│   ├── generator.py     ← Claude answer generation
│   └── models.py        ← Pydantic models
├── static/
│   └── index.html       ← Full web UI
├── scripts/
│   ├── ingest_local.py  ← Standalone ingest script
│   └── eval_simple.py   ← Evaluation runner
├── data/
│   ├── docs/            ← Put your documents here
│   ├── faiss.index      ← Generated after ingest
│   └── chunks.json      ← Generated after ingest
├── eval/
│   ├── goldens/
│   │   └── goldens.json ← 15 golden Q&A pairs
│   └── report/          ← Evaluation outputs
├── requirements.txt
├── .env.example
└── README.md
```

---

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/` | Web UI |
| GET | `/health` | Index status + model info |
| POST | `/ingest` | Ingest from local path |
| POST | `/upload` | Upload files + ingest |
| POST | `/query` | Ask a question |
| GET | `/documents` | List indexed sources |
| DELETE | `/documents` | Clear index |
| GET | `/eval/sample` | Random chunks for golden writing |
| GET | `/docs` | Swagger UI |

---

## Supported File Types

| Format | Extension |
|---|---|
| PDF | `.pdf` |
| Word Document | `.docx` |
| HTML | `.html`, `.htm` |
| Plain Text | `.txt` |

---

## Troubleshooting

**"Index not built" error**
→ Upload documents first via the UI or run `python scripts/ingest_local.py`

**OpenAI API error during ingest**

**Anthropic API error during query**
→ Check your `ANTHROPIC_API_KEY` in `.env`

**SSL errors on corporate networks**
→ Add `NODE_TLS_REJECT_UNAUTHORIZED=0` to `.env` (Cognizant/enterprise proxy fix)

**Port 8000 already in use**
→ Use a different port: `uvicorn app.main:app --reload --port 8001`

---

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | required | Claude API key |
| `CLAUDE_MODEL` | `claude-sonnet-4-5` | Claude model to use |
| `FAISS_INDEX_PATH` | `data/faiss.index` | Index file location |
| `CHUNKS_PATH` | `data/chunks.json` | Chunk store location |
| `TOP_K` | `5` | Default chunks to retrieve |
| `MAX_TOKENS` | `1024` | Max tokens in answer |
