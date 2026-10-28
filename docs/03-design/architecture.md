# Architecture (v2, 2026-10-28; v1 15 Oct. Decisions D-002…D-019; v2 changes from council review 1 are proposals D-012…D-019)

> Working name: **Pramaan** (प्रमाण, "proof"). It's a risk-based authentication layer that sits **on top of** mandated 2FA. It never replaces it.

## Components
```
 ┌──────────────────────────────┐
 │ Client: demo UPI-style app    │  txn + device + session behaviour
 │ + Pramaan SDK (JS / mobile)   │  (paste, dwell, app switches, on-call*, screen-share*, SIM-change*)
 └──────────────┬───────────────┘   *mobile-only signals: simulated by toggles in the web prototype
                │ via bank backend (API key) → POST /v1/authenticate, before the PIN step
                ▼
 ┌──────────────────────────────┐      ┌──────────────────────────────────┐
 │ 1. API layer (FastAPI)        │      │ Profile store (cache + Supabase)  │
 │    key · rate limit · validate│◀────▶│  baseline from TRUSTED payees,    │
 │    · request id · timing      │      │  probation payees, devices, hours,│
 └──────────────┬───────────────┘      │  payee fan-in · payee reputation   │
                ▼                       │  (mock FRI / mule list)            │
 ┌──────────────────────────────┐      └──────────────────────────────────┘
 │ 2. Feature builder            │  ~25 features: ratios & z-scores vs the user's OWN baseline
 └──────────────┬───────────────┘
                ▼
 ┌──────────────────────────────────────────────────────────────┐
 │ 3. Risk engine (v2, risk-engine.md)                           │
 │   a. Hard rules ── the ONLY blocks (flagged payee, card-testing│
 │      velocity, owner taps "Not me") + fast-path allow on       │
 │      server facts only                                         │
 │   b. Rules scorecard (H8) → multiclass LightGBM (H30):         │
 │      legit / IMP / SCAM / BOT / MULE; risk rank =              │
 │      max(server-only model, full model)                        │
 │   c. Reasons ── pred_contrib: φ_type − φ_legit (non-allow only)│
 │   d. Friction budget per amount band (bank-set) → challenge?  │
 │   e. Policy table by risk type → which challenge (stacked if  │
 │      two types are close)                                     │
 └──────────────┬───────────────────────────────────────────────┘
                ▼
   Response: decision · risk_rank · risk_type · challenge · reasons[] (top 3, REAL/SIMULATED tag) · latency_ms
                │
     ┌──────────┴────────────┐
     ▼                       ▼
 Event log + holds +        Holds & approvals: Telegram bot (in FastAPI,
 contacts (Supabase)        long polling) → contact Approve/Cancel; registered
     │                      phone polls pending passkey approvals; analyst
     │                      release ("bank call-back"); feedback = labels only
     ▼
 Dashboard (React): live decision feed · "why" panel (analyst key) · budget sliders · rules-vs-model
```

## Request flow
1. The user taps Pay. The app's SDK collects context and the bank backend calls `/v1/authenticate` (with the bank's API key) **before** the UPI PIN screen. The SDK owns the timeout (150 ms in production, 3 s in the demo).
2. The API validates the payload (`api-contract.md`) and starts a timer.
3. The feature builder loads the user/device/payee profile and computes features against the user's **own** history (trusted payees only; probation payees don't count).
4. Hard rules run first: a hit blocks (flagged payee, card testing, "Not me"), or the fast path allows on server facts only.
5. The scorecard (later LightGBM) gives a risk rank and a risk type; reasons come from `pred_contrib` only when the decision isn't allow.
6. The amount band's friction budget decides challenge vs allow; the policy table decides which challenge (passkey on the registered phone / hold + trusted contact / bank call-back).
7. On a hold, the app doesn't open the PIN step. Release (contact Approve or bank analyst for scam holds) unlocks the PIN step and the owner completes the payment herself. Holds, approvals and contacts are stored in Supabase.
8. **Fail-safe:** if the API is unreachable or slow, the SDK falls back to the bank's standard 2FA; for first-time ₹10k+ transfers the bank applies its RBI-style hold meanwhile (degrade to the regulator's rule, not to nothing). If the engine itself errors or misses its 150 ms deadline it returns `standard_2fa` (D-006, D-017).

## Latency budget (target p95 < 50 ms server-side; hard deadline 150 ms)
| Step | Budget |
|---|---|
| Validation + parsing | 2 ms |
| Profile lookups (in-memory cache backed by Supabase) | 5 ms |
| Feature building | 3 ms |
| LightGBM predict (+ `pred_contrib` only for non-allow) | timed on Render at H8 (0.1 CPU); throwaway 1-thread benchmark: ~2 ms p50 for 100 rounds × 5 classes, depth 5 |
| Policy + response | 1 ms |
| Headroom / network | remainder |

## Deployment ("cloud authentication service")
- API: Docker → **Render free tier** (no card). It sleeps after 15 min idle and wipes memory, so profiles are loaded from Supabase on startup and memory is only a cache; ping `/v1/health` every 10–14 min on demo day (D-009, D-011).
- Guardian loop: **Telegram bot inside FastAPI** (long polling, one poller per token; D-016, replaces n8n). Scam holds send Approve / Cancel to the verified trusted contact; only the contact or a bank analyst releases a scam hold; silence keeps it held (reminder at 10 min, call-back queue at 30 min). In the demo a judge can be "Rohan" through a no-install web page.
- Impostor level: **WebAuthn passkey on the owner's registered phone**, challenge = hash(nonce, txn_id, amount, payee); RP ID = the frontend's full production hostname; within 72 h of a SIM change a passkey alone isn't enough (threat model).
- Event log, holds, approvals, contacts: **Supabase Postgres** (the team already knows it; touch it daily so the free project doesn't pause).
- Dashboard + demo app: React (Vite) on **Vercel**.
- Model artefact (`model.txt` + feature list + version) trained offline and shipped inside the image; the Render runtime is FastAPI + LightGBM Booster + numpy only (512 MB). `engine` / `model_version` are in every response.
- Deploys come from a `demo` branch (Vercel production branch), so `main` can move freely.

## Scaling story (for judges)
The API is stateless, and the state lives in the profile store, so it scales horizontally behind any autoscaler (the prototype runs on Render; production would use something like Cloud Run). UPI averages ~9,300 txns/sec (Sept 2026). A small GBDT scores in a few ms per core (to be timed on Render), so the bottleneck is profile I/O, which a managed cache such as Redis handles at this scale (the prototype keeps profiles in memory, backed by Supabase).

## Privacy by design
We store derived features only (no keystrokes, audio or call content; "on a call" is a yes/no), salted-hash device IDs, a profile retention window, and an audit log of every decision and its reasons (DPDP Act 2023; under the DPDP Rules, notice/consent duties apply from ~14 May 2027 and consent managers from Nov 2026).
