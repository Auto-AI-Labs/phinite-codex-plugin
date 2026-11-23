# Phinite plugin for Codex and ChatGPT

Build, ship, run and debug **Phinite agent graphs** from Codex or ChatGPT. The plugin connects to the hosted
**Phinite MCP server** (`https://mcp.dev.phinite.ai/mcp`, streamable HTTP) and adds the Phinite playbook as skills. It uses the portable
Agent Plugins layout (`plugin.json` + `mcp.json` + `skills/` at the root) with OpenAI's presentation fields under
`extensions["com.openai"]`. It is generated from the Claude plugin, so both behave the same.

## Install

- **Codex (marketplace):** `codex plugin marketplace add Auto-AI-Labs/phinite-codex-plugin` (this repo has
  `.agents/plugins/marketplace.json` at its root), then
  install **phinite** from the plugin directory in the Codex or ChatGPT desktop app.
- **Local:** copy this folder to `~/.codex/plugins/phinite`, add it to `~/.agents/plugins/marketplace.json`
  (`"source": {"source": "local", "path": "./phinite"}`, path relative to that marketplace's root), and restart
  the app.
- **ChatGPT (web or app):** chatgpt.com/plugins → **+** (New Plugin) → drop a zip of this repo (`plugin.json` at the
  zip's root; `tests/` and `.agents/` can be left out) → **Add plugin**. That installs the skills and the `phinite`
  server together; then connect it to sign in. Only the server, without the skills: **Add custom MCP server** →
  URL `https://mcp.dev.phinite.ai/mcp`, OAuth.
  - In ChatGPT, steps that need files on your disk (deploying a project folder, sending knowledge files or a
    `.env`) only work where the project script can run on your machine, i.e. in Codex. Designing, building with
    `upsert_tool` / `set_flow_graph`, testing, logs and shipping work anywhere.

You need Python 3.9+ for the project script. No Phinite token is stored on your machine.

To use another server (production or a local one), change `url` in `mcp.json`.

## Sign in

The client signs in with OAuth on install or first use (the server supports discovery and dynamic client
registration, so there is nothing to configure). In Codex CLI: `codex mcp login phinite`. A browser opens on the
Phinite login (Google, GitHub or email); pick the org and the workspace.

## Update

- **Codex:** Codex has no documented plugin auto-update or update command. Run the `codex plugin marketplace add`
  command again to fetch the latest version, reinstall **phinite** from the plugin directory, and start a new
  session.
- **ChatGPT (uploaded zip):** upload a zip of the latest version: **Plugins → Phinite → Edit Plugin**.
- **Workspace admins (team GitHub marketplace):** the marketplace checks for updates daily; **Admin → Plugins →
  Marketplaces → Sync now** updates right away.
- **Local copy:** `git pull` in the copy, then restart the app.

## Use

Codex and ChatGPT have no slash commands for plugins, so each Claude command is a skill. Ask for the task, or name
the skill:

| Skill | What it does |
|---|---|
| `phinite-login` | Sign in, check the sign-in, or switch workspace |
| `phinite-new-use-case` | Scaffold a use-case project and design its graphs with you |
| `phinite-deploy-project` | Dry run, then deploy: tools upserted (published when changed), graphs created or updated |
| `phinite-test-graph` | Chat or run the graph autonomously, read the log, diagnose |
| `phinite-ship-graph` | Publish → build → environment → trigger or Chat API → API key → call it from outside |
| `phinite-explain-run` | Explain a run: tool calls, inputs, outputs, variables, errors |
| `phinite-evaluate-graph` | Evaluate with LLM judges: cases, metrics, failures and fixes |
| `phinite-build-use-case` | A whole use case end to end (the Claude plugin's `phinite-builder` agent) |

## What's inside

- **Skills** (the same as the Claude plugin's, from the Phinite playbook): `phinite-design`, `phinite-ship`,
  `phinite-debug`, `phinite-api` and `phinite-project`, plus `phinite-quality` for evaluations (LLM judges,
  custom metrics, production scoring).
- **A local project script**, `skills/phinite-project/scripts/phinite_project.py` (Python 3.9+, standard library
  only). It scaffolds use-case folders from `templates/`, runs `graph_spec.py` on your machine, bundles the project
  and sends files through the server's upload links. The hosted server can't read your disk.
- `tests/`: the offline tests for that script: `python -m unittest tests/test_offline.py`.

The MCP tools and the rules they apply are listed in the
[phinite-mcp README](https://github.com/Auto-AI-Labs/phinite-mcp#tools).

## Security notes

- **No secrets on disk or in the chat.** The client stores and refreshes the OAuth token; the assistant never sees
  it. Env values go from your `.env` straight to an upload link and never pass through the chat.
- **Upload links** are valid for 15 minutes and for one purpose. The project script accepts only https links on
  `*.phinite.ai` (or the `PHINITE_MCP_URL` host, or localhost) with the `/u/<ticket>` path, so a link from a file,
  web page or message is refused.
- **Hidden files** (`.env`, `.git`…) in folders are never sent, and `KNOWLEDGE` files must be inside the project.
- **`.env` values never reach the chat.** A hook refuses file reads and shell commands that would print a `.env`
  (`cat .env`, `grep KEY .env`…) in every task, Phinite or not; names-only commands (`cut -d= -f1 .env`) and
  writing to a `.env` still work. Set `PHINITE_ALLOW_ENV_READ=1` before starting the client to turn it off.
- **Env variables are workspace-wide**: prefix a use case's own secrets (`MYUC_SLACK_TOKEN`), and the assistant asks
  before overwriting a name that already exists.
- **`graph_spec.py` is code**: `deploy` runs it on your machine. Deploy only projects you wrote or trust.
