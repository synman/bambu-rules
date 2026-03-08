# bambu-mqtt (bmt) AI Agent Guidelines

## ⚠️ PRINTER WRITE PROTECTION — ABSOLUTE, NO EXCEPTIONS, NEVER BYPASSED

See global rules (`~/.copilot/copilot-instructions.md`). This rule applies in all operating modes without exception.

## ⚠️ CONTAINER API AUTH — MANDATORY, NO EXCEPTIONS, EVERY CALL

See global rules (`~/.copilot/copilot-instructions.md`). Key: `secrets.py get bpm_api_auth`. Never use `security find-internet-password`.

---

> **Global rules** in `~/.copilot/copilot-instructions.md` are always in effect. This file extends them with project-specific guidance. Both must be read and applied together.

## Session Start Protocol (Mandatory)

At the start of every task, **both** the global rules file and this repo-specific file MUST be read and applied together before any tool call related to the task.

**Hard requirements:**
- Read `~/.copilot/copilot-instructions.md` (global rules) — always in effect for every session, every repo.
- Read `.github/copilot-instructions.md` in the repo root — repo-specific rules extend or override global rules but do NOT replace them.
- Both files must be synergized: all rules from both are simultaneously active.
- Failure to read either file is a rules violation.

### Mandatory Preflight Enforcement (No Exceptions)

- **Mandatory Preflight Evidence**: Before any non-read action, the agent must first output a `RULES_PRECHECK` line that names both files read.
- **Fail-Closed**: If either rules file is unreadable or not read in this turn, the agent must stop and perform no further actions.
- **Invalidation Rule**: Any task action before `RULES_PRECHECK` is automatically invalid and must be re-run from the start.
- **No Memory Substitution**: Prior-session memory of rules does not satisfy this requirement; each new task requires a fresh read.

## Root Cause Fix Rule (Mandatory)

See global rules (`~/.copilot/copilot-instructions.md`). Fix bugs at their source; never paper over them with workarounds.

## Code Style

**Language**: Python 3.11+ (shebang targets `~/.virtualenvs/bpm/bin/python`)

**Type Hints**: Use full type annotations where applicable. Use `|` for unions (PEP 604).

**Linting**: Follow the same Ruff configuration used in `bambu-printer-manager` (line length 90, rules B, C, E, F, I, W).

**Naming**:
- Functions/variables: `snake_case`
- Constants: `SCREAMING_SNAKE_CASE`
- Private attributes: `_leading_underscore`

## Architecture

This is a standalone Python MQTT research/utility project. It is **not** a Flask app, library, or containerized service.

**Main entry point**: `bambu-mqtt.py` — connects directly to the printer via MQTT and FTPS for experimentation, debugging, and protocol research.

**Communication**:
- **MQTT**: Direct Paho MQTT client connecting to the printer. Topics follow `device/{serial}/request` (commands) and `device/{serial}/report` (telemetry).
- **FTPS**: `ftpsclient/` module handles file transfers directly to/from printer SD card.

**Utility scripts**:
- `get_all_hms.py` — fetches and decodes HMS error codes
- `get_filament_data.py` — fetches filament/spool data

**Data files**:
- `filament.json` — filament catalog data
- `hms_en_master.json` — HMS error code lookup table
- `extrusion_cali.gcode` — extrusion calibration gcode

**go2rtc**: `go2rtc.yaml` configures the go2rtc camera proxy (binary excluded from repo via `.gitignore`).

**Virtualenv**: `~/.virtualenvs/bpm/` (shared with bambu-printer-manager).

## Cross-Model Compatibility Policy

See bpa rules for the full policy. Short form: default to Bambu-Studio-compatible behavior. Add model-specific overrides only when the existing logic is proven to fail for that model with direct evidence.

## Integration Points

**MQTT Topics**: Commands to `device/{serial}/request`, telemetry from `device/{serial}/report`.

**Credentials**: Printer IP, access code, and serial are stored in `~/.bpm_secrets`. Retrieve via:
```bash
python ~/bambu-printer-manager/secrets.py get bambu-h2d-printer_ip
python ~/bambu-printer-manager/secrets.py get bambu-h2d-printer_access_code
python ~/bambu-printer-manager/secrets.py get bambu-h2d-printer_serial
```
Same pattern for `bambu-a1-printer_*`.

**Printer serials:**
- H2D: `0948AD5B2700913` (LAN IP: `bambu-h2d-printer.shellware.com` / 10.151.51.110)
- A1:  `03919A3B2000654` (LAN IP: `bambu-a1-printer.shellware.com` / 10.151.51.135)

## Reference Implementations

For telemetry and protocol questions, consult the same authoritative sources as bpa:

- **[BambuStudio](https://github.com/bambulab/BambuStudio)**: Official client — complete telemetry mapping and protocol definitions
- **[ha-bambulab pybambu](https://github.com/greghesp/ha-bambulab/tree/main/custom_components/bambu_lab/pybambu)**: Low-level Python MQTT parsing
- **[OpenBambuAPI](https://github.com/Doridian/OpenBambuAPI)**: Alternative implementation with detailed protocol docs
- **[OrcaSlicer](https://github.com/OrcaSlicer/OrcaSlicer)**: Community fork with enhanced telemetry handling
- **[bambu-node](https://github.com/THE-SIMPLE-MARK/bambu-node)**: Node.js cross-language verification

## Security & Sensitive Areas

**Credentials**: Access codes, IPs, and tokens are 8-character or short strings. Never log or display them.

**`token` file**: Contains a sensitive credential. Excluded from repo via `.gitignore`. Never commit, display, or reference its contents.

**`printer.cer.fake.cer`**: Excluded from repo. Never commit certificate files.

**Secrets management**: All secrets stored in `~/.bpm_secrets` via `secrets.py`. Never hard-code or inline real values.

**`BPM_SECRETS_PASS`**: Master password exported in `~/.zshenv`. Scripts source it at startup.

**SSL/TLS**: MQTT connections use SSL with certificate validation.
