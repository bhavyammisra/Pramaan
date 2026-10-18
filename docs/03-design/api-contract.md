# API contract (v1, 2026-10-15)

Base URL: `https://<render-service>.onrender.com/v1` (Render free tier, D-009) · JSON · every response carries `request_id` and `latency_ms`.

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
                   "keystroke_mean_ms": 210, "confirm_dwell_ms": 18000, "payee_pasted": true,
                   "on_call": true, "screen_share": false, "remote_access_app": false },
  "user_context":{ "age_band": "70+", "trusted_contact_set": true }
}
```
Only `transaction` is required. Missing device/session fields are treated as "unknown" (a feature value of its own), not as safe.

### Response
```json
{
  "request_id": "req_9f2",
  "transaction_id": "txn_123",
  "decision": "challenge",
  "risk_score": 0.62,
  "risk_type": "SCAM",
  "challenge": { "type": "trusted_contact", "hold_minutes": 30,
                 "message": "Police, CBI or RBI never ask you to move money over a call. We've paused this payment for 30 minutes and asked your trusted contact to confirm. You can cancel anytime." },
  "reasons": [
    { "code": "ON_CALL",      "contribution": 0.21, "message": "You're on a phone call while sending money" },
    { "code": "AMOUNT_SPIKE", "contribution": 0.17, "message": "Amount is 71x your usual ₹1,200" },
    { "code": "NEW_PAYEE",    "contribution": 0.11, "message": "First payment to this receiver" }
  ],
  "latency_ms": 14,
  "model_version": "lgbm-v1-synth-2026-10-15"
}
```
- `decision`: `allow | challenge | block`
- `risk_type`: `NONE | IMPOSTOR | SCAM | BOT | MULE`
- `challenge.type`: `biometric_step_up | otp | cooling_off | trusted_contact | standard_2fa`
- `reasons[].contribution`: the TreeSHAP value (log-odds), or `null` with `"source": "rule"` for rule hits. Add `?detail=true` to get every feature's contribution.

## `POST /v1/feedback`
`{ "transaction_id": "txn_123", "label": "fraud | legit", "source": "analyst | challenge_outcome | chargeback" }` → `204`. This updates the profile (e.g. adds a payee) and stores the label for retraining.

## `GET /v1/decisions?limit=50`: recent decisions for the dashboard (also streamed via Supabase realtime).
## `GET /v1/health`: `{ "status": "ok", "model_version": "...", "p95_latency_ms": 12 }`

## Errors & fail-safe (D-006)
`400` invalid payload · `500` internal error. If the engine errors or misses its 150 ms internal deadline, it returns
`{ "decision": "challenge", "challenge": {"type": "standard_2fa"}, "reasons": [{"code": "ENGINE_UNAVAILABLE"}] }`.
The payment then falls back to the bank's normal mandated PIN/OTP flow: never weaker than today's check (but no scam protection during the outage), and it never blocks a user because of our own outage.
