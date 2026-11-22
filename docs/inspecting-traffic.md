# Inspecting Traffic Yourself

The most reliable way to know what an agent sends is to look.

## Option A: Session logs (easiest)

Most agents keep local logs or session transcripts that contain the exact tool calls and results. Look in the product's config/state directory (e.g. `~/.copilot/`, `~/.claude/`, VS Code output panel → the extension's log channel). Many CLIs also support a `--log-level debug` or similar flag.

## Option B: HTTPS proxy with mitmproxy

> Only do this on your own machine and accounts. Some products use certificate pinning and will refuse the proxy.

```bash
pip install mitmproxy
mitmweb --listen-port 8080          # opens a web UI on :8081
```

Trust the mitmproxy CA (`~/.mitmproxy/mitmproxy-ca-cert.pem`) for the agent's runtime, then route it through the proxy:

```bash
# Node.js-based agents
export HTTPS_PROXY=http://127.0.0.1:8080
export NODE_EXTRA_CA_CERTS=~/.mitmproxy/mitmproxy-ca-cert.pem

# Python-based agents
export HTTPS_PROXY=http://127.0.0.1:8080
export REQUESTS_CA_BUNDLE=~/.mitmproxy/mitmproxy-ca-cert.pem
export SSL_CERT_FILE=~/.mitmproxy/mitmproxy-ca-cert.pem

<your-agent-command>
```

In the mitmweb UI, filter for the model endpoint (look for `/chat/completions`, `/v1/messages`, `/responses` or similar) and inspect the JSON body. You will see the `messages` / `input` array with every tool result.

## Option C: Canary test

1. Create a file `canary.txt` containing a unique string, e.g. `CANARY-7f3a9c`.
2. Ask the agent to do an unrelated task, then one that reads the file.
3. Search proxy captures for `CANARY-7f3a9c` to confirm exactly when it was transmitted.

## What to look for

- Which hosts are contacted (model API, telemetry, auth, others)?
- Does any request contain file contents you didn't expect?
- Are files sent before the agent "reads" them (indicating indexing/upload)?
- What's in telemetry payloads?
