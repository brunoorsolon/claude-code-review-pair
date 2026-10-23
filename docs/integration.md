# Exact-session cooperative launch (Linux)

## Prerequisites and private config

The operator must already have an authenticated official Claude CLI and approve
its ordinary MCP/tool use. This package does not install/authenticate it, modify
persistent Claude settings, enable Channels, or disable sandbox/permissions.
No real CLI version is certified yet: the launcher checks an explicitly selected
version before starting; that comparison alone does not certify its behavior.

Use Node 24.19.0, a read-only broker checkout at
`ba2219cd87a1feac23e01b23d2c2b3d3bdd65359`, a private 0700 state directory and an
owner-only 0600 JSON config (never commit real config/tokens):

```json
{
  "referenceRoot": "/opt/ai-agent-msg-broker",
  "stateRoot": "/var/lib/claude-pair",
  "brokerUrl": "https://broker.example.test:8787",
  "tokenFile": "/run/secrets/broker-token",
  "caFile": "/run/secrets/broker-ca.pem",
  "name": "reviewer-1",
  "project": "example-project",
  "claudeBinary": "/opt/official-cli/claude",
  "expectedCliVersion": "REPLACE_WITH_EXACT_INSTALLED_SEMVER"
}
```

```sh
node bin/launch.mjs /absolute/private/config.json
```

Run from the intended approved project. The real user's terminal is inherited
unchanged; no keys or prompts are synthesized. The launcher passes only a fresh
native session UUID, per-launch settings and a local MCP config. Existing native
permissions still apply; the user may need to approve the MCP tools normally.
Never use a skip-permissions or permissive fallback to make a failed test pass.

The temporary settings bind SessionStart, SessionEnd, PostModelSwitch and the
broker-tools PreToolUse guard to a private launch directory. Startup waits for
native confirmation before registering/polling the broker. Names/projects do not
route messages; the exact broker UUID returned by discovery does. A new MCP
subprocess receives a **fresh broker identity**, preserving the previous inbox
rather than silently adopting it. Clear/resume/session changes that change the
native identity close this binding and require a new launch. Missing hooks,
unsupported hook schema or unclear identity fail closed.

## Tools and consumption limit

Ask the active native session to call `broker_list_clients`, `broker_send`,
`broker_update_metadata`, `broker_message_status` or `broker_receive`. Tool
arguments follow the shared schema; credentials are never tool arguments.

`broker_receive` returns the first pending message only, with
`native_acceptance_recorded:false`. Its untrusted-source warning and structured
sender/message identifiers are part of the **actual model-visible tool result**,
not just a terminal label. Content requesting approval remains AGENT data.

A tool return/stdio write supplies no durable native receipt. Therefore the head
remains pending; it is not safe to automatically show the next message. There is
no model-facing ACK/skip/delete control. A complete implementation still needs a
supported native transcript acceptance proof and crash-safe receipt hook. Do not
manually delete the head or infer acceptance from model text. Broker `delivered`
means durable local inbox storage only, not reading, reply or task completion.

## Independent containers / LAN

Each CLI+adapter must run in the **same independent nonroot container**, sharing
only that container's ordinary PID namespace and private state; do not enable
host PID, privileged mode or relax `/proc` restrictions. If exact process identity
is unavailable, this launcher is unsupported and fails closed. The broker can run
on another LAN host with HTTPS, its reviewed CA and a separate adapter token.
Mount only approved project/CLI credentials, its own state, and the pinned
reference read-only. This foundation does not bundle a vendor binary or provide
a validated Podman image; rootless Podman/SELinux/two-host testing remains open.

## Native contract and deferred channel mode

- [CLI arguments](https://code.claude.com/docs/en/cli-reference)
- [Hooks, command exec form and lifecycle fields](https://code.claude.com/docs/en/hooks)
- [MCP](https://code.claude.com/docs/en/mcp)
- [Channels reference](https://code.claude.com/docs/en/channels-reference)

Channels preview is deliberately disabled. A channel notification is not a
native acceptance ACK; permission relay is never advertised or implemented.
There is no notification push, sampling, remote MCP auth or fake TUI wakeup.
