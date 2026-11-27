# DevOS IDE

A Code-OSS fork of `microsoft/vscode` that ships TGC DevOS as the primary
experience: a FOSS Devin-class agent platform on top of the VS Code
workbench.

## What this fork changes

Deliberately little. Upstream merges must stay cheap, so changes are
confined to:

1. `product.json` — DevOS branding, data folder, protocol, Open VSX
   gallery (done in the initial branding commit).
2. `extensions/devos-agent` — the DevOS agent extension, adapted from the
   in-tree `extensions/copilot` (MIT). See `REBIND.md`.
3. `extensions/devos-control` (planned) — the DevOS control-plane
   extension: sessions inspector, approvals inbox, handoff controls.
4. `devos/` — this directory: fork policy, rebind map, build notes.

**Do not** patch `src/vs` workbench code. If a feature seems to require
a workbench patch, prefer the extension API first; the proposed API
surface (chat sessions, agent sessions) is enabled for our built-in
extensions via `devos-agent`'s `enabledApiProposals`.

## Remotes and merge policy

- `origin` — `djhome50/devos-ide` (this repo)
- `upstream` — `microsoft/vscode`

Merge policy: take upstream `main` into a `devos/upstream-merge` branch,
resolve conflicts only in the areas listed above, run a smoke launch,
then fast-forward `main`. Never rebase shared `main`.

## The agent stack we inherit

`microsoft/vscode` ships the Copilot agent stack in-tree at
`extensions/copilot` (MIT): agent mode, chat sessions providers, BYOK
model providers, external-agent harnesses, plan→implement delegation,
and cloud-session architecture. DevOS adapts it rather than rebuilding —
see `REBIND.md` for the per-module binding map.

## Build notes (first pass)

Full installer builds use the VSCodium-style pipeline (later milestone).
For development:

```bash
# requires Node (see .nvmrc) and the platform toolchain
npm install        # or pnpm/yarn per repo root docs
npm run watch      # compile the workbench
# run Code - OSS from sources:
./scripts/code.sh  # macOS/Linux
.\scripts\code.bat # Windows
```

Building DevOS IDE installers (Windows/macOS/Linux) is roadmap S2 in
`TGC-DevOS` — tracked in `docs/ROADMAP.md` there.
