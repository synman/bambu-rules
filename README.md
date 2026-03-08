# github-rules

This repository is the **authoritative remote backup** for all Copilot behavioral rules and workspace baselines. It serves as the `origin` for rules files in the same way that a git remote serves as the origin for source code.

## Purpose

Rules files govern how the GitHub Copilot CLI agent behaves across all workspace projects — what it can and cannot do, how it makes decisions, how it handles safety-critical operations, and how it maintains consistency across sessions. These rules are treated as code: versioned, baselined, diffed, and kept in sync with the same rigor as source code.

This repo provides:
- **Remote backup** — rules files are mirrored here so they can be restored even if the local machine is lost
- **Audit trail** — every rule change is a commit with a message explaining why
- **Baseline snapshots** — point-in-time captures of all rules files alongside the code repo SHAs they correspond to, enabling full environment restore

## Structure

```
github-rules/
├── .github/
│   └── copilot-instructions.md         # Copilot hook for this repo itself
├── global/
│   └── copilot-instructions.md         # Universal rules — apply to all repos, all sessions
├── projects/
│   ├── bambu-printer-app/
│   │   └── copilot-instructions.md     # bpa — project-specific extensions
│   ├── bambu-printer-manager/
│   │   └── copilot-instructions.md     # bpm — project-specific extensions
│   ├── bambu-mcp/
│   │   └── copilot-instructions.md     # MCP server — project-specific extensions
│   ├── bambu-fw-fetch/
│   │   └── copilot-instructions.md     # Firmware fetch utility
│   ├── bambu-mqtt/
│   │   └── copilot-instructions.md     # bmt — MQTT research/utilities
│   └── webcamd/
│       └── copilot-instructions.md     # wcd — MJPEG server / RTSPS ingestion
└── baselines/
    └── <name>.<repo>.copilot-instructions.md   # Point-in-time rules snapshots
    └── copilot-instructions.<name>.md          # Global rules snapshot for each baseline
```

## How rules work

**Two layers apply simultaneously:**

1. `global/copilot-instructions.md` — universal behavioral rules (safety, KISS, verification, git policy, etc.). Always in effect.
2. `projects/<repo>/copilot-instructions.md` — project-specific extensions (architecture, build commands, API endpoints, local conventions). Extend but never contradict global rules.

**Local source of truth:** The live files on the local machine are always the source of truth. This repo is a mirror — never edit files here directly as the primary edit.

| This repo | Local path |
|-----------|-----------|
| `global/copilot-instructions.md` | `~/.copilot/copilot-instructions.md` |
| `projects/<repo>/copilot-instructions.md` | `~/<repo>/.github/copilot-instructions.md` |
| `baselines/` | `~/.copilot/baselines/` |

## Baselines

A baseline is a named point-in-time snapshot capturing:
- All 7 rules files (global + 6 project)
- The HEAD SHA of every tracked code repo at capture time

Baselines enable full environment restore — rules state and code state together. The global rules file documents the known baselines table and the pre-flight requirements for capturing a valid one.

**Restoring from this repo** (if local `~/.copilot/baselines/` is unavailable):
```bash
cd ~/GitHub/github-rules && git pull
cp baselines/copilot-instructions.<name>.md ~/.copilot/copilot-instructions.md
cp baselines/<name>.bambu-printer-app.copilot-instructions.md ~/bambu-printer-app/.github/copilot-instructions.md
# ... repeat for all 6 project repos
# Then reset each code repo to its captured SHA (see global rules Known Baselines table)
```

## Rules reference

### Global rules (`global/copilot-instructions.md`)

These apply universally across every repo and session.

**Infrastructure**
- **Baselines** — Named point-in-time snapshots of all 7 rules files + 6 code repo SHAs. Pre-flight requires clean + pushed repos. Must be synced to this repo in the same turn. Restore procedure documented for both local and remote (this repo) sources. Restore paradox: sync obligation is suspended during a restore until intent is confirmed.
- **Rules as Code** — Rules files carry the same versioning, baselining, and delta-assessment rigor as source code. Rules-only deltas warrant new baselines. This repo is `origin` for rules files.
- **Rules File Maintenance** — Global vs. project-specific placement rules. No-duplication rule: never copy a global rule verbatim into a project file. Sync to this repo is mandatory after any edit.
- **Session Start Protocol + RULES_PRECHECK** — Both rules files must be read in full in the current turn before any non-read action. `RULES_PRECHECK` line must appear in visible response text as a user-auditable compliance gate.
- **Scope Gate** — Active repo scope declared in every RULES_PRECHECK line. Cannot change without explicit user instruction. Cross-repo touches require explicit approval.

**Safety (absolute, no exceptions)**
- **Printer Write Protection** — No write or destructive operation to a physical printer without explicit user text in the current turn. No `write_bash` confirmation prompts, no "dry-run" pretexts.
- **Container API Auth** — All live printer state queries must use credentials from `~/.bpm_secrets` via `secrets.py`. Never use macOS Keychain, never hard-code credentials.
- **Security & Privacy** — No secrets in logs or commits. RTSPS URLs always redacted. Test fixture data treated as private. SSL/TLS certificate validation required.

