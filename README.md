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

Use these direct files from this repository:

- [GitHub Actions workflow (`ciao-sync.yml`)](https://raw.githubusercontent.com/176Anjali/CIAO---Code_In_Architecture_Out/main/github_sync/ciao-sync.yml)
- [Standalone generator (`ciao_github_sync.py`)](https://raw.githubusercontent.com/176Anjali/CIAO---Code_In_Architecture_Out/main/github_sync/ciao_github_sync.py)

### Install in the repository you want to keep synchronized

1. Save the workflow file in the target repository at `.github/workflows/ciao-sync.yml` (create the folders if needed).
2. Save the generator file at `github_sync/ciao_github_sync.py`.
3. Commit and push both files. That first push triggers the workflow; future pushes regenerate the deliverables. You can also run it from that repository's **Actions** tab with **Run workflow**.
4. The generated files are committed under `docs/`: `CIAO_DOCUMENTATION.md`, `CIAO_DOCUMENTATION.pdf`, and `CIAO_ARCHITECTURE.svg`.

### GitHub repository settings

- In **Settings → Actions → General**, make sure Actions are enabled and allowed for the repository.
- Under **Workflow permissions**, allow read and write access for repository contents. The workflow requests only `contents: write` so it can commit the generated files. Organization policies may prevent repository owners from changing this setting; an organization administrator may need to enable it.
- No personal access token is needed. The workflow uses the automatic, repository-scoped `GITHUB_TOKEN`.

### Select the model API

The default is Ollama with `llama3.2:3b`. On GitHub-hosted Ubuntu, the workflow installs Ollama and downloads the model each run; the first run can take several minutes. For faster repeated runs, install Ollama and the model on a self-hosted runner, then change `runs-on: ubuntu-latest` to `runs-on: self-hosted` in the workflow.

For a remote Ollama server, add the repository Actions variable `CIAO_OLLAMA_HOST` with the API root (for example `https://ollama.example.com`, without `/api/chat`). A server bound only to your laptop's `localhost` cannot be reached by GitHub-hosted runners. If the remote endpoint requires bearer authentication, store its token in the Actions secret `CIAO_API_KEY`.

For OpenAI or another OpenAI-compatible API, open **Settings → Secrets and variables → Actions** and add these repository variables:

- `CIAO_PROVIDER` = `openai-compatible`
- `CIAO_MODEL` = a model name supported by your provider
- `CIAO_API_BASE_URL` = `https://api.openai.com/v1` for OpenAI, or the compatible API root for your provider

Add `CIAO_API_KEY` as a **secret**, never as a variable or in the workflow file. API use may incur provider charges. Repository source excerpts are sent to the configured model API. To switch back to default Ollama, remove or change `CIAO_PROVIDER` and `CIAO_API_BASE_URL` and remove the API key if not needed.

The workflow's generated commit does not start a duplicate run. See [GitHub's GITHUB_TOKEN documentation](https://docs.github.com/en/actions/concepts/security/github_token) for details.
