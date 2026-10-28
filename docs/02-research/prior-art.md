# Prior art: what exists and what's missing

> Researched 2026-10-15. Feeds deck slide 2 ("Existing Approaches & Gap").

## A. Static two-factor authentication (the mandated baseline)
**What it does:** UPI uses device binding plus a UPI PIN. Cards use 3-D Secure plus an SMS OTP. Every transaction gets the same check. RBI's 2025 Directions keep 2FA as the floor from 1 Apr 2026 [1].
**Gap:**
- It verifies **the credential, not the context**. APP-scam victims type their own PIN/OTP, and RBI says APP is now most fraud [2].
- OTPs get intercepted by SIM swap [3] or read out over screen-share. Google has had to pilot screen-share alerts inside GPay/Paytm/Navi for exactly this [4].
- One-size friction: a ₹50 chai and a ₹2 lakh first-time transfer get the same check.

## B. Commercial risk engines (Stripe Radar, Sift, Feedzai, BioCatch, issuer ACS risk engines)
**What they do:**
- **Stripe Radar:** an ML model over "hundreds of risk factors" plus network data. It outputs a 0–99 score (elevated ≥ 65, high ≥ 75 by default). Highest risk is blocked, elevated goes to manual review, and a rule can request 3DS. "Risk insights" explanations exist only on the paid Fraud Teams tier [5].
- **Sift:** a 0–100 score, with customer-set thresholds for accept/review/step-up/block [6].
- **EMV 3DS 2.x:** issuers' ACS risk engines get 41 required + 150 optional data elements (10x 3DS 1.0) to run a "frictionless" vs "challenge" flow [7]. In Japan after its 2025 mandate, ~60% of 3DS transactions went frictionless [8].
- **BioCatch:** behavioural biometrics. In remote-access scams, the victim is **on an active phone call in up to 70% of sessions, vs < 1% of genuine sessions** [9] (BioCatch's own wording and data: a vendor claim, and "up to" is a ceiling, not an average). In India, ATO was 55% of fraud in its 2024 data, and 9/10 mule accounts went undetected at one partner bank [10]. ⚠ This clashes with RBI calling ATO "now negligible" [2]; see the judge Q&A in `../05-pitch/README.md`.
- **Razorpay ACS:** risk-based authentication for Indian cards, using customer, merchant and device parameters [11].

**Gap:**
- Proprietary and siloed. Behaviour, device and transaction intelligence are often **separate products**.
- Outputs are a **score for an analyst**, not a decision with plain-language reasons for the customer and the bank.
- The "challenge" is generic (OTP/3DS) **whatever the risk type**. An OTP challenge does nothing against a coached scam victim.
- Expensive or inaccessible for small banks, co-ops and fintech PSPs.

## C. National / ecosystem intelligence (India)
**What exists:**
- **NPCI FRM:** a real-time AI/ML scoring service offered to member banks, with alerts on a web portal [12].
- **DoT Financial Fraud Risk Indicator (FRI, May 2025):** rates mobile numbers Medium/High/Very-High. PhonePe declines Very-High. PhonePe, Paytm and GPay (> 90% of UPI) are integrating it [13].
- **RBI MuleHunter.AI:** mule-account detection, live in 26 banks by Mar 2026 [14].
- **IDPIC / Digital Payment Intelligence Platform:** shared fraud intelligence, piloting in 8 banks as of Sep 2026 [14].
- **RBI Apr 2026 proposals:** a 1-hour lag on first-time account-to-account transfers above ₹10k (merchant and whitelisted payees exempt), trusted-person approval for 70+ users above ₹50k, a ₹25 lakh annual credit cap on unverified accounts, and a kill switch [2].
- **Google in-call scam protection (Android 11+, pilots in UK, US, Brazil and India with GPay, Paytm, Navi):** opening a participating financial app while screen-sharing on a call with an unsaved number shows a warning, a one-tap "end call and stop sharing", and a 30-second pause [16]. On-device Gemini Nano call scam detection is Pixel 9+ only in India [4].

**Gap:**
- These are **reputation lists after the fact**. A number or account has to be reported first, and scam numbers live only a few days [13].
- They score the **beneficiary**, not *this session's* behaviour.
- The RBI lag is a **blanket rule for its class**: it delays every honest first-time ₹10k+ transfer, misses splits under ₹10k and repeat payments to a now-whitelisted scam payee, and a 1-hour wait is easy to coach around in a digital arrest that lasts days [15].
- Google's protection is a **phone-level warning**: it needs screen-sharing + an unsaved caller, doesn't see the amount, payee or the user's history, doesn't hold the money and doesn't involve family.

## The gap we are addressing
> **No lightweight, real-time service fuses behaviour + device + transaction context into an explained allow / challenge / block decision, and picks the *right kind* of challenge (approval on the registered phone for a possible impostor; a hold that only a trusted contact or the bank can release, plus a scam warning, for a possibly-coached genuine user). Most honest payments need to flow untouched.**

## What we borrow
| From | We borrow |
|---|---|
| Stripe Radar | Score → three-way action, adjustable thresholds, analyst feedback loop |
| 3DS 2.x | Frictionless vs challenge split driven by rich context |
| BioCatch | Session-behaviour signals: active call, remote access, typing cadence |
| FRI / MuleHunter | Beneficiary-reputation feature as one input (mocked in prototype) |
| RBI 2026 paper | Cooling-off delay and trusted-contact challenge, but **risk-targeted, not blanket** |
| Revolut scam intervention (2024) | ML flags a likely scam → questions + scam stories + specialist chat ("break the spell"); ~30% fewer investment-scam card losses |
| US FINRA Rules 2165 / 4512 (since 2018) | Firms may temporarily hold a 65+ client's payout on suspected exploitation and notify a pre-named **trusted contact person**: legal precedent for our hold + family loop |
| Google in-call protection | Call + screen-share state as a signal (proof it's obtainable on Android); we add the payment context, the hold and the family loop |
| PSD2 dynamic linking / W3C Secure Payment Confirmation | The approval is cryptographically bound to *this* amount and payee (we hash amount + payee + txn id into the passkey challenge) |

## Sources
1. RBI Authentication Directions press release: https://www.rbi.org.in/Scripts/BS_PressReleaseDisplay.aspx?prid=61282
2. RBI Discussion Paper, 9 Apr 2026: https://www.rbi.org.in/SCRIPTs/PublicationsView.aspx?id=23810
3. Outlook Money, Karnataka HC SIM-swap: https://www.outlookmoney.com/news/banks-must-take-steps-to-prevent-sim-swap-fraud-says-karnataka-high-court
4. TechCrunch, Google scam protection India (Nov 2025): https://techcrunch.com/2025/11/20/google-steps-up-ai-scam-protection-in-india-but-gaps-remain
5. Stripe Radar risk evaluation docs: https://docs.stripe.com/radar/risk-evaluation
6. Sift, fraud score explainer: https://sift.com/blog/fraud-score-how-ai-calculates-transaction-risk-in-real-time/
7. Mastercard, Top 10 things about 3DS: https://www.mastercard.com/content/dam/public/mastercardcom/globalrisk/pdf/Top-10-Things-to-Know-About-3DS.pdf
8. Stripe, 3DS trends in regulated markets (Aug 2025): https://stripe.com/blog/3ds-trends-in-regulated-markets
9. BioCatch, remote access scams: https://www.biocatch.com/blog/remote-access-scam-protect-customers
10. BioCatch India 2024 report (CXOToday): https://cxotoday.com/press-release/biocatch-releases-comprehensive-analysis-of-digital-banking-fraud-trends-in-india/
11. Razorpay ACS risk engine: https://razorpay.com/blog/razorpay-acs-risk-engine-achieving-up-to-95-authentication-success-while-minimizing-fraud/
12. NPCI FRM: https://www.npci.org.in/who-we-are/risk-management/fraud-risk-management
13. Business Standard, DoT FRI: https://www.business-standard.com/amp/industry/news/dot-launches-financial-fraud-risk-indicator-to-aid-cybercrime-detection-125052101912_1.html
14. IBTimes, IDPIC pilot / MuleHunter (Sep 2026): https://www.ibtimes.co.in/rbi-speeds-idpic-rollout-combat-digital-payment-frauds-pilot-running-8-banks-906120
15. Daily Mirror feature on RBI paper: https://www.dailymirror.lk/print/international/Indians-lost-25bn-to-digital-fraud-in-2025-now-its-central-bank-is-fighting-back/107-339105
16. Google, "Android expands pilot for in-call scam protection for financial apps" (3 Dec 2025): https://blog.google/security/android-expands-pilot-in-call-scam-protection-financial-apps/
17. FINRA Rule 2165 (Financial Exploitation of Specified Adults): https://www.finra.org/rules-guidance/rulebooks/finra-rules/2165
18. tech.eu, Revolut AI scam feature (Feb 2024): https://tech.eu/2024/02/15/revolut-launches-ai-scam-features-to-break-scammers-spell/
19. Chrome, Secure Payment Confirmation: https://developer.chrome.com/blog/spc-on-android
