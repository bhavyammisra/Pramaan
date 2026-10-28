# Methods research: scoring, explanations, thresholds, latency, privacy

> **Update 28 Oct (council review 1):** the design now uses a risk *rank* + bank-set friction budget (not k / amount thresholds), a multiclass model with LightGBM's built-in `pred_contrib` (not the `shap` library), and only hard rules block. See `../03-design/risk-engine.md` v2 and D-012…D-014. The research below is kept as background.

> Researched 2026-10-15. Answers open questions 2 and 3 in `README.md`.

## 1. Rules vs ML vs hybrid → **hybrid**
| Option | Pros | Cons |
|---|---|---|
| Rules only | Fully explainable, no data needed | Brittle, easy to game, can't weigh 20 weak signals together |
| ML only (GBDT) | Learns interactions, strong on tabular data | Needs labels; reasons aren't free; can miss "never allow X" policies |
| **Hybrid** | Hard rules for policy/known-bad + a GBDT for weighing + per-user baselines | Two things to maintain (fine at our scale) |

This is what production systems do. Stripe Radar is an ML score plus merchant rules (block / review / request 3DS) [1]. Sift is a score plus customer thresholds [2].

## 2. Model choice → **LightGBM (gradient-boosted trees)**
- Tabular data is GBDTs' home ground. They're fast and deterministic.
- A 2026 payment-fraud benchmark measured **p99 0.15 ms per transaction on one CPU core** for a GBDT. A simulated LLM scorer was ~1.2 s p99 and gave non-reproducible scores. Its conclusion: **GBDTs own the synchronous hot path, and LLMs only help asynchronously (case notes)** [3].
- So **no LLM in the decision path**. Reason messages are deterministic templates.

## 3. Explanations → **TreeSHAP contributions → reason codes → templated messages**
- TreeSHAP gives exact Shapley values for tree ensembles in polynomial time, O(T·L·D²) [4]. Reported CPU cost for medium models is ~0.1–1.5 ms per row on a 40-core box [4]. A small model (≈200 trees, depth ≤ 6) fits in our budget, and we'll measure it.
- Each feature maps to a **reason code** (e.g. `NEW_DEVICE`, `ON_CALL`, `AMOUNT_SPIKE`) with a message template filled with the user's own numbers: "Amount is 8.1× your usual ₹2,300."
- We show the top 3 positive contributors to users and all contributions to analysts.
- Rule hits are reasons too (contribution = override).

## 4. Thresholds → **cost-based, amount-aware, three-way**
Instance-dependent cost-sensitive learning (Höppner, Baesens et al., EJOR 2020) shows the optimal fraud threshold for transaction *i* is **t*ᵢ = c_f / Aᵢ**: a fixed handling cost over the amount. Big transfers get flagged at lower probabilities, and this saves more money than fixed-threshold models [5]. We extend that to three actions with expected cost:

```
cost(allow)     = p · A
cost(challenge) = c_friction + p · A · (1 − e_type)     # e_type = how well this challenge stops this fraud type
cost(block)     = (1 − p) · c_good_customer             # losing a good payment/customer
decision = argmin(cost), then hard-rule overrides
```
- **e_type is the key idea.** An OTP/biometric step-up stops an *impostor* (e≈0.9) but barely stops a *coached victim* (e≈0.1), since they'll enter it. A cooling-off delay + scam-specific warning + trusted-contact confirmation has much higher e for SCAM. So the engine picks the challenge **type** by risk type, not just "challenge yes/no".
- We also keep a **challenge-rate budget** (e.g. ≤ 5% of legit traffic) as a guardrail when tuning c_friction.
- For reference, Stripe's defaults are elevated ≥ 65/99 and high ≥ 75/99 [1]. Ours are learned from costs instead of picked by hand.

## 5. Latency budget
- Industry authorisation paths expect fraud decisions in **≤ 100 ms** [2][3]. NPCI cut the response-time limit for UPI "request pay" and related APIs to 15 s [6]. That's a timeout, not a decision window; the risk call must be a tiny slice of it.
- Our target: **p95 < 50 ms server-side** (stretch < 20 ms) with in-memory profiles. We measure it in the demo and show it on the dashboard.
- Throughput reference: UPI averages ~9,300 txns/sec (Sept 2026). The service is stateless apart from the profile store, so it scales horizontally.

## 6. Privacy (DPDP Act 2023 / Rules 2025)
- The DPDP Rules were published on **14 Nov 2025** (the Data Protection Board provisions started 13 Nov 2025). They phase in: the consent-manager rule (Rule 4) from **Nov 2026**, and the core duties (notice, consent, security, breach reporting, user rights: Rules 3, 5–16) from **about 14 May 2027**, 18 months after publication [7][8]. Check the Gazette for the exact day before quoting it.
- Our design: **derived features only**. We store typing *statistics*, not keystrokes, and no raw content. Device IDs are salted hashes, the profile store has a retention window, and every decision has a purpose-limited audit log.
- Section 17 exempts processing for *prevention/detection of offences* [7], but we design for consent + minimisation anyway.

## 7. Evaluation metrics we'll report
- Recall at a fixed challenge rate (by fraud type: IMP / SCAM / BOT)
- False-positive rate on legit users (challenged % and blocked %)
- **₹ saved** = fraud ₹ stopped − friction cost (cost-based, as in [5])
- p50/p95 latency

## Sources
1. Stripe Radar risk evaluation: https://docs.stripe.com/radar/risk-evaluation
2. Sift fraud score: https://sift.com/blog/fraud-score-how-ai-calculates-transaction-risk-in-real-time/
3. TDS, "The hot path belongs to GBDTs" (2026): https://towardsdatascience.com/the-hot-path-belongs-to-gbdts-agents-own-the-cold-path-a-payment-fraud-benchmark/
4. GPUTreeShap (PeerJ CS) with CPU TreeSHAP baselines: https://peerj.com/articles/cs-880
5. Höppner et al., Instance-dependent cost-sensitive learning for transfer fraud: https://arxiv.org/pdf/2005.02488
6. MediaNama, NPCI response-time circular: https://www.medianama.com/2025/06/223-npci-upi-changes-transaction-speed-fraud-prevention/
7. Hogan Lovells on DPDP Rules: https://ca.hoganlovells.com/en/publications/indias-digital-personal-data-protection-act-2023-brought-into-force-
8. JSA Prism, DPDP Rules phase-in (Nov 2025): https://www.jsalaw.com/?p=55573 · Lawrbit summary: https://www.lawrbit.com/article/digital-personal-data-protection-rules-2025
