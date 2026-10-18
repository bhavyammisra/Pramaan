# Pitch & demo

## Screening deck (submitted 2026-10-15)
- **Final: `Pramaan_BIOS_Screening_v4.pptx`** (+ `.pdf`), built with `scripts/build_deck_v3.py`. The files below (`Pramaan_BIOS_Screening.pptx`, `_v2`, `_v3`) are earlier drafts kept for history.
- **Revised: `Pramaan_BIOS_Screening_v5.pptx`** (+ `.pdf`), built with `python scripts/build_deck_v5.py <template.pptx> <out.pptx>` (no icon files needed). Fixes from the v4 review: verified stats with sources, no made-up risk scores or latency, consistent worked example (71x usual ₹1,200), slide 2 ends with the template's "The gap we are addressing is:", a plain team diagram on slide 4 (template: simple beats polished), 48-hour plan (the hack runs 30 Oct to 1 Nov), fewer boxes and no icon badges.
- **`Pramaan_BIOS_Screening.pptx`**: the 5-slide deck in the official BIOS V2 template, with speaker notes (~45–60 s per slide) to read for the video. `.pdf` is a preview render.
- Rebuild: `python scripts/build_deck.py <BIOS_V2_CORE_FRONTIER_Screening_Template.pptx> docs/05-pitch/Pramaan_BIOS_Screening.pptx` (needs `python-pptx`). The template is kept locally, not in git.
- **Before submitting:** fill in the **Challenge ID** and **4 roll numbers** in the strip at the top of slide 1.

| Slide | Message | Key evidence |
|---|---|---|
| 1 Problem, Stakeholder & SDGs | Fraud has moved past OTPs: victims approve the payment themselves | ₹22,495 cr lost in 2025 (MHA); ~85% from investment/digital-arrest scams; 98.5% of fraud value > ₹10k; RBI: ATO "now negligible" |
| 2 Existing Approaches & Gap | Today's defences check the credential or the receiver, never the moment | Static 2FA · Radar/Sift/BioCatch · NPCI FRM/FRI/MuleHunter → gap statement |
| 3 Proposed Solution | OTP checks the person; we check the situation | Same ₹85k payment in 4 situations → allow / biometric / hold + trusted contact / block |
| 4 Technical Approach | One API call: context in, explained decision out | Input → API → features → rules + LightGBM + TreeSHAP + cost policy → output |
| 5 Build Plan, Risks & Impact | Deployed prototype, honest limits | 4 risks + mitigations; metrics: ≥80% scam catch at ≤5% challenge, ≥90% of honest ₹10k+ not delayed, p95 < 50 ms |

## Promo video (60 s)
- `promo/Pramaan_Promo_60s.mp4`: silent animated promo of the 4 situations; timed VO script in `promo/SCRIPT.md`; source + re-render in `promo/README.md`.

## Video (record from the deck)
1. Screen-record the deck in slideshow mode and read each slide's speaker notes (≈ 4–5 min total).
2. Each person can take one slide and Bhavyam two, or one narrator throughout.
3. Optional 15 s opener: "Asha, 72, is on a call with 'CBI'. She types her own UPI PIN. Which check stops that?"

## Demo script (for the hack round)
1. Legit purchase from a known device → **allow**, instant
2. Same user, new phone + SIM changed yesterday + new city, ₹85k → **challenge: biometric** (impostor)
3. Own phone at home, **on a call**, new UPI ID **pasted**, ₹85k → **challenge: 30-min hold + trusted contact** (scam), with reasons
4. Emulator + VPN, 15 attempts in 10 min → **block** (bot)
5. Dashboard: live feed, "why" panel, latency chart; drag the `k` slider to show the fraud-caught vs friction trade-off

## Likely judge questions (with answers)
- **How do you avoid blocking good users?** Block only at p ≥ 0.90 or a hard rule; challenges are capped at ≤ 5% of legit traffic; amount-aware thresholds; cancel/whitelist; feedback loop.
- **Your data is synthetic. Why trust the model?** No public dataset has these signals together (and real data is PII). We inject hard negatives, report recall per attack type, and sanity-check on IEEE-CIS/PaySim. A pilot would retrain on the bank's labelled history.
- **How do you know the user is on a call?** On Android, a mobile SDK can read call/screen-share state (Google already pilots screen-share alerts in GPay/Paytm/Navi). In the web prototype it's a toggle, and missing signals count as "unknown", never "safe".
- **Why not just use OTP / RBI's 1-hour delay?** OTP doesn't stop a coached victim. A blanket delay hits every honest ₹10k+ transfer. We delay only the risky ones.
- **Why these thresholds?** Amount-aware (t* = c / amount) from cost-sensitive fraud research (Höppner et al. 2020); block fixed high; tuned on data to a ≤ 5% challenge budget.
- **How does it scale?** Stateless API on Cloud Run; GBDT < 1 ms/core; Redis profiles; UPI average ~9,300 txns/s.
- **Privacy?** Derived features only, hashed device IDs, retention window, audit log; designed for DPDP Act consent rules (from 13 Nov 2026).
- **Why not an LLM?** ~1 s and non-deterministic. GBDT + TreeSHAP is ms-fast and reproducible. LLMs could later draft analyst case notes off the hot path.
