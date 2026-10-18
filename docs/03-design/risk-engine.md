# Risk engine (v1, 2026-10-15)

> How signals → score → risk type → decision → reasons. Research behind it: `../02-research/methods.md`.
> All numeric parameters below are **initial values** to tune on the synthetic data. They live in `src/risk_engine/config.yaml`.

## 1. Features (P0 set, computed against the user's own baseline)
| Code | Formula | Family |
|---|---|---|
| `amount_ratio` | amount / user.median_amount_30d | TXN |
| `amount_z` | (log amount − mean log amt) / std (robust, MAD) | TXN |
| `new_payee` | payee ∉ user.payees | TXN |
| `new_payee_high_amt` | new_payee ∧ amount_ratio > 5 | SCAM |
| `balance_drain` | amount / balance | SCAM |
| `round_amount` | amount % 1000 == 0 | SCAM |
| `on_call` | SDK: active phone call | SCAM |
| `screen_share` | SDK: screen-share / remote-access app | SCAM |
| `payee_pasted` | SDK: payee field pasted | SCAM |
| `confirm_dwell_z` | z of time on confirm screen | SCAM |
| `new_device` | device_hash ∉ user.devices | IMP |
| `sim_change_recent` | hours since SIM change < 72 | IMP |
| `geo_velocity_kmh` | distance(last_geo, geo) / Δt | IMP |
| `unusual_hour` | hour ∉ user's 95% hour interval | IMP |
| `typing_z` | z of mean inter-key interval vs user | IMP |
| `failed_pin_24h` | count | IMP |
| `login_to_pay_ms` | from SDK | BOT |
| `txn_count_1h` | velocity | BOT |
| `is_emulator`, `is_vpn` | SDK / IP intel | BOT / IMP |
| `payee_risk` | mock FRI tier 0–3 + mule score | MULE |
| `payee_age_days`, `payee_fanin_24h` | receiver stats | MULE |
| `vulnerable_user` | age ≥ 70 | context (lowers thresholds, not a fraud signal by itself) |

## 2. Scoring: hybrid
**a. Hard rules (evaluated first):**
- `payee_risk == VERY_HIGH` → **block** (as PhonePe does with FRI Very-High).
- `is_emulator ∧ txn_count_1h ≥ 10` → **block** (card testing).
- **Fast-path allow:** known device ∧ known payee ∧ amount_ratio < 2 ∧ no SCAM/IMP flags → allow without calling the model. This keeps most traffic frictionless and fast.

**b. Model:** LightGBM binary classifier, ~200 trees, depth ≤ 6, trained on the synthetic data (`../02-research/datasets.md`), with class weights for the imbalance. Output p = P(fraud | features), calibrated with isotonic regression so the cost maths is meaningful.

**c. Contributions:** TreeSHAP values φⱼ for each feature (in log-odds). `risk_score` = calibrated p.

**d. Risk type:** sum the positive φⱼ by family (IMP, SCAM, BOT, MULE). The family with the largest sum is the `risk_type`. This drives the challenge type.

## 3. Decision policy: amount-aware thresholds + cost-based challenge type
Inputs: calibrated p, amount A (₹), risk type t, vulnerable_user.

