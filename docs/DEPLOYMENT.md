# Deployment

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3000` | HTTP server port |
| `HOST` | `0.0.0.0` | HTTP server host |
| `AGENTFLOW_DB_PATH` | `agentflow.db` | SQLite database path |
| `ANTHROPIC_API_KEY` | — | API key for Claude (preflight + SDK) |

## System Service (Linux)

Create `/etc/systemd/system/agentflow.service`:

```ini
[Unit]
Description=agentflow
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/opt/agentflow
ExecStart=/opt/agentflow/agentflow
Environment=PORT=3000
Environment=HOST=0.0.0.0
Environment=AGENTFLOW_DB_PATH=/opt/agentflow/data/agentflow.db
Environment=ANTHROPIC_API_KEY=sk-ant-...
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now agentflow
```

## Single Binary

```bash
bun build src/index.ts --compile --target=bun --outfile=agentflow
```

This produces a self-contained executable at `./agentflow`. Copy it to your VM and run:

```bash
./agentflow
```

## Security

- API keys are **encrypted at rest** in SQLite via `credentials_encrypted`
- Credentials are **never exposed** through the API
- Tasks run in **isolated git worktrees** — agents can't touch `main`
- The `permissionMode: 'acceptEdits'` option on Claude Code auto-accepts file edits without prompts in automated mode
- Consider running agentflow behind a reverse proxy (nginx, Caddy) with TLS and optional auth
