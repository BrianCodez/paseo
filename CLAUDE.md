# CLAUDE.md

Paseo is a mobile app for monitoring and controlling your local AI coding agents from anywhere. Your dev environment, in your pocket. Connects directly to your actual development environment — your code stays on your machine.

**Supported agents:** Claude Code, Codex, and OpenCode.

## Repository map

This is an npm workspace monorepo:

- `packages/server` — Daemon: agent lifecycle, WebSocket API, MCP server
- `packages/app` — Mobile + web client (Expo)
- `packages/cli` — Docker-style CLI (`paseo run/ls/logs/wait`)
- `packages/relay` — E2E encrypted relay for remote access
- `packages/desktop` — Electron desktop wrapper
- `packages/website` — Marketing site (paseo.sh)

## Documentation

| Doc | What's in it |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | System design, package layering, WebSocket protocol, agent lifecycle, data flow |
| [docs/CODING_STANDARDS.md](docs/CODING_STANDARDS.md) | Type hygiene, error handling, state design, React patterns, file organization |
| [docs/TESTING.md](docs/TESTING.md) | TDD workflow, determinism, real dependencies over mocks, test organization |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | Dev server, build sync gotchas, CLI reference, agent state, Playwright MCP |
| [docs/RELEASE.md](docs/RELEASE.md) | Release playbook, draft releases, completion checklist |
| [docs/ANDROID.md](docs/ANDROID.md) | App variants, local/cloud builds, EAS workflows |
| [docs/DESIGN.md](docs/DESIGN.md) | How to design features before implementation |
| [SECURITY.md](SECURITY.md) | Relay threat model, E2E encryption, DNS rebinding, agent auth |

## Quick start

```bash
npm run dev                          # Start daemon + Expo in Tmux
npm run cli -- ls -a -g              # List all agents
npm run cli -- daemon status         # Check daemon status
npm run typecheck                    # Always run after changes
```

See [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) for full setup, build sync requirements, and debugging.

## Critical rules

- **NEVER restart the main Paseo daemon on port 6767 without permission** — it manages all running agents. If you're an agent, restarting it kills your own process.
- **NEVER assume a timeout means the service needs restarting** — timeouts can be transient.
- **NEVER add auth checks to tests** — agent providers handle their own auth.
- **Always run typecheck after every change.**

## Fork maintenance

This repo may be maintained on a personal fork with a small hardening patch stack on a dedicated branch such as `security-hardening`.

- Treat `upstream` as the source project and `origin` as the personal fork.
- Keep the hardening branch small and focused. Prefer one commit per concern.
- Do not mix unrelated feature work into the hardening branch.
- If upstream lands an equivalent fix, drop the local patch instead of reimplementing it.

When updating the hardening branch:

```bash
git status --short --branch
git remote -v
git branch --show-current
git fetch upstream
git checkout security-hardening
git rebase upstream/main
```

After rebases or security edits, run this repo's required verification:

```bash
npm run build --workspace=@getpaseo/highlight
npx vitest run packages/server/src/server/file-explorer/service.test.ts packages/server/src/server/websocket-server.relay-reconnect.test.ts
npm run typecheck
```

When pushing a rebased hardening branch, use:

```bash
git push --force-with-lease
```

When creating it for the first time, use:

```bash
git push -u origin security-hardening
```

Conflict rule:
- Prefer upstream structure.
- Reapply only the minimum local logic needed to preserve the hardening behavior.
- Re-check auth, relay, file access, shell execution, and desktop bridge code if upstream changed those areas.

If a `PATCHES.md` file exists, read it before rebasing or editing the hardening branch.

## Debugging


Find the complete daemon logs and traces in the $PASEO_HOME/daemon.log
