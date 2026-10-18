# Decision log

> Append-only. Format: ID · date · decision · why · alternatives considered · owner.

## D-001 · 2026-10-14 · Problem statement chosen
**Decision:** Fast and Secure Online Transaction Authentication.
**Why:** Team choice. Clear demoable output (decision + reasons) with strong ML/systems depth.
**Owner:** Team

## D-002 · 2026-10-15 · Tech stack
**Decision:** Python 3.11 + **FastAPI** (Pydantic v2) · **LightGBM** + `shap` (TreeSHAP) · profiles in-memory with an optional **Redis** · event log in **Supabase Postgres** (realtime to the dashboard) · dashboard + demo app in **React (Vite) on Vercel** · API as Docker on **Google Cloud Run** (Render fallback).
**Why:** Fast to build; GBDT p99 ≈ 0.15 ms/txn on one core; the team already knows Supabase/Vercel; Cloud Run gives a real "cloud service" with autoscaling and a public URL.
**Alternatives:** Node + Express (weaker ML ecosystem); Streamlit dashboard (faster, but can't host the demo payment app nicely).
**Owner:** Team (proposed by Claude; confirm)

## D-003 · 2026-10-15 · Scoring approach: hybrid
**Decision:** Hard rules (policy + fast-path allow) → LightGBM p(fraud), calibrated → TreeSHAP reason codes. **No LLM in the decision path.**
**Why:** Rules give guaranteed policy and explainability; GBDT weighs many weak signals; LLM scoring is ~1 s and non-deterministic (see `02-research/methods.md`).
**Alternatives:** rules only (brittle), ML only (no policy guarantees), deep sequence models (no time, harder to explain).

## D-004 · 2026-10-15 · Decision thresholds
**Decision:** Block at p ≥ 0.90 or a hard rule. Challenge at p ≥ clip(₹500 / amount, 0.05, 0.60), ×0.6 for users aged 70+. Challenge **type** = argmin expected cost given risk type.
**Why:** Amount-aware thresholds are optimal under instance-dependent costs (Höppner et al. 2020); a high fixed block threshold protects genuine users; typed challenges fix the "OTP doesn't stop coached victims" gap.
**Alternatives:** fixed score cut-offs like Stripe's 65/75 (ignore amount); blanket delay on ≥ ₹10k (RBI 2026 proposal; too much friction).

## D-005 · 2026-10-15 · Training data
**Decision:** Calibrated synthetic generator with labelled attack types (IMP, SCAM, BOT, MULE) and hard negatives. IEEE-CIS/PaySim are used only as sanity checks.
**Why:** No public dataset has behaviour + device + transaction + scam context. Real data would be PII.

## D-006 · 2026-10-15 · Fail-safe policy
**Decision:** On error or timeout (150 ms internal deadline), return `challenge: standard_2fa`, which falls back to the bank's normal mandated PIN/OTP.
**Why:** Never fail open (security) and never hard-block on our own outage (customer harm). We are a layer on top of mandated 2FA.

## D-007 · 2026-10-15 · Mobile-only signals in the prototype
**Decision:** `on_call`, `screen_share`, `sim_change_hours` are simulated with demo toggles. The JS SDK captures the real web signals (typing cadence, dwell, paste, login→pay).
**Why:** A browser can't read call state. On Android these come from a mobile SDK (as in Google's screen-share alerts in GPay/Paytm). We're honest about this on the slides.

## D-008 · 2026-10-15 · Working product name
**Decision:** "Pramaan" (प्रमाण, "proof"). Can change.

## D-009 · 2026-10-15 · Four response levels made real; ₹0 stack
**Decision:** Pitch and build 4 levels: allow · fingerprint (WebAuthn passkey) · pause + family approval (n8n self-hosted + Telegram bot: hold → Approve/Cancel → timeout → label) · block. Add a replay simulator (our policy vs RBI's blanket 1-hour delay). Host the API on Render free tier (no card) instead of Cloud Run; in-memory profiles instead of Redis. n8n is only used *after* a decision, never in the scoring path.
**Why:** Matches the promo video; makes the "typed challenge" idea visible in a demo; keeps running cost at ₹0 with no paid APIs (SMS/WhatsApp avoided). Supersedes the hosting/Redis parts of D-002.

## D-010 · 2026-10-18 · Screening deck v5
**Decision:** Replace the v4 deck with `Pramaan_BIOS_Screening_v5.pptx` (`scripts/build_deck_v5.py`): text-led layout, no icon badges or pill buttons, no invented risk scores or latency, consistent worked example (71x usual ₹1,200), slide 2 ends with the template's "The gap we are addressing is:", a plain team diagram on slide 4, and a 48-hour plan.
**Why:** The template says a simple diagram the team understands beats a polished generated one, and Frontier judging includes live technical questions, so every number must be sourced or labelled as a target. The hack runs 48 h (30 Oct to 1 Nov), not 24.
**Open:** The team decides whether to resubmit v5 before screening closes (19 Oct), if the dashboard allows it.
**Owner:** Bhavyam (+ Claude)

## D-011 · 2026-10-18 · Deck v6 + review fixes
**Decision:** `Pramaan_BIOS_Screening_v6.pptx` (`scripts/build_deck_v6.py`) replaces v5. (1) Impostor challenge = approve with a **device-bound passkey on the registered phone**, not "fingerprint" (a fingerprint on the new phone only proves the thief's own finger). (2) Family hold: the payment goes **only if the trusted contact approves**; no reply in 30 min cancels it (slide 3 said both "family can cancel" and "no reply cancels"). (3) Bot row = "15 PIN retries in 10 min" (card testing doesn't fit an ₹85k story). (4) "SIM changed yesterday" everywhere; "UPI's 15-second response limit", not "decision window". (5) Slide 5 targets labelled as tuned together. Fail-safe wording: "never weaker than today's check", not "never fails open".
**Docs:** DPDP dates corrected (core duties ~14 May 2027, consent managers Nov 2026); Render free-tier sleep + memory wipe added as risk 7 (profiles backed by Supabase, keep-alive ping); threshold-vs-target risk 8; stale Cloud Run/Redis, 24 h plan and TBD dates removed; ATO and loss-trend caveats added.
**Why:** Frontier judging includes live technical questions; each of these is an easy catch.
**Owner:** Shefali (+ Claude)
