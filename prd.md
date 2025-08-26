# Drama Guard — MVP Product Requirements Document (PRD)

**Version:** 0.1 (MVP)
**Date:** August 25, 2025
**Owner:** Product
**Stakeholders:** Eng, Design, Security/Privacy, Customer Pilot Champions

---

## 1) Summary

Drama Guard MVP is a **Slack bot** that can be **invited** into public channels, private channels, and DMs (with the bot). When present, it detects messages that fall **below the line** (Drama Triangle roles: Persecutor, Victim, Rescuer) and sends **private nudges** to the author to help them shift **above the line** (ownership, clarity, curiosity).

> **Out of scope for MVP:** Org-wide assessments, dashboards, KPI tiles, exports/Discovery, long-term storage of message content, cross‑platform support (e.g., Microsoft Teams), and any per-user scorecards.

**Why now**: Teams want lightweight, non-judgmental coaching *in the moment* without governance overhead. MVP focuses solely on real‑time nudges where the bot is explicitly present.

---

## 2) Goals & Non‑Goals

**Goals**

* Provide **real-time, private coaching** via ephemerals/DM to reduce drama language in Slack.
* Be **easy to install and invite** by any motivated individual (subject to workspace settings).
* Maintain **privacy by design** (consent-first coverage; minimal data retention).

**Non‑Goals (MVP)**

* No org-wide analytics or health reports.
* No admin dashboards beyond install/config basics.
* No multi-tenant exports or Discovery API usage.

---

## 3) Scope

**In scope**

* Slack app with Bot user using granular permissions.
* Real-time detection of below-the-line language in conversations the bot can see.
* Private coaching nudges with suggested rewrites and micro-prompts.
* Minimal controls: slash commands, App Home toggles, thumbs up/down feedback.

**Out of scope**

* KPI charts, role mix trends, leaderboards, heatmaps, digests.
* Storage of raw message bodies beyond transient processing window.
* MS Teams or other platforms.

---

## 4) Personas

* **Teammate (Author):** Wants help phrasing messages constructively; fears public shaming. Receives private nudges.
* **Channel Champion/Manager:** Invites the bot into high-friction threads/channels; ensures team buy‑in. Needs simple on/off and pause controls.
* **Workspace Admin:** Installs/approves the app; cares that it is consent-first, minimally invasive, and compliant.

---

## 5) User Stories (MVP)

**Must‑have**

1. *As a teammate*, I can **invite** Drama Guard to a public or private channel so it can coach me in that space.
2. *As a teammate*, when I write a message that appears **below the line**, I get a **private nudge** (ephemeral if possible) with:

   * A plain-language reflection (what might be happening)
   * A **rewrite suggestion**
   * A **micro-prompt** (e.g., “What outcome do I want?”)
   * **Insert** / **Try again** actions
3. *As a teammate*, I can **pause/resume** nudges for myself.
4. *As a champion*, I can **pause/resume** nudges for a channel.
5. *As an admin*, I can **install** the app (or approve end‑user installs per workspace policy) and see a basic **App Home** with settings and help.

**Should‑have**
6\. *As a teammate*, I can give **thumbs up/down** on suggestions to improve future nudges.
7\. *As a teammate*, I can use a **message action** (e.g., “Get coaching”) to request a one‑off rewrite even if the bot didn’t auto‑nudge.

**Could‑have** (post-MVP)
8\. Nudge **tone styles** (concise, warm, formal).
9\. **Practice mode** in DM with the bot for training outside of live channels.

---

## 6) Core Experience

### 6.1 Trigger & Detection

* Bot listens to message events in conversations where it is present (public channel joined, private channel invited, DM with bot or MPIM including bot).
* Each new message is scored by a lightweight classifier for **Persecutor/Victim/Rescuer** cues and overall **“below-the-line” confidence**.
* If above threshold and not rate-limited, proceed to nudge.

### 6.2 Nudge Delivery

* **Primary:** `chat.postEphemeral` to the **author** in-channel (private to them).
* **Fallback:** DM the author if ephemeral cannot be delivered (user inactive).
* Nudge content:

  * Short reflection (e.g., “This reads like blame…”)
  * 1–2 rewrite options
  * Micro-prompt
  * Buttons: **Insert rewrite** (posts an edited draft in composer via `chat.postMessage`/copy workflow), **Rephrase again** (regenerates), **Dismiss**.

### 6.3 Controls

* Slash commands (tentative):

  * `/drama help` — quick usage & links
  * `/drama pause` / `/drama resume` — per-user toggle
  * `/drama pause here` / `/drama resume here` — channel toggle (requires permission)
  * `/drama coach` — open modal for on-demand coaching of selected text
* App Home: global pause, learn the model, privacy details, feedback switch.

### 6.4 Guardrails

* **Rate limiting:** max N nudges per user per hour; per-channel cooldown windows; dedupe on thread.
* **Confidence thresholds:** only nudge when confident, else silently log a low-confidence event (aggregate counts only).
* **Safety language:** empathetic tone; never shame; provide opt-out link in DM.

---

## 7) Functional Requirements

### 7.1 Slack Integration

* **Events:** `message.channels`, `message.groups`, `message.mpim`, `app_mention` (and `message.im` only if DM coaching is enabled).
* **Scopes (granular bot):**

  * Read: `channels:read`, `groups:read`, `channels:history`, `groups:history`, `mpim:history`, (`im:history` if DM with bot), `users:read`.
  * Write: `chat:write`, `chat:write.public` (optional), `reactions:write` (for feedback), `commands`.
