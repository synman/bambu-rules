# bambu-printer-app (bpa) AI Agent Guidelines

## ⚠️ PRINTER WRITE PROTECTION — ABSOLUTE, NO EXCEPTIONS, NEVER BYPASSED

**NEVER execute, run, pipe input to, or interact with any command that sends a write or destructive operation to a physical printer — under any circumstances — without the user typing explicit permission in plain text in the current conversation turn.**

This means:
- **Do NOT run `bambu_fw_upgrade.py`** (or any variant) without the user saying so explicitly in their message.
- **Do NOT use `write_bash`, `echo "YES" |`, or any other mechanism to answer a confirmation prompt on behalf of the user.** The confirmation prompt exists for humans only.
- **Do NOT run it as a "test", "dry-run validation", "connection check", or any other pretext.** `--dry-run` is safe; anything else is not.

Prohibited operations (not exhaustive):
- Firmware update commands (`upgrade.start`, any `upgrade.*`)
- MQTT publish to any `device/*/request` topic
- GCode commands
- Configuration changes
- Any FTP/file upload to a printer

This rule applies in **all operating modes** without exception: interactive, autopilot, background agents, scripted execution. Violating this rule has already caused accidental firmware flash attempts on a physical printer twice. There will be no third time.

## ⚠️ CONTAINER API AUTH — MANDATORY, NO EXCEPTIONS, EVERY CALL

See global rules (`~/.copilot/copilot-instructions.md`). Key: `secrets.py get bpm_api_auth`. Never use `security find-internet-password`.

---

> **Global rules** in `~/.copilot/copilot-instructions.md` are always in effect. This file extends them with project-specific guidance. Both must be read and applied together.

## Session Start Protocol (Mandatory)

Both the global rules file and this repo-specific file MUST be read together before any tool call.
See **Session Start Protocol + RULES_PRECHECK** in `~/.copilot/copilot-instructions.md` for the
full enforcement gate, `RULES_PRECHECK` output format, and invalidation rules — they apply here
without exception.

## Root Cause Fix Rule (Mandatory)

When the root cause of a problem has been identified in a specific piece of code, **fix that code**. Do not introduce workarounds, shims, compensating logic, or structural changes elsewhere to paper over a bug when a direct fix is available.

**Hard requirements:**
- If you know where the bug is, fix it there. Full stop.
- Do not add infrastructure, build stages, extra processes, or architectural indirection to compensate for broken code.
- Adding resource cost (CPU, memory, network, build time) to work around a software defect is never acceptable when the defect can be fixed directly.
- Rationalization trap: if you find yourself building a case for NOT fixing the root cause — stop. The existence of the rationalization is itself a signal you are about to make the wrong decision.

**Anti-patterns (never do these):**
- ❌ "I'll add a Dockerfile build stage so the display script doesn't have to handle this edge case"
- ❌ "I'll wrap the call to avoid fixing the underlying function"
- ❌ "This workaround restores expected behavior" — restoring behavior via a workaround is not the same as fixing the bug

**Required pre-fix gate:**
1. Have I identified the file and line(s) where the defect lives? If yes — fix it there.
2. Am I about to change something OTHER than the defective code? If yes — stop and explain why a direct fix is impossible before proceeding.

## Telemetry Mapping Parity Rule (Mandatory)

When adding or changing support for a telemetry field that belongs to an existing family (for example print_option flags), the implementation MUST follow the proven pattern used by sibling fields unless direct evidence proves otherwise.

**Hard requirements:**
- Use the nearest working sibling field as the baseline reference (for example `nozzle_blob_detect`).
- Verify where sibling values are sourced (for example `home_flag` bitfield, `cfg`, `xcam`, or explicit key) before coding.
- If sibling print_option state is sourced from `home_flag`, default new sibling steady-state mapping to `home_flag` as well unless direct evidence proves a different source.
- Do not introduce a new parsing path (for example command-ack-only tracking) unless verified evidence shows sibling parity is invalid.
- If a user says to use a specific field as a reference point, treat that as a mandatory implementation constraint.

**Evidence requirements before coding telemetry mappings:**
1. Confirm upstream behavior in at least one authoritative source (BambuStudio preferred).
2. Confirm current project behavior for sibling fields in source code.
3. Confirm runtime evidence (logs/payloads) and classify it correctly:
	- command ack payloads confirm command acceptance,
	- status payloads or bitfields confirm steady-state telemetry source.
4. Implement using the source that represents steady-state truth unless proven otherwise.

**Anti-fabrication guardrails:**
- Never invent a new telemetry lifecycle model when a proven local pattern already exists.
- Never treat command ack events as a complete substitute for continuous status mapping unless explicitly verified.
- If evidence is mixed, stop and state the uncertainty, then request/collect the missing payload needed to resolve it.

