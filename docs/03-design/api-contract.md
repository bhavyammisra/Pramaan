# API contract (v2, 2026-10-28; v1 15 Oct)

> v2 changes (council review 1, proposed D-012…D-017): API key auth, `risk_rank` instead of a probability, no `detail` for customers, feedback can't whitelist payees, hold/approval endpoints. See `threat-model.md`.

Base URL: `https://<render-service>.onrender.com/v1` (Render free tier, D-009); runtime-configurable in the demo app so the API can move to a laptop tunnel · JSON · every response carries `request_id` and `latency_ms`.

**Auth:** every call carries `X-Api-Key` (one key per integrating bank; analyst endpoints need a separate analyst key). The demo app never holds a key: it calls a thin "bank backend" route (`/bank/*` in the same FastAPI app) that adds it. Rate limit per `user_id` and `device_hash`.

## `POST /v1/authenticate`
Called by the bank/PSP **before** the normal PIN/OTP step.

### Request
```json
{
  "transaction": { "id": "txn_123", "user_id": "u_42", "amount": 85000.0, "currency": "INR",
                   "type": "upi_p2p | upi_merchant | card | transfer",
                   "payee_id": "asha.helper@okaxis", "merchant_category": null,
                   "timestamp": "2026-10-15T23:10:00+05:30", "balance": 120000.0 },
  "device":      { "device_hash": "fp_abc", "ip": "203.0.113.5", "geo": {"lat": 30.35, "lon": 76.37},
                   "is_emulator": false, "is_vpn": false, "sim_change_hours": null },
  "session":     { "login_to_pay_ms": 41000, "failed_pin_24h": 0,
                   "confirm_dwell_ms": 18000, "payee_pasted": true, "app_switches": 3,
                   "on_call": true, "on_call_source": "toggle | android_audio_mode",
                   "screen_share": false, "remote_access_app": false },
  "user_context":{ "age_band": "70+", "trusted_contact_set": true }
}
```
`device.device_binding_id`: the id the bank bound to this phone at onboarding (server-side state, checked against the profile; in the demo it's seeded by Reset). A WebAuthn assertion, when present, comes through the approval endpoints below, not this call.

Only `transaction` is required. Missing device/session fields are treated as "unknown" (a feature value of its own), not as safe. **Client-reported signals can only raise risk**: `on_call: false` is treated like "unknown" (D-017). Keystroke timing was dropped for mobile web (Android keyboards report key code 229).

### Response
```json
{
  "request_id": "req_9f2",
  "transaction_id": "txn_123",
  "decision": "challenge",
  "risk_rank": 0.995,
  "risk_type": "SCAM",
  "challenge": { "type": "trusted_contact", "hold_id": "hold_77", "release": "contact_approval",
                 "message": "Police, CBI or RBI never ask you to move money over a call. We've paused this payment and asked your trusted contact to confirm. You can cancel anytime." },
  "reasons": [
    { "code": "ON_CALL",      "message": "You're on a phone call while sending money", "signal": "simulated" },
    { "code": "AMOUNT_SPIKE", "message": "Amount is 71x your usual ₹1,200", "signal": "server" },
    { "code": "NEW_PAYEE",    "message": "First payment to this receiver", "signal": "server" }
  ],
  "latency_ms": 14,
  "engine": "rules-v1 | lgbm-v2",
  "model_version": "lgbm-v2-synth"
}
```
- `decision`: `allow | challenge | block` (block only from hard rules, D-013)
- `risk_rank`: percentile of this payment's risk among honest traffic in the same amount band (not a probability, D-012)
- `risk_type`: `NONE | IMPOSTOR | SCAM | BOT | MULE`
- `challenge.type`: `passkey_registered_phone | trusted_contact | bank_callback | cooling_off | standard_2fa`, or a list when two risk types are close (e.g. `["passkey_registered_phone", "trusted_contact"]`)
- `reasons[]`: top 3, customer-facing, each tagged `signal: server | web | simulated`. No contributions here.

## `GET /v1/analyst/decisions/{transaction_id}` (analyst key)
All feature values and `pred_contrib` contributions for one decision, plus rules fired. For the dashboard's "why" panel.

## Holds and approvals
- `POST /v1/holds/{hold_id}/cancel` (owner, one tap) → `204`.
- `POST /v1/holds/{hold_id}/contact-decision` `{ "decision": "approve | cancel", "contact_token": "..." }` → releases or cancels a `trusted_contact` hold. **Authorised only by the contact's per-hold one-time token** (sent only to the contact's Rohan page / Telegram chat) or by the bot's verified Telegram `from.id`; the bank key alone is not enough, and this call never goes through `/bank/*`, so a scammer with remote access to Asha's phone (who can see `hold_id`) can't approve it. Never releases a hard-flagged payee.
- `POST /v1/holds/{hold_id}/owner-release` `{ "webauthn_assertion": {...} }` → releases **only a `cooling_off` hold** (non-scam soft hold on a risky payee; **never any SCAM hold**), only after the cool-off time, with a passkey assertion from the **registered** credential over `hash(nonce, txn_id, amount, payee)` (one-time server nonce), only if the amount is ≤ the bank's cap and there was no SIM change / new device in the last 72 h. Otherwise → `409 needs_contact_or_bank`.
- `GET /v1/approvals/pending?user_id=` → the registered phone's Pramaan page polls this; each pending approval carries `{nonce, txn_id, amount, payee}` and the challenge to sign.
- `POST /v1/approvals/{txn_id}` `{ "webauthn_assertion": {...} }` or `{ "not_me": true }` → approve this payment, or block it ("Not me" is a hard rule).
- `POST /v1/holds/{hold_id}/analyst-release` (analyst key) → the bank call-back path ("bank call-back (simulated)" button in the demo dashboard). Never releases a hard-flagged payee.
- Contact silence or a Telegram outage never changes a hold: reminder at 10 min, call-back queue at 30 min.
- Holds, approvals and contacts are **persisted in Supabase** (never only in memory), so a Render restart or a switch to the laptop loses nothing.
- `POST /v1/contacts` (enrol) → the contact completes Telegram `/start`, shares their number with the `request_contact` button (we check `contact.user_id == from.id`) and enters a one-time code; active after 72 h; changes frozen 72 h and the old contact is told (demo mode waives the 72 h, shown on screen).

