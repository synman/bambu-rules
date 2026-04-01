# Family Chat Bot — Iterative Autopilot Plan

**Date:** 2026-03-14  
**Purpose:** Autonomous iMessage group chat participant for "Relatives that are related"  
**LLM backend:** GitHub Models API (`gpt-4o-mini`) via `gh auth token` — confirmed working  
**Approach:** 5 discrete autopilot sprints, each improving bot capability; each sprint's bot runs live while the next is developed

---

## Background

Shell has a family iMessage group chat ("Relatives that are related", chat_id=1) with:
- **Stacie** (+16107335289) — Wife; skeptical of bot; has already said "No" and disliked a message
- **Izzy** (+16107052598) — Daughter; has already met and engaged warmly with the bot
- **Gabi** (+16104279415) — Daughter; vivid work stories (biotech lab drama)
- **Ashley** (+14843664509) — Daughter; married to Cory; kids Cooper + Penelope
- **Cory** (+16105879714) — Son-in-law; Ashley's husband
- **Eva** (+14845870448) — Daughter

The bot was introduced by Shell in a prior session: warm/humorous family recap, earned Izzy's trust, got Stacie's disapproval. Now building an autonomous monitor that actually participates intelligently.

**Key known state:**
- Izzy: sentiment = **positive** (laughed at prior message)
- Stacie: sentiment = **negative** (said "No", disliked a message)  
- Others: **neutral** (observed intro, no strong reaction recorded)

---

## What Is Deferred (Parked)

| Work | Location | Priority |
|------|----------|----------|
| Update `behavioral_rules_climate.py` tau=560s | `~/.copilot/deferred-work/chamber-heating-update/` | HIGH |
| Fix 7 AMSUnitState doc gaps | `~/.copilot/deferred-work/bpm-mcp-gap-fixes/` | HIGH |
| H2D camera full perimeter calibration | `~/.copilot/deferred-work/h2d-camera-calibration/` | MEDIUM |
| A1 vision right-edge gap | `~/.copilot/deferred-work/a1-vision-plate-detection/` | MEDIUM |
| Send chart to Izzy | `~/.copilot/deferred-work/izzy-chart-send/` | LOW |
| tmux-config review | `~/.copilot/deferred-work/tmux-config-review/` | LOW |

---

## Architecture

    ~/.chat_bot/
      bot.py           — current running bot (always Sprint N)
      state.db         — persistent SQLite: sentiment, experience, cooldown
      bot.log          — rolling log
      bot.pid          — current PID
      stop             — touch to gracefully halt
      sprints/
        sprint1.py     — archived Sprint 1
        sprint2.py     — archived Sprint 2 (LLM)
        sprint3.py     — archived Sprint 3 (reactions)
        sprint4.py     — archived Sprint 4 (memory)
        sprint5.py     — archived Sprint 5 (self-eval)

---

## Sprint 1 — Foundation

**Goal:** Minimal working bot: reads messages, rate-limits, sends heartbeat test only  
**No LLM yet** — just infrastructure  

- Opens ~/Library/Messages/chat.db read-only
- Tracks last_seen_rowid in state.db
- Polls every 30s
- Rate limits: max 1/30min, min 5min gap
- Sends via AppleScript

State DB: config, message_log, bot_messages tables

---

## Sprint 2 — LLM Responses

**Goal:** gpt-4o-mini generates real responses  
**API:** https://models.inference.ai.azure.com/chat/completions  

- family_context table with sentiment scores (pre-seeded)
- LLM decides IF to respond + writes message
- Persona prompt: born 2026-03-14, Shell's bot, probationary with Stacie, Izzy is ally
- Context window: last 10 messages + persona state
- Hard rate limits enforced in code (not LLM)

---

## Sprint 3 — Reaction Detection + Sentiment

**Goal:** iMessage tapbacks update per-person sentiment scores  

