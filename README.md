# Triggo Runtime

**Workflow runtime for AI agents. Turn approved workflows into MCP and REST tools agents can call safely.**

---

## What this is

AI agents need to take real actions — write to a CRM, post to a channel, update a sheet, hit a partner API. The hard part isn't the call. The hard part is letting an agent loose without it duplicating writes on a retry, draining a quota, leaking a credential into a log, or silently chewing through your billing because a partner API started 500ing at 3am.

Triggo Runtime is the execution layer between an agent and the systems it touches. You define a workflow once. The runtime exposes it as an MCP tool, a REST endpoint, or both. Every call goes through the same envelope: **per-tool scoped API keys**, **dry-run mode**, **idempotency keys**, an **ON CONFLICT event journal**, a **per-integration circuit breaker**, **auto-pause after N consecutive failures**, and a full **audit journal**. The agent gets a narrow, named capability. You get a replayable record of every step.

This is not "an MCP server with connectors." It's a deterministic workflow runtime where MCP is one surface. If you're using Activepieces or n8n with MCP enabled, the difference shows up the first time a flaky downstream API causes a retry storm — Triggo's circuit breaker trips per-integration, the journal de-duplicates writes via ON CONFLICT, and auto-pause stops a misbehaving workflow before it burns through your credits. Depth on governance, not breadth on connectors. That's the trade.

## Quick start

No signup. Pull, run, hit it.

```bash
docker run -p 8080:8080 \
  -e ENCRYPTION_KEY=$(openssl rand -hex 32) \
  -e BETTER_AUTH_SECRET=$(openssl rand -hex 32) \
  ghcr.io/triggoapp/triggo-runtime:dev-preview
```

Define a workflow (`hello.yaml`):

```yaml
slug: hello-world
name: Hello, agent
trigger:
  type: manual
steps:
  - id: greet
    action: http-request.send
    input:
      method: POST
      url: https://example.com/hooks/hello
      body:
        message: "{{ input.message }}"
```

Register it:

```bash
curl -X POST http://localhost:8080/api/v1/runtime/workflows \
  -H "Authorization: Bearer $TRIGGO_API_KEY" \
  -H "Content-Type: application/yaml" \
  --data-binary @hello.yaml
```

Call it as a tool:

```bash
curl -X POST http://localhost:8080/api/v1/runtime/actions/hello-world/run \
  -H "Authorization: Bearer $TRIGGO_API_KEY" \
  -H "Idempotency-Key: $(uuidgen)" \
  -H "Content-Type: application/json" \
  -d '{"input": {"message": "hi"}, "dryRun": false}'
```

### Claude Desktop / Cursor MCP config

```json
{
  "mcpServers": {
    "triggo": {
      "transport": "http",
      "url": "http://localhost:8080/mcp",
      "headers": {
        "Authorization": "Bearer ${TRIGGO_API_KEY}"
      }
    }
  }
}
```

The agent now sees `list_workflows`, `run_action`, `get_run_status`, `approve_run`, and the workflows you've registered — each gated by the scopes minted on the API key.

## Concepts

- **Action** — A single named operation against an external system. Has an input schema, an output schema, and a handler. Examples: `http-request.send`, `google-sheets.append-row`.
- **Workflow** — A DAG of steps (actions, conditions, loops) with a trigger. Registered, versioned, callable by slug.
- **Tool** — A workflow exposed over MCP or REST. The agent sees the input schema; the handler is the workflow.
- **Scope** — A capability minted on an API key. Tools check required scopes before executing. No scope, no call.
- **Dry-run** — Execute the workflow against the journal and validators, skip side-effecting calls. Returns the planned operations and inputs the runtime would have used.
- **Idempotency key** — Header sent with a run request. The runtime de-duplicates retries against the journal: same key, same input, returns the original result. Different input with the same key is rejected.
- **Circuit breaker** — Per-integration, Redis-backed. After N failures in a window, calls to that integration fast-fail until the breaker half-opens.
- **Auto-pause** — A workflow with three consecutive failed runs is paused. Resume is explicit.
- **Journal** — Event-sourced record of every step: input, output, error, timestamp. ON CONFLICT semantics on the write path make replays safe. Used for audit, debugging, and replay.