**Step 1: block or not.** Block if a hard rule fires or **p ≥ τ_block = 0.90**. Blocking a genuine person is the worst outcome (they can't pay rent or fees, and complain to the regulator), so this threshold is high and fixed.

**Step 2: challenge or allow.** This is the amount-aware threshold from instance-dependent cost-sensitive learning (t* = c / A, Höppner et al. 2020):
```
τ_challenge(A) = clip( k / A , τ_min = 0.05 , τ_max = 0.60 )     k = ₹500 (friction budget)
vulnerable_user → τ_challenge × 0.6
challenge if p ≥ τ_challenge(A), else allow
```
| Amount | τ_challenge | Meaning |
|---|---|---|
| ₹40 | 0.60 | small payments are challenged only when very suspicious |
| ₹1,000 | 0.50 | |
| ₹5,000 | 0.10 | |
| ≥ ₹10,000 | 0.05 | where ~98.5% of fraud value sits (RBI 2026), so we're strict |

**Step 3: which challenge.** Pick the challenge c that minimises expected cost for this risk type:
`cost(c) = friction[c] + p · A · (1 − eff[c][t])` (only challenges the user has set up are eligible)

| Challenge c | friction[c] (₹-equiv.) | eff vs IMP | eff vs SCAM | eff vs BOT | eff vs MULE |
|---|---|---|---|---|---|
| `biometric_step_up` (passkey bound to the user's **registered** phone, not the phone making the payment) | 3 | 0.90 | 0.10 | 0.95 | 0.10 |
| `otp` (fallback) | 5 | 0.60 | 0.05 | 0.90 | 0.05 |
| `cooling_off` (hold 30–60 min + scam-specific warning, cancel anytime) | 25 | 0.30 | 0.70 | 0.50 | 0.60 |
| `trusted_contact` (payment held; it goes only if the nominated person approves; no reply in 30 min = cancelled) | 40 | 0.60 | 0.85 | 0.50 | 0.70 |

What this gives us:
- A **coached victim (SCAM)** gets cooling-off / trusted contact, never an OTP they'd just type in. A likely **impostor (IMP)** must approve with a passkey on the owner's **registered** phone. A fingerprint on the new phone would prove nothing, since the attacker unlocks that phone with their own finger. Use device-bound passkeys (not cloud-synced) for this; if the owner really has a new phone, fall back to the bank's re-registration flow. A **bot** gets a cheap step-up it can't do.
- Most traffic (known device, known payee, normal amount) is allowed, often by the fast-path rule, so it never sees friction.
- **Guardrail:** we tune `k` (and `τ_min`) so the challenge rate on legit synthetic traffic stays ≤ 5% **and** ≥ 90% of honest ≥ ₹10k transfers pass without delay (`../04-plan/risks-and-metrics.md`). With τ_min = 0.05 at ≥ ₹10k, honest first-time big payments may cross it, so check this first once the model runs. Dashboard sliders for `k` and `τ_block` show the fraud-caught vs friction trade-off live.
- Compare RBI's proposed blanket rule (delay *every* APP transfer ≥ ₹10k): we only delay the ones whose context is risky.

## 4. Explanations
- Each feature has a reason code and a message template filled with the user's own numbers:
  - `AMOUNT_SPIKE`: "Amount is {ratio}× your usual ₹{median}"
  - `ON_CALL`: "You're on a phone call while sending money"
  - `NEW_PAYEE`: "First payment to this receiver"
  - `NEW_DEVICE`: "First time this phone is used on your account"
  - `SIM_CHANGE`: "Your SIM was changed {h} hours ago"
  - `PAYEE_FLAGGED`: "This receiver's number is flagged for fraud"
- The response has the top 3 positive reasons (customer-facing) plus all contributions (`detail=true`, for analysts).
- Rule hits are reasons too, with `contribution: null, source: "rule"`.
- The challenge screen shows a **type-specific** message. For SCAM: "Police, CBI or RBI never ask you to transfer money over a call. We've paused this payment for 30 min. Cancel anytime."

## 5. Evaluation (what we report)
- Recall per fraud type at the operating point; false challenge rate and false block rate on legit traffic.
- ₹ saved = fraud ₹ prevented − Σ friction cost (cost-based, not just F1).
- Latency p50 / p95 from a 1,000-request load test (`scripts/load_test.py`).
- Baselines to beat: (i) "OTP everything" (100% friction, ~0% SCAM catch), (ii) a fixed 0.5 threshold, (iii) the RBI-style blanket 1-hour delay on ≥ ₹10k transfers.

## 6. Worked example (used in the deck and demo)
Asha, 72, usually pays ₹1,200. She sends **₹85,000** to a **new UPI ID** from **her own phone, at home**, while **on a call**, with the payee **pasted**.
→ The device and location look clean (an OTP would pass). Model p = 0.62, top family **SCAM**.
→ Policy: p = 0.62 < τ_block 0.90 → not blocked. τ_challenge(85,000) = 0.05 × 0.6 (vulnerable) = 0.03 → challenge. Type: trusted_contact costs 40 + 0.62·85,000·0.15 ≈ ₹7.9k, cooling_off ≈ ₹15.8k, biometric ≈ ₹47k → **trusted_contact with a 30-min hold** (cooling_off if no contact is set).
→ Reasons: "You're on a call while sending money" · "Amount is 71× your usual ₹1,200" · "First payment to this receiver".
