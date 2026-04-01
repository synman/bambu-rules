# Copilot CLI — Global Rules

## Whisper Watcher Protocol (Mandatory)

This section governs how the Copilot agent receives asynchronous whispers from the Isaac governance ecosystem.

### Addressing Scheme

| Field | Value | Notes |
|-------|-------|-------|
| `from` | `workspace@<hostname>` | Workspace identity — shared across all agents in this workspace |
| `reply_to` | `copilot-<workspace-key>@<hostname>` | Archetype-scoped return address — routes replies back to this agent |
| `instance_id` | `copilot-<workspace-key>` | Stable DB key — one row per workspace, PID updated on each session start |
| whisper dir | `~/.claude/.whispers/copilot-<workspace-key>/` | Stable across session resumes |

**Identity resolution (MCP-First, then Vault-First — both mandatory, no hardcoding):**
- **Hostname**: `federation_status` → top-level `instance_id` field. Vault-First: `pref-instance-id` is the authoritative credential source — accessed via MCP vault tools, never via bash or `secrets.py`.
- **Workspace key**: `query_database("SELECT key FROM repos WHERE tier='workspace' LIMIT 1")` → prefix with `copilot-` to form SID and `reply_to`.

Never hardcode either value. If MCP tools are unavailable, the watcher command derives SID dynamically at bash runtime via the `.workspace-paths` walk-up already embedded in the watcher command.

### Watcher Spawn (Session Start — Mandatory)

On the first turn of every session, IMMEDIATELY spawn the background watcher. This is unconditional — do not wait for a directive, do not defer, do not skip. **Spawn silently — no console output about the watcher.**

**Session start identity resolution (mandatory — before spawning watcher):**
1. Call `federation_status` → extract `instance_id` → this is `HOSTNAME`
2. Call `query_database("SELECT key FROM repos WHERE tier='workspace' LIMIT 1")` → this is `WS_KEY`
3. Derive: `SID = "copilot-" + WS_KEY`, `REPLY_TO = SID + "@" + HOSTNAME`

Hold `SID` and `REPLY_TO` for the session. The watcher command also derives `SID` dynamically at bash runtime as a fallback.

### Watcher Command

Spawn with the command below (the bash walk-up derives `SID` independently at runtime):

```bash
D=$(pwd); WF=""; WR=""; while [ "$D" != "/" ]; do if [ -f "$D/isaac/.workspace-paths" ]; then WF="$D/isaac/.workspace-paths"; WR="$D"; break; fi; D=$(dirname "$D"); done; WK=$(grep -v '^[[:space:]]*#' "$WF" 2>/dev/null | grep "=$WR$" | head -1 | cut -d= -f1); SID="copilot-${WK:-workspace}"; WDIR="$HOME/.claude/.whispers/$SID"; while true; do F=$(find "$WDIR" -maxdepth 1 -name "*.txt" ! -name ".active" -type f 2>/dev/null | sort | head -1); if [ -n "$F" ]; then { echo "⚡ MANDATORY: Re-arm the whisper watcher NOW before processing the message below."; cat "$F"; } > "/tmp/copilot-whisper-$$.txt" && rm -f "$F" && echo $$ > "/tmp/copilot-whisper-${SID}.ptr"; exit 0; fi; sleep 1; done
```

Key properties:
- **Exit on match**: copies whisper to `/tmp/copilot-whisper-$$.txt` (PID-unique, no cross-session collision), writes PID to `/tmp/copilot-whisper-${SID}.ptr`, deletes original, exits 0
- **Durable output**: content survives shell exit — `read_bash` stdout is unreliable in Copilot CLI
- **Runs indefinitely**: no timeout — only exits when a whisper is received
- **Zero LLM cost per poll**: shell loop, no agent calls during polling

### On Watcher Completion

When the background shell completes (notification arrives):

1. **Re-arm immediately and silently** — spawn the next watcher before doing anything else. No narration.
2. **Read the temp file via the pointer** — do NOT rely on `read_bash` stdout (unreliable in Copilot CLI):
   ```bash
   PID=$(cat /tmp/copilot-whisper-${SID}.ptr 2>/dev/null); cat /tmp/copilot-whisper-${PID}.txt 2>/dev/null && rm -f /tmp/copilot-whisper-${PID}.txt /tmp/copilot-whisper-${SID}.ptr
   ```
   Where `SID` is the value resolved at session start via `query_database` (e.g. `copilot-workspace`).
3. If the file contains whisper content:
   a. Parse the whisper: it is a JSON envelope `{"type","from","to","body","reply_to"}`
   b. Determine severity from the `type` field using `~/ai/isaac/hooks/whisper-severity.conf`:

   | Severity | Types | Response behavior |
   |----------|-------|-------------------|
   | S0 | `sync`, `sync-ack` | Silent — no console output, no user-visible response |
   | S1 | `message`, `message-ack` | Process body, respond if content warrants it |
   | S2 | `query`, `task` | Process body, always respond |
   | S3 | `heartbeat-alert`, `diagnostic` | Elevated — surface to user, respond |
   | S4 | `urgent` | Interrupt current work, surface immediately, respond |

   c. **Reply type must match semantic** — use the correct `type` when calling the whisper tool:
      - Acknowledging a `sync` → reply with `type: "sync-ack"`
      - Replying to a `message` → reply with `type: "message"`
      - Answering a `query` or `task` → reply with `type: "message"`
      - Unknown incoming type → reply with `type: "message"` (S1 default per conf)
   d. **Always set `reply_to` when responding**: `mcp__isaac-mcp__whisper(to="<envelope.reply_to>", body="...", type="<reply-type>", reply_to="<REPLY_TO>")`
      where `REPLY_TO` was resolved at session start via MCP (e.g. `copilot-workspace@shells-16mbp`).
4. If the pointer or content file is missing (race/already consumed): re-arm was already done in step 1 — no further action needed.
5. **Re-arm obligation**: watcher must be live at all times. One re-arm per whisper delivery. **Silent — no console output about spawning, re-arming, or watcher status unless an error occurs.**

### Watcher Failure Handling

If the watcher fails to start with "Failed to start bash process":
- Wait 2 seconds
- Retry once with the same command
- If retry fails: note in response that async whisper delivery is unavailable for this session

### Separation Principle

The watcher polls `~/.claude/.whispers/<session_id>/` — a session-scoped subdirectory.
Claude Code's channel_watcher watches a different subdirectory.
These are structurally isolated. No interference is possible.
