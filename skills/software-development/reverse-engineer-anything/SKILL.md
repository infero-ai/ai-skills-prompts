---
name: reverse-engineer-anything
description: "Use REA (rea-agents) to reverse engineer binaries, JS/Electron apps, APKs, .NET, EVM, WASM, and websites — evidence-backed, runs locally."
version: 0.1.0
author: celebez (0xcelebez), Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [reverse-engineering, mcp, rea, binaries, electron, android]
    related_skills: [hermes-agent]
---

# REA — Reverse Engineer Anything (rea-agents)

REA (github.com/morluto/rea, npm `rea-agents`) is one MCP/CLI that connects an
agent to reverse-engineering tooling: native binaries (Hopper/Ghidra/IDA),
JavaScript/Electron apps, Android APKs + devices (adb), .NET assemblies, WASM,
EVM bytecode, SQLite, websites, network captures (HAR/SAZ), process/runtime
capture. Every finding ships with **evidence and stated limitations**. Analysis
runs locally; nothing is uploaded.

## When to Use
- "Reverse engineer / understand how X works" for a binary, app, APK, or site
- Recover a feature's logic from an app **without source code** and rebuild it
- Inspect Electron/JS bundles, ASAR, IPC boundaries, source maps
- Static .NET/WASM/EVM/SQLite inspection without running anything

## Sandbox Install (this machine)
- Location: `~/workspace/rea/rea-agents` (extracted from npm tarball because
  `npm install` hangs in this sandbox; see Pitfalls).
- Node requirement: 22.19+/24.11+/26+ — sandbox has Node 26.7.0 (OK).
- Entry points: `~/workspace/rea/rea-agents/dist/cli.js` (bin `rea`),
  `bin/rea` when node_modules is installed.
- Run: `node ~/workspace/rea/rea-agents/dist/cli.js --help`
- Update: download a newer tarball from registry.npmjs.org and re-extract —
  do NOT rely on `rea update` / `npm install -g` here.

## MCP in Hermes
Registered in `~/.hermes/config.yaml` under `mcp_servers.rea` (stdio), verified
handshake + `tools/list` = 207 tools:
```yaml
mcp_servers:
  rea:
    command: "node"
    args: ["~/workspace/rea/rea-agents/scripts/rea.mjs", "mcp"]
    timeout: 300
    connect_timeout: 120
```
MCP mode is `rea.mjs mcp` (or `--mcp`); tools appear as `mcp_rea_<name>`.
Restart Hermes to load. Smoke test by hand:
`printf '{"jsonrpc":"2.0","id":1,...initialize...}' | node scripts/rea.mjs mcp`.

## CLI Quick Reference
```bash
# JavaScript / Electron app (no native engine needed)
rea analyze-javascript-application /abs/path/to/app-or-asar --json

# Native binary — needs a provider (Hopper/Ghidra/IDA); see provider config
rea analyze-native-binary /abs/path/to/binary

# Offline ELF diagnostics (caller-supplied pwntools), Go/Rust static facts
# .NET (static), WASM (WABT), EVM (raw hex), SQLite snapshot, HAR/SAZ captures
rea --help            # full command list
```
Native analysis provider is selected/configured per `docs/cli.md` in the repo;
Hopper can be auto-installed by `rea` setup WITH user approval (never assume).

## Evidence Discipline
- REA returns pseudocode/assembly/relations + evidence. Quote the evidence
  when explaining behavior; do not overstate beyond what the tool returned.
- Static inspection ≠ runtime behavior: say which one produced a claim.
- Legal: only analyze software you own or are authorized to examine.

## Pitfalls
- Sandbox: `npm install` (any form, incl. `--ignore-scripts`) stalls forever on
  proxy — install REA from the registry tarball. Deps: `python3
  ~/workspace/rea/fetch-deps.py` (BFS registry GET installer, re-runnable,
  skips optional/peer; 212 pkgs installed 2026-10-15, ~4 min).
- Scripts/postinstall never ran (node-pty, playwright-core native bits) →
  runtime browser/electron/process capture may fail; static JS/.NET/SQLite/WASM
  analysis is unaffected (verified via inspect-sqlite-database).
- ai-skills-prompts sync uses `rsync --delete` — this skill lives only locally;
  canonical backup at `~/workspace/rea/SKILL.md`. Re-copy after each sync until
  it's added to the 0xcelebez/ai-skills-prompts repo.
- Linux needs a procps-compatible `ps` on PATH for JS/Electron process checks.
- Native deep analysis needs a provider (Hopper/Ghidra/IDA) — none installed in
  sandbox; `capabilities` shows native providers unavailable (host or missing).
