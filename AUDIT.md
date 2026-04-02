# AUDIT.md

Use this file when auditing upstream changes for security and deployment readiness.

## Scope

Focus review on changes that affect:

- daemon network exposure
- relay trust boundaries and pairing flows
- websocket and MCP control surfaces
- shell execution and git command invocation
- file explorer and file download path handling
- desktop bridge code that can reach the daemon
- config defaults that widen access

## Required Reads

Read these before auditing:

- `CLAUDE.md`
- `SECURITY.md`
- `docs/DEVELOPMENT.md`
- changed files in `packages/server`
- changed files in `packages/desktop` if daemon startup or pairing changed

## Audit Questions

For each relevant change, answer:

1. Does this expose a new control surface or widen an existing one?
2. Does this rely on network reachability as a trust boundary? If yes, is that explicit and acceptable?
3. Does this add or change any shell execution, command interpolation, or git ref handling?
4. Does this change file path resolution, symlink handling, downloads, or workspace boundaries?
5. Does this change relay, pairing, auth assumptions, or local daemon trust boundaries?
6. Does this change defaults in a way that makes remote access easier or less safe?
7. Are tests present for the security-sensitive behavior?

## Verification

After security-relevant changes, run:

```bash
npm run build --workspace=@getpaseo/highlight
npx vitest run packages/server/src/server/file-explorer/service.test.ts packages/server/src/server/websocket-server.relay-reconnect.test.ts
npm run typecheck
```

## Review Output

When reviewing, report:

- what changed
- whether the change narrows risk, widens risk, or is neutral
- what assumptions the code is making about deployment
- what tests cover it
- any follow-up hardening you would keep on a fork rather than upstream

## Deployment Rule

Assume the intended safe setup is:

- daemon local-only by default
- remote access through relay, Tailscale, SSH tunnel, or equivalent controlled network path

Do not treat direct public daemon exposure as a safe default.
