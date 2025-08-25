# Drama Guard — Slack AI Coach PRD (Slack-based AI coach to reduce drama)

**Doc owner:** Product (you)

**Version:** v1.0  
**Last updated:** 2025-08-25
**Stakeholders:** Product Managers, Developers, UX Designers, Slack Workspace Admins, Team Members, HR/People Ops

---

## 1) Executive Summary
Drama Guard is a Slack-native AI coach that helps teams notice and shift out of drama in written communication. It listens (with consent) to public channels, private channels it’s invited to, and DMs (where permitted), detects “below‑the‑line” drama roles (Victim, Persecutor, Rescuer), and delivers private, compassionate nudges with concrete rewrites. At the org level, it surfaces an anonymized “drama health” scorecard and trends so leaders can build healthier, more collaborative cultures.

**Primary outcomes**
- Reduce frequency and duration of drama-laden exchanges.
- Increase reframe/adoption rate of constructive language.
- Improve perceived psychological safety and clarity of requests.

**Non‑goals**
- Not a surveillance tool for disciplinary action.
- Not a replacement for HR, therapy, or legal counsel.
- Not a general-purpose toxicity detector; it is purpose-built for drama dynamics.

---

## 2) Problem, Goals, and Principles
**Problem**: Written comms often drift into blame, helplessness, or over-functioning. Once threads go “below the line,” they waste time, erode trust, and create churn.

**Goals**
1. Detect drama patterns accurately in near‑real time (<2s P95 end-to-end for ephemeral nudges).
2. Coach individuals privately with respectful, actionable guidance and suggested rewrites.
3. Provide opt‑in team and org dashboards with anonymized trend metrics.
4. Bake in privacy-by-design and consent-first controls.

**Design principles**
- *Consent and transparency*: Clear signals where the bot is active. Easy /pause and channel‑level controls.
- *Private first*: Use ephemeral messages and DMs by default.
- *Actionable coaching*: Offer 1–2 click rewrites, short prompts, and learning bites.
- *Low friction*: No new surface if possible; everything runs in Slack.
- *Ethical by default*: Aggregate reporting with k‑anonymity; no individual scorecards by default.


## 2A) Scope, Assumptions & Dependencies
**In scope**: Slack bot integration for real-time monitoring and coaching; aggregate health checks; user/admin controls for consent and customization.

**Out of scope (v1)**: Other chat platforms (e.g., Microsoft Teams, Discord); advanced AI training interfaces; hardware dependencies.

**Target audience**: Organizations on Slack—especially remote tech teams, agencies, and culture-forward enterprises.

**Assumptions**: Users have basic Slack proficiency; organizations value psychological safety; detection quality improves with opt‑in, anonymized feedback data.

**Dependencies**: Slack APIs; third‑party LLM services; cloud hosting (AWS/GCP/Azure).

**Risks & mitigations**: False positives → conservative thresholds + easy `/pause`; privacy concerns → opt‑in channels, transparency, private-first nudges.

---

## 3) Background: Drama Dynamics (operationalized)
Informed by Transactional Analysis (Karpman Drama Triangle) and Conscious Leadership (e.g., *The 15 Commitments of Conscious Leadership*), we map conversational cues to roles and coach toward ownership, curiosity, and clear agreements.
We operationalize “below‑the‑line” roles:
- **Victim**: Powerless framing, externalizing agency. Markers: “I can’t…”, “No one told me…”, “This always happens to me…”.
- **Persecutor**: Blame, attack, shaming. Markers: “You never…”, “This is stupid…”, sarcasm-as-attack.
- **Rescuer**: Over-functioning, disempowering help. Markers: “Let me just do it for you…”, unsolicited fixes without consent.

We coach to “above‑the‑line” postures: ownership (“What can I do?”), curiosity (“What’s the request?”), and collaboration (clear boundaries/agreements). The model translates this framing into concrete linguistic features and conversation-state patterns.

---

## 4) Personas
1. **Ava – IC Engineer (end-user)**
   - Pain: Threads get heated; wants quick prompts to rephrase.
   - Success: Private nudge suggests a rewrite she can one-click send.

2. **Marco – Team Lead / Manager**
   - Pain: Recurrent drama spikes in certain channels.
   - Success: Weekly drama health report + coaching tips for retros.

3. **Rina – People Ops / HRBP**
   - Pain: Wants culture signals without surveilling individuals.
   - Success: Anonymized trendline by org unit; opt-in pilots; policy alignment.

4. **Sam – Security/Compliance Admin**
   - Pain: Data access & retention risk.
   - Success: Clear scopes, data minimization, SOC 2 roadmap, admin controls.

5. **Dana – Executive Sponsor**
   - Pain: Cultural drag on velocity.
   - Success: Company-level score with targets tied to engagement and retention.

