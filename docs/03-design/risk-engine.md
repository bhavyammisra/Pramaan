# Risk engine (v2, 2026-10-28, after council review 1)

> How signals → risk rank + risk type → decision → reasons. v1 (15 Oct) is in git history; what changed and why is in `decisions.md` D-012…D-018.
> Parameters live in `src/risk_engine/config.yaml`. Every number below is a **policy choice or an assumption**, labelled as such.

## What changed from v1 (one screen)
| v1 | v2 | Why |
|---|---|---|
| Calibrated "p(fraud)" and τ = ₹500 / amount | **Risk rank + a friction budget per amount band** | Our synthetic fraud rate (2–3%) is ~3,000× real UPI (~7 per million), so p isn't a real probability; v1's k (₹500) also didn't match the friction costs (₹3–40) |
| Block if p ≥ 0.90 | **Only verified server-side facts block**; the model can only challenge | The model is trained on our own generator; a judge-proof rule: no one is blocked on a guess |
| Risk type = sum of SHAP by family | **Multiclass model** (legit / IMP / SCAM / BOT / MULE) | Correlated features split SHAP credit arbitrarily; the generator already has type labels |
| Challenge type = argmin cost over invented efficacy | **Policy table by risk type**, with the evidence for each | The efficacy numbers had no source |
| `shap` library | LightGBM's built-in `pred_contrib`, small model (start at 40 rounds × 5 classes, depth 4) | No numba; fits Render free (0.1 CPU, 512 MB). Throwaway benchmark on 1 thread: 100 trees / depth 5 ≈ 2 ms p50 / 4 ms p95 for probabilities + contributions; 200 / depth 6 ≈ 22 ms p50. Expect ~10× slower on 0.1 CPU: **time it on Render at H8 and quote only that** |
| Payee becomes "known" after one payment | **Probation payees** + scam-aware profile updates | Investment scams (76% of losses) are repeat payments |
| Client signals trusted | **Client signals can only raise risk** | Unsigned JSON can be forged |

## 1. Features (computed against the user's own baseline)
Column **Source** = where the value comes from in the hackathon demo. Server-side facts can't be forged by the client.

| Code | Formula | Type | Source in demo |
|---|---|---|---|
| `amount_ratio` | amount / user.median_amount_30d (median over *trusted* payments only) | TXN | server |
| `new_payee` | payee ∉ user.trusted_payees | TXN | server |
| `probation_payee` | payee not yet trusted (< 30 days, < 2 payments, total > usual monthly outflow, or under complaint) | SCAM | server |
| `new_payee_outflow_24h` | Σ amount to new/probation payees in 24 h (**splits under ₹10k**) | SCAM | server |
| `new_payee_outflow_30d_ratio` | Σ to probation payees in 30 d / usual monthly outflow (**escalating investment payments**) | SCAM | server |
| `bait_profit` | credit from a new counterparty in last 7 d ∧ now paying more than that to a new payee | SCAM | server |
| `payee_risky_senders_24h` | # unrelated users who paid this payee in 24 h under risky context (**collective immunity**) | MULE/SCAM | server |
| `payee_risk` | mock DoT FRI tier 0–3 + mule list | MULE | server (mocked feed) |
| `balance_drain` | amount / balance | SCAM | server |
| `unusual_hour` | hour ∉ user's 95% interval | IMP | server |
| `geo_velocity_kmh`, `new_ip_asn` | distance / Δt; ASN not seen for user | IMP | server (IP) |
| `txn_count_10m`, `new_payees_10m` | velocity | BOT | server |
| `bound_device` | request comes from the device the bank bound at onboarding (`device_binding_id` matches the profile); a passkey on that device strengthens it once the impostor path ships | IMP (lowers risk) | server state (seeded in demo) |
| `payee_pasted` | `input` event with `inputType = insertFromPaste` | SCAM | web SDK (real) |
| `confirm_dwell_z` | time on confirm screen vs user | SCAM | web SDK (real) |
| `app_switches` | `visibilitychange` hidden→visible during the payment flow | SCAM | web SDK (real) |
| `login_to_pay_ms` | from SDK | BOT | web SDK (real) |
| `on_call` | cellular **or VoIP** call active (Android audio mode) | SCAM | Android wrapper (stretch) else **SIMULATED** toggle |
| `screen_share` | screen recording/sharing active (Android 15 callback) | SCAM/IMP | Android wrapper (stretch) else **SIMULATED** |
| `sim_change_hours`, `new_device` | from the bank's device binding | IMP | **SIMULATED** (bank data) |
| `is_emulator`, `is_vpn` | SDK / IP intel | BOT | SIMULATED / IP |
| `vulnerable_user` | age ≥ 70 or user opted in | context: tightens the budget, never a fraud signal by itself | server |

