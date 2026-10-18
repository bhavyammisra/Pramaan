# Architecture (v1, 2026-10-15; decisions logged in `decisions.md` D-002…D-008)

> Working name: **Pramaan** (प्रमाण, "proof"). It's a risk-based authentication layer that sits **on top of** mandated 2FA. It never replaces it.

## Components
```
 ┌──────────────────────────────┐
 │ Client: demo UPI-style app    │  txn + device + session behaviour
 │ + Pramaan SDK (JS / mobile)   │  (typing cadence, dwell, paste, on-call*, screen-share*, SIM-change*)
 └──────────────┬───────────────┘   *mobile-only signals: simulated by toggles in the web prototype
                │ POST /v1/authenticate   (HTTPS, JSON)
                ▼
 ┌──────────────────────────────┐      ┌──────────────────────────────────┐
 │ 1. API layer (FastAPI)        │      │ Profile store (in-memory → Redis) │
 │    validate · request id ·    │◀────▶│  user baseline: median amt, known │
 │    timing                     │      │  devices/payees, home geo, hours, │
 └──────────────┬───────────────┘      │  typing stats · payee reputation   │
                ▼                       │  (mock FRI / mule list)            │
 ┌──────────────────────────────┐      └──────────────────────────────────┘
 │ 2. Feature builder            │  ~25 features: ratios & z-scores vs the user's OWN baseline
 └──────────────┬───────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────┐
 │ 3. Risk engine                                                │
 │   a. Hard rules ── policy overrides (very-high-risk payee, bot │
 │      combo) and a fast-path allow for clearly normal payments  │
 │   b. LightGBM model ── p(fraud) in [0,1]                      │
 │   c. TreeSHAP ── per-signal contributions → reason codes      │
 │   d. Risk type ── IMPOSTOR / SCAM / BOT / MULE (strongest      │
 │      signal family)                                           │
 │   e. Decision policy ── expected-cost, amount-aware:          │
 │      allow | challenge(type) | block                          │
 └──────────────┬───────────────────────────────────────────────┘
                ▼
   Response: decision · risk_score · risk_type · challenge · reasons[] · latency_ms
                │
     ┌──────────┴────────────┐
     ▼                       ▼
 Event log (Postgres/       Feedback API  POST /v1/feedback
 Supabase, realtime)        (analyst or challenge outcome → profile update,
     │                       labels for retraining)
     ▼
 Dashboard (React): live decision feed · "why" panel · latency chart · threshold sliders
```

## Request flow
1. The SDK collects context and the client calls `/v1/authenticate` **before** the normal PIN/OTP step.
2. The API validates the payload (`api-contract.md`) and starts a timer.
3. The feature builder loads the user/device/payee profile and computes features against the user's **own** history.
4. Hard rules run first. A hit can force block/challenge or fast-path allow.
5. LightGBM → p(fraud). TreeSHAP gives each feature's contribution. Contributions are grouped by signal family to get the **risk type**.
6. The decision policy minimises expected cost for this amount, which picks allow / challenge / block. The challenge **type** is chosen by risk type (see `risk-engine.md`).
7. The response returns in milliseconds. The event is logged asynchronously and the dashboard updates live. After an allowed payment completes, the profile updates (new payee, rolling median).
8. **Fail-safe:** if the engine errors or misses its 150 ms internal deadline, the response is `challenge` with `type: standard_2fa`. The payment then goes through the normal mandated PIN/OTP flow, so we never fail *open* and never block a user because of our own outage (D-006).

## Latency budget (target p95 < 50 ms server-side; hard deadline 150 ms)
| Step | Budget |
|---|---|
| Validation + parsing | 2 ms |
| Profile lookups (in-memory / Redis) | 5 ms |
| Feature building | 3 ms |
| LightGBM + TreeSHAP | 10 ms |
| Policy + response | 1 ms |
| Headroom / network | remainder |

## Deployment ("cloud authentication service")
- API: Docker → **Render free tier** (no card). It sleeps after 15 min idle and wipes memory, so profiles are loaded from Supabase on startup and memory is only a cache; ping `/v1/health` every 10–14 min on demo day (D-009, D-011).
- Guardian loop: **n8n (self-hosted) + Telegram bot**. Scam-risk holds trigger an Approve/Cancel message to the user's family member; the payment goes only on Approve, and no reply in 30 min cancels it; the outcome becomes a label.
- Impostor level: **WebAuthn passkey on the owner's registered phone** (device-bound), not a fingerprint on the phone making the payment.
- Event log: **Supabase Postgres** (the team already knows it). Realtime channel feeds the dashboard.
- Dashboard + demo app: React (Vite) on **Vercel**.
- Model artefact (`model.txt` + feature list + version) ships inside the image. `model_version` is in every response.

## Scaling story (for judges)
The API is stateless, and the state lives in the profile store, so it scales horizontally behind any autoscaler (the prototype runs on Render; production would use something like Cloud Run). UPI averages ~9,300 txns/sec (Sept 2026). A GBDT scores in < 1 ms per core, so the bottleneck is profile I/O, which a managed cache such as Redis handles at this scale (the prototype keeps profiles in memory, backed by Supabase).

## Privacy by design
We store derived features only (typing *statistics*, not keystrokes), salted-hash device IDs, a profile retention window, and an audit log of every decision and its reasons (DPDP Act 2023; under the DPDP Rules, notice/consent duties apply from ~14 May 2027 and consent managers from Nov 2026).
