# Examples

Simplified, annotated payloads in the common OpenAI-style chat format. Real payloads vary by vendor and include more fields.

- [request-turn-1.json](request-turn-1.json) — initial request with system prompt, environment and user message
- [response-tool-call.json](response-tool-call.json) — model asks the client to read a file
- [request-turn-2.json](request-turn-2.json) — follow-up request: note the **file contents** in the `tool` message
