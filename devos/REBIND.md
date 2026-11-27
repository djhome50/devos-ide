# Rebind map: extensions/copilot → extensions/devos-agent

The in-tree `extensions/copilot` is the full Copilot agent stack (MIT).
`devos-agent` starts as a copy and rebinds four seams: auth, models,
session backends, telemetry. Everything else — intents, tool calling,
prompt infrastructure, session UI integration — is reused as-is.

## Binding map

| `extensions/copilot` area | What it does | DevOS rebind |
|---|---|---|
| `authentication/` | GitHub/Copilot sign-in | DevOS control-plane token (local: none/dev token; hosted: OIDC) |
| `byok/` | BYOK model providers → Copilot model service | `languageModelChatProviders` → LiteLLM gateway + Ollama |
| `chatSessions/` | Session list providers (local CLI + cloud) | DevOS task/run providers over `devos-api` |
| `chatSessions/copilotcli` | Local background sessions, worktree isolation | DevOS local runner + worktree policy (workspace-contracts) |
| `chatSessions` cloud paths | Delegate to GitHub coding-agent backend → PRs | Delegate to DevOS workspace manager → remote run → PR via GitHub App |
| `externalAgents/` | Claude Code / Codex-style harness integration | Keep — external engines become DevOS adapters (ACP seam) |
| `intents/` | Agent-mode intent + tool-call loop | Reuse; DevOS objectives/plans injected via context |
| `conversation/`, `prompt/` | Request handling, prompts | Reuse; DevOS instructions/knowledge layered as context |
| `mcp/` | MCP tool servers | Reuse; DevOS tools published as MCP where useful |
| `otel/`, telemetry | Copilot telemetry | Disable by default; DevOS usage events → control-plane metering only |
| `completions*/`, `inlineEdits/` | Inline suggestions | Optional — strip initially (needs Copilot backend); revisit later |
| `onboardDebug`, `getting-started` | Copilot onboarding UX | Replace with DevOS first-run (connect control plane, pick model) |

## Working rules

- Keep upstream file layout inside `extensions/devos-agent` so subtree
  merges from `extensions/copilot` stay reviewable.
- Rebind through seam interfaces (service/endpoint indirection), not by
  deleting code paths — upstream diffs then touch seams, not core logic.
- No Copilot endpoints, keys, or telemetry in DevOS builds: gate behind
  `devos` product config, not runtime checks.
- `enabledApiProposals` in the built-in extension may use proposed APIs
  (chat sessions); the standalone `devos-control` extension must stay on
  stable API so it also installs into stock VS Code.

## First adaptation tasks

1. Copy `extensions/copilot` → `extensions/devos-agent`; rename
   publisher/name/displayName; strip Copilot branding assets.
2. Point `byok` + chat model resolution at LiteLLM (`tgc-devos.litellm.*`
   settings shared with the standalone extension).
3. Rewire `chatSessions` providers to read DevOS runs from
   `devos-api` (`GET /api/v1/runs`, `GET /runs/{id}/events/stream`).
4. Replace auth provider registration with the DevOS session token.
5. Disable telemetry pipeline; emit `usage.reported`-equivalent events
   to the control plane instead.
