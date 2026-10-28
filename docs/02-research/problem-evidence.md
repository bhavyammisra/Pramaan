# Problem evidence, stakeholders & SDGs

> Researched 2026-10-15. Every number has a source at the bottom. Feeds deck slide 1.

## 1. Headline numbers (use these on slide 1)

| Fact | Number | Source |
|---|---|---|
| Cyber-fraud losses reported on NCRP, 2025 | **~₹22,495–22,931 crore, ~28 lakh complaints** (cases up ~24% vs 2024) | MHA via ThePrint [1]; RBI discussion paper [2] |
| Same, 2024 | ₹22,845 crore lost (vs ₹7,465 cr in 2023, ~3x) | MHA, Lok Sabha [3] |
| Growth since 2021 | 2.6 lakh frauds / ₹551 cr (2021) → 28 lakh / ₹22,931 cr (2025) | RBI discussion paper [2] |
| UPI fraud | FY24 ₹1,087 cr (13.4 lakh cases), FY25 ₹981 cr (12.6 lakh), FY26 Apr–Nov ₹805 cr (10.6 lakh) | MoS Finance, Lok Sabha [4] |
| Bank-reported card/internet fraud (≥ ₹1 lakh only) | FY25: 13,516 cases, ₹520 cr = **56.5% of all bank fraud cases** | RBI Annual Report FY25 [5] |
| Seniors & women, 2025 | Seniors: 1.03 lakh complaints, ₹4,005 cr lost. Women: 4.63 lakh, ₹3,765 cr | MHA, Rajya Sabha [6] |
| Scale we must be fast at | UPI Sept 2026: **24.07 bn txns, ~802 M/day** (~9,300 txns/sec average) | NPCI data via StartupTalky [7] |
| UPI time budget | NPCI cut "request pay" response time from 30 s → 15 s (Jun 2025) | MediaNama [8] |

## 2. The key insight: the fraud has moved past what OTPs can stop

- RBI (Apr 2026): **"frauds attributed to account take-over are now negligible"**. Most fraud is now **Authorised Push Payment (APP)**: the *real* user, on their own phone, enters their own PIN/OTP, because a scammer is coaching them [2].
- In 2025, **investment scams = ~76% of money lost** and "digital arrest" scams ~9% [1]. In both, the victim pays themselves.
- Transactions **> ₹10,000 = ~45% of fraud cases but ~98.5% of fraud value**. ~92% of fraud value is in transactions > ₹50,000 [2].
- SIM-swap still beats OTP: Karnataka HC case, ₹87.7 lakh drained in 7 transfers using intercepted OTPs; the court suggested delays for large transfers after a SIM swap [9].
- **Careful with trends:** 2025 losses (₹22,495 cr) are slightly *below* 2024 (₹22,845 cr); only complaints rose (~24%). Don't say "losses are rising". Say losses stay at ~₹22,500 cr a year while complaints keep growing.
- **Careful with ATO:** RBI's "ATO now negligible" is about bank-reported fraud classification. BioCatch reported ATO as 55% of fraud in its 2024 India session data (`prior-art.md`). Different data sets; we pitch both IMP and SCAM handling, so we don't depend on either.
- **So the question isn't just "is this the right person?" It's "is this payment normal for this person, right now, and are they being manipulated?"** PIN/OTP answers the first question only.

## 2b. How the two big scams actually run (feeds features + demo; added 28 Oct)

- **Digital arrest runs on WhatsApp/Skype *video* calls, for hours or days.** E.g. a Telangana couple was kept on a WhatsApp video call for ~50 h and paid ₹30 lakh [16]. Android's phone-state API sees only cellular calls; during a VoIP call the *calling app* switches Android's audio mode to "in communication", which other apps can read (to verify on our phones with `adb shell dumpsys audio`). I4C (5 Oct 2024): **"CBI/Police/Custom/ED/Judges DO NOT arrest you on video call"**; report on 1930 / cybercrime.gov.in [17].
- **Investment scams are repeat, escalating payments, often with no call.** Pattern: small first deposit → a fake "profit" credited *back* to the victim → bigger deposits → a "tax/fee to withdraw". The Gwalior case (70-year-old, ₹21 cr) went through 4 layers and ~20,500 mule accounts [18]. So after payment #1 the payee looks "known"; the signal is *escalating outflow to recent payees* and *money in from a stranger, then more money out*.
- **Behaviour of a coached victim** (industry): longer sessions with aimless movement, segmented entry (reading digits out), hesitation before Submit, frequent device reorientation [19]. Revolut's ML + "scam intervention flow" (questions, education, a specialist chat) cut investment-scam card losses ~30% [20].
- **Base rate:** FY25 UPI = 185.85 bn transactions (₹2,605.6 lakh crore) [21] vs 12.6 lakh UPI frauds (₹981 cr) [4] ⇒ **~7 frauds per million transactions**. Our synthetic data uses 2–3% fraud for stable training, so model scores are a *ranking*, not a real-world probability.
- **UPI PIN:** 3 wrong attempts in 24 h block UPI payments for 24 h [22], so "15 PIN retries" can't happen on UPI.
- **Refunds won't save Asha:** RBI's compensation framework (notified Jun 2026, effective 1 Jan 2027; coverage of scams the victim authorised herself is unclear) pays at most 85% / ₹25,000, once per lifetime, only for losses up to ₹50,000 [23]. An ₹85,000 digital-arrest loss is outside it, so prevention is the only protection.

