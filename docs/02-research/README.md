# Research hub

> **Update 28 Oct:** design v2 (council review 1) replaces TreeSHAP-by-family and cost thresholds with a multiclass model, `pred_contrib` reasons and bank-set friction budgets: see `../03-design/risk-engine.md`.

| File | What's in it | Status |
|---|---|---|
| `problem-evidence.md` | Fraud stats (MHA/RBI/NPCI), the APP-scam shift, friction cost, stakeholders, SDGs | Done |
| `prior-art.md` | Static 2FA, commercial risk engines, India's ecosystem tools, and the gap statement | Done |
| `signals.md` | Signal catalogue by family, with priority and risk type | Done (v2) |
| `datasets.md` | Public datasets compared, plus the synthetic-generator plan | Done |
| `methods.md` | Hybrid scoring, LightGBM + TreeSHAP, cost-based thresholds, latency, privacy | Done |

## Answers to the open research questions
1. **Which public dataset covers device + behaviour + transactions?** None does. IEEE-CIS has transaction + device, PaySim has transfer patterns, BAF has device/session priors. → **Calibrated synthetic generator with labelled attack types** (`datasets.md`).
2. **Rules vs ML vs hybrid?** → **Hybrid**: hard rules + per-user baselines + LightGBM, explained with TreeSHAP reason codes. No LLM in the hot path (`methods.md` §1–3).
3. **How do real systems pick thresholds?** Stripe uses default score cut-offs (65/75 of 99) that merchants tune. Research shows **amount-aware, cost-based thresholds** save more. → **Expected-cost decision over allow/challenge/block, with challenge *type* chosen by risk type** (`methods.md` §4).

## The one-line insight
RBI (Apr 2026): account takeover is "now negligible"; most fraud is **authorised push payment**, where the real user enters their own PIN/OTP while being coached. **OTP checks the person; we check the situation.**