---

## 5) Key User Stories & Acceptance Criteria
### 5.1 Real-time private coaching
- **As an IC**, when I compose or send a message with drama markers, I receive a private, ephemeral nudge with:
  - (a) a brief label (e.g., “Blame detected”),
  - (b) 1–2 suggested rewrites, and
  - (c) a micro‑prompt (e.g., “What’s your specific request?”).
**Acceptance**: 95% of nudges appear within 2 seconds (P95). Users can insert a rewrite into the thread with one click. Nudges are visible only to the author.

### 5.2 Thread-level de‑escalation
- **As a participant**, if a thread escalates across multiple messages, I can click a message action → “Get coaching.”
**Acceptance**: App posts an ephemeral summary of patterns and suggested next message templates (e.g., clarifying request, boundary statement). No content is posted publicly unless user chooses.

### 5.3 Opt-in channel coverage
- **As a channel admin**, I can invite Drama Guard to a channel and set sensitivity (Low/Medium/High), privacy mode (ephemeral‑only vs allow public prompts), and keywords to ignore.
**Acceptance**: Settings persist; the bot displays a pinned notice and `/drama status` shows current mode.

### 5.4 Org health check (anonymized)
- **As People Ops**, I can view a dashboard summarizing drama indicators by org and time window.
**Acceptance**: Metrics meet k‑anonymity thresholds (e.g., no slice < 10 active users). No individual names shown by default. Export CSV of aggregated stats.

### 5.5 Manager weekly digest
- **As a manager**, I receive a weekly DM with top patterns in my team’s channels and recommended coaching prompts for retros.
**Acceptance**: Only includes channels where the bot was invited. No quotes of private DMs. Opt-out available.

### 5.6 Consent & control
- **[M]** As a Team Member, I can opt‑in to personal monitoring and choose nudge style (emoji‑only, concise, rich) from App Home.
- **As any user**, I can `/drama pause 1h|1d|7d`, `/drama resume`, `/drama delete-my-data`.
**Acceptance**: Commands execute immediately and are auditable in Admin logs.

### 5.7 Admin governance
- **As a Compliance Admin**, I can set org-wide data retention (0–30 days), export DPAs, and restrict features (e.g., disable DM scanning).
**Acceptance**: Settings enforced across workspaces/Org Grid; changes logged.

---

## 6) Functional Requirements
### 6.1 Slack integration
- OAuth 2.0 with granular bot scopes. Minimum:
  - `channels:read`, `channels:history`, `groups:history` (for private channels where invited), `im:history` (configurable), `mpim:history`, `chat:write`, `chat:write.public`, `commands`, `reactions:write`, `users:read`, `team:read`.
- Events API / Socket Mode for message events, edits, reactions, app_home opened, and channel join/leave.
- Use `chat.postEphemeral` for nudges; `chat.scheduleMessage` for digests.
- Message Actions & Shortcuts: “Get coaching”, “Summarize thread”, “Set channel sensitivity”.
- Slash commands: `/drama help|status|pause|resume|settings|analyze-thread|delete-my-data` (alias: `/dramaguard`).

### 6.2 Detection engine (NLP)
- Multi-label classifier for roles (Victim/Persecutor/Rescuer) + *intensity* (0–1) and *confidence*.
- Features: sentiment, modality, blame/agency patterns, hedging, directives, rescue markers, rhetorical questions, absolutist terms, second-person accusations, negations, sarcasm cues, emoji/emoji-as-tone.
- Context window: last N messages in thread (configurable; default 12) + speaker turns + reply relationships.
- Language support: v1 English; roadmap: ES, FR, DE. Detect language and fall back gracefully.
- Continual learning loop with human feedback (opt-in thumbs up/down on nudges).
- Safety layer: profanity/harassment filter, PII redaction before storage.

### 6.3 Coaching and content system
- Rule-based templates + LLM rewriters produce:
  - 1–2 rewrites preserving intent but shifting tone
  - Micro‑prompts (questions to move “above the line”)
  - Education snippets (e.g., “Ask vs. Accuse” 15-second reads)
- Personalization knobs: user preferred tone (direct, warm, formal), reading level, emoji use, and nudge display style (emoji‑only subtle mode, concise, or rich).
- Hallucination guardrails: rewrites must not fabricate facts; emphasize requests/agreements.

### 6.4 Health scoring & analytics
- **Message-level Drama Index**: 0–10 per message/thread; rolled up into channel/team composites.
- **Indicators**: drama incidence rate (per 1k messages), role distribution, escalation streak length, recovery rate (nudge→reframe), time-to-resolution, opt-out rates.
- **Score**: Weighted composite (0–100) per channel/team with trend vs prior period.
- **Privacy**: k-anonymity on all slices; DP noise optional for large orgs.
- **Exports**: Aggregated CSV/JSON/PDF; webhook to BI tools; optional connectors to Google Sheets/HRIS (Phase 2).

