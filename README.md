# SearXNG LLM Tools

Local LLM integration with web search and persistent memory via llama.cpp and SearXNG.

## Modules

| Module | Description | Docs |
|--------|-------------|------|
| **Search with Memory** | Chat with web search and SQLite & Vector memory | [MEMORY.md](MEMORY.md) |
| **Deep Research** | Multi-phase agentic research with task anchoring | [DEEP_RESEARCH.md](DEEP_RESEARCH.md) |

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (for SearXNG + Valkey)
- llama.cpp server running with your model
- Python 3.10+

## Setup

### 1. Generate a secret key

```powershell
.\setup.ps1 -GenerateSecretKey
```

This generates a random key and writes it to `searxng/settings.yml`. You can also generate one manually:

```powershell
$bytes = New-Object byte[] 32
(New-Object Security.Cryptography.RNGCryptoServiceProvider).GetBytes($bytes)
-join ($bytes | ForEach-Object { "{0:x2}" -f $_ })
```

Then paste the value into `searxng/settings.yml` under `server.secret_key`.

### 2. Configure search engines

Edit `searxng/settings.yml` to enable the engines you want. The `engines:` section controls which upstream search services SearXNG queries. The default configuration includes:

| Engine | Module | Description |
|--------|--------|-------------|
| Google | `google` | General web search |
| DuckDuckGo | `duckduckgo` | Privacy-focused web search |
| Bing | `bing` | General web search |
| Wikipedia | `wikipedia` | Encyclopedia articles |
| GitHub | `github` | Code repositories |
| Stack Overflow | `stackexchange` | Programming Q&A (uses `search_site: stackoverflow.com`) |
| arXiv | `arxiv` | Scientific preprints |
| Google Scholar | `google_scholar` | Academic papers |

To add or remove engines, edit the `engines:` list in `searxng/settings.yml`. Each entry needs at minimum:

```yaml
engines:
  - name: example
    engine: engine_module_name
    shortcut: ex
    disabled: false
```

See the [SearXNG engine list](https://docs.searxng.org/user/configured_engines.html) for all available engines and their module names.

### 3. Start the containers

```powershell
.\setup.ps1 -Start
# Or directly:
docker compose up -d
```

SearXNG will be available at `http://localhost:8888`. The container listens internally on port 8080 and is mapped to host port 8888.

### 4. Verify the setup

```powershell
.\setup.ps1 -Test
# Or manually:
curl "http://localhost:8888/search?q=test&format=json"
```

### 5. Start llama.cpp

```bash
llama-server -m /path/to/model.gguf --ctx-size 16384 --port 8080
```

### 6. Install Python dependencies

```bash
pip install openai requests instructor pydantic
```

## Quick Start

```bash
# Simple conversational search
python simple_llm_search.py --llm http://localhost:8080 --searxng http://localhost:8888

# Full search with vector memory
python llm_memory_bridge.py --llm-url http://localhost:8080 --searxng-url http://localhost:8888

# Deep multi-step research
python deep_research.py --mode thorough "your research question"
```

## Container Management

```powershell
.\setup.ps1 -Start      # Start containers
.\setup.ps1 -Stop       # Stop containers
.\setup.ps1 -Restart    # Restart containers
.\setup.ps1 -Logs       # Follow container logs
.\setup.ps1 -Status     # Show container status
.\setup.ps1 -Test       # Test JSON API endpoint
```
