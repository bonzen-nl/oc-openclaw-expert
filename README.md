# OpenClaw Expert — OpenClaw Skill

**Lokaal RAG-systeem (Retrieval-Augmented Generation) voor volledige OpenClaw documentatie.**

Semantische zoeken + lokale LLM = snelle, nauwkeurige antwoorden op vragen over OpenClaw. Volledig privacy-preserved, geen cloud afhankelijkheden.

---

## 🎯 Wat doet OpenClaw Expert?

OpenClaw Expert is een **Retrieval-Augmented Generation (RAG)** systeem dat:

- **Alle OpenClaw docs indexeert** — 99 markdown files van https://docs.openclaw.ai/
- **Semantische zoeking** — Vindt relevante docs via vector embeddings
- **Lokale LLM antwoorden** — Genereert antwoorden via Mistral (Ollama, geen cloud)
- **Geciteerde bronnen** — Elke antwoord bevat links naar bron-docs
- **Volledig privé** — Alles lokaal, geen external API calls

### 🔄 Workflow

```
User Question
    ↓
nomic-embed-text (local)
    ↓
ChromaDB Vector Search
    ↓
Top 5 matching docs
    ↓
Mistral Small 24B (local)
    ↓
Generated Answer + Source URLs
```

---

## 📦 Afhankelijkheden

### Systeemvereisten
- **Python:** 3.8+
- **Ollama:** https://ollama.ai (nodig voor LLM + embeddings)

### Ollama Models (automatisch gedownload)
```
nomic-embed-text:latest       # Embeddings (768-dim vectors)
mistral-small3.1:24b          # Language model (7B parameters)
```

### Python Dependencies

```
chromadb>=0.3.21              # Vector database
ollama>=0.1.0                 # Ollama Python client
beautifulsoup4>=4.11.0        # HTML parsing (voor docs scraping)
requests>=2.28.0              # HTTP requests
markdownify>=0.11.0           # Convert HTML to Markdown
python-dotenv>=0.20.0         # .env loading
```

### Hardware Requirements
- **RAM:** 8GB minimum (Mistral 24B = ~16GB)
- **Disk:** 20GB (embeddings + models cache)
- **CPU:** Modern processor (Intel i7+, Apple Silicon, AMD Ryzen+)

---

## ⚡ Quickstart

### 1. Installatie

```bash
# Clone de repository
git clone https://github.com/bonzen-nl/oc-openclaw-expert
cd oc-openclaw-expert

# Maak virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Installeer dependencies
pip install -r requirements.txt
```

### 2. Ollama Setup

```bash
# Download Ollama (één keer)
brew install ollama

# Start Ollama daemon
ollama serve

# In aparte terminal, pull models
ollama pull nomic-embed-text
ollama pull mistral-small3.1:24b
```

### 3. Configuratie

```bash
# Kopieer template
cp .env.example .env

# Instellingen (optioneel):
# OLLAMA_BASE_URL=http://127.0.0.1:11434
# CHROMADB_PATH=./index_data
```

### 4. Database Initialiseren

```bash
# Download + index OpenClaw docs (één keer, ~5 min)
python3 scripts/initialize_rag.py

# Output: ✅ 99 docs indexed, 99 vectors created
```

### 5. Test Query

```bash
python3 scripts/query_expert.py "Hoe configureer ik een agent?"

# Output:
# [Antwoord in Nederlands]
# 
# 📚 Bronnen:
# - https://docs.openclaw.ai/agents/configuration
# - https://docs.openclaw.ai/agents/overview
# ...
```

---

## 🚀 Gebruik

### Via CLI

```bash
# Query directly
python3 scripts/query_expert.py "Wat zijn de vereisten voor OpenClaw?"

# Output formatted (JSON)
python3 scripts/query_expert.py \
  "Hoe debug ik een skill?" \
  --format json

# Advanced: Custom model parameters
python3 scripts/query_expert.py \
  "Verklaar het RAG concept" \
  --temperature 0.5 \
  --top-k 10
```

### Via Python API

```python
from lib.expert import OpenClawExpert

expert = OpenClawExpert()

# Simple query
answer = expert.query("Wat is een skill?")
print(answer["text"])
print(answer["sources"])  # URLs

# Advanced
result = expert.query_advanced(
    question="OAuth2 implementatie",
    top_k=5,
    temperature=0.3
)
```

### Integratie met Software-Architect

```python
# Architect kan expert raadplegen
architect = SoftwareArchitectExpert()
context = architect.consult_expert(
    question="Wat zijn de best practices voor error handling?"
)
# → Context wordt toegevoegd aan complex task routing
```

---

## 🏗️ Projectstructuur

```
oc-openclaw-expert/
├── SKILL.md                          # Skill documentatie
├── README.md                         # Dit bestand
├── requirements.txt                  # Python dependencies
├── .env.example                      # Configuration template
├── .gitignore                        # Git security
├── LICENSE                           # MIT
├── config/
│   ├── ollama_config.json            # Model parameters
│   └── openclaw_expert_config.json   # Expert settings
├── scripts/
│   ├── initialize_rag.py             # Download + index docs
│   ├── query_expert.py               # CLI query interface
│   └── scrape_docs.py                # Fetch OpenClaw docs
├── lib/
│   └── expert.py                     # Core RAG engine
├── index_data/
│   └── openclaw_chroma_local.json    # Vector database (generated)
└── .venv/                            # Virtual environment
```