## Example: Telegram → CRM → AI summary, MCP-callable

A common shape: an agent receives a Telegram message, looks up the contact in a CRM, summarizes the conversation, writes the summary back. You want the agent to be able to invoke this as one tool, with one scope, and one audit trail.

`workflows/telegram-triage.yaml`:

```yaml
slug: telegram-triage
name: Triage Telegram message and update CRM
trigger:
  type: manual
input:
  type: object
  required: [chat_id, message]
  properties:
    chat_id: { type: string }
    message: { type: string }
steps:
  - id: lookup-contact
    action: crm.find-contact-by-chat-id
    input:
      chat_id: "{{ input.chat_id }}"
  - id: summarize
    action: openai.complete
    input:
      model: gpt-4.1-mini
      prompt: |
        Summarize this customer message in one line:
        {{ input.message }}
  - id: update-contact
    action: crm.append-note
    input:
      contact_id: "{{ steps.lookup-contact.output.id }}"
      note: "{{ steps.summarize.output.text }}"
```

Mint a key with only the scopes this workflow needs:

```bash
curl -X POST http://localhost:8080/api/v1/runtime/api-keys \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -d '{
    "name": "agent: telegram-triage",
    "scopes": ["actions:run:telegram-triage", "runs:read"]
  }'
```

The agent's MCP client sees exactly one workflow tool, `telegram-triage`. It cannot list other workflows. It cannot read other runs. It cannot mint keys. The journal records every invocation; replay any run from the dashboard.

## Roadmap

**Developer Preview today**

- MCP HTTP transport, REST runtime API
- Per-tool scoped API keys, idempotency keys, dry-run
- Event journal with ON CONFLICT writes, replay
- Circuit breaker, auto-pause, per-step + per-pipeline timeouts
- Connectors: HTTP, webhook, schedule, Google Sheets, Gmail, OpenAI, Slack, Notion, HubSpot
- Self-host via Docker; managed cloud at usetriggo.com

**In progress**

- MCP stdio transport
- Workflow signing + signed runs
- Streaming step outputs over MCP
- Connector SDK for community-maintained connectors

**Not yet — and we'll be honest when we ship**

- A formal connector certification program
- Multi-region self-host orchestration
- A managed offering for compliance-bound workloads

This is a Developer Preview. APIs will change. The journal format is stable; the surrounding tooling is not. Don't put it in the critical path of revenue without talking to us first.

## Issues / contributing

File issues on GitHub. The most useful feedback is:

- A workflow you tried to build and the exact friction point
- An MCP client + model + scope combination that didn't behave as expected
- A retry / idempotency / circuit breaker scenario where the runtime did the wrong thing

PRs welcome on connectors, docs, examples. Core runtime PRs should start with an issue.

## Security

Credentials are AES-256-GCM encrypted at rest with a key the operator provides (`ENCRYPTION_KEY`). Logs are sanitized for known credential and PII patterns before journal writes. Every API key carries an explicit scope set; tools enforce required scopes before executing, and runs without sufficient scope fail closed.

Found something? Email `security@usetriggo.com`. Don't open a public issue for vulnerabilities.

## Pricing

Self-host is free. Managed cloud pricing lives at [usetriggo.com/pricing](https://usetriggo.com/pricing) — Developer Preview is free, Builder and Team / Pilot tiers above that, AI top-ups available. Pricing is intentionally not duplicated in the README so it stays in one place.

## Status

**Developer Preview.** Not actively onboarding new EU teams beyond invited builders right now. The MCP demo, the GitHub repo, and the docs remain available; managed-cloud pilot slots are gated.
