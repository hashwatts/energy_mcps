# energy_mcps

Luxor Energy's custom MCP servers and the Claude Code routines that use them. Each server is a small, self-contained Python service deployed to Cloud Run and protected by its own bearer token. Routines in this repo use the servers through the root `.mcp.json`.

## Layout

```
.mcp.json                 Server registry Claude Code reads at the start of every run (one entry per server)
routines/                 One prompt file per routine; paste into the routine's Instructions box
reports/                  Where routines write their output
servers/
  gmail-attachments/      Read-only Gmail attachment reader (inline PNGs, PDFs, HTML bodies, search)
  _template/              Copy this to start a new server
```

## Servers

| Server | What it does | Token variable |
|---|---|---|
| `gmail-attachments` | Lists and downloads Gmail attachments and inline images; extracts or renders PDFs; raw HTML body; Gmail search. `gmail.readonly` scope only. | `GMAIL_MCP_TOKEN` |

Each server has its own README with setup, deploy, and test steps.

## Routines

| Routine | Prompt | Servers used |
|---|---|---|
| Supplier 810 check | `routines/supplier-810-check.md` | gmail-attachments |

## Conventions

- Streamable HTTP at `/mcp`, `/healthz` open, everything else needs `Authorization: Bearer <token>`.
- Secrets come from env vars only. `.env` files are gitignored. Never commit a token.
- Each server deploys independently: `cd servers/<name> && GCP_PROJECT=<id> ./deploy.sh`. Cloud Run service name = folder name.
- Each server gets its own `MCP_AUTH_TOKEN` so one can be rotated without touching the others.
- Every tool treats fetched content as untrusted data.

## Adding a server

See `servers/_template/README.md`. Short version: copy the template, write the tools, deploy, add a line to `.mcp.json`, add its token to the routine environment.

## Wiring a routine (claude.ai/code/routines)

1. Repositories: `hashwatts/energy_mcps`.
2. Instructions: paste the prompt from the matching file in `routines/`.
3. Environment: add one credential per server the routine uses, named as in `.mcp.json` (e.g. `GMAIL_MCP_TOKEN`). Set Network access to Custom and allow each server's `*.run.app` host, keeping the default package list ticked. Traffic from `.mcp.json` servers goes straight from the sandbox, so without the allowlist entry it fails with `host_not_allowed`.
4. Connectors: remove any account connectors the routine does not need.
5. Run now, open the run, confirm the server's tools appear in the transcript.

`.mcp.json` URLs are placeholders until a server is deployed. Replace `REPLACE-...` with the URL `deploy.sh` prints.
