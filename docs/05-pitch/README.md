# Pitch & demo

## Screening deck (submitted 2026-10-15)
- **Final: `Pramaan_BIOS_Screening_v4.pptx`** (+ `.pdf`), built with `scripts/build_deck_v3.py`. The files below (`Pramaan_BIOS_Screening.pptx`, `_v2`, `_v3`) are earlier drafts kept for history.
- **Latest: `Pramaan_BIOS_Screening_v6.pptx`** (+ `.pdf`): v5 + review fixes (D-011). Impostor challenge = passkey on the **registered** phone (not a fingerprint on the new phone); family hold = goes only if the son approves (deck wording "no reply in 30 min cancels" is superseded by D-017: silence keeps it held), no reply in 30 min cancels; bot row = 15 PIN retries; "SIM changed yesterday" everywhere; "UPI's 15-second response limit"; targets labelled as tuned together. Built with `scripts/build_deck_v6.py <template.pptx> <out.pptx>`. **Resubmit this one by 19 Oct if the dashboard allows.**
- **Revised: `Pramaan_BIOS_Screening_v5.pptx`** (+ `.pdf`), built with `python scripts/build_deck_v5.py <template.pptx> <out.pptx>` (no icon files needed). Fixes from the v4 review: verified stats with sources, no made-up risk scores or latency, consistent worked example (71x usual ₹1,200), slide 2 ends with the template's "The gap we are addressing is:", a plain team diagram on slide 4 (template: simple beats polished), 48-hour plan (the hack runs 30 Oct to 1 Nov), fewer boxes and no icon badges.
- **`Pramaan_BIOS_Screening.pptx`**: the 5-slide deck in the official BIOS V2 template, with speaker notes (~45–60 s per slide) to read for the video. `.pdf` is a preview render.
- Rebuild: `python scripts/build_deck.py <BIOS_V2_CORE_FRONTIER_Screening_Template.pptx> docs/05-pitch/Pramaan_BIOS_Screening.pptx` (needs `python-pptx`). The template is kept locally, not in git.
- **Before submitting:** fill in the **Challenge ID** and **4 roll numbers** in the strip at the top of slide 1.

*Screening-round deck (historical; corrected claims for the finals are in the Q&A below and decisions D-012…D-019; deck v7 fixes them):*

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

## Demo script v3 (for the hack round; council review 1)
Full plan and reliability rules: `../04-plan/finals-playbook.md`. 3 minutes, one hero flow, all three outcomes (allow · challenge · block) and all three signal families (behaviour · device · transaction):
1. **Story first, 20 s, no badges yet:** "Asha, 72, gets a video call from a fake CBI officer. She types her own UPI PIN. Every check passes."
2. **Split screen (10 s):** the same ₹85k. OTP-only: "OTP verified ✔, ₹85,000 gone". Pramaan: "Held, here's why".
3. **Allow:** Asha's usual ₹1,200 → allowed instantly (fast path, server facts only).
4. **Challenge (hero):** Asha is on a call (SIMULATED toggle with badge; a live WhatsApp call only if the 29 Oct `adb shell dumpsys audio` test passed and the micro-app was built), pastes a new UPI ID, sends ₹85,000 → **held**. The hold screen plays our **recorded Hindi/English warning** ("CBI/Police/ED/Judges DO NOT arrest you on video call", I4C) and shows 3 spell-breaker questions + the top-3 reasons.
5. **The judge is Rohan:** a QR on the stand opens a **no-install web page** ("You are Rohan, Asha's son"; one-time token per hold in the URL) that shows Approve / Cancel / Call Asha (we don't promise a buzz: iOS Safari can't vibrate) → Cancel → Asha's screen shows "₹85,000 saved". The real product channel (Telegram) runs on a teammate's phone held up beside it. Badge: "DEMO MODE: 72 h contact enrolment waived". Fallback: the demo-only in-app "Rohan" panel.
6. **Device (15 s, SIMULATED badge):** the same ₹85k from a new phone with the SIM changed yesterday → "Approve on your registered phone" → the owner taps **Not me** → blocked. (This is the "device information" leg of the problem statement.)
7. **Block (10 s):** Asha tries to pay a UPI ID that's on the (mock) DoT FRI Very-High list → **Blocked**, with the reason. (Card testing stays in the replay and Q&A.)
8. **Close on one spoken line:** "Of 100 honest first-time ₹10k+ payments, RBI's proposed lag holds 100 and releases them all after an hour, even the coached ones; at our default we hold 10, keep every scam hold until someone breaks the spell, and caught X of Y held-out scam variants." Then one honest failure case. (Recall-vs-budget curve + replay chart are backup slides.)
- **Floor-walk:** "Play Asha" 60-second guided mode with REAL/SIMULATED badges and Reset, plus a 20-s investment-scam scene (a "profit" credit from a stranger, then a bigger payment → held by probation).
- **Fail-safe on purpose:** "kill API" switch → SDK times out → standard 2FA ("never weaker than today").
- **Backup:** a recorded 60-s run on a second device; the full backup video from H40.
- **Leave-behind:** a QR card + README with the live link, one architecture diagram, the REAL/SIMULATED table, the three measured numbers, and the commit timeline (shows it was built on-site). Judges compare ~20 teams afterwards.