* **Coverage reality:** Bot **only** sees conversations it’s a member of; private channels require invite; no reading users’ DMs unless it’s a DM **with the bot**.

### 7.2 Detection Engine

* **Classifier:** heuristic + LLM few-shot prompt for role detection (P, V, R) with confidence; lightweight profanity/hostility filter.
* **Latency target:** nudge composed and sent **≤ 3s P95** from message timestamp.
* **Content policy:** reject generating targeted harassment; always offer neutral tone rewrites.

### 7.3 Nudge Generator

* Prompt templates per role with **rewrite macros** (facts → request; blame → concern; helpless → ownership; rescuing → coaching).
* Personalization: remember user’s last chosen tone (if enabled).
* Insert flow: provide copy-to-clipboard or prefilled draft via DM; for public posting, always require **user action** (never auto-post on their behalf).

### 7.4 Controls & Feedback

* Slash commands + App Home toggles; per-user and per-channel state.
* Feedback: 👍/👎 attaches to message-id + anonymized label for model tuning (no raw text stored past retention window).

### 7.5 Data & Privacy

* Processing: **in-memory/ephemeral**; store only minimal **event metadata** + **model labels** for short retention (e.g., 0–7 days, default 0).
* No storage of raw message bodies beyond transient feature extraction buffer (<60s).
* Pseudonymous telemetry only (counts of nudges, acceptance, thumbs up/down). No individual scorecards.
* Clear `/drama delete-my-data` path to purge any retained user-specific preferences.

---

## 8) Non‑Functional Requirements

* **Performance:** P95 nudge latency ≤ 3s; 99th ≤ 5s.
* **Reliability:** ≥ 99.5% weekly success rate on eligible nudges; retries on Slack HTTP 429s per backoff.
* **Security:** TLS in transit; encrypted storage at rest; least-privilege IAM; signed releases.
* **Compatibility:** English only; Slack Free/Pro/B+/Enterprise. Install flow must work for user‑initiated installs where workspace policy allows; otherwise request admin approval.
* **Observability:** basic metrics (nudge attempts/sent/skipped; reasons), structured logs (no message bodies), alerting on error rate.

---

## 9) Success Metrics (MVP)

* **Activation:** ≥ 20 pilot workspaces install; ≥ 50 channels invited.
* **Engagement:** ≥ 30% weekly active authors (received ≥1 nudge).
* **Quality:** ≥ 65% positive feedback (👍) on nudges; **false positive rate** ≤ 15% (measured via thumbs down + manual spot checks).
* **Latency:** P95 ≤ 3s in production week 2.

---

## 10) Milestones & Deliverables

**M0 — Tech Spike (1–2 weeks)**

* Slack app scaffold; event subscription; ephemeral + DM fallback; minimal classifier stub; `/drama pause/resume`.

**M1 — Alpha (3–4 weeks)**

* Role detector v1 (LLM + heuristics), nudge templates v1, per-user/channel state, rate limiting, App Home basics, feedback 👍/👎.

**M2 — Beta (2–3 weeks)**

* Tone preference (optional), message action “Get coaching,” improved prompts, observability, security review, pilot docs.

**Launch criteria**

* Meets Success Metrics; zero P0 security issues; clear privacy policy.

---

## 11) Risks & Mitigations

* **False positives** → Conservative thresholds; quick “Dismiss” and per-user pause; continuous template tuning.
* **User backlash** → Empathetic language; easy opt-out; channel-level pause; clear install comms.
* **Rate limits** → Batch classification, jitter, shared queue with backoff.
* **Workspace policy variance** → Document that some installs require admin approval; provide request‑approval flow.

---

## 12) Appendix

### 12.1 Example Nudge Templates

* **Persecutor → Challenger**

  * *Reflection:* “This might read as blame.”
  * *Rewrite:* “I’m concerned about **\[specific]**. Can we align on **\[criteria]** and next steps?”
  * *Prompt:* “What outcome do I want?”
* **Victim → Ownership**

  * *Reflection:* “This might read as powerless.”
  * *Rewrite:* “I missed **\[context]**. I can deliver **\[X]** by **\[time]** if we clarify **\[Y]**—can someone confirm?”
  * *Prompt:* “What can I do in the next 24h?”
* **Rescuer → Coach**

  * *Reflection:* “This might jump into fixing.”
  * *Rewrite:* “Happy to help—what have you tried, and where would feedback be most useful?”
  * *Prompt:* “What support was asked for?”

### 12.2 Minimal Manifest (indicative)

```json
{
  "display_information": { "name": "Drama Guard" },
  "features": {
    "bot_user": { "display_name": "Drama Guard", "always_online": false },
    "slash_commands": [{ "command": "/drama", "url": "https://app.example.com/slack/commands", "description": "Drama Guard controls" }]
  },
  "oauth_config": {
    "redirect_urls": ["https://app.example.com/slack/oauth/callback"],
    "scopes": {
      "bot": [
        "commands",
        "chat:write",
        "chat:write.public",
        "channels:read", "channels:history",
        "groups:read", "groups:history",
        "mpim:history",
        "im:history",
        "reactions:write",
        "users:read"
      ]
    }
  },
  "settings": {
    "event_subscriptions": {
      "request_url": "https://app.example.com/slack/events",
      "bot_events": ["message.channels", "message.groups", "message.mpim", "app_mention"]
    },
    "interactivity": { "is_enabled": true, "request_url": "https://app.example.com/slack/interactive" }
  }
}
```

---

**End of MVP PRD**