## 3. Friction is also a cost (why we can't just challenge everything)

- RBI's own proposal (Apr 2026) is a **1-hour lag on every first-time account-to-account transfer above ₹10,000** (merchant, bill, recurring and whitelisted payees exempt; the sender can cancel in the lag), plus trusted-person approval for 70+ citizens and persons with disabilities above ₹50,000 (opt-out allowed) [2][14][15]. Even an ex-RBI official calls the value benefit "minuscule" [10]. It delays **every** honest first-time ₹10k+ transfer (rent to a new landlord, fees, a new supplier), and critics note it misses splits below ₹10k and digital arrests that last longer than an hour [14]. *(Corrected 28 Oct: earlier docs said "every APP transfer ≥ ₹10k"; the lag targets first-time / non-whitelisted beneficiaries.)*
- Merchants reject 2.8–5.2% of orders on fraud suspicion, more than the 0.3–4% of orders that are actually fraud (MRC 2026) [11]. Classic Javelin study: false declines cost ~$118 bn vs ~$9 bn actual card fraud, and 39% of wrongly declined cardholders stop using that card [12] (old, 2015, so use it as directional only).
- RBI's 2025 Authentication Directions (in force **1 Apr 2026**) keep 2FA as the minimum but explicitly let issuers add checks **"based on the fraud risk perception of the underlying transaction"** and encourage new factors [13]. This is a regulatory opening for risk-based authentication.

## 4. Stakeholders

| Who | Pain | What they need from us |
|---|---|---|
| **Primary: issuers & PSPs.** Banks (especially co-op/small banks with thin fraud teams), UPI apps, payment gateways | Fraud losses, regulator pressure, customer churn from friction | A plug-in API: decision + reasons in milliseconds |
| **End users.** Everyday digital-payment users, especially seniors and first-time users | Scams they don't recognise; OTP fatigue | Stopped *only* when it matters, with a clear "why" |
| Fraud/ops analysts | Black-box scores they can't justify | Per-signal contributions, an audit trail |
| Regulator / LEA (RBI, I4C) | Need explainable, auditable decisions | Logged reasons, reason codes |

## 5. SDGs (slide 1 asks for 1 primary + up to 2 supporting)

- **Primary: SDG 16 Peace, Justice & Strong Institutions.** Target **16.4**: "significantly reduce illicit financial … flows, strengthen the recovery and return of stolen assets and combat all forms of organized crime." Scam rings and mule networks are organised crime, and stopping the payment *before* it leaves is the cheapest form of asset recovery.
- **Supporting: SDG 8 Decent Work & Economic Growth.** Target **8.10**: expand access to banking and financial services for all. Trust and low friction keep first-time, rural and senior users in digital finance.
- **Supporting: SDG 9 Industry, Innovation & Infrastructure.** Target **9.1**: reliable, resilient infrastructure. UPI is digital public infrastructure, and a fast risk layer keeps it trustworthy at 800 M txns/day.

