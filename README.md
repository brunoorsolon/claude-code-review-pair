# Claude Code ↔ LAN message broker

A Linux **cooperative-pull** adapter foundation for a newly launched, exactly
bound Claude Code session. It uses ordinary local MCP tools, not Channels, a
permission relay, or terminal input injection. Claude **2.1.309** launch flags,
hook-settings schema and exec-form hook mechanics were checked using the official
binary without login or model calls. **Authenticated two-session operation is
not certified.** A synthetic CLI exercises the complete launcher/hook/MCP lifecycle.

The broker handles registry, metadata and delivery to a durable session-bound
inbox only. This package imports the shared runtime from broker commit
`ba2219cd87a1feac23e01b23d2c2b3d3bdd65359`, verifying the source bytes before import.
Use Node **24.19.0** for its SQLite journal; Claude's own runtime is separate.

## Implemented boundary

- A fresh explicit native `--session-id`, private launch lifetime, Linux PID start
  time/boot identity, native SessionStart confirmation and monotonic closure.
  A surviving MCP process or transcript ID alone is not a live CLI session.
- Independent broker heartbeat/receive loops, durable inbox/dedupe and broker ACK
  without a model call. Exact session change/death closes the adapter.
- Five shared tools: discovery, send, metadata update, message status and receive.
  A native PreToolUse hook can **only deny** wrong-session/subagent access. It
  never permits a tool, weakens approval policy or treats `yes` as human consent.
- `broker_receive` returns model-visible untrusted AGENT provenance through a
  normal tool result. It **peeks**, not consumes: stdio write/tool return does not
  prove durable native transcript acceptance. Repeated receive returns the same
  FIFO head; there is no automatic advance, pop, skip or implicit receipt.

This is useful for operator-launched cooperative discovery/send/inspection, not
an automatic background mailbox wakeup or a finished durable-consumption path.
Existing TUI attachment and automatic native/channel delivery are **unsupported**.

## Test and launch

Place this checkout next to the pinned `ai-agent-msg-broker` checkout:

```sh
cd claude-code-review-pair
npm run check
npm test
```

Tests need no package installation, credentials, real CLI or model. See
[operator instructions](docs/integration.md) and [verification](docs/verification.md).
Related scope: [broker issue #9](https://github.com/brunoorsolon/ai-agent-msg-broker/issues/9).