Dropped for mobile web: keystroke timing (Android keyboards report key code 229 / "Unidentified", so it's noise). Kept for desktop only.

## 2. Trust rules
1. **Client signals can only raise risk.** A client value of `on_call: false` is treated the same as "unknown". For the model this needs enforcing, because a tree model *can* lower a score when a client field looks "clean": we also train a **server-only model** (no client-reported fields) and take **score = max(server-only score, full score)** (both as percentiles), then **set the band thresholds on that max itself**, ranked against held-out honest traffic (the max of two percentiles isn't itself a percentile: without re-ranking, a 10% budget could become up to 20%). Tested with a spoofed "clean" client in the held-out set.
2. **Fast-path allow uses only server-side facts:** trusted payee ∧ amount_ratio < 2 ∧ bound device ∧ no risky payee flags. ~Most traffic stops here and never reaches the model.
3. **Missing = unknown**, never safe.
4. Production: the SDK payload is signed with a per-install key in Android Keystore and checked with Play Integrity (free); see `threat-model.md`.

## 3. Scoring
**Stage 1 (live by H8): rules scorecard.** Transparent points per feature (e.g. on_call +30, probation payee +20, amount_ratio > 20 +25 …), summed per risk type. The demo never depends on the model.
**Stage 2 (H30): multiclass LightGBM** (start 40 rounds, depth 4, 1 thread; time it on Render at H8 — if p95 > 50 ms there, the rules decide and the model's view is shown beside them) over the same features, classes legit / IMP / SCAM / BOT / MULE.
- `risk = 1 − P(legit)` → used only as a **rank** (percentile among honest traffic in the same amount band).
- `risk_type = argmax` over the fraud classes. If the top two fraud types are within 1.5× of each other (e.g. IMP vs SCAM), **stack both challenges** (passkey on the registered phone *and* hold) instead of a coin flip.
- `pred_contrib` runs **only for non-allow decisions** (allowed payments don't need reasons), so the common path is a plain predict. At H8, time a dummy model of the final size on Render, measuring predict and `pred_contrib` separately (contribution cost grows with trees × leaves × depth², which is why 200 rounds / depth 6 was ~10× slower than 100 / depth 5).
- Reasons, exactly: softmax gives `log P(k)/P(legit) = z_k − z_legit`, and `pred_contrib` splits each raw score `z_k` over features, so **reason weight of feature j = φ_{k,j} − φ_{legit,j}**; these weights plus the baseline term (bias_k − bias_legit, the last `pred_contrib` column) sum exactly to log P(k)/P(legit).
The dashboard shows "rules said / model said" side by side.

**Hard rules (the only things that block):** payee on the flagged list (mock DoT FRI Very-High, like PhonePe; this is the demo's block scene) · card-testing velocity (≥ 20 `type: card` attempts under ₹10 from one device/IP in 5 min; UPI itself blocks after 3 wrong PINs and emulators can't pass UPI SIM binding, so the bot story is card testing on the same API) · the owner taps **"Not me"** on the registered phone, or a passkey assertion arrives with an invalid signature / unknown credential. (A *failed* biometric is not visible to the server: WebAuthn returns the same `NotAllowedError` for cancel, timeout and a wrong finger, so we never block on "failure", only on an explicit "Not me" or a bad signature.)

## 4. Decision policy: friction budget per amount band
The bank chooses how much friction it will spend; we put it where the money is. Thresholds = the risk-rank percentile that gives this challenge rate on held-out honest synthetic traffic. **These are policy targets, not measured friction.** What we *measure* is the catch rate at that budget on held-out attacks.

| Band | Max share of honest payments challenged (default) |
|---|---|
| < ₹2,000 | 0.2% |
| ₹2,000–10,000 | 1% |
| ≥ ₹10,000, trusted payee | 2% |
| ≥ ₹10,000, new / probation payee | 10% (RBI's proposed lag: 100% of first-time ₹10k+ A2A) |

`vulnerable_user` doubles the budget in each band. Budgets are sliders on the dashboard. **The 10% default** is a starting point (one-tenth of the payments RBI's proposed lag would hold); after the first run we move it to the knee of the recall-vs-budget curve on held-out attacks and say so.

## 5. Which challenge (policy table by risk type)
| Risk type | Challenge | Evidence |
|---|---|---|
| IMPOSTOR | Approve **this payment** with the passkey on the **registered phone** (the phone's Pramaan page polls for pending approvals; challenge = hash(one-time server nonce, txn_id, amount, payee)); "Not me" blocks | A SIM-swapper doesn't hold the original phone's key; amount + payee are bound into what's signed (PSD2 dynamic-linking idea) |
| SCAM | **Hold** + spell-breaker screen (3 questions, I4C line, recorded warning) + **trusted contact** approval; bank call-back if no contact | Revolut intervention flow (~30% fewer investment-scam losses); Google's 30-s pause; FINRA 2165 hold + trusted contact; RBI's trusted-person proposal |
| BOT | Block (hard rule) or passkey (bots can't do user-verification) | — |
| MULE (payee) | Hold + warning that names the payee risk; feeds collective immunity | DoT FRI / PhonePe practice |
Fallback if the user has no registered passkey: hold + bank call-back. `cooling_off` (owner can release after the cool-off with a passkey) is used **only** for non-scam soft holds on a risky payee.

**Where the hold sits in the UPI flow (two integration points):**
1. **In the payer's own bank/PSP app (our demo):** Pramaan is called when the user taps Pay, **before** the UPI PIN screen (the PIN is entered in NPCI's library). A hold = the app doesn't open the PIN step; release = the app unlocks it and the owner completes the payment herself. Money never moves without her own PIN.
2. **At her bank, for payments from other UPI apps:** the remitter bank only sees the request *after* the PIN, via NPCI, so it can't hold before the PIN: it **declines with the reason and notifies** her and the contact; once the contact approves, that payee + amount is pre-approved for a short window and she retries.

**Release rules (no hold ever releases on a timer; see `threat-model.md`):**
- *Scam hold with a contact:* released only by the contact's Approve or a bank analyst ("bank call-back", simulated in the demo). The owner can always cancel but **never releases a scam hold alone** ("hang up, release, call back" is the coaching). Silence or a Telegram outage keeps it held: reminder at 10 min, bank call-back queue at 30 min.
- *Scam hold with no contact enrolled:* bank analyst only ("bank call-back"). **One rule for every SCAM hold: only the trusted contact or the bank releases it**; a coached victim can always "hang up first", so the owner's passkey + a no-call re-check is not enough.
- *Cooling-off hold (non-scam soft holds only: a collective-immunity or soft mule flag on the payee, no scam signals in the session):* the owner can release after the cool-off with the passkey on the registered phone ∧ amount ≤ the bank's cap.
- *Within 72 h of a SIM change / new device:* a passkey alone is not enough; held → bank call-back.
- *Hard-flagged payee:* nobody can release.

## 6. Profiles and labels (scam-aware)
- A payee becomes **trusted** only after 30 days ∧ ≥ 2 payments ∧ total sent to them ≤ the user's usual monthly outflow ∧ no complaint (multi-week investment scams would pass a 7-day probation). Held/challenged/probation payments never update the median or the payee list.
- Labels: final only after a 30-day complaint window or a confirmed 1930/NCRP report. A contact's approval = weak "legit". `/v1/feedback` can't whitelist payees.

## 7. Explanations
Reason templates filled with the user's own numbers (`AMOUNT_SPIKE`: "Amount is {ratio}× your usual ₹{median}", `ON_CALL`, `NEW_PAYEE`, `SPLIT_PAYMENTS`: "₹{sum} sent to new receivers in 24 h", `BAIT_PROFIT`: "You received ₹{x} from a stranger this week and are now sending more", `PAYEE_MANY_VICTIMS`: "{n} other people paid this receiver today while on a call", `NEW_DEVICE`, `SIM_CHANGE`, `PAYEE_FLAGGED`). Customers see the top 3; analysts (separate key) see all contributions.

## 8. Evaluation (what we report, measured)
- **Held-out attack variants, written blind:** the generator (≈300 users × 60 days: 30 warm-up for baselines/probation + 30 eval) has variant parameters; the held-out variants are specified by a teammate who doesn't build the model (victim told to hang up first, payee typed, ₹9,999 splits, repeat investment payments after a bait profit, spoofed "clean" client). A second fully independent generator is talk-only unless time allows.
- **Ablation:** all signals · without call/screen-share (server + real web signals only) · rules-only.
- **Unseen attacks** (not in training): victim told to hang up first · payee typed not pasted · ₹9,999 splits · repeat investment payments after a bait profit.
- **Recall-vs-budget curve** per attack type (the honest-challenge rate is the budget we set, so we never call it "measured friction").
- Replay on the same traffic, reporting **payments intercepted** (held/challenged), **still stopped if the victim is coached to wait out any timer** (a labelled assumption: a timer-based hold like RBI's lag releases after an hour; an approval-based hold doesn't), and honest payments held, for OTP-only · RBI lag (first-time ₹10k+ A2A → 1 h, cancellable) · rules-only · Pramaan. Headline the after-coaching column. We don't claim "₹ prevented": that needs intervention efficacy we can't measure.
- Latency: server decision time p50/p95 (measured) and client round trip, reported separately.
- Optional: transaction+device sub-model AUC on IEEE-CIS (real data). If not done, don't claim it.

## 9. Worked example (deck + demo)
Asha, 72, usually pays ₹1,200. She sends **₹85,000** to a **new UPI ID** (pasted) from **her own phone at home** while **on a WhatsApp call**.
→ Fast path fails (new payee, 71× amount). Device and location look clean: an OTP would pass.
→ Rank: top 0.5% of honest ₹10k+ new-payee payments, above the 10% budget line (20% for a 70+ user) → **challenge**. Type: P(SCAM) highest → **hold + trusted contact**.
→ Reasons: "You're on a call while sending money" · "Amount is 71× your usual ₹1,200" · "First payment to this receiver".
*(Numbers are illustrative until measured; the deck must say so.)*
