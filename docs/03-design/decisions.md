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

---
> **D-012 … D-019 are PROPOSED** (council review 1, 28 Oct, `../06-review/council-review-1.md`). The team confirms or rejects each before the hack; change "Proposed" to "Accepted" when agreed.

> Notes on earlier decisions: D-011's "no reply in 30 min cancels" is superseded by D-017 (silence keeps a hold held); D-009's n8n is superseded by D-016; D-004's thresholds by D-012/D-013.

## D-012 · 2026-10-28 · Rank + friction budget instead of a "probability" threshold (Proposed)
**Decision:** The model's score is used as a rank. Challenge thresholds = the percentile that gives a bank-set challenge budget per amount band on held-out honest traffic (0.2% < ₹2k · 1% ₹2–10k · 2% ₹10k+ trusted payee · 10% ₹10k+ new payee; doubled for 70+). Supersedes the τ = k / amount rule of D-004.
**Why:** Synthetic fraud (2–3%) is ~3,000× the real UPI rate (~7 per million, FY25), so scores aren't real probabilities; v1's k = ₹500 also didn't match the friction costs (₹3–40).
**Alternatives:** prior-shift correction of p (needs a reliable real base rate for large new-payee payments, which we don't have).

## D-013 · 2026-10-28 · Only verified facts block; the model only challenges (Proposed)
**Decision:** Block only on hard rules (flagged payee, card-testing velocity, the owner's "Not me" on the registered phone or an invalid passkey signature; a failed biometric isn't visible to the server). Supersedes "block at p ≥ 0.90" in D-004.
**Why:** No one is blocked on a model trained on our own generator; removes the block-vs-cost contradiction.

## D-014 · 2026-10-28 · Multiclass model + policy table by risk type (Proposed)
**Decision:** LightGBM multiclass (legit / IMP / SCAM / BOT / MULE), small (start 40 rounds, depth 4; timed on Render at H8); reasons from built-in `pred_contrib` as φ_{k,j} − φ_{legit,j} (with the bias term, sums exactly to log P(k)/P(legit)); if the top two fraud types are within 1.5×, stack both challenges; challenge type from a policy table by risk type with cited evidence. Supersedes "SHAP summed by family" and the cost-argmin over efficacy values in D-003/D-004.
**Why:** SHAP credit splits arbitrarily across correlated features; efficacy values were unsourced; `shap` pulls in numba (RAM/cold start on Render's 0.1 CPU / 512 MB). Throwaway benchmark: 100 trees / depth 5 ≈ 2 ms p50 on 1 thread; 200 / depth 6 ≈ 22 ms.

## D-015 · 2026-10-28 · Rules scorecard first (Proposed)
**Decision:** A transparent points scorecard is the live engine by H8; LightGBM replaces it behind the same API by ~H30, shown side by side.
**Why:** The demo never depends on the model being ready.

## D-016 · 2026-10-28 · Telegram bot inside FastAPI instead of n8n (Proposed)
**Decision:** ~40 lines of Telegram Bot API on **long polling everywhere** (no webhook URL, so the laptop fallback runs the same code) with inline Approve / Cancel buttons (Asha's number in the message text, since URL buttons can't open `tel:`); the contact's decision is authorised by a per-hold one-time token or the verified `from.id`, never by the bank key alone; contacts verified with `request_contact` (`contact.user_id == from.id`) + a one-time code. Supersedes the n8n part of D-009.
**Why:** n8n needs an always-on host (₹0 options sleep), and its 30-min wait dies with a sleeping host.

## D-017 · 2026-10-28 · Trust rules for holds, contacts and passkeys (Proposed)
**Decision:** See `threat-model.md`. No hold releases on a timer; every SCAM hold is released only by the trusted contact or a bank analyst (the owner can cancel, never release; "hang up first" defeats owner-side re-checks); owner cool-off release only for non-scam soft holds (passkey + amount ≤ cap); holds/approvals/contacts persisted in Supabase; contact active after 72 h, changes frozen 72 h; hard-flagged payees unreleasable; passkey challenge bound to (one-time nonce, txn_id, amount, payee); score = max(server-only model, full model), thresholds set on that max, so forged client fields can't lower it; within 72 h of a SIM change a passkey alone can't release (bank call-back); client signals only raise risk; fail-safe timeout in the caller's SDK. Refines D-006 and D-011.
**Why:** Each closes an attack the council found (coach-to-wait, coach-to-change-contact, SIM-swap re-registration, forged client JSON, sleeping server).

## D-018 · 2026-10-28 · Investment-scam features + scam-aware profiles (Proposed)
**Decision:** Probation payees (trusted after 30 d ∧ ≥ 2 payments ∧ total ≤ usual monthly outflow ∧ no complaint); held payments don't update the baseline; features for 24 h new-payee outflow (splits), 30 d escalating outflow, bait-profit, and payee fan-in across users (collective immunity, hold-only, decays in 24 h).
**Why:** Investment scams are ~76% of losses and are repeat payments, which v1 would have learned as "normal".

## D-019 · 2026-10-28 · Corrected claims (Proposed)
**Decision:** RBI's lag = every first-time account-to-account (A2A) transfer above ₹10k (whitelisted/merchant exempt), not "every payment". Google's in-call protection = an app-level warning + 30-s pause, no per-payment hold. The demo's block scene = a payee on the (mock) DoT FRI Very-High list; card testing (not "15 PIN retries": UPI blocks after 3 wrong PINs, and emulators can't pass UPI SIM binding) stays in the replay and Q&A. Challenge budgets are policy targets, not "measured friction"; we show recall at each budget. Replay reports "intercepted", not "₹ prevented". No "unsaved caller" claim for WhatsApp. Positioning: "Google warns. RBI's proposed lag holds every first-time ₹10k+ transfer. Pramaan holds only the risky payment, says why, and asks the right person."
**Why:** Each was a fact a judge could disprove.