**Behavioral standards**
- **KISS Principle** — Minimum effective change. Reuse before creating. No speculative architecture. KISS pre-implementation gate must run before writing any code.
- **Quality-First Mode** — Correctness over speed. Verify assumptions with source evidence before editing.
- **Verification First** — Never infer from patterns. Read actual implementations. Trace data flow line-by-line. Cite specific line numbers. Truncated file read trap: absence in truncated output ≠ absence in file.
- **Strict Execution Mode** — Verify → patch minimally → validate immediately → no speculation.
- **Protocol Field Mapping Parity** — New protocol fields must follow proven sibling field patterns. Evidence hierarchy: steady-state status > command ack. Parity checklist required before finalizing.
- **Authoritative Sources** — Establish source hierarchy before any protocol/API work: official vendor → platform integrations → protocol specs → community implementations → cross-language implementations.
- **Ask for Clarification** — If deducing a domain-specific value costs more than a one-sentence user answer, ask first.
- **Post-Audit Rules Update Obligation** — Every audit gap must produce a rules patch before the audit closes. Rules files are the only durable output; session notes and TODOs are not sufficient.

**Operational**
- **Git Commit Policy** — Never commit without explicit user instruction. Stage and stop. No-attribution rule: no `Co-authored-by` or any authorship credit. Exhaustive list of trap phrases that do not authorize committing.
- **BPM Write Scope Lock** — `bambu-printer-manager` permits only `git add`, `pre-commit run`, and read-only git ops. Absolute, no delegation, no exception, no override by any argument.
- **Premium Requests** — Ask user before any web search or premium model invocation. `ask_user` response to an explicit named question is the only valid authorization.
- **MCP Tool Reconnect** — Run `~/bin/mcp-reload` immediately when tools go missing. Do not wait to be told.
- **Process State Awareness** — Check running processes before assuming code changes are live. Multiple instances = stale processes.
- **In-Memory Cache Trap** — A consumer restart never reloads server-side in-memory content. Identify which process owns the cache and restart that.
- **Response Endings** — Close with task result only. No unsolicited follow-up offers, next-step suggestions, or optional cleanups.
- **Tool Output Visibility** — Reproduce important output in response text. Users cannot expand tool output summaries in the CLI UI.

---

### Project rules (per-repo extensions)

Each file in `projects/<repo>/` extends the global rules for that specific codebase.

**`bambu-printer-app` (bpa)**
- Docker build & deployment — Watchtower, container lifecycle, image tagging
- Container API endpoints — full route reference for the Flask REST API
- Root Cause Fix Rule — fix the actual cause, not the symptom; no workarounds without documented root cause
- Telemetry Mapping Parity — new telemetry fields must follow proven sibling field patterns (extension of global Protocol Parity)
- Terminal Housekeeping — clean up temp files, background processes, and open ports after every task
- Cross-Model Compatibility Policy — H2D dual-extruder vs. single-extruder parity requirements
- Architecture — Flask app structure, bpm library integration, Docker networking

**`bambu-printer-manager` (bpm)**
- GIT WRITE PROTECTION — absolute scope lock (extends global BPM Write Scope Lock); stage + pre-commit only
- Root Cause Fix Rule, Telemetry Mapping Parity, Verification First — same as bpa
- Cross-Model Compatibility Policy — single vs. dual extruder parity
- Architecture — library structure, telemetry, MQTT session, BambuState dataclass
- Reference Implementations — canonical patterns for telemetry fields, HMS, AMS, home_flag bits
- Querying Live Printer State — always use container API, not direct bpm calls

**`bambu-mcp` (mcp)**
- Git Flow — agent-managed (stage + commit + push all permitted)
- Dependency Updates — hard stop, consult user before any version bump
- Versioning Policy — SemVer; patch/minor/major rules with explicit criteria
- Camera Streaming Architecture — RTSPS vs. TCP-TLS, MJPEG server, freeze recovery, portal tab targeting
- MCP Server Restart Procedure — two-phase: kill+relaunch process, then reconnect MCP client
- Pre-Print Confirmation Gate — mandatory 3-step summary before `print_file` is ever called
- Ephemeral Port Pool — IANA range 49152–49251, shared across REST API and MJPEG streams
- Swagger/OpenAPI Maintenance Standard — docstring completeness, deprecated scaffolding annotation, dual-layer sync
- Pervasive Logging Standard — entry/exit on every method, I/O event logging, all exceptions with `exc_info=True`
- BPM MCP Coverage Standard — every BPM public method must have MCP tool coverage; audit required on every BPM-touching PR
- Camera Feature Tier Parity — RTSPS and TCP-TLS tiers must stay functionally equivalent
- Visual Design Baseline — HUD components, overlay layout, color/typography standards
- FastMCP Response Size Constraint — compressed response threshold, pagination patterns
- Knowledge Module Maintenance Standard — when and how to update baked-in knowledge modules
- Veil of Ignorance Testing Protocol — restricted mode for MCP tool stress-testing; veil state persisted to `~/bambu-mcp/.veil_state`; Veil Threshold Citation Requirement for all numeric detection constants
- Disk Persistence Pattern Standard — how state is persisted across MCP server restarts

**`bambu-fw-fetch` (fwf)**
- Printer Write Protection — firmware upgrade commands require explicit user permission
- Build/Run procedures — dylib loading, cloud binding prerequisite
- libbambu_networking.dylib verified function signatures
- Known API behavior — Cloudflare bypass for download URLs, offline zip structure, MQTT upgrade limitations in LAN mode
- Git Workflow — standard commit/push (no special lock)

**`bambu-mqtt` (bmt)**
- Printer Write Protection + Container API Auth
- Root Cause Fix Rule
- Architecture — standalone MQTT research/utility, ftpsclient, protocol experimentation
- Cross-Model Compatibility Policy

**`webcamd` (wcd)**
- Printer Write Protection + Container API Auth
- Root Cause Fix Rule
- Architecture — MJPEG server, RTSPS ingestion, HAProxy, systemd service
- Cross-Model Compatibility Policy