## `POST /v1/feedback`
(analyst key) `{ "transaction_id": "txn_123", "label": "fraud | legit", "source": "analyst | complaint_1930 | chargeback" }` → `204`. Stores a label for retraining. **It never whitelists a payee**: payees become trusted only through probation (30 d, ≥ 2 payments, total ≤ usual monthly outflow, no complaint). Labels become final after a 30-day complaint window; a contact's approval is only a weak "legit".

## `GET /v1/decisions?limit=50`: recent decisions for the dashboard (also streamed via Supabase realtime).
## `GET /v1/health`: `{ "status": "ok", "model_version": "...", "p95_latency_ms": 12 }`

## Errors & fail-safe (D-006, refined by D-017)
`400` invalid payload · `401` bad key · `429` rate limit · `500` internal error. **The caller's SDK owns the timeout** (150 ms in production; 3 s network timeout in the demo), so a sleeping or dead server still falls back. The server's internal deadline is an env var (`ENGINE_DEADLINE_MS`: 150 in production, ~2,000 in the demo on Render's 0.1 CPU, so the live hero call never randomly drops to `standard_2fa`); we quote only the measured p95. While Pramaan is unreachable, the bank applies its RBI-style hold to first-time ₹10k+ transfers (degrade to the regulator's rule, not to nothing). If the engine itself errors or misses its 150 ms internal deadline, it returns
`{ "decision": "challenge", "challenge": {"type": "standard_2fa"}, "reasons": [{"code": "ENGINE_UNAVAILABLE"}] }`.
The payment then falls back to the bank's normal mandated PIN/OTP flow: never weaker than today's check (but no scam protection during the outage), and it never blocks a user because of our own outage.
