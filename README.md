# How Coding Agents Work With Your Local Repo

A plain-language guide to what an AI coding agent (e.g. GitHub Copilot CLI, Copilot in VS Code agent mode, Claude Code, Cursor, Aider) actually does when you point it at a local repository — **what runs on your machine, what gets sent to remote servers, and how you can verify it yourself.**

> ⚠️ This is a vendor-neutral conceptual guide. Exact behavior, data retention and telemetry differ by product, plan and settings. Always check your vendor's official privacy/trust documentation (links in [docs/privacy-and-security.md](docs/privacy-and-security.md)).

## TL;DR

- The **agent runs locally** (a CLI or IDE extension). The **model runs remotely** (an LLM API).
- The model **cannot see your disk**. It only sees what the local agent puts into the request.
- The agent works in a **loop**: send context → model replies with text or a *tool call* → agent executes the tool locally → sends the result back → repeat.
- Therefore, **anything a tool reads or prints can be sent to the server**: file contents, `grep` hits, command output, test logs, error messages, environment details — including secrets if they appear there.
- Your whole repo is **not** uploaded wholesale (for typical agents), but over a session a large portion of it may be sent piece by piece.
- Separate from model traffic, there may be **telemetry** (usage events, errors) and **auth** traffic.

## The core loop in one picture

```
 Your machine                                         Vendor cloud
┌───────────────────────────────────────────┐        ┌─────────────────────────┐
│  You ──prompt──▶  Agent (CLI / IDE ext.)   │        │                         │
│                    │                       │ HTTPS  │   LLM inference API     │
│                    │ 1. build request ─────┼───────▶│   (system prompt +      │
│                    │                       │        │    history + tool       │
│                    │ 2. response  ◀────────┼────────│    results)             │
│                    │    "call tool X"      │        │                         │
│                    ▼                       │        └─────────────────────────┘
│   Local tools: read file, grep, edit,      │
│   run shell cmd, git, tests ...            │        ┌─────────────────────────┐
│                    │                       │        │ Telemetry / auth / MCP  │
│                    └─ 3. tool output ──────┼──┐     │ servers (optional)      │
│                       appended to history  │  └────▶│                         │
│                       → back to step 1     │        └─────────────────────────┘
└───────────────────────────────────────────┘
```

## Contents

| Doc | What it covers |
|-----|----------------|
| [docs/architecture.md](docs/architecture.md) | Components: client, model, tools, context window, MCP servers |
| [docs/agent-loop.md](docs/agent-loop.md) | Step-by-step walkthrough of a real task |
| [docs/data-flow.md](docs/data-flow.md) | **Exactly what data is sent, and when** |
| [docs/privacy-and-security.md](docs/privacy-and-security.md) | Secrets, content exclusion, retention, risks, mitigations |
| [docs/inspecting-traffic.md](docs/inspecting-traffic.md) | How to observe the traffic yourself with a proxy |
| [examples/](examples/) | Annotated example request/response payloads |

## Contributing

Corrections and product-specific notes are welcome — please cite official documentation in PRs.

## License

[MIT](LICENSE)