### 6.5 Admin & governance
- Org-level install (Enterprise Grid) with workspace allowlist.
- Data retention policy (default 7 days; min 0-day streaming mode).
- Access controls: Admin vs Manager vs User capabilities.
- Audit logs: settings changes, data deletions, exports.
- DSR/DSAR endpoints for personal data deletion and export.

---

## 7) Non-Functional Requirements
- **Latency**: Real-time nudges P95 < 2s; thread analyses P95 < 5s.
- **Availability**: 99.9% monthly (excluding Slack outages).
- **Scalability**: 100k daily active users, 10M msgs/day; autoscaling model servers.
- **Security**: TLS 1.2+, encryption at rest; key rotation; SSO/SAML for web dashboard.
- **Compliance roadmap**: SOC 2 Type II within 9–12 months; HIPAA/PHI not supported v1.
- **Observability**: Traces on event→nudge pipeline; redaction in logs; PII scrub.

- **Accessibility**: Web dashboard meets WCAG 2.1 AA.
- **Compliance**: GDPR/CCPA readiness; consent audits and data processing records.
- **Reliability**: Graceful fallbacks (e.g., “Unable to analyze—try again”), with retries respecting Slack rate limits.

---

## 8) UX Flows (Slack-first)
1. **Onboarding**
   - Admin installs app → selects default policies → chooses pilot channels → posts intro message + pinned note.
2. **First nudge**
   - User sends edgy message → ephemeral nudge with reason + rewrite → 1-click insert.
3. **Thread coaching**
   - Message action → ephemeral summary + suggested next move → optional mini‑lesson link in App Home.
4. **Weekly digest (Manager)**
   - DM with top patterns, channels trending worse/better, and 3 prompts for retro.
5. **App Home**
   - Personal stats (private), saved lessons, nudge preferences, snooze/pause.

---

## 9) Sample Coaching Content (v1 library)
- **Victim → Ownership**
  - Prompt: “What can I do in the next 24h?”
  - Rewrite: “I’m blocked because X. I can do Y by EOD if Z is clarified—can you confirm?”
- **Persecutor → Challenger**
  - Prompt: “What outcome do I want?”
  - Rewrite: “I’m concerned about [specific]. Can we agree on criteria and next steps?”
- **Rescuer → Coach**
  - Prompt: “What support was asked for?”
  - Rewrite: “Happy to help. What have you tried, and where do you want input?”

---

## 10) Architecture & Implementation Details
**High-level**
- Slack App → Events API / Socket Mode → Ingestion (HTTP/WS) → Queue (Pub/Sub or Kafka) → Feature Extractor → Classifier/LLM Rewriter → Policy Engine → Slack Delivery (ephemeral/DM) → Metrics Store + Analytics API → Web Dashboard.

**Components**
- **Ingestion**: Stateless workers validate Slack signatures; batch/stream to queue.
- **Feature extraction**: Python service (spaCy/transformers) for tokenization, dependency parsing, role markers, context assembly.
- **Model serving**: Role classifier (fine-tuned transformer) + rule-based features; LLM rewriter with prompt templates and safety constraints.
- **Policy engine**: Applies channel/user settings, sensitivity thresholds, rate limits, privacy rules.
- **Delivery**: Slack Web API calls; exponential backoff; idempotency keys per message.
- **Storage**: Message content optional; default is feature vectors + hashed IDs; retention window enforced via TTL.
- **Analytics**: Aggregation jobs compute incidence and scores; precomputed tiles for dashboard.
- **Dashboard**: Lightweight web app (Next.js) for admins/managers; SSO; RBAC.

**Data schema (simplified)**
- `message_event`: team_id, channel_id, user_id_hash, ts, text_redacted, features, lang, policy_snapshot.
- `nudge_event`: message_ts, user_id_hash, nudge_type, latency_ms, accepted_rewrite(bool).
- `channel_metrics_daily`: counts, incidence, role_mix, score.

**Latency budget (P95)**
- Slack event delivery: ~200ms
- Feature + classify: ~300–600ms
- Rewriter: ~600–900ms
- Policy + postEphemeral: ~200–300ms

**Rate limiting & backoff**
- Respect Slack rate limits; shard per team; queue buffering; drop to summary-only mode on sustained overload.

**Testing**
- Offline eval set (10k labeled messages) per role; target F1 ≥ 0.85 overall, ≥ 0.80 per role.
- Red‑team eval: sarcasm, code snippets, quotes, emojis, multi-language.
- Shadow mode pilots (analyze-only) before enabling nudges.

---

