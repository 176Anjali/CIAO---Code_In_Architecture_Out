# CIAO — Code In, Architecture Out

CIAO analyzes software using a local Llama model served by Ollama. Each successful analysis provides two separate deliverables: a PDF documentation report and an SVG architecture diagram. Source code is sent only to the configured Ollama host, and CIAO never executes submitted code.

## Inputs and outputs

Inputs:

- Public GitHub repository URL (Git must be available on `PATH`)
- ZIP archive up to 20 MB
- One supported source or documentation text file up to 128 KB

Outputs:

- **Documentation PDF:** executive summary, system purpose, components, data flows, technologies, entry points, risks, and open questions
- **Architecture diagram:** separate SVG with component boxes and labeled connections

Large repositories are sampled for speed and memory use: CIAO reads at most 100 files and 8,000 source characters, skipping common dependency, build, and version-control directories. A single file may be up to 128 KB, with up to 8,000 characters sent for analysis. The report and diagram are kept concise, and Ollama keeps the selected model warm for 10 minutes to speed up subsequent analyses.

## Requirements

- Python 3.11 or newer
- Ollama with a Llama model installed
- Git on `PATH` for GitHub URLs

## Windows setup

From the repository root in PowerShell:

```powershell
python -m pip install -r requirements.txt
ollama pull llama3.2:3b
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

Open <http://127.0.0.1:8000>. CIAO starts `ollama serve` automatically if the local Ollama CLI is available and the local server is down. If another Llama model is already installed, CIAO selects an available Llama model automatically; set `OLLAMA_MODEL` before launch to pin a specific model. For a remote Ollama server, set `OLLAMA_HOST` (default `http://127.0.0.1:11434`). CIAO does not try to start a remote server.

If Ollama is installed but the website still reports it offline, check that Ollama is running and that `OLLAMA_HOST` points to the same machine and port. The default model can be installed with `ollama pull llama3.2:3b`.

## GitHub Actions: keep a repository's docs in sync

The website's **Download GitHub sync setup** button provides a workflow and standalone generator for another repository. Copy the ZIP contents into that repository, preserving `.github/workflows/ciao-sync.yml` and `github_sync/ciao_github_sync.py`, then commit and push. The workflow runs on that push and subsequent pushes, generates `docs/CIAO_DOCUMENTATION.md`, `docs/CIAO_DOCUMENTATION.pdf`, and `docs/CIAO_ARCHITECTURE.svg`, and commits changed files back. It can also be started manually from the Actions tab. See [github_sync/README.md](github_sync/README.md) for setup details.

The default workflow installs Ollama and `llama3.2:3b` on a GitHub-hosted runner each run. A self-hosted runner with Ollama already installed can keep the model warm and avoid repeated downloads. A remote Ollama API must be reachable by the runner; `localhost` on a developer's laptop is not reachable from GitHub-hosted runners. OpenAI and other OpenAI-compatible APIs are supported by setting repository Actions variables `CIAO_PROVIDER=openai-compatible`, `CIAO_MODEL`, and `CIAO_API_BASE_URL`, and storing `CIAO_API_KEY` as an Actions secret. The repository source is sent to the selected model API. The workflow needs `contents: write`; generated commits use GitHub's repository-scoped token and do not recursively trigger another run.
