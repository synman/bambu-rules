# BPM → bambu-mcp Traceability & Coverage Audit Report

## Audit Header

| Field | Value |
|-------|-------|
| **Date** | 2026-03-10T22:02:05Z |
| **Targeted baseline** | `rest-compliance-printer-param-2` · v1.0.3 · REST compliance + Agent B fixes + all gaps closed |
| **Audit version** | v4.1 (Full — not delta; all gaps resolved) |
| **BPM package path** | `~/bambu-mcp/.venv/lib/python3.12/site-packages/bpm/` |

## Repo SHAs

| Repo | Branch | HEAD SHA |
|------|--------|----------|
| `~/bambu-mcp` | `main` | `f5b138efbab8c9ffd7a911e7d9363d14d406822e` |
| `~/bambu-printer-manager` | `devel` | `4c3ac821c6498ba1c8c11020acdf9ec6c16329b6` |
| `~/bambu-printer-app` | `main` | `7d8c60fecb41d2aa7f80e755033e1317e080c25c` |
| `~/bambu-fw-fetch` | `main` | `080005040db31734eeac01d8bf958a5860eb7397` |
| `~/GitHub/bambu-mqtt` | `main` | `faa3ba00144125513a00f8d1e75c1a0cecc14cb5` |
| `~/GitHub/webcamd` | `bambu` | `4aac6d07e9c5e60d0aadf4301e0e21ff67806686` |

---

## Phase 1 Audit Findings

| Agent | Required Findings Statement |
|-------|-----------------------------|
| **A1 — Post-audit** | Post-audit complete: 0 behavioral gaps found, 0 rules written. |
| **A2 — BPM coverage** | Coverage audit complete: 95 bpm items checked, 8 gaps found, all resolved / 9 items added to exclusions table. |
| **Agent B — MCP↔HTTP contract** | MCP↔HTTP contract audit complete: 65+47+10 items checked, 3 failures found, all resolved. Type I: all HUD components documented in start_stream() docstring. |
| **Agent C — Repo state** | Repo state: all 6 repos clean and pushed. |

---

## Agent B Fixes (3 resolved)

| Type | Route / Item | Finding | Fix Applied |
|------|-------------|---------|-------------|
| **D** | `GET /api/analyze_active_job` | Used `_rargs().get("printer","")` directly — bypassed `_get_printer()` injection; `printer` showed `required=false` in OpenAPI | Changed to `_get_printer(_rargs())` — `printer` now `required=true`; adds DEFAULT_PRINTER fallback and early 400 on no connection |
| **Write guard** | `GET,POST /api/download_file_from_printer` | Knowledge section missing `⚠️ WRITE OPERATION` note despite MCP `download_file` having `user_permission` | Added `⚠️ WRITE OPERATION` note to `http_api_files.py` |
| **Write guard** | `POST /api/set_spool_k_factor` | Knowledge section missing `⚠️ WRITE OPERATION` note | Added `⚠️ WRITE OPERATION` note to `http_api_ams.py` |

**Gate 3 result:** 0 missing printer params, 58 routes with `printer` param, all `required=true` ✅

---

## A2 Gaps Resolved (all 9 items)

| Item | Type | Resolution |
|------|------|-----------|
| `BambuPrinter.cached_sd_card_contents` | C | Added to exclusions — internal cache, surfaced via `list_sdcard_files` |
| `BambuPrinter.cached_sd_card_3mf_files` | C | Added to exclusions — internal cache, surfaced via SD card tools |
| `BambuPrinter.service_state` | C | Added to exclusions — surfaced via `get_printer_connection_status` |
| `BambuPrinter.skipped_objects` (deprecated) | C | Added to exclusions — deprecated, data in `get_printer_state.skipped_objects` |
| `BambuPrinter.config` property | A | Added to exclusions — internal accessor, data in `/api/printer` JSON |
| `bambutools.cache_write/read/delete/make_cache_key` | A | Added to exclusions — internal cache utilities |
| E7 — AMS humidity visibility threshold | I | **Fixed** — `behavioral_rules_camera.py`: threshold `hIdx in {1,2}` documented with color codes |
| E8 — Heating animation (`htg()`) | I | **Fixed** — `behavioral_rules_camera.py`: CSS `heating` pulse class documented with trigger condition |
| `BambuState.stat` / `BambuState.fun` | H | Added to exclusions — opaque firmware fields, no agent semantics |

---

## Knowledge Consistency Fixes (2 resolved)

| Item | Fix |
|------|-----|
| `api_reference_ams.py` — humidity index | Added "**Higher numbers mean DRIER — the scale is counterintuitive.**" matching tool docstrings |
| `behavioral_rules_print_state.py` — stage code table | Added "action-relevant stages" note + `get_job_info()` cross-reference for complete enumeration |

---

## All Checks Summary

| Check | Result |
|-------|--------|
| Gate 1: Knowledge content placement | ✅ All content inside `*_TEXT` variables |
| Gate 3: Printer param parity | ✅ 0 missing, 58 routes, all `required=true` |
| Gate 4: HTTP method correctness | ✅ No GET routes performing writes |
| Write guard parity (47 routes) | ✅ All guarded write routes have `⚠️` in knowledge |
| Dual-layer completeness (65 routes) | ✅ All routes in knowledge files |
| Discovery route completeness | ✅ `/api/printers` + `/api/default_printer` present |
| Type I — UI traceability | ✅ All HUD components documented (E7/E8 now fixed) |
| Rules↔knowledge consistency | ✅ Humidity counterintuitive warning + stage table note fixed |
| Proactive guidance completeness | ✅ 10 alert types, 18 action guidance lines |
| A2 BPM coverage | ✅ 95 items, 0 High gaps, 9 exclusions |
| Repo state | ✅ All 6 repos clean and pushed |