## Sources
1. ThePrint: "Cybercrime saw 24% spike in 2025, Indians lost Rs 22,495 crore" https://theprint.in/india/cybercrime-saw-24-spike-in-2025-indians-lost-rs-22495-crore-mainly-in-investment-scams/2859930/
2. RBI Discussion Paper "Exploring safeguards in digital payments to curb frauds", 9 Apr 2026: https://www.rbi.org.in/SCRIPTs/PublicationsView.aspx?id=23810
3. Angel One / MHA Lok Sabha reply (Jul 2025): https://www.angelone.in/news/market-updates/cyber-fraud-costs-indians-22-845-crore-in-2024-government
4. IANS, MoS Finance Lok Sabha reply (15 Dec 2025): https://ianslive.in/upi-frauds-worth-rs-805-crore-witnessed-this-fiscal-so-far-minister--20251215175703
5. Business Standard on RBI Annual Report FY25: https://www.business-standard.com/amp/finance/news/bank-fraud-amount-triples-in-fy25-despite-drop-in-number-of-cases-rbi-125052900696_1.html
6. Asianet Newsable, MHA Rajya Sabha reply (Aug 2026): https://newsable.asianetnews.com/india/senior-citizens-women-lost-rs-7-769-crore-to-cyber-fraud-in-2025-articleshow-u9oad8w
7. StartupTalky, UPI Sept 2026: https://startuptalky.com/upi-transactions-september-2026/
8. MediaNama, NPCI response-time circular: https://www.medianama.com/2025/06/223-npci-upi-changes-transaction-speed-fraud-prevention/
9. Outlook Money, Karnataka HC SIM-swap: https://www.outlookmoney.com/news/banks-must-take-steps-to-prevent-sim-swap-fraud-says-karnataka-high-court
10. Daily Mirror (Reuters-style feature on RBI paper): https://www.dailymirror.lk/print/international/Indians-lost-25bn-to-digital-fraud-in-2025-now-its-central-bank-is-fighting-back/107-339105 (its "$25bn" headline figure doesn't match MHA data; don't use it)
11. Chargeback Gurus summary of MRC 2026 report: https://www.chargebackgurus.com/blog/fraud-statistics-from-mrc-report
12. Javelin via Digital Transactions (2015): https://www.digitaltransactions.net/lost-sales-from-unfounded-rejections-dwarfs-actual-fraud-losses-report-says/
13. RBI press release, Authentication Directions 2025 (25 Sep 2025): https://www.rbi.org.in/Scripts/BS_PressReleaseDisplay.aspx?prid=61282
14. Moneylife, "RBI's proposed digital payment safeguards: what do they mean for you?" (lag scope, exemptions, criticism): https://moneylife.in/article/rbis-proposed-digital-payment-safeguards-what-do-they-mean-for-you/80218.html
15. IASGyan, RBI proposed safeguards (first-time A2A transfers > ₹10k, trusted person 70+, kill switch, ₹25 lakh credit cap): https://www.iasgyan.in/daily-current-affairs/rbi-proposed-safeguards-for-digital-payment-frauds
16. ETV Bharat, Telangana couple under ~50-hour "digital arrest" on a WhatsApp video call: https://www.etvbharat.com/en/!state/under-50-hour-digital-arrest-telangana-couple-lose-rs-30-lakh-to-e-fraud-enn25090402492
17. ETV Bharat, I4C advisory (6 Oct 2024): https://www.etvbharat.com/en/!bharat/cbi-ed-police-dont-arrest-people-through-video-calls-indian-cyber-crime-coordination-centre-enn24100602996
18. Sumsub, India trading-scam probe identifies 20,507 mule accounts (Jul 2026): https://sumsub.com/media/news/india-trading-scam-probe-identifies-mule-accounts/
19. The Paypers, APP fraud and digital behaviour: https://thepaypers.com/fraud-and-fincrime/expert-views/authorised-push-payment-fraud-how-digital-behaviour-can-uncover-social-engineering-scams
20. tech.eu, Revolut AI scam feature (Feb 2024): https://tech.eu/2024/02/15/revolut-launches-ai-scam-features-to-break-scammers-spell/
21. Business Standard, UPI FY25 totals: https://www.business-standard.com/amp/finance/news/upi-value-hits-record-rs-24-77-trillion-in-march-marks-new-high-in-fy25-125040100744_1.html
22. Google Pay help, incorrect UPI PIN: https://support.google.com/pay/india/answer/7430541?hl=en
23. Business Today, RBI's digital-fraud compensation directions (26 Jun 2026; effective 1 Jan 2027): 85% of net loss capped at ₹25,000, only for losses ≤ ₹50,000, once per lifetime; RBI + the customer's bank (+ beneficiary bank for domestic) share the cost: https://www.businesstoday.in/personal-finance/news/story/rbis-new-digital-fraud-compensation-rules-from-january-2027-how-victims-can-recover-up-to-rs25000-539435-2026-06-26 (draft terms: Outlook India, https://www.outlookindia.com/national/rbis-digital-scam-compensation-scheme-customers-to-get-up-to-25000-relief)