## Likely judge questions (with answers)
- **How do you avoid blocking good users?** *(updated 28 Oct)* Only verified facts block (flagged payee, card testing, the owner's "Not me"); the model can only challenge; the bank sets a challenge budget per amount band; cancel is one tap; probation instead of instant whitelisting.
- **Your data is synthetic. Why trust the model?** *(updated 28 Oct)* No public dataset has these signals together (and real data is PII). We use scores only as a rank, report recall per attack type on **held-out variants the model never saw** (victim told to hang up, payee typed, splits, repeat investment payments, a spoofed "clean" client), show an ablation without the simulated call signal, and (if time) a real-data check on IEEE-CIS. A bank pilot would retrain on its labelled history.
- **How do you know the user is on a call?** On Android, a mobile SDK can read call/screen-share state (Google already pilots screen-share alerts in GPay/Paytm/Navi). In the web prototype it's a toggle, and missing signals count as "unknown", never "safe".
- **Why not just use OTP / RBI's 1-hour delay?** OTP doesn't stop a coached victim. RBI's proposed lag holds *every* first-time account-to-account transfer above ₹10k (whitelisted and merchant payees exempt), misses splits under ₹10k and repeat payments to a now-whitelisted payee, and one hour doesn't stop a days-long digital arrest. We hold only the risky payment, say why, and ask the right person.
- **Why these thresholds?** *(updated 28 Oct)* The bank sets a friction budget per amount band (0.2% under ₹2k … 10% for first-time ₹10k+), because our synthetic scores aren't real probabilities; we show recall at every budget and move the default to the knee of that curve.
- **How does it scale?** *(updated 28 Oct)* The API is stateless apart from the profile store, so it scales horizontally; a small GBDT takes a few ms per payment (we quote only what we time on Render). The prototype runs on Render's free tier with profiles cached in memory and backed by Supabase; production would use a managed cache (e.g. Redis) and autoscaling. UPI averages ~9,300 txns/s.
- **Why a passkey and not a fingerprint for the impostor?** *(updated 28 Oct)* The fingerprint check runs on whatever phone it's on, so a thief unlocks their own phone with their own finger. We ask for approval on the **registered** phone, bound to this payment. Honest limit: Android passkeys usually sync with the Google account, so within 72 h of a SIM change or new device a passkey alone isn't enough and the bank calls back.
- **RBI says account takeover is negligible, but BioCatch says 55%?** Different data: RBI counts bank-reported fraud, BioCatch measures sessions at its partner banks. We handle both impostors and scams, so the pitch doesn't depend on either number.
- **What happens if your service is down?** *(updated 28 Oct)* The app's SDK times out and falls back to the bank's standard PIN/OTP, and for first-time ₹10k+ transfers the bank applies its RBI-style hold meanwhile: we degrade to the regulator's rule, never to nothing. We show this live with a "kill API" switch.
- **Privacy?** Derived features only, hashed device IDs, retention window, audit log; designed for the DPDP Rules, whose notice/consent duties apply from ~14 May 2027 (consent managers from Nov 2026).
- **Why not an LLM?** ~1 s and non-deterministic. A small GBDT with built-in contributions is ms-fast and reproducible. LLMs could later draft analyst case notes off the hot path.

## New Q&A after council review 1 (owner = who answers; confirm owners in the team chat)
| Question | Answer | Owner |
|---|---|---|
| Your scores come from fake data? | We use them as a **rank**, not a probability (synthetic fraud is ~3,000× real UPI's ~7 per million). The bank sets a challenge budget per amount band; we report recall at that budget on held-out attack variants, with the call signal removed too. | Abhijay |
| What exactly does the user approve? | This payment: a one-time server nonce + amount + payee + txn id are hashed into the passkey challenge and shown on the registered phone; "Not me" blocks it. | Gunagye |
| The scammer makes her change the trusted contact? | A new contact is active only after 72 h, changes are frozen 72 h and the old contact is told; hard-flagged payees can't be released by anyone. | Shefali |
| The scammer says "just wait 30 minutes" or "hang up and release it"? | No hold releases on a timer, and the owner can never release a scam hold: only the trusted contact or the bank can. | Shefali |
| Digital arrests run on WhatsApp, not phone calls. | *If the 29 Oct adb test passed:* "During a WhatsApp call the calling app switches Android's audio mode to 'in communication', which other apps can read; we tested it." *If not:* "A bank's mobile SDK would read call and screen-share state, as Google already does for GPay/Paytm; in our web demo it's a labelled toggle." Never claim to know who's calling. | Abhijay |
| Doesn't Google / RBI already do this? | Google gives an app-level warning + a 30-s pause when a finance app is opened during screen-share with an unsaved caller: no amount, payee or history, no per-payment decision. RBI's proposed lag holds every first-time A2A transfer above ₹10k. We decide per payment, with reasons and family. | Shefali |
| Who's liable for a wrong hold? | We hold, never decline on a guess; cancel is one tap; every release is logged for the bank's grievance process. And prevention matters more than refunds: RBI's compensation framework (notified Jun 2026, effective 1 Jan 2027; whether it covers scams the victim authorised herself is unclear) pays, even where it applies, at most 85% / ₹25,000, once per lifetime, only for losses up to ₹50,000. **Asha's ₹85,000 is outside that cap; only prevention protects her.** | Bhavyam |
| Privacy / DPDP? | Derived features only (no keystrokes, audio or call content; "on a call" is yes/no), hashed device ids, retention window, audit log. | Gunagye |
| What does the friction cost honest users? | The bank sets it: at our default, 10 of 100 honest first-time ₹10k+ payments are held, vs 100 under RBI's proposed lag; here's the recall at each setting. | Abhijay |
| Can a scammer probe your API, or fake a "clean" phone? | Customers get decision + top-3 reasons only; contributions need an analyst key; calls are rate-limited. Client fields can only raise risk: our risk rank is the max of a server-only model and the full model, tested with a spoofed client. | Gunagye |
| How does a bank plug this in, and what does it cost? | The bank's app adds our small SDK and calls `/v1/authenticate` before the PIN step; one API call per payment; the server is stateless apart from the profile store, so it scales out horizontally (UPI averages ~9,300 txns/s). Our ask: a pilot with one co-operative or small bank on its first-time ₹10k+ transfers. We'll quote p95 and cost per call only from our measured run. | Bhavyam |
| Where does the hold sit in the UPI flow? | Before the PIN. Pramaan is called when Asha taps Pay; on a hold the app doesn't open the UPI PIN screen. When Rohan approves, the app unlocks the PIN step and Asha completes the payment herself: money only moves when the owner enters her PIN after release. | Gunagye |
| What if she just switches to another UPI app? | Two integration points. In her bank's own app we hold *before* the PIN. Through another UPI app, her bank only sees the payment after the PIN (via NPCI), so it **declines with the reason** and alerts her and Rohan; once Rohan approves, that payee + amount is pre-approved for a short window and she retries. | Shefali |
| Your promo video says "fingerprint". | Made before our review: a fingerprint on the thief's phone only proves the thief's finger, so we moved to a passkey on the registered phone and approve-only release. | Bhavyam |

*Owner load: Bhavyam 3 · Abhijay 3 · Gunagye 3 · Shefali 3 (plus the older rows above, split the same way).*
