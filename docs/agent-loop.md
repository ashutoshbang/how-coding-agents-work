# The Agent Loop: A Walkthrough

Task: *"Fix the failing test in `utils/date.test.js`."*

### Turn 1 — your prompt

Client sends to the model:

- System prompt (agent's built-in instructions, tool definitions)
- Environment info (OS, current directory, maybe git branch/status)
- Repo instruction files if present (e.g. `AGENTS.md`)
- Your message

Model replies: *tool call* `bash("npm test -- utils/date.test.js")`

### Turn 2 — run tests locally

Client (maybe after asking your permission) runs the command **on your machine**.
The **full stdout/stderr** — test names, stack traces, file paths, any logged values — is appended to history and sent back.

Model replies: tool call `view("utils/date.js")`

### Turn 3 — read source

Client reads the file locally and sends its **contents** to the model.

Model replies: tool call `edit("utils/date.js", old, new)`

### Turn 4 — edit

Client applies the edit locally. Sends back "edit applied".

Model replies: tool call `bash("npm test -- utils/date.test.js")` → passes → final text answer to you.

## What left your machine in this session

1. Your prompt
2. Environment metadata
3. Test output (including stack traces and anything the tests printed)
4. Full contents of `utils/date.js`
5. The edit (old and new strings)
6. Second test run output
7. Telemetry events (e.g. "session started", "tool X used", token counts) — product dependent

What did **not** leave: every other file in the repo, your `.git` objects, files that were never read or grepped.

> Note: each turn re-sends the **entire** history so far, so the same file content is transmitted multiple times within a session (often deduplicated via prompt caching on the server side).