---

## 📚 Volledige Documentatie

- **[SKILL.md](./SKILL.md)** — Skill-definitie en workflows
- **[config/](./config/)** — Configuration options
- **[scripts/](./scripts/)** — Command-line tools

---

## 🔍 RAG System Details

### Vector Database (ChromaDB)

```json
{
  "99 documents": [
    {
      "id": "doc_001",
      "source_url": "https://docs.openclaw.ai/agents/overview",
      "embedding": [0.123, 0.456, ...],  // 768-dimensional vector
      "metadata": {"title": "...", "category": "agents"}
    }
  ]
}
```

### Query Process

1. **Embed Query** — User's vraag → nomic-embed-text → vector
2. **Search** — ChromaDB finds top-5 most similar docs
3. **Augment** — Combine docs + query for context
4. **Generate** — Mistral produces answer
5. **Cite** — Attach source URLs

### Performance

- **Query latency:** ~3-5 seconds (local, no network)
- **Embedding quality:** High (nomic-embed-text trained on OpenClaw-like docs)
- **Token efficiency:** ~200-500 tokens per response
- **Throughput:** ~10-15 queries/minute (single GPU/CPU)

---

## 🔐 Veiligheid & Privacy

### Local Processing
✅ Alle verwerking lokaal (geen cloud API calls)
✅ Documenten niet geupload
✅ Queries niet gelogd naar externe servers
✅ Volledig privacy-preserved

### Data Files
```bash
.env                           # NEVER commit (contains secrets)
index_data/                    # Safe (public OpenClaw docs)
.venv/                         # Never commit
```

---

## 🧪 Testing & Maintenance

### Rebuild Index

```bash
# If docs updated on docs.openclaw.ai
python3 scripts/initialize_rag.py --force

# Check index integrity
python3 scripts/query_expert.py --check-db
```

### Performance Check

```bash
# Benchmark query speed
python3 -c "
from lib.expert import OpenClawExpert
import time

expert = OpenClawExpert()
start = time.time()
result = expert.query('Test query')
print(f'Latency: {time.time() - start:.2f}s')
"
```

---

## 🐛 Troubleshooting

### Ollama Not Running
```bash
# Start daemon
ollama serve

# Or in background
nohup ollama serve > /tmp/ollama.log 2>&1 &
```

### Models Not Downloaded
```bash
ollama pull nomic-embed-text
ollama pull mistral-small3.1:24b

# Verify
ollama list
```

### "No documents found"
- Rebuild index: `python3 scripts/initialize_rag.py --force`
- Check ChromaDB: `ls -la index_data/openclaw_chroma_local.json`

### Slow Responses
- Increase RAM available to Ollama
- Check: `ollama ps` (running models)
- Reduce `top_k` parameter (fewer docs to search)

---

## 🔗 Sub-Projecten & Integraties

OpenClaw Expert is onderdeel van het **OpenClaw Skills Ecosystem**:

### Master Hub
- **[oc-overzicht](https://github.com/bonzen-nl/oc-overzicht)** — Centrale index

### Gerelateerde Skills
- **[oc-software-architect](https://github.com/bonzen-nl/oc-software-architect)** — Roept expert aan voor context
- **[oc-github-manager](https://github.com/bonzen-nl/oc-github-manager)** — Integreert expert in issue-beschrijvingen
- **[oc-server-status](https://github.com/bonzen-nl/oc-server-status)** — Health monitoring
- **[oc-ram-guardian](https://github.com/bonzen-nl/oc-ram-guardian)** — Memory optimization

### Integration Points

**Software-Architect consults Expert:**
```python
# Architect checks expert voor context
expert_answer = expert.query("Wat zijn de OpenClaw skill best practices?")
# → Used to inform complex decision-making
```

**GitHub Manager uses Expert:**
```python
# Auto-generate issue descriptions based on expert knowledge
issue_template = expert.query("Template voor feature request")
```

---

## 📊 Index Statistics

- **Total Documents:** 99
- **Total Vectors:** 99 (768-dim each)
- **Index Size:** ~2.5 MB
- **Vector Dimension:** 768 (nomic-embed-text)
- **Last Updated:** [Auto-synced weekly]

---

## 🚀 Deployment

### Standalone
```bash
# Run Expert service (listens on 8000)
python3 scripts/serve_expert.py
curl http://localhost:8000/query?q=What+is+a+skill?
```

### With OpenClaw
```bash
# Expert auto-available to software-architect
# No extra setup needed
```

---

## 📝 Licentie

MIT © 2026 Bonzen

---

## 📬 Ondersteuning

- **Issues:** [oc-openclaw-expert/issues](https://github.com/bonzen-nl/oc-openclaw-expert/issues)
- **Architecture Question:** Zie [oc-software-architect](https://github.com/bonzen-nl/oc-software-architect)

---

**Onderdeel van:** [OpenClaw Skills Suite](https://github.com/bonzen-nl/oc-overzicht)