**Parity checklist (must all be true before finalizing):**
- Did I compare against at least one sibling field implementation line-by-line?
- Did I verify the same source-of-truth path for both fields?
- Did I avoid adding special-case logic without explicit evidence?
- Did I document why this field matches or intentionally differs from sibling behavior?

## Verification First - Before Any Work

**Critical Principle**: Never infer architecture, features, behavior, or implementation details from partial code patterns. **ALWAYS verify against actual source code before acting.** This applies to everything: documentation, code analysis, bug fixes, feature work, reverse engineering.

**User assertion handling (mandatory):**
- Treat user technical assertions as hypotheses to verify, not as implementation facts.
- Before changing code based on an assertion, confirm it in source (or authoritative runtime evidence) and record the evidence path.
- If verification is missing or contradictory, do not patch yet; gather the missing proof first or ask a focused clarification.
- Do not claim verification/validation success unless the exact validating tool for that stack has run successfully.

**When in doubt, verify first**:
- ✗ "I see ANNOUNCE_PUSH and pushing namespace, so a push topic probably exists"
- ✓ "I found ANNOUNCE_PUSH. Let me check actual client.subscribe() calls to verify the topic is used"

**Verification checklist for ANY claim about the codebase:**
1. **What does the code actually do?** (Read the implementation, not inferred architecture)
2. **Is this feature really used?** (Search for actual calls/invocations, not just template definitions)
3. **Does this path actually execute?** (Check conditionals, handlers, try/catch blocks - don't assume)
4. **Are there edge cases I missed?** (Search for all usages, not just one example)
5. **What do authoritative reference implementations do?** (Cross-check with ha-bambulab, BambuStudio, OrcaSlicer if unsure)

**Telemetry absence rule (mandatory):**
- For telemetry-key claims, explicitly search local runtime evidence (`tests/` fixtures, logs, captured payloads) for the exact key.
- If the key is absent locally, treat **"missing in local telemetry"** as a first-class interpretation and state it before proposing alternatives.
- Do not imply local presence from upstream references alone; upstream presence can justify parser support but not claim local telemetry coverage.

**Pattern Matching Trap**: Finding evidence of a pattern (e.g., command template, enum value, function name) does NOT mean the feature is implemented or used. Each claim requires independent verification:
- Template exists ≠ command is sent
- Message type defined ≠ message is parsed
- Function exists ≠ function is called
- Namespace present ≠ all sub-features exist

**Truncated File Read Trap (mandatory):**
- A file read that returns truncated output is a **partial read**, not a complete read. Absence of a symbol in truncated output is NOT evidence the symbol is absent from the file.
- When a large file is truncated, always follow up with a **targeted symbol search** (grep for the exact name) before concluding the symbol does not exist.
- Required steps when a file read is truncated:
  1. Note that the read was truncated.
  2. Run a targeted code search for the specific symbol or key.
  3. Only conclude absence if the targeted search also returns no results.

**Prior Session Evidence Rule (mandatory):**
- When prior work cites specific file paths, variable names, or line numbers, treat those as **direct search queries** — look them up verbatim before evaluating credibility.
- Do not perform a broad independent search, fail to find something, and then dismiss the prior evidence. Exhaust targeted verification first.
- Specific cited names (variable names, file paths, function names) are higher-signal starting points than a fresh broad search.

### Mandatory Verification Depth Requirements

**For ANY method/function documentation or analysis:**
1. ✓ **Read the method implementation** - Find the actual implementation in source code
2. ✓ **Trace data flow** - Read line-by-line how data is constructed/transformed (assignments, operations, conditionals)
3. ✓ **Document actual values** - Use real examples from the code (e.g., `f"M104 S{value}\n"`), not placeholders like `{0-280}`
4. ✓ **Check conditionals** - Document when behavior changes based on conditions (e.g., T parameter only when tool_num >= 0)
5. ✗ **Never assume from names or templates** - Finding a template/constant does NOT tell you what actual values are used

**For method signatures and parameters:**
1. ✓ **Read the def line** - Check actual parameter names, types, defaults, order
2. ✓ **Verify parameter order** - Order matters; never rearrange based on assumptions
3. ✓ **Check parameter usage** - Verify parameters are actually used in the implementation
4. ✗ **Never infer from field names** - Field names in data structures ≠ method parameter names/order

**For feature existence claims (topics, endpoints, commands, etc.):**
1. ✓ **Search for actual usage** - Find where it's called, subscribed, registered, or invoked
2. ✓ **Search for implementation** - Verify the actual code that handles/processes it
3. ✓ **Check registration/initialization** - Read where it's set up (e.g., `client.subscribe()`, route decorators)
4. ✗ **Never assume from patterns** - Templates/constants/enums don't prove the feature is active

**Insufficient Verification Examples:**
- ❌ "Found `SET_CHAMBER_AC_MODE` constant → method `set_chamber_ac_mode()` exists"
- ❌ "Saw enum value `ANNOUNCE_PUSH` → feature X is implemented"
- ❌ "Method has `tool_num` parameter → value is always used in output"
- ❌ "Data structure has `ams_id` field → method parameter order is `(ams_id, slot_id)`"
- ❌ "File has `@route('/api/status')` → endpoint returns status data"
- ❌ "Class has `_cache` attribute → caching is implemented"

**Sufficient Verification Examples:**
- ✅ "Read `set_chamber_temp_target()` lines 363-398 → sends `SET_CHAMBER_AC_MODE` internally, no public method"
- ✅ "Grepped for feature usage → only called in test file line 150, not in production code"
- ✅ "Read line 344 → parameter used conditionally: `{'' if tool_num == -1 else ' T' + str(tool_num)}`"
- ✅ "Checked def line 482 → `load_filament(slot_id: int, ams_id: int = 0)` - slot first, not ams"
- ✅ "Read route handler lines 89-102 → returns `{'error': 'not implemented'}`, endpoint is a stub"
- ✅ "Searched for `_cache` usage → only initialized line 45, never read/written, unused attribute"

### Verification Enforcement

**Before making ANY technical claim about the codebase, you MUST:**

1. **Execute verification tools** - Use `read_file`, `grep_search` to gather actual code
2. **Quote exact code** - Reference specific line numbers and actual code snippets in your reasoning
3. **Trace execution paths** - For any feature, trace from invocation → implementation → output
4. **Document conditionals** - Note any if/else that changes behavior (examples: T parameter only when tool_num >= 0, SET_CHAMBER_AC_MODE only when has_chamber_temp)
5. **Verify user-provided assertions** - Confirm user claims in code before editing behavior that depends on them.

**Self-check before claiming anything is true:**
- [ ] Did I read the actual implementation method/function/handler?
- [ ] Did I trace how the data/message/output is constructed line-by-line?
- [ ] Did I check for conditionals that change behavior?
- [ ] Can I cite specific line numbers where this behavior occurs?
- [ ] Did I use actual values from the code, not placeholder syntax?
- [ ] For telemetry keys, did I explicitly check local fixtures/logs and state whether the key is missing locally?
- [ ] For user-provided assertions, did I verify them in source before patching?

**If you cannot check all required boxes above, your verification is INSUFFICIENT.**

## Strict Execution Mode (Mandatory)

Use this workflow for every non-trivial request:
1. **Verify first**: gather concrete evidence with `grep_search` + `read_file` before proposing or applying changes.
2. **Patch minimally**: implement the smallest targeted change that satisfies verified requirements.
3. **Validate immediately**: run diagnostics/tests relevant to touched files right after editing.
   - For Python source files in `bambu-printer-manager/`: run `pre-commit run` from the `bambu-printer-manager/` repo root. This is the authoritative staging check and must pass clean (exit 0).
   - For React/Vite frontend files: `yarn build` is the authoritative parse/build validation.
4. **Do not speculate**: if evidence is insufficient or contradictory, stop and collect the missing payload/code path before changing logic.

**Hard requirements:**
- No speculative architecture or inferred behavior changes without source proof.
- No broad multi-area edits when a single focused patch is sufficient.
- For protocol/telemetry work, always establish source-of-truth hierarchy first (steady-state status/bitfield over command ack).
- After changes, report what was verified, what was changed, and what validation ran.
- For React/Vite frontend files, `yarn build` is the authoritative parse/build validation.
- Do not treat editor diagnostics alone as sufficient when build tooling can validate more accurately.

## Terminal Housekeeping (Mandatory)

Kill every background terminal spawned with `isBackground=true` after its task is complete. Use `kill_terminal` with the terminal ID returned by `run_in_terminal`.

**Hard requirements:**
- Never leave background terminals running after their work is done.
- Kill the terminal immediately after `await_terminal` confirms completion (exit code 0 or handled error).
- Do not batch kills at the end of a session — kill each terminal as soon as it is no longer needed.

## Code Style

**Language**: Python 3.11+ (strict minimum, see `pyproject.toml`)

**Type Hints**: Always use full type annotations. Use `|` for unions (PEP 604), `typing_extensions` for Python <3.12.

**Linting**: Ruff is the authority. Configuration in `pyproject.toml`:
- Line length: 90 characters
- Target: Python 3.11
- Selected rules: B, C, E, F, I, W, B9 (strict enforcement)
- Ignored: E501 (line length is disabled), C901 (complexity), E701
- **Import organization**: Alphabetically sorted, first-party imports under `bpm` namespace

**Docstrings**: Google-style docstrings with triple quotes. See examples in `src/bpm/bambutools.py` and `src/bpm/bambuprinter.py`.

**Private Attributes**: Use underscore prefix (`_hostname`, `_mqtt_port`). Dataclasses use field names with underscores to maintain backing store for properties.

## Cross-Model Compatibility Policy

This project targets **all Bambu Lab printers** (library + frontend), not a single model.

**Default behavior**:
- Preserve existing/legacy cross-model behavior unless there is verified evidence it fails.
- Prefer Bambu Studio-compatible behavior as the broad baseline across models.

**Override gate (strict)**:
- Add model-specific overrides only when the current/legacy logic is proven to fail for that model.
- Missing fields alone are not a legacy-logic failure. Example: if `ams_mapping2` is missing, add it alongside legacy `ams_mapping` first.
- Do not alter existing `ams_mapping` semantics unless breakage is demonstrated with direct evidence.

## Architecture

**Central Class**: `BambuPrinter` (see `src/bpm/bambuprinter.py`) is the main abstraction layer. All printer interaction flows through this class.

**Configuration System**: `BambuConfig` (see `src/bpm/bambuconfig.py`) now uses Python dataclasses (@dataclass). Fields are private (_field_name) with inline docstrings.

**State Management**: `BambuState` (see `src/bpm/bambustate.py`) holds telemetry state. Updated via MQTT telemetry parsing.

**Communication Layers**:
- **MQTT**: Primary for telemetry subscriptions and command transmission (Paho MQTT client)
- **FTPS**: File operations via `src/bpm/ftpsclient/ftpsclient.py`

**Enums & Constants**:
- `bambutools.py` contains all IntEnum/Enum definitions (PrinterModel, Stage, etc.)
- `bambucommands.py` contains MQTT command templates and constants
- Use descriptive enum values with docstrings (see ActiveTool, AirConditioningMode patterns)

**Logging**: All modules use logger = logging.getLogger(LoggerName) where LoggerName = "bpm" (defined in bambutools.py).

## Docker Build & Deployment

**Registry**: `docker-registry.shellware.com/bambu-printer-manager`

**Registry Semantics**:
- The registry is a private Docker Registry v2 instance at `https://docker-registry.shellware.com`.
- Every `dockermake.sh` run produces **two tags**: a dated label (e.g. `2026-03-03-14_01_21`) and `latest`.
- `latest` is always a copy of (same digest as) the most recent dated tag — it is promoted by the `skopeo copy` step.
- The Registry v2 API is available without authentication (no `--creds` flag needed for `curl`). Manifest digests can be fetched with: `curl -s -I -H "Accept: application/vnd.docker.distribution.manifest.v2+json" https://docker-registry.shellware.com/v2/bambu-printer-manager/manifests/<tag> | grep -i docker-content-digest`
- Tags can be listed with: `curl -s https://docker-registry.shellware.com/v2/bambu-printer-manager/tags/list`
- **Deleting a manifest by digest removes all tags sharing that digest.** Always compare digests before deleting to ensure `latest` and its paired dated tag are never inadvertently removed.

**Pre-requisites** (one-time builder setup):
```bash
# Local node
docker buildx create --name multi-platform-builder \
  --node local_builder-<arch> --platform linux/<arch>

# Remote node (append)
docker buildx create --name multi-platform-builder \
  --append --node remote_builder-<arch> --platform linux/<arch> \
  ssh://user@host
```

**Build & push** (run from repo root):
```bash
ssh-add ~/.ssh/docker/builder          # load build SSH key first
docker buildx use multi-platform-builder
label=$(date +%Y-%m-%d-%H_%M_%S)
docker buildx build --no-cache \
  --platform=linux/amd64,linux/arm64 \
  --push \
  -t docker-registry.shellware.com/bambu-printer-manager:$label .
```

**Promote label → latest**:
```bash
skopeo copy \
  docker://docker-registry.shellware.com/bambu-printer-manager:$label \
  docker://docker-registry.shellware.com/bambu-printer-manager:latest \
  --override-os linux --multi-arch all
```

**Sync to Docker Hub** (Docker Hub Release — separate lifecycle event):
```bash
BPM_SECRETS_PASS=... ./dockerhub_release.sh
```

**Canonical helper**: `./dockermake.sh` automates the ssh-add → buildx → skopeo copy (promote to latest) → **Watchtower API trigger** sequence. After the image is pushed, `dockermake.sh` immediately calls the Watchtower HTTP API to trigger a refresh and then waits ~30 seconds before printing Watchtower session logs to confirm containers were updated. `BPM_SECRETS_PASS` is sourced from `~/.zshenv` automatically.

**Docker Hub Release** (separate lifecycle event): run `./dockerhub_release.sh` to sync all tags from the private registry to Docker Hub. Requires `BPM_SECRETS_PASS` in the environment.

**Container host**: `bpm-a1` and `bpm-h2d` run on `ds1621` (Synology NAS, 10.151.51.24). Docker binary is `/usr/local/bin/docker`. SSH as default user. Example: `ssh ds1621 "/usr/local/bin/docker restart bpm-a1 bpm-h2d"`

**Container log monitoring (mandatory)**: Always use `docker exec` on `ds1621` to read container logs — never the host's `~/bambu-printer-app/api/output.log` (stale bind-mount). The live log is at `/bambu-printer-app/api/output.log` inside the container:
- Snapshot: `ssh ds1621 "/usr/local/bin/docker exec bpm-a1 tail -100 /bambu-printer-app/api/output.log"`
- Live tail: `ssh ds1621 "/usr/local/bin/docker exec bpm-a1 tail -f /bambu-printer-app/api/output.log"`
- Same pattern for `bpm-h2d`. The `/api/dump_log` HTTP endpoint serves the same file but is less convenient for tailing.
- **Verbosity**: Default log level is WARNING. `GET /api/toggle_verbosity?verbose=true` is the finest-grained level — enables DEBUG logging AND sets `config.verbose=true` for full MQTT message detail. It is a **blind toggle**: each call flips between DEBUG and INFO regardless of `?verbose=true`. Call it repeatedly until the response shows `"level": "DEBUG"` AND `"verbose": true` — that combination is the finest-grained mode. Calling without `?verbose=true` will set verbose=false on that call even if level becomes DEBUG.

**Watchtower (mandatory awareness)**: Watchtower runs on `ds1621` and automatically pulls updated images and restarts containers when a new `latest` tag is pushed to the registry. `WATCHTOWER_POLL_INTERVAL=60` — it checks every 60 seconds, so containers are updated within ~1 minute of `dockermake.sh` completing. **Do not manually `docker pull` + `docker restart` after a `dockermake.sh` run** — Watchtower will handle it. Manual restarts are only needed for config-only changes (e.g. nginx volume edits) that don't involve a new image.

To trigger an immediate Watchtower update instead of waiting for the 60s poll, use the HTTP API. First the Watchtower container must be configured with `WATCHTOWER_HTTP_API_TOKEN=<token>` and the `--http-api-periodic-polls` flag (so periodic polling continues alongside manual triggers).

Store/retrieve the Watchtower API token using the encrypted secrets store (`~/.bpm_secrets`) the same way Docker Hub credentials are handled:
```bash
# Store once
BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py set watchtower_http_api_token <token>

# Retrieve at runtime
WT_TOKEN=$(BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py get watchtower_http_api_token)
```

Preferred host endpoint (current deployment):
```bash
WT_TOKEN=$(BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py get watchtower_http_api_token)
curl -sf -X POST -H "Authorization: Bearer $WT_TOKEN" http://10.151.51.24:9180/v1/update
```

Equivalent command from the container network on `ds1621`:
```bash
WT_TOKEN=$(BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py get watchtower_http_api_token)
ssh ds1621 "/usr/local/bin/docker exec bpm-a1 curl -sf -X POST -H 'Authorization: Bearer $WT_TOKEN' http://watchtower:8080/v1/update"
```

To verify that the trigger was accepted and processed, check Watchtower logs for the HTTP API trigger and session completion:
```bash
ssh ds1621 '/usr/local/bin/docker logs --tail 200 watchtower 2>&1 | grep -Ei "Updates triggered by HTTP API request|Session done|Checking all containers"'
```

**API server (gunicorn — mandatory)**: `api/api.py` MUST be served by `gunicorn` with the `gthread` worker class. `waitress` does not expose the raw socket in the WSGI environ, so `simple_websocket` (used by flask-socketio) cannot perform WebSocket upgrades. The correct launch in `docker/run.sh` is:
```bash
gunicorn --worker-class gthread --workers 1 --threads 8 --bind 127.0.0.1:5000 api:app
```
`gunicorn` MUST be present in the Dockerfile (`RUN pip install --no-cache gunicorn`). Do not revert to `waitress` — polling works with waitress but WebSocket upgrades will always fail (`RuntimeError: Cannot obtain socket from WSGI environment`). If the API returns 502, check: (1) is `gunicorn` installed in the container venv, (2) is port 5000 listening (`netstat -tlnp | grep 5000`), (3) check `output.log` for startup errors.

**Remote builder node**: The AMD64 build node is `shells-mac-mini.local` (10.151.51.189, user `shell`). If `dockermake.sh` fails with `kex_exchange_identification: Connection reset by peer`, sshd on the Mac mini has crashed. Recovery steps:
1. Verify the host is up: `ping shells-mac-mini.local`
2. Open Screen Sharing: `open vnc://shells-mac-mini.local`
3. In a terminal on the remote machine: `sudo launchctl kickstart -k system/com.openssh.sshd`
4. Re-run `./dockermake.sh`

**Registry Housekeeping (mandatory — after every `dockermake.sh` run and when explicitly requested)**: The registry accumulates one dated tag per `dockermake.sh` run. After every successful build+push, prune all dated tags except the one paired with `latest` (they share a digest — confirm before deleting). Also execute when explicitly asked. Safe pruning procedure:
```bash
REGISTRY="docker-registry.shellware.com"
REPO="bambu-printer-manager"
LATEST_DIGEST=$(curl -s -I -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  https://$REGISTRY/v2/$REPO/manifests/latest | grep -i docker-content-digest | awk '{print $2}' | tr -d '[:space:]')

for tag in $(curl -s https://$REGISTRY/v2/$REPO/tags/list | python3 -c \
  "import sys,json; [print(t) for t in json.load(sys.stdin).get('tags',[]) if not t.startswith('latest')]"); do
  DIGEST=$(curl -s -I -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    https://$REGISTRY/v2/$REPO/manifests/$tag | grep -i docker-content-digest | awk '{print $2}' | tr -d '[:space:]')
  if [ "$DIGEST" = "$LATEST_DIGEST" ]; then
    echo "KEEP $tag (paired with latest)"
  else
    echo "DELETE $tag"
    curl -s -o /dev/null -w "%{http_code}\n" -X DELETE \
      -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
      "https://$REGISTRY/v2/$REPO/manifests/$DIGEST"
  fi
done
```

---

## Build and Test

**Git Workflow**:
- **Never run `git commit` or `git push`.** These are the user's actions, not yours.
- `git add`/`git rm` and pre-commit hooks are permitted. Stop there.
- Report what is staged and ready — do not commit under any circumstances.

**Frontend Validation (Mandatory):** After any JS/JSX/TS/TSX edit, `yarn build` is the authoritative validation — do not claim success until it passes clean.

**Frontend Build (bambu-printer-app):**
```bash
yarn build
```

**Package Manager Policy (bambu-printer-app):**
- Use `yarn` commands for frontend dependency and build tasks.
- Do not substitute `npm` commands.

**Build**:
```bash
./make.sh
# Cleans dist/, builds with python -m build via setuptools
```

**Staging Validation for bambu-printer-manager (Mandatory)**:
```bash
cd /path/to/bambu-printer-manager && pre-commit run
# Run after any Python source edit in bambu-printer-manager/.
# Must pass clean (exit 0) — ruff lint + format + all configured hooks.
```

**Test Execution**:
- Test files in `tests/` directory (e.g., `tests/h2d-unit-test.py`)
- Test JSON fixtures for printer telemetry states provided (h2d-*.json, a1-*.json)
- Run with: `python tests/<test-file>.py`

**Documentation Build & Validation**:
```bash
mkdocs build --clean  # Generates from mkdocs.yml with full validation
```
**Critical**: After creating or modifying any `.md` files in `docs/` or any Python docstrings (since `mkdocstrings-python` renders them into the site), ALWAYS validate with `mkdocs build --clean` to catch broken links, missing anchors, and formatting issues before committing. No INFO-level warnings should be present in the output.

## Project Conventions

**Naming Patterns**:
- Classes: PascalCase (BambuPrinter, BambuConfig, PrinterCapabilities)
- Methods/Functions: snake_case
- Constants: SCREAMING_SNAKE_CASE in bambucommands.py and bambutools.py
- Private attributes: _leading_underscore

**Enum Usage**: All state values are IntEnum/Enum to provide type safety. Always reference via enum rather than raw integers (e.g., `PrinterModel.H2D` not `2`). Document each enum value with docstring commentary on its meaning.

**Dataclass Fields**: Private backing fields with field() for non-init state. Example from bambuconfig.py:
```python
_firmware_version: str = field(default="", init=False, repr=False)
_auto_recovery: bool = field(default=True, init=False, repr=False)
```

**Paths**: Use `pathlib.Path` for all filesystem operations. See bpm_cache_path pattern in BambuConfig.

**Deprecated Features**: Mark with `@deprecated` from typing_extensions. See bambuprinter.py for examples.

## Container API Endpoints

**Mandatory interface rule (no exceptions):** To query live printer state, always use the container API over HTTPS. Do **not** instantiate `BambuPrinter` directly for read-only state queries. The containers (`bpm-h2d`, `bpm-a1`) already hold active MQTT sessions with the printers. Opening a second `BambuPrinter` session from an agent script creates a duplicate MQTT client, is wasteful, and risks MQTT interference. The only legitimate reason to instantiate `BambuPrinter` directly is to send a command that has no container API endpoint (e.g. `send_anything()`).

**Correct pattern for reading live state:**
```bash
AUTH=$(python ~/bambu-printer-manager/secrets.py get bpm_api_auth)
# bpm_api_auth is already base64-encoded user:password — use as Authorization header directly
curl -sk -H "Authorization: Basic $AUTH" "https://bambu-h2d.shellware.com/api/printer"
```
Use `https://bambu-a1.shellware.com/api/printer` for the A1.

The bambu-printer-app runs in containers accessible at these addresses on the local network:

| Printer | Host | Resolved IP |
|---------|------|-------------|
| H2D | `bambu-h2d.shellware.com` | 10.151.51.24 |
| A1  | `bambu-a1.shellware.com`  | 10.151.51.x  |

**Access pattern (mandatory):**
- Protocol: HTTPS only. Port 80 serves a redirect page; use **port 443**.
- Context root: `/api/` — all Flask routes are under `/api/...` (e.g. `https://bambu-h2d.shellware.com/api/printer`)
- Auth: HTTP Basic. Credentials are stored in `~/.bpm_secrets` under key `bpm_api_auth` (format `user:password`):
  ```bash
  AUTH=$(python ~/bambu-printer-manager/secrets.py get bpm_api_auth)
  # bpm_api_auth is already base64-encoded user:password — use as Authorization header directly
  ```
- Use with curl: `curl -sk -H "Authorization: Basic $AUTH" "https://bambu-h2d.shellware.com/api/printer"`

**Printer serials:**
- H2D: `0948AD5B2700913` (LAN IP: `bambu-h2d-printer.shellware.com` / 10.151.51.110)
- A1:  `03919A3B2000654` (LAN IP: `bambu-a1-printer.shellware.com` / 10.151.51.135)

**Sending arbitrary MQTT commands**: The container API has no `send_anything` endpoint. Use `BambuPrinter.send_anything()` from `bambu-printer-manager` directly:
```python
import time, sys
sys.path.insert(0, '/path/to/bambu-printer-manager/src')
from bpm.bambuprinter import BambuPrinter
from bpm.bambuconfig import BambuConfig

cfg = BambuConfig(hostname=IP, access_code=ACCESS, serial_number=SERIAL)
p = BambuPrinter(config=cfg)
p.start_session()
time.sleep(3)
p.send_anything('{"update": {"name": "Shell\'s A1", "sequence_id": "0"}}')
time.sleep(2)
```
Credentials via `secrets.py`: `bambu-a1-printer_ip`, `bambu-a1-printer_access_code`, `bambu-a1-printer_serial` (same pattern for h2d).

**Printer rename MQTT message**: `{"update": {"name": "<new name>", "sequence_id": "0"}}` — published to `device/{serial}/request`. Printer responds with `{"update": {"name": "...", "reason": "success", "result": "success", "sequence_id": "0"}}`. The container logs this as `WARNING: unknown message type` — no handler exists for `update` message type.

**SSDP discovery (port 2021)** `[VERIFIED: bpm source — bambudiscovery.py:153, sock.bind(("", 2021))]`: Printers broadcast on UDP port 2021. Use `BambuDiscovery` or bind directly. Key fields: `dev_name`, `dev_version`, `location` (printer IP), `dev_bind`, `dev_connect`. The `BambuDiscovery` class deduplicates by USN — for monitoring changes, bind the socket directly and compare fields across broadcasts. `DiscoveredPrinter.fromData()` parses raw UDP payloads.

**H2D firmware upgrade state fields** (in `push_status`) `[PROVISIONAL: ha-bambulab models.py — UPGRADE_SUCCESS and dis_state:1/3 confirmed; dis_state:2 and FLASH_START not found in ha-bambulab examples; resolve before acting on these specific values]`: `upgrade_state.status` (`FLASH_START`, `UPGRADE_SUCCESS`), `upgrade_state.progress` (0–100), `upgrade_state.module` (`ap` = main Linux image), `upgrade_state.message`. `dis_state: 2` = actively upgrading; `dis_state: 3` = complete/failed.

**trigger_printer_refresh**: `GET /api/trigger_printer_refresh` forces the container to re-query the printer. Always call this after a printer reboot (firmware upgrade, etc.) before reading `firmware_version` — the cached value is stale until refreshed.

## Integration Points

**MQTT Topics** `[VERIFIED: ha-bambulab bambu_client.py:613-624]`: Commands sent to `device/{serial}/request` namespace. Telemetry (including push_status updates) subscribed via `device/{serial}/report` only. Note: "push" refers to message types within the report topic, not a separate subscription.

**FTPS Operations**: IoTFTPSClient handles FTPS file transfers. Used for 3MF uploads via `src/bpm/bambuprinter.py`.

**External Dependencies**:
- `paho-mqtt`: MQTT client library
- `webcolors`: Color name/hex conversions
- `mkdocstrings-python`: Doc generation
- `typing-extensions`: Backport typing features for Python <3.12

**File Structure**: Source in `src/`, tests in `tests/`. Excluded from packaging: `bpm/ftpsclient` (see pyproject.toml setuptools config).

## Reference Implementations

For telemetry and data mapping questions, consult these authoritative public repositories:

- **[Official bpm/bpa documentation](https://synman.github.io/bambu-printer-manager/)**: Official mkdocs-generated API reference for all public classes in `bambu-printer-manager` and `bambu-printer-app`. **Consult this first for any bpm API question — before reading installed package source.** Authoritative for method signatures, parameter semantics, behavior differences between printer models, and configuration field descriptions.
- **[BambuStudio](https://github.com/bambulab/BambuStudio)**: Official Bambu Lab client implementation with complete telemetry mapping and protocol definitions
- **[OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer)**: Community fork with enhanced telemetry handling and data structure examples
- **[ha-bambulab](https://github.com/greghesp/ha-bambulab)**: Home Assistant integration with comprehensive MQTT topic and payload documentation
- **[ha-bambulab pybambu](https://github.com/greghesp/ha-bambulab/tree/main/custom_components/bambu_lab/pybambu)**: Low-level Python MQTT client implementation and message parsing
- **[Bambu-HomeAssistant-Flows](https://github.com/WolfwithSword/Bambu-HomeAssistant-Flows)**: Workflow patterns and integration examples using Bambu Lab printers
- **[OpenBambuAPI](https://github.com/Doridian/OpenBambuAPI)**: Alternative API implementation with detailed protocol documentation
- **[X1Plus](https://github.com/X1Plus/X1Plus)**: Community firmware and protocol analysis for extended printer capabilities
- **[bambu-node](https://github.com/THE-SIMPLE-MARK/bambu-node)**: Node.js implementation providing cross-language verification and alternative patterns

These references are valuable for understanding MQTT message structures, telemetry field mappings, printer state transitions, and edge cases across multiple implementations.

## Security & Sensitive Areas

**Credentials**: Access codes are 8-character strings passed via BambuConfig. Never log or display access_code values.

**Credential Exposure Rule (Mandatory)**: Never write real passwords, tokens, API keys, or secrets into any documentation, instructions, or source file. Always use a placeholder (e.g., `<dockerhub-token>`, `<password>`) in documentation and code examples. This applies to all files including copilot-instructions.md, README.md, and any other docs.

**Secrets Management**: Project secrets (Docker Hub token, access codes, etc.) are stored in `~/.bpm_secrets` using `../bambu-printer-manager/secrets.py`. The file is Fernet-encrypted (AES-128-CBC + HMAC-SHA256) with a PBKDF2-derived key and is chmod 600. Master password is read from the `BPM_SECRETS_PASS` env var or an interactive prompt. Never hard-code or inline real secret values; retrieve them at runtime via `secrets.py get <key>`.

**`BPM_SECRETS_PASS` location**: The master password is exported in `~/.zshenv` (sourced automatically by zsh for all invocations). Scripts that need it (`dockermake.sh`, `dockerhub_release.sh`) source `~/.zshenv` explicitly at startup to ensure it is available regardless of how they are invoked.

```bash
# Store a secret
BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py set dockerhub_token <value>

# Retrieve at runtime (e.g. in a shell script)
TOKEN=$(BPM_SECRETS_PASS=... python ../bambu-printer-manager/secrets.py get dockerhub_token)

# List all keys
python ../bambu-printer-manager/secrets.py list
```

**SSL/TLS**: MQTT connections use SSL with certificate validation. See ssl.create_default_context() usage in BambuPrinter.__init__.

**Path Traversal**: Validate all user-provided paths, especially in FTPS operations (ftpsclient/_client.py).