## 11) Security, Privacy, Ethics
- **Consent**: Explicit channel opt-in; clear `/drama status` banner; DM scanning off by default.
- **Transparency**: Bot announces on join; help center explains detection and limits.
- **Privacy**: Default no-content storage; features-only with 7-day TTL; optional 0-day streaming mode.
- **Controls**: User `/drama delete-my-data`; admin hard-delete API; export logs.
- **Anonymity**: Aggregates only when ≥10 unique users; DP noise option for large slices.
- **Misuse prevention**: No individual scorecards; managers cannot see individual nudges by default.
- **Legal**: DPA, SCCs for cross-border; SOC 2 roadmap; not designed for PHI/PII heavy contexts.

---

## 12) Metrics & KPIs
- **Core**: drama incidence (per 1k msgs), reframe adoption rate, time-to-repair (first nudge → constructive reply), escalation streak length, opt-out rate, false positive rate.
- **Business**: pilot NPS, weekly active nudged users (WANU), team retention correlation (exploratory), digest open/click.
- **Quality**: model F1, latency P95, nudge acceptance CTR.

---

## 13) Rollout Plan & Milestones
**M0 – Discovery (2–3 weeks)**
- Stakeholder interviews; labeled data collection (internal + synthetic); privacy review.

**M1 – Alpha (6–8 weeks)**
- Slack app with event ingestion, basic classifier, ephemeral nudges, `/drama` commands, App Home.
- Shadow mode in 3 pilot channels; weekly qualitative reviews.

**M2 – Beta (8–10 weeks)**
- Rewriter v1, manager digest, health dashboard (anonymized), admin console, retention controls, audits.
- Multi-tenant hardening, SOC 2 readiness controls.

**M3 – GA (6–8 weeks)**
- Language detection, improved sarcasm handling, DP option, Grid support, SSO, RBAC.
- Pricing & billing (per-seat or per-workspace), support playbooks.

**M4 – Post‑GA Enhancements**
- Additional languages, Jira/GitHub linking for context, custom coaching modules, mobile-optimized App Home.

**Timeline framing:** MVP target Q4 2025 (core detection, coaching, basic health checks); Phase 2 Q1 2026 (dashboard, customizations, integrations/connectors); Phase 3 Q2 2026 (feedback loops, multi‑platform exploration).

---

## 14) Pricing & Packaging (placeholder)
- **Starter**: up to 50 users, channel-based pricing, basic dashboards.
- **Growth**: per-seat, full dashboards, manager digests, retention controls.
- **Enterprise**: Grid, SSO, DPA/SCCs, custom data residency, advanced privacy.

---

## 15) Risks & Mitigations
- **False positives → user frustration**: Start in shadow mode; conservative thresholds; easy `/pause`.
- **Perceived surveillance**: Private-first, opt-in channels, no individual reporting.
- **Cultural/locale mismatch**: Localization; tone presets; human-in-the-loop feedback.
- **Slack API changes**: Version pin; integration tests; feature flags.
- **Adversarial use**: Abuse detection; rate limits; admin overrides.

---

## 16) Open Questions
- Should we allow optional public prompts (visible to all) in retrospect threads? Default off.
- How to handle legal hold/eDiscovery requests while honoring privacy promises?
- Should managers be able to see *aggregated* reframe adoption rates by team?
- What is the minimum viable accuracy that users will tolerate in early pilots?

---

## 17) Glossary
- **Below the line**: Drama posture (Victim/Persecutor/Rescuer).
- **Nudge**: Private ephemeral coaching message to the author.
- **Reframe**: A constructive rewrite or prompt that shifts posture.
- **k‑anonymity**: Minimum cohort size for aggregated reporting.

---

## 18) Appendices
### A. Slack scopes checklist
- Read history (where installed/invited), write messages (ephemeral + DM), shortcuts/commands, user/team metadata (non-sensitive), reactions.

### B. Sample Admin Policy Presets
- **Conservative**: Low sensitivity, no DM scanning, 0-day retention (features only).
- **Balanced**: Medium sensitivity, 7-day retention, manager digest enabled.
- **Proactive**: High sensitivity, 14-day retention, DP noise for analytics.
- **Ultra‑Privacy**: Opt-in analysis only; no raw message storage; store only consents and aggregated analytics.

### C. Evaluation dataset plan
- Collect 10–15k labeled snippets (balanced by role/intensity, plus neutral), seed from public corpora where permitted, augment with paraphrases, and include sarcasm + emoji cases.



### D. References
- Transactional Analysis (Karpman Drama Triangle)
- *The 15 Commitments of Conscious Leadership*

### E. Open Items
- Pricing model details (e.g., freemium vs seat-based)
- Detailed wireframes for App Home, manager digest, and dashboard

