# Architecture

A coding agent is made of a few distinct pieces. Knowing which piece runs *where* is the key to understanding what leaves your machine.

## 1. The local client (runs on your machine)

The CLI binary or IDE extension. It:

- Reads your prompt.
- Holds the **conversation history** in memory (and often on disk, e.g. a session folder).
- Builds each request to the model: system instructions + history + tool definitions.
- **Executes tools locally** — reading files, running `grep`, editing files, running shell commands, calling `git`.
- Enforces permissions (e.g. "allow this command?" prompts, allowed directories, sandboxes).

The client has the same filesystem and network access as **your user account**, unless it is sandboxed.

## 2. The model (runs remotely)

A large language model hosted by the vendor or a model provider (OpenAI, Anthropic, Google, etc.), usually reached through the vendor's proxy/API.

- It is **stateless** between requests: each request contains the full context it needs (history is re-sent, sometimes with server-side prompt caching).
- It never touches your files. It can only *ask* the client to run a tool, by returning a structured "tool call".
- It sees **only** what is in the request.

## 3. Tools

Tools are functions the client advertises to the model as JSON schemas, e.g.:

| Tool | Runs where | What the model gets back |
|------|-----------|--------------------------|
| `view` / `read_file` | local | file contents (possibly truncated) |
| `grep` / `glob` | local | matching lines / file paths |
| `edit` / `create` | local | success/failure (sometimes a diff) |
| `bash` / `run_terminal` | local | stdout + stderr of the command |
| `web_fetch` / `web_search` | remote (vendor or 3rd party) | page text / search results |
| GitHub API tools | remote (GitHub) | issues, PRs, files from GitHub |

## 4. Context window

The model has a finite input size (tokens). The client decides what to include. When the conversation gets too long, clients typically **summarize or compact** older history — that summarization itself is usually another model call.

## 5. Optional extras

- **MCP servers** (Model Context Protocol): plug-in tool providers. Local MCP servers run as processes on your machine; remote ones are network services. Their output is fed back to the model like any other tool result.
- **Sub-agents**: the main agent can spawn helper agents, each making their own model calls with their own context.
- **Code search / embeddings indexes**: some products build a semantic index of your repo, either locally or by uploading chunks to a server. Check your product's docs — this is a significant difference between tools.
- **Instruction files**: `AGENTS.md`, `.github/copilot-instructions.md`, `CLAUDE.md`, `.cursorrules` etc. are read locally and **inserted into the prompt**.
