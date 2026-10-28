# Risks, limitations & success metrics (feeds deck slide 5)

## What we will realistically build during the hack
A deployed cloud API (`/v1/authenticate`) that scores a payment in milliseconds and returns allow / challenge(type) / block with reasons. Also: a demo UPI-style payment app with a behaviour SDK, a live dashboard, and a synthetic dataset with 4 labelled attack types. **Not built:** a real bank/NPCI integration, a real mobile SDK (on-call/screen-share are toggles), real FRI/MuleHunter feeds (mocked).

## Genuine risks / limitations
| # | Risk | Why it's real | Mitigation |
|---|---|---|---|
| 1 | **Data realism**: the model learns our generator, not real fraud | No public behaviour+payment dataset exists | Hard negatives (travel, new phone, big rent); report metrics per fraud type; sanity-check on IEEE-CIS/PaySim; a real pilot would retrain on the bank's labelled history |
| 2 | **Signal availability / adoption**: on-call & screen-share need OS permissions and app integration | A browser can't see them; apps must adopt an SDK | Missing = "unknown", never "safe"; the model still works on txn + device signals; precedent: Google's screen-share alerts already in GPay/Paytm/Navi |
| 3 | **False positives hurt real people**: delaying a genuine urgent payment | Friction drives users away; blocking is worst | Only verified facts block (D-013); bank-set challenge budget per amount band (D-012); cancel is one tap; probation instead of instant whitelisting; release logged |
| 4 | **Adversaries adapt**: scammers tell victims to hang up, or to wait out the hold | Social engineering is adaptive | No single signal decides; trusted-contact approval; retrain on feedback |
| 5 | **Privacy (DPDP Act)**: behavioural data is personal data; core notice/consent duties apply from ~14 May 2027 (consent managers from Nov 2026) | Legal | Derived features only, hashed IDs, retention window, audit log |
| 6 | **Latency at UPI scale** (~9,300 txns/s average) | Shared 15 s UPI timeout | In-memory cache backed by Supabase, small GBDT (timed on Render at H8), fast-path rules, `pred_contrib` only for non-allow, SDK-side timeout → standard 2FA |
| 7 | **Demo hosting: Render free tier sleeps** | Spins down after 15 min without traffic; waking takes up to ~1 min, and local memory/files are wiped ([Render docs](https://render.com/docs/free)) | Load profiles from Supabase on startup (memory is only a cache); ping `/v1/health` every 10–14 min from demo day morning; warm it up before judging; report server-side `latency_ms`, not the first cold request |
| 8 | **Friction budget vs catch rate**: a tighter budget on first-time ₹10k+ payments holds fewer honest people but may miss scams | Our own policy choice (D-012) | Show recall at every budget per band (2% trusted / 10% new payee by default); move the default to the knee of the curve and say which way we traded |

## Success metrics (v2, council review 1: measured vs set, kept separate)
**Set by policy (we choose them, never call them "measured"):**
- Challenge budget per amount band: 0.2% < ₹2k · 1% ₹2–10k · 2% ₹10k+ trusted payee · 10% ₹10k+ new payee (doubled for 70+). Spoken as: *"of 100 honest first-time ₹10k+ payments, RBI's proposed lag holds 100; at our default we hold 10."*

**Measured (held-out synthetic attack variants + a timed run on Render):**
1. **Recall at each budget, per attack type** (curve; quote the default point). Includes the ablation without call/screen-share signals and unseen attacks (victim told to hang up, payee typed, ₹9,999 splits, repeat investment payments).
2. **Interceptions** in the replay for OTP-only · RBI lag · rules-only · Pramaan (held/challenged, not "₹ prevented").
3. **Server decision time** p50/p95 timed on Render free (0.1 CPU) at H8 and again at H36; client round trip reported separately.
4. **One honest failure case**, stated on stage.

**The three closing numbers** (held-out variants): SCAM recall at the 10% budget **with `on_call` removed** · the model's lift over the rules scorecard · % of honest payments held. Optional real-data check: transaction + device sub-model AUC on IEEE-CIS.

## Added risks (council review 1)
| # | Risk | Mitigation |
|---|---|---|
| 9 | On-site statement differs / pre-written code not allowed | Ask organisers 29 Oct; no project code until they say yes; H0 gate maps any statement onto the decision kernel (`finals-playbook.md` §1) |
| 10 | Scope too big for 48 h | MoSCoW tiers, hero flow by H20, cut order fixed in advance |
| 11 | Demo-day hosting (Render 0.1 CPU sleeps, Supabase pauses) | Keep-alive, `demo` branch, laptop + hotspot + quick tunnel, backup video by H40, kill-API switch shows the fail-safe |
| 12 | A judge asks something only one teammate can answer | Q&A owners per row; rehearsals at H36, H44, H47 |
