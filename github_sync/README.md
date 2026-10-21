# Enable CIAO GitHub sync

This bundle installs a GitHub Actions workflow in your repository. On every code push it analyzes the repository, creates or updates `docs/CIAO_DOCUMENTATION.md`, `docs/CIAO_DOCUMENTATION.pdf`, and `docs/CIAO_ARCHITECTURE.svg`, then commits changes back to the same repository. The first push containing the workflow generates the first set of deliverables; later pushes refresh them. You can also run it manually from the Actions tab.

## Install

1. Extract the downloaded ZIP into the root of your repository, preserving its folder structure. It places the workflow at `.github/workflows/ciao-sync.yml` and the generator at `github_sync/ciao_github_sync.py`.
2. Commit and push those files. GitHub Actions will start on that push.
3. Open the repository's **Actions** tab to see progress and errors.

The workflow needs `contents: write` to commit generated files. If repository settings restrict workflow token permissions, enable write access for repository contents under **Settings → Actions → General → Workflow permissions**. The workflow uses the repository-scoped `GITHUB_TOKEN`; its generated commit does not recursively start another run.

## Model setup

The default provider is Ollama with `llama3.2:3b`. On GitHub-hosted Ubuntu, the workflow installs Ollama, starts it, and pulls the model each run. The first run may take several minutes. For quicker repeated runs, use a self-hosted GitHub runner with Ollama and the model already installed, and change `runs-on: ubuntu-latest` to `runs-on: self-hosted` in the workflow.

### Remote Ollama API

If using an Ollama server other than the runner's local service, add repository variable `CIAO_OLLAMA_HOST` with the API root, for example `https://ollama.example.com` (do not append `/api/chat`). Set `CIAO_MODEL` to an installed model such as `llama3.2:3b`. If the server requires bearer authentication, add repository secret `CIAO_API_KEY`. The API must be reachable from GitHub Actions over HTTPS; an Ollama instance bound only to `localhost` on your personal computer cannot be reached by GitHub-hosted runners.

### OpenAI or another OpenAI-compatible API

In **Settings → Secrets and variables → Actions**, add:

- Variable `CIAO_PROVIDER` = `openai-compatible`
- Variable `CIAO_MODEL` = the provider's model name, for example `gpt-4o-mini`
- Variable `CIAO_API_BASE_URL` = `https://api.openai.com/v1` (or the provider's compatible API root)
- Secret `CIAO_API_KEY` = your API key

API usage may incur provider charges. Keep API keys in GitHub Actions secrets, never in the workflow file or repository. The repository source excerpts are sent to the configured model API when the workflow runs.

## Website connection

The CIAO website provides the setup bundle for download. Add its workflow and generator files to the repository as described above; the website does not need to store a GitHub password or API key. Once the workflow is pushed, GitHub handles future syncs automatically. To change provider settings later, edit repository Actions variables/secrets; to change the runner or trigger, edit `.github/workflows/ciao-sync.yml`.
