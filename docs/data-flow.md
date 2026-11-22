# Data Flow: What Is Sent to Servers

## High-level diagram

```mermaid
flowchart LR
    U(["👤 You"])

    subgraph LOCAL["💻 Your machine"]
        direction TB
        A["🤖 Agent client<br/>CLI / IDE extension<br/><i>holds conversation history</i>"]
        T["🛠️ Local tools<br/>read · grep · edit · shell · git · tests"]
        R[("📁 Your repo & filesystem<br/>source, configs, .env, logs")]
        L[("🗒️ Session logs<br/>on disk")]
    end

    subgraph VENDOR["☁️ Vendor cloud"]
        direction TB
        M["🧠 Model inference API<br/><i>stateless LLM</i>"]
        AU["🔑 Auth / entitlement"]
        TE["📊 Telemetry"]
        IX["🔎 Code index<br/><i>some products only</i>"]
    end

    subgraph THIRD["🌐 Third-party services"]
        direction TB
        X["Web search / fetch<br/>Remote MCP servers<br/>GitHub API"]
    end

    U -- "① prompt" --> A
    A == "② system prompt + instructions<br/>+ your messages + full history<br/>+ <b>tool results (file contents, output)</b>" ==> M
    M == "③ text or tool-call request" ==> A
    A -- "④ tool call" --> T
    T -- "⑤ reads / writes" --> R
    R -- "⑥ data" --> T
    T -- "⑦ tool result → back to ②" --> A
    A -- "⑧ final answer & diffs" --> U

    A -- "Ⓐ tokens, client version" --> AU
    A -. "Ⓑ usage events, errors" .-> TE
    A -. "Ⓒ file chunks / embeddings" .-> IX
    A -- "Ⓓ queries & arguments<br/>chosen by the model" --> X
    A -. "Ⓔ transcript" .-> L

    classDef local fill:#e8f5e9,stroke:#2e7d32,color:#1b5e20
    classDef cloud fill:#e3f2fd,stroke:#1565c0,color:#0d47a1
    classDef third fill:#fff3e0,stroke:#ef6c00,color:#e65100
    classDef model fill:#ede7f6,stroke:#5e35b1,color:#311b92,stroke-width:2px
    class A,T,R,L local
    class AU,TE,IX cloud
    class M model
    class X third
    linkStyle 1 stroke:#c62828,stroke-width:3px
    style LOCAL fill:#f6fff6,stroke:#2e7d32
    style VENDOR fill:#f5f9ff,stroke:#1565c0
    style THIRD fill:#fffaf2,stroke:#ef6c00
```

**Legend:** 🟥 thick red = the main path where your code leaves the machine · ━ thick = model round-trip · ┄ dotted = optional / product-dependent · 🟩 local · 🟦 vendor · 🟧 third-party

### Reading the diagram

**Main loop (numbers)** — repeats ②→⑦ until the task is done:

| # | Step | Leaves your machine? |
|---|------|----------------------|
| ① | You type a prompt into the agent | No |
| ② | Agent sends system prompt, repo instructions, full history **and all tool results so far** to the model | **Yes — this is where your code goes** |
| ③ | Model replies with text or a request to call a tool | (incoming) |
| ④ | Agent runs the tool locally (may ask your permission) | No |
| ⑤ | Tool reads or writes files / runs commands | No |
| ⑥ | Files, search matches or command output come back to the tool | No |
| ⑦ | Tool result is appended to history → next request goes back to ② | Sent on the next ② |
| ⑧ | Agent shows you the final answer and file changes | No |

**Side channels (letters)** — happen alongside the loop:

| | Channel | Notes |
|---|---------|-------|
| Ⓐ | Auth / entitlement | Login tokens, client version |
| Ⓑ | Telemetry | Usage events, errors; configurable in some products |
| Ⓒ | Code index | Only some products upload file chunks/embeddings |
| Ⓓ | Third-party tools | Whatever arguments the model chose (search queries, code snippets, etc.) |
| Ⓔ | Session logs | Local disk only |

> The model never touches your disk. It only sees what the agent client puts in step ②, and the most sensitive part of that is **tool results**: file contents, search matches and command output.

## Destinations

| Destination | Purpose | Typical content |
|-------------|---------|-----------------|
| **Model inference API** | Generate responses | Prompts, history, tool results (file contents, command output) |
| **Auth service** | Login, token refresh, entitlement checks | Account tokens, client version |
| **Telemetry / analytics** | Product metrics, error reports | Event names, durations, token counts, error stacks, client version, OS; *may* include snippets depending on settings |
| **Tool backends** | Web search, GitHub API, remote MCP servers | Queries and parameters chosen by the model |
| **Code indexing service** (some products) | Semantic search | Chunks or embeddings of repo files |

## Model request contents (per turn)

| Item | Sent? | Notes |
|------|-------|-------|
| System prompt + tool schemas | ✅ every turn | Vendor-authored |
| Your messages | ✅ | |
| Environment info | ✅ usually | cwd, OS, shell, date, sometimes git branch/remote |
| Repo instruction files | ✅ if present | `AGENTS.md`, `copilot-instructions.md`, etc. |
| Files the agent **read** | ✅ | Full or truncated content |
| `grep`/search **matches** | ✅ | Matching lines, which may include secrets |
| Shell command **output** | ✅ | `env`, `cat .env`, `printenv` would leak secrets |
| Files the agent **edited** | ✅ partially | Edit arguments contain old/new text |
| Files **never touched** | ❌ | Unless an indexing feature uploaded them |
| Open editor tabs / selection | ⚠️ IDE agents | Often added as implicit context |
| Images / screenshots | ✅ if attached | |

## What the server does with it

This is policy, not technology, and varies by product and plan. Questions to look up for your tool:

1. Is prompt/response data **retained**? For how long?
2. Is it used to **train models**? Is that opt-in or opt-out? Does it differ for business/enterprise plans?
3. Which **third-party model providers** see the data, and under what terms (zero data retention agreements)?
4. Where is data processed (data residency)?
5. Is telemetry configurable / can it be disabled?

See [privacy-and-security.md](privacy-and-security.md) for links.

## Key mental model

> **The model's view of your repo = the union of everything the agent's tools have returned during the session.**

If you wouldn't paste it into a chat window, don't let the agent `cat` it.
