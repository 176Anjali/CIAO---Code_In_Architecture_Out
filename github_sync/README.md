# Install CIAO GitHub sync

Download the two files from the CIAO repository:

- [Workflow YAML](https://raw.githubusercontent.com/176Anjali/CIAO---Code_In_Architecture_Out/main/github_sync/ciao-sync.yml)
- [Generator Python file](https://raw.githubusercontent.com/176Anjali/CIAO---Code_In_Architecture_Out/main/github_sync/ciao_github_sync.py)

In the repository you want to synchronize, save them to these paths:

- `.github/workflows/ciao-sync.yml`
- `github_sync/ciao_github_sync.py`

Commit and push both files. The first push runs the workflow, and later pushes update `docs/CIAO_DOCUMENTATION.md`, `docs/CIAO_DOCUMENTATION.pdf`, and `docs/CIAO_ARCHITECTURE.svg`. You can also run it manually from the repository's Actions tab.

The Markdown and PDF reports cover the project overview, purpose and scope, features and user workflows, architecture and component responsibilities, runtime/data flow, technologies and entry points, setup/configuration, testing, and source-supported limitations. The generator asks the model to cite concrete paths, symbols, commands, and dependencies. If a draft is too short or generic, it requests one rewrite; if the configured model still cannot provide adequate detail, the workflow reports an actionable error instead of committing another one-line report. A larger model may be needed for repositories whose details are difficult for the default `llama3.2:3b` model to infer.

## GitHub settings

1. Open **Settings → Actions → General** and make sure Actions are enabled and permitted.
2. Under **Workflow permissions**, allow read and write access for repository contents. The workflow requests `contents: write` so it can add or update the deliverables. An organization policy may require an organization administrator to change this setting.
3. No personal access token is required. GitHub supplies the workflow's repository-scoped `GITHUB_TOKEN` automatically.

## Default provider: Ollama

The default is Ollama with `llama3.2:3b`. A GitHub-hosted runner installs Ollama and downloads the model on each run, so the first run may take several minutes. For faster repeat runs, configure a self-hosted runner with Ollama and the model installed, and change `runs-on: ubuntu-latest` to `runs-on: self-hosted` in the workflow.

For a remote Ollama API, add repository variable `CIAO_OLLAMA_HOST` with the API root, such as `https://ollama.example.com` (do not add `/api/chat`). It must be reachable by the runner. An Ollama process listening only on your laptop's `localhost` cannot be reached by GitHub-hosted runners. If the remote Ollama server requires bearer authentication, save its token as the Actions secret `CIAO_API_KEY`.

## OpenAI or compatible API

Under **Settings → Secrets and variables → Actions**, add:

- Variable `CIAO_PROVIDER` = `openai-compatible`
- Variable `CIAO_MODEL` = a model name supported by the API provider
- Variable `CIAO_API_BASE_URL` = `https://api.openai.com/v1` for OpenAI, or the compatible API root
- Secret `CIAO_API_KEY` = your API key

Keep API keys in secrets, never in the workflow file or repository. API usage may incur charges. Repository source excerpts are sent to the selected model API. To use the default local-on-runner Ollama setup again, remove `CIAO_PROVIDER`, `CIAO_API_BASE_URL`, and `CIAO_API_KEY`.
