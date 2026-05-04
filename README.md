# ContextCut

**Stop wasting tokens. Inject only what matters.**

ContextCut is a transparent semantic RAG proxy for Ollama, OpenClaw, and any OpenAI-compatible local LLM endpoint. Drop it in front of your LLM — zero application changes required.

---

## Quick Install

```bash
curl -fsSL https://raw.githubusercontent.com/StevoKeano/ContextCut/main/install.sh -o /tmp/install.sh
chmod +x /tmp/install.sh
bash /tmp/install.sh
```

---

## Why ContextCut

Most RAG implementations stuff your entire knowledge base into every prompt. ContextCut uses vector similarity to inject **only the chunks that are actually relevant** to each query — and skips injection entirely when nothing scores above your threshold.

| Query                      | Without ContextCut       | With ContextCut               |
| -------------------------- | ------------------------ | ----------------------------- |
| "What are the guardrails?" | 3,000+ tokens (all docs) | 806 tokens (1 relevant chunk) |
| "Explain quantum physics"  | 3,000+ tokens (junk)     | ~5 tokens (nothing relevant)  |

**Result: 50–90% token reduction on real workloads.**

![Dashboard](dashboard.png)

> Note: This is the local AI context optimizer for Ollama + Qdrant. There is another unrelated repo with a similar name.

---

## How it works

1. Request arrives at the proxy
2. User message is embedded and compared against your knowledge base
3. Only chunks above your relevance threshold are retrieved
4. Relevant chunks are prepended to the system prompt
5. Enriched request is forwarded to your LLM endpoint
6. Response returned to caller unchanged

---

## Requirements

- Python 3.10+
- [Voyage AI API key](https://dash.voyageai.com) (free tier works)
- [Qdrant](https://qdrant.tech) running locally or on your LAN
- Ollama or any OpenAI-compatible LLM endpoint

---

## Install (macOS / Linux)

```bash
curl -fsSL https://raw.githubusercontent.com/StevoKeano/ContextCut/main/install.sh -o /tmp/install.sh
chmod +x /tmp/install.sh
bash /tmp/install.sh
```

The installer will ask for:

- Voyage AI API key
- Ollama host and port
- Qdrant host and port
- Path to your markdown knowledge base
- Proxy and dashboard ports

On macOS, services are registered as launchd agents and start automatically on login.
On Linux, a `start.sh` script is generated.

---

## Manual Install

```bash
git clone https://github.com/StevoKeano/ContextCut
cd ContextCut
python3 -m venv venv
source venv/bin/activate
pip install voyageai qdrant-client watchdog tiktoken

export VOYAGE_API_KEY=your-key-here
export CONTEXTCUT_UPSTREAM=http://localhost:11434
export CONTEXTCUT_QDRANT_HOST=localhost
export CONTEXTCUT_KB_DIR=/path/to/your/markdown/files

python ingest.py
python qdrant_proxy.py
```

---

## Configuration

All settings via environment variables:

| Variable                    | Default                  | Description                            |
| --------------------------- | ------------------------ | -------------------------------------- |
| `VOYAGE_API_KEY`            | _(required)_             | Voyage AI API key                      |
| `CONTEXTCUT_UPSTREAM`       | `http://localhost:11434` | Ollama or OpenAI-compatible endpoint   |
| `CONTEXTCUT_QDRANT_HOST`    | `localhost`              | Qdrant host                            |
| `CONTEXTCUT_QDRANT_PORT`    | `6333`                   | Qdrant port                            |
| `CONTEXTCUT_COLLECTION`     | `contextcut`             | Qdrant collection name                 |
| `CONTEXTCUT_KB_DIR`         | `~/contextcut/knowledge` | Knowledge base directory (ingest only) |
| `CONTEXTCUT_PROXY_PORT`     | `18788`                  | Proxy listen port                      |
| `CONTEXTCUT_DASHBOARD_PORT` | `18787`                  | Dashboard port                         |
| `CONTEXTCUT_CTX_LIMIT`      | `8192`                   | Model context window (for % display)   |
| `CONTEXTCUT_TOP_K`          | `5`                      | Max chunks to retrieve                 |
| `CONTEXTCUT_MIN_SCORE`      | `0.30`                   | Minimum relevance threshold (0.0–1.0)  |

---

## Ingest Tool

```bash
python ingest.py                         # one-shot ingest all .md files
python ingest.py --watch                 # ingest then watch for file changes
python ingest.py --query "guardrails"    # test semantic search
python ingest.py --clear                 # wipe collection and start fresh
```

Backup files are skipped automatically.

> **Note:** Voyage AI free tier has rate limits. ContextCut handles this automatically.

---

## Dashboard

Open `http://localhost:18787` to see:

- Real-time context usage bar (green → yellow → red)
- Per-request token counts before and after injection
- Qdrant hit sources and relevance scores
- Peak token usage across the session

---

## Tuning MIN_SCORE

```bash
python ingest.py --query "your typical query"
```

- Above `0.35` — highly relevant, inject
- `0.20–0.35` — tangentially related, use with caution
- Below `0.20` — noise, skip

Start at `0.30` and adjust for your domain.

---

## Want More?

ContextCut-PRO adds a split-panel live dashboard, integrated streaming chat, per-message token analytics, and priority support.

[Upgrade to PRO →](https://5984630877416.gumroad.com/l/ContextCut-Pro)

---

## License

MIT License — free for personal, educational, and open source use.

Commercial deployments require a Pro License. See above.