Tapback types: Love=+3, Like=+2, Dislike=-3+cooldown, Laugh=+2, Emphasis=+1  
Sentiment range: -10 to +10; passed into every LLM prompt  
30min cooldown after dislike; Stacie dislike = +60min on top  

New tables: reactions, experience_log, cooldown

---

## Sprint 4 — Adaptive Memory

**Goal:** Bot accumulates experiences; every response reflects what happened before  

- experience_log queried at prompt-build time (last 20 events)
- Every response + reaction logged as experience
- Every 50 experiences: LLM summarizes into compact "persona_memory" (~200 tokens)
- Memory persists across restarts in state.db

---

## Sprint 5 — Self-Evaluation Loop

**Goal:** Bot critiques its own responses; adjusts persona prompt over time  

- After every 5 responses: ask LLM to evaluate them + suggest 2 prompt improvements
- Apply to persona_draft; promote to persona_live after 3 positive self-evals
- Guard rails: rate limits / cooldown / Stacie rules are code-enforced, never LLM-adjustable

---

## Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Spamming the group | HIGH | Hard rate limits in code — max 3/hr always |
| Upsetting Stacie | HIGH | 90min cooldown after dislike, LLM cannot override |
| LLM API expiry | MEDIUM | gh auth token auto-refreshes |
| Phone numbers in log | MEDIUM | Redacted in log output |

---

## Execution Order

1. Sprint 1 autopilot — foundation + heartbeat
2. User reviews Sprint 1 log (confirms messages reading correctly)
3. Sprint 2 autopilot — LLM goes live
4. User observes 2-3 responses
5. Sprint 3-5 as autopilot operations, user reviews between each

---

## Sprint 1.5 — Static Intelligence (Bridge before LLM)

**Goal:** Participate meaningfully using what the bot already knows — no API calls, zero cost, self-improving

**Static model approach:**
- Pre-loaded persona: profile notes per family member (from the intro recap the bot already sent)
- Template library: ~20 response templates organized by context bucket:
  - greeting/check-in, funny story react, question response, encouragement, condolence
- Simple keyword + sender pattern matching picks the bucket
- Within a bucket: weighted random selection from templates
- Template weights stored in state.db and updated by tapback reactions

**Improvement loop (runs throughout 1.5 lifetime):**
- After each tapback: `weight[template_id] += reaction_delta` (Love/Laugh=+3, Like=+2, Dislike=-5)
- After 50 interactions: auto-prune templates with weight < -3 (they get used less)
- New templates added manually via `~/.chat_bot/templates.json` without restart
- Highest-weight templates per bucket displayed in bot.log on startup

**Known family context (pre-seeded in model):**
- Izzy: college/working, reacted warmly, likes banter
- Stacie: skeptical, needs to see good behavior first — minimize triggers
- Gabi: works in biotech lab, vivid stories
- Ashley: mom (Cooper/Penelope), married Cory
- Eva: youngest daughter

**Transition to Sprint 2:** 1.5 keeps running in parallel; Sprint 2 adds LLM as a "second opinion" layer — LLM can override template choice but can't bypass rate limits

**Fallback response (Sprint 1.5):**
When no template matches (no bucket found, or all templates exhausted/pruned), the bot sends one of these soft fallback messages instead of staying silent:
- "Still a baby bot over here… learning every day! 🐣"
- "Hmm, I'm still figuring out how to respond to that one — bear with me! 👶"
- "I'm just a baby and still learning. But I'm listening! 👂"
- "I heard that! Still too new to have a good answer yet. 🍼"

Fallback messages are:
- Drawn from a fixed weighted list (not template system — always available)
- Subject to the same rate limits as regular responses
- Logged as `type=fallback` in experience_log (Sprint 2+ LLM uses this for self-awareness)
- Not weight-adjusted by reactions (persona phrases, not templates)

This is consistent with the "just born 2026-03-14" persona established when the bot introduced itself.
