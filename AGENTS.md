# AGENTS.md

Guidance for AI agents working in this repository.

## Cursor Cloud specific instructions

### Product

Odysseus is a single self-hosted FastAPI app with a static ES-module frontend (`static/js/`). There is no separate frontend build step for the main UI.

### One-time VM packages (not in update script)

On Debian/Ubuntu cloud VMs, create the venv once:

```bash
sudo apt-get install -y python3.12-venv tmux
```

`tmux` is optional for the web UI but required for Cookbook download/serve on native Linux.

### Dependency refresh (automatic on session start)

The cloud update script installs Python deps into `./venv`. After it runs:

```bash
source venv/bin/activate
```

If `data/auth.json` does not exist yet, run `python setup.py` once (non-interactive; prints a temporary admin password).

### Running the app (native dev — preferred in Cloud Agent VMs)

```bash
source venv/bin/activate
python -m uvicorn app:app --host 127.0.0.1 --port 7000
```

Use a tmux session for long-running servers, e.g. session name `odysseus-dev`.

First login: admin user from `setup.py` output, or set `ODYSSEUS_ADMIN_USER` / `ODYSSEUS_ADMIN_PASSWORD` before setup.

### Docker Compose (optional full stack)

For parity with production (ChromaDB, SearXNG, ntfy), use `docker compose up -d --build` after copying `.env.example` to `.env`. Odysseus waits for SearXNG health before accepting traffic. This path needs Docker installed on the VM; it is not part of the minimal update script.

### Checks

| Check | Command |
|-------|---------|
| Tests | `source venv/bin/activate && python -m pytest` |
| Python syntax | `python -m py_compile app.py routes/*.py` |
| JS syntax (changed files) | `node --check static/js/<file>.js` |

There is no project-wide Ruff/mypy config; follow `CONTRIBUTING.md`.

### Gotchas

- **ChromaDB / SearXNG**: Not required for `pytest` or basic UI login; memory/search features degrade or need external config when those services are down.
- **LLM**: Chat/agent flows need a configured model in Settings or env; the shell and auth still work without one.
- **Re-running setup**: `python setup.py` is safe; it skips existing `data/auth.json` and `.env`.
- **Port**: Default `7000`; override with `APP_PORT` in `.env`.

See `README.md` and `CONTRIBUTING.md` for feature-level setup (Ollama, email, CalDAV, optional `requirements-optional.txt`).
