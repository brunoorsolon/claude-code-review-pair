# Verification and remaining gates

2026-10-23 cloud Linux, Node 24.19.0 / SQLite 3.53.3:
`npm run check` and **12/12 tests passed** against shared broker commit
`e015a6f5d7cc32d0c0a90bd633fab8d94195417a`.

The end-to-end fixture spawns a **synthetic Claude CLI**, real native-hook helper
processes, the real standard MCP subprocess, shared HTTP broker and SQLite inbox.
It checks lifecycle confirmation, exact ownership, delivery ACK, model-visible
AGENT warning, cooperative non-consumption and clean native shutdown. Focused
tests cover main/subagent guards, unknown/switched lifetime, PID reuse defense,
model metadata, EOF during registration, bounded arguments and tamper-rejected
reference source. The fake executable's `0.0.0` version is not a vendor version.

Not run: actual Claude binary/TUI, authenticated provider calls, real permission
UI, native hook ordering under clear/resume/compact/model switch, rootless Podman
or two-host TLS LAN. Ordinary MCP tools may require human approval; none was
bypassed. This PR is not a full issue #9 completion claim.

Required next gates:

1. Pin an official CLI version and verify all launch/hook events against it,
   including root session and subagent tool routing and MCP restart.
2. Confirm actual provider-visible tool-result provenance with hostile body text
   (`yes`, fake channel boundaries and prompt-injection strings).
3. Keep human drafts, active work and approvals untouched. Cooperative tools are
   invoked only in native flow; no automatic event may inject a prompt.
4. Establish a supported durable native acceptance receipt before consuming any
   inbox head. Channel opt-in alone is insufficient and permission relay must
   remain disabled.
5. Package and test separate CLI+adapter rootless containers and two-host LAN.
