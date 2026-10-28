# PLAN.md — Pramaan, the whole plan in one place

> **Read this first.** This is the single source of truth for *what* we are building and *why*, written so any teammate (or any teammate's Claude) understands it in 5 minutes. Details live in `docs/` (links at the end); if a detail doc disagrees with this file, this file wins and the detail doc is out of date.
> How we execute it, step by step: `docs/04-plan/roadmap.md`. Last rewritten: 28 Oct 2026 (after council review 1).

---

## 1. The problem in one paragraph
**BIOS Hackathon finals, Frontier mode, PS-04: "Fast and Secure Online Transaction Authentication."** Build a cloud service that uses **behaviour, device information and transaction patterns** to **allow, challenge or block** an online payment in real time, and **explain the signals behind the decision**.

## 2. The insight everything rests on
Fraud has moved past what OTP/PIN can stop.
- RBI (Apr 2026): account-takeover fraud is "now negligible". Most money is lost to **scams where the real user pays the scammer herself**, on her own phone, with her own PIN, while being coached (digital arrest, fake investment).
- So the right question isn't only *"is this the right person?"* but **"is this payment normal for this person, right now, and is someone pushing her?"**
- India lost ~₹22,500 crore to cyber fraud in 2025; ~98.5% of fraud value is in payments above ₹10,000.
- Refunds won't save victims: RBI's compensation framework (from 1 Jan 2027) pays at most ₹25,000, only for losses up to ₹50,000. **Only prevention protects a victim who loses ₹85,000.**

## 3. The solution: Pramaan (प्रमाण, "proof")
A risk service a bank calls **when the user taps Pay, before the UPI PIN screen**. In milliseconds it returns **allow / challenge / block + the top 3 reasons in plain language**. It sits on top of mandated 2FA and never replaces it.

### The one principle
> **Money is untrusted until it's proven safe, and the challenge must fit the danger.**

Everything else is an example of this principle.

### The three ideas judges should remember
1. **Compared with *her own* normal**, not a global average. ₹85,000 to a new UPI ID is ordinary for one person and 71× normal for Asha.
2. **The challenge fits the risk** (our core contribution):

| What we see | Likely risk | What Pramaan does |
|---|---|---|
| Usual payee, own phone, usual amount | none | **Allow** instantly |
| New phone, SIM changed yesterday | **Impostor** | Approve on the owner's **registered phone** (bound to this exact payment) or tap "Not me" → block |
| Own phone, at home, on a call, pasted a new UPI ID, huge amount | **Coached victim** | **Hold** + scam warning; only her **trusted contact** (or the bank) can release it |
| Payee on a fraud list, or bot-like velocity | **Fraud for sure** | **Block** |

An OTP would pass the coached victim. A blanket delay annoys everyone. We hold only the risky payment and ask the right person.
3. **The decision explains itself**: "You're on a call while sending money · Amount is 71× your usual ₹1,200 · First payment to this receiver."

### Positioning (one line)
**Google warns. RBI's proposed rule holds every first-time ₹10k+ transfer. Pramaan holds only the risky payment, says why, and asks the right person.**

---

## 4. How it works (the engine, plain version)
1. **Features against her own history**: amount vs her usual, new vs trusted payee, device bound by her bank, time, velocity, paste, hesitation, app switches, call state.
2. **Hard rules first.** Only verified facts can **block** (flagged payee, card-testing velocity, owner taps "Not me"). Clearly normal payments are **fast-path allowed** using server facts only.
3. **Risk score** from a transparent rules scorecard (built first), later a small LightGBM model that also says **which kind** of risk (impostor / scam / bot / mule). The model can only *challenge*, never block.
4. **How much friction is allowed** is a bank setting per amount band, e.g. hold at most 10% of honest first-time ₹10k+ payments (RBI's proposal holds 100%).
5. **Which challenge** comes from the table in §3.
6. **Reasons** come from the model's built-in per-feature contributions, filled into templates with her own numbers.

Safety rules that keep it honest:
- **Phone-reported signals can only raise risk, never lower it** (a scammer can fake a "clean" phone).
- **No hold ever releases on a timer.** A scam hold is released only by the trusted contact or the bank; the victim can always cancel but can't release it herself ("hang up and release it" is exactly what scammers say).
- **New payees are "on probation" for 30 days**, so repeat investment-scam payments don't become "normal".
- **If Pramaan is down**, the bank falls back to normal 2FA, plus its RBI-style hold for first-time ₹10k+ transfers: never weaker than today.

## 5. Where it plugs into UPI (be precise)
1. **Inside the bank's own app (strongest, our demo):** checked **before the PIN**; a hold means the PIN screen doesn't open until release.
2. **Payments from GPay/PhonePe:** the bank only sees them **after the PIN** (via NPCI), so it **declines with the reason**, alerts the contact, and pre-approves the retry once the contact says yes.
- **vs RBI's DPIP** (national fraud-intelligence platform): DPIP shares intelligence *between* banks; Pramaan is the decision *at the moment of payment*. DPIP's flags would be one more input to us.
- **First customer:** a co-operative or small bank with no fraud team, piloting first-time ₹10k+ transfers in its own app.

## 6. When there's no trusted contact
Fallback chain: **contact → second contact → bank call-back desk.**
- Only scam-type holds above ₹10k reach a human at the bank; we estimate that volume from the replay ("N calls per 1 lakh payments") against the loss avoided per call. That's the business case, not a hidden cost.
- Contact is offered at onboarding for 60+ users (RBI's own proposal suggests a trusted person for 70+); a family member can set it up from their own phone. A contact becomes active after 72 h, and changes are frozen for 72 h (stops a scammer becoming the "son").
- Optional **safe word**: a phrase she agreed with her bank earlier; the hold screen reminds her that real officials never ask for it, and the bank call-back uses it.

## 7. What's REAL vs SIMULATED (we say this out loud)
| Signal | In our demo |
|---|---|
| Pasted UPI ID, hesitation on confirm, app switching, time to pay, amount, payee history, velocity, IP | **REAL** |
| On a phone/WhatsApp call | **REAL if** the 29 Oct `adb shell dumpsys audio` test passes (tiny Android helper app), else a labelled **SIMULATED** toggle |
| Screen-sharing, SIM change, new device | **SIMULATED** (bank/mobile-SDK data). Proof it's obtainable: Google already reads call + screen-share state for GPay/Paytm |

**The hero decision must hold even with the call signal switched off** (paste + hesitation + new payee + 71× amount). We show that on stage, then switch it on to show the reasons get stronger.

## 8. How we prove it works (evidence, not promises)
1. **Synthetic data, honestly framed:** no public dataset has these signals together. Scores are used as a *ranking*, not a probability (real UPI fraud is ~7 per million).
2. **Blind test:** a teammate who doesn't build the model writes scam variants it never saw (victim told to hang up, payee typed, ₹9,999 splits, repeat investment payments, a faked "clean" phone). We report "caught X of Y" and one that got through.
3. **One real dataset:** the transaction + device part on **IEEE-CIS** (real card-fraud data), one number.
4. **Hall user test:** 10–15 people play "Asha" while a teammate coaches them on a call; count how many cancel at the warning screen.
5. **Closest outside evidence:** Revolut's warning-and-chat flow cut investment-scam losses ~30%; US FINRA Rule 2165 already allows holds + a trusted contact for 65+ clients.

## 9. The demo (3 minutes, one story)
On stage, **only three things**: Asha's story → "the challenge fits the risk" → **the judge cancels the scam on their own phone.**
1. **Story (20 s):** "Asha, 72, is on a video call with a fake CBI officer. She types her own PIN. Every check passes."
2. **Split screen (10 s):** same ₹85,000: OTP-only "verified ✔, money gone" vs Pramaan "held, here's why".
3. **Allow:** her usual ₹1,200 goes through instantly.
4. **Hold (hero):** she pastes a new UPI ID and sends ₹85,000 → held, with a **recorded Hindi/English warning** ("CBI, police, ED or judges never arrest anyone on a video call") and the top 3 reasons. Shown first with the call signal off, then on.
5. **The judge is Rohan:** a QR opens a no-install page on the judge's phone → Approve / Cancel → they press Cancel → "₹85,000 saved".
6. **Device (15 s):** same payment from a new phone + new SIM → "Approve on your registered phone" → "Not me" → blocked.
7. **Block (10 s):** payee on the fraud list → blocked with the reason.
8. **Close (one line):** "Of 100 honest first-time ₹10k+ payments, RBI's rule holds all 100 and releases them after an hour, even the coached ones. We hold 10, keep every scam hold until someone breaks the spell, and caught X of Y scams we'd never seen." Then one honest failure case.

Everything deeper (threat model, probation, cross-victim flags, integration) goes on a **one-page depth card** handed to judges who dig in.

## 10. Scope for the 48 hours
- **MUST (by H20):** demo payment app + reset · rules scorecard + API on Render · hold screen + recorded warning · Rohan page (judge QR) + Telegram for the real channel · reasons panel · split screen · device scene · fraud-list block · mini data generator + replay · keep-alive + laptop fallback.
- **SHOULD (H20–H36, in order):** blind-test results + call-off ablation → LightGBM → probation + investment-scam scene → real passkey on the registered phone.
- **COULD:** real call detection app (only if the adb test passed) · IEEE-CIS number · hall user test · cross-victim payee flags.
- **TALK-ONLY:** SDK signing, bank call-back desk, DPIP/FRI feeds.

## 11. What can still lose it (and the plan for each)
| Risk | Plan |
|---|---|
| The on-site problem statement is different | Ask the organisers 29 Oct; 30-min gate at H0 maps any statement onto the same pattern: score → typed intervention → human approval → reasons (`docs/04-plan/finals-playbook.md` §1) |
| Pre-written code isn't allowed | Write no project code until they say yes; prepare everything else |
| The build slips | Skeleton by H8, hero flow by H20, checkpoints at H24/H30, fixed cut order, one integrator |
| A teammate can't answer a judge | Every Q&A row has an owner; rehearsals at H36, H44, H47 |
| Free hosting sleeps / Wi-Fi dies | Keep-alive pings, laptop + hotspot fallback, recorded backup video |

---

## Where the details are
| Need | Read |
|---|---|
| Step-by-step execution order | `docs/04-plan/roadmap.md` |
| Hour plan, owners, demo-day reliability | `docs/04-plan/finals-playbook.md` |
| Live status | `STATUS.md` |
| Engine details (features, thresholds, maths) | `docs/03-design/risk-engine.md` |
| Attacks and defences | `docs/03-design/threat-model.md` |
| API | `docs/03-design/api-contract.md` |
| Why we chose what we chose | `docs/03-design/decisions.md` (D-012…D-019 are proposals until the team accepts them) |
| Evidence and sources | `docs/02-research/problem-evidence.md`, `prior-art.md` |
| Demo script + judge Q&A | `docs/05-pitch/README.md` |
| How we got here (council review) | `docs/06-review/council-review-1.md` |
