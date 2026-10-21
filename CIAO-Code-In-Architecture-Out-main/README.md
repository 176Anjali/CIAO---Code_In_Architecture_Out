# CIAO — Code In, Architecture Out

CIAO analyzes a software repository and produces two separate deliverables: a PDF documentation report and a rendered SVG architecture diagram. This fresh implementation uses a small model served by Ollama on your own computer. It does not use an OpenAI API key or send source code to a hosted language model.

The web app accepts a public GitHub repository URL or a ZIP archive. It collects a bounded set of source files and asks Ollama for evidence-based documentation and architecture relationships. CIAO draws the diagram as a separate image; diagram markup is not included in the report. CIAO reads repository files as text and does not execute project code.

## Requirements

- Python 3.11 or newer
- Ollama for Windows
- Git on `PATH` when analyzing a GitHub URL

## Windows setup

From the repository root in PowerShell or Command Prompt:

```powershell
python -m pip install -r requirements.txt
ollama pull llama3.2:3b
python -m uvicorn app:app --host 127.0.0.1 --port 8000
```

Open <http://127.0.0.1:8000>. The Ollama desktop app normally starts its local server automatically. If the website says Ollama is offline, run `ollama serve` in another terminal and refresh the page.

To select a different installed model, set `OLLAMA_MODEL` before starting CIAO. To use a non-default Ollama server, set `OLLAMA_HOST` (default `http://127.0.0.1:11434`).

## What the report contains

- PDF documentation: executive summary, system purpose, components, data flows, technologies, entry points, risks, and open questions
- A separate SVG architecture diagram with component boxes and connection arrows

The analyzer limits file count, individual file size, and total source context. It skips common dependency, build, and version-control directories. Large repositories may be represented by a source sample rather than every file.

## Command-line launch

The original nested `CIAO-system/main.py` path remains as a small launcher for the new web app:

```powershell
python CIAO-system/main.py
```

This starts the same local site at <http://127.0.0.1:8000>.
