# Council review 1 (28 Oct 2026)

> The whole project was put through our LLM council (`.claude/skills/llm-council/`: 4 Claude members with different models and lenses → anonymous peer ranking → chairman), the weak spots were researched on the web, fixed, and re-reviewed four times until the scores stopped rising and every member said the remaining risk is execution, not the plan. The fixes live in the design docs; this file is the record. **Everything in D-012…D-019 is a proposal until the team accepts it.**

## How it ran
| Round | What the council did | Result |
|---|---|---|
| 1 | Found weak spots in the whole project (all repo docs + deck v6) | 12 weak spots (4 critical) |
| — | Web research on each weak spot (RBI paper, UPI rules, Android APIs, Google's protection, WebAuthn, hosting limits, scam patterns) | fact-checks below |
| 2 | Graded solutions v1 | 4–6.5 / 10: right direction, ~2× too much scope, a few contradictions and over-claims |
| 3 | Graded solutions v3 (scope tiers, one trust rule set, exact maths) + chairman verdict on v4 | 5 / 7 / 7 / 7; chairman **7.5 / 10** after the v4 fixes |
| 4 | Final check of the updated repo docs themselves (consistency + facts) | **7.5 / 7 / 7 / 5**; all four: "the remaining risk is execution, not the plan"; their last fixes (contact-decision auth, thresholds on the max score, two UPI integration points, device scene, leftover v1 lines) are applied |

Round 1 leaderboard (anonymous peer ranking, lower = better): First-principles (opus) 2.0 · Judge view (sonnet) 2.5 · Pragmatist (sonnet) 2.5 · Skeptic (haiku) 3.0.

## The 12 weak spots (round 1, chairman's synthesis) → where each is fixed
| # | Weak spot | Severity | Fixed in |
|---|---|---|---|
| 1 | On-site statement and pre-written-code rules unknown; all work tied to PS-04 | critical | `04-plan/finals-playbook.md` §0–1 (ask organisers, no code until allowed, H0 gate, decision kernel) |
| 2 | Nothing built; a serial plan with no slack | critical | playbook §2–3 (MoSCoW tiers, mock-API first, rules by H8, hero flow by H20) |
| 3 | ₹0 hosting chain can fail during judging (Render 0.1 CPU sleeps, `shap` heavy, n8n homeless, Supabase pauses, fail-safe inside the sleeping server) | critical | playbook §4; D-014, D-016, D-017 |
| 4 | Scam catch rests on a toggle and is graded by our own generator | critical | `03-design/risk-engine.md` §1, §8 (REAL/SIMULATED badges, ablation, unseen attacks, recall-vs-budget) |
| 5 | Pitch misread RBI ("every ₹10k+ payment") and ignored Google's in-call protection | high | `02-research/problem-evidence.md`, `prior-art.md`; D-019 |
| 6 | Threshold maths didn't add up (k vs friction, block outside the cost model, fake base rate, unsourced efficacy) | high | risk-engine §3–5; D-012, D-013, D-014 |
| 7 | API had no trust boundary (forgeable client JSON, no auth, probing via `detail`, feedback could whitelist a mule) | high | `03-design/threat-model.md` 1–3; `api-contract.md` v2 |
| 8 | Impostor passkey path didn't hold (can't force device-bound, not bound to the payment, SIM-swap re-registration hole) | high | threat-model 6–8; D-017 |
| 9 | Blind to investment scams (76% of losses); labels poison themselves | high | risk-engine §1, §6; D-018 |
| 10 | Shallow Q&A depth; outdated claims (fingerprint, "15 PIN retries", ATO-negligible vs 2 of 4 scenes) | high | `05-pitch/README.md` Q&A owners; D-019 |
| 11 | No memorable demo moment | high | `05-pitch/README.md` demo script v3 |
| 12 | Trusted-contact loop could be gamed (unverified enrolment, coach-to-wait, outage = blocked payments) | medium | threat-model 4–5, 12; D-017 |

**Where the council disagreed:** build risk vs technical flaws first (chair: execution first, but most technical fixes are doc/slide edits, so do them now); how deep to take the maths fixes (chair: cheap versions only); "omitted fields pass" (rejected: the docs already treat missing as unknown; the real attack is forged values).

## Fact-checks from web research (all sourced in the docs above)
- RBI's Apr 2026 lag targets every **first-time** account-to-account transfer above ₹10k (merchant/whitelisted exempt), not every payment.
- UPI blocks payments for 24 h after 3 wrong PINs; emulators can't pass UPI SIM binding.
- Real UPI fraud ≈ 7 per million transactions (FY25), ~3,000× rarer than our synthetic data.
- Digital arrests run on WhatsApp/Skype **video** calls for hours or days; I4C: "CBI/Police/Custom/ED/Judges DO NOT arrest you on video call".
- Investment scams: small deposit → fake "profit" credited back → escalating payments → "tax to withdraw"; one case used ~20,500 mule accounts.
- Google's in-call protection (Android 11+, piloted in India with GPay/Paytm/Navi) warns + 30-s pause when a finance app is opened while screen-sharing on a call with an unsaved number.
- Android: VoIP calls show in the audio mode (to verify with `adb shell dumpsys audio`); Android 15 screen-recording callback; WebView has no `speechSynthesis`; WebView passkeys need extra setup.
- WebAuthn BE/BS flags tell synced from device-bound passkeys (readable, not enforceable).
- Render free = 0.1 CPU / 512 MB, sleeps after 15 min; Hugging Face Docker Spaces need a paid plan; Supabase free pauses when idle; Cloudflare quick tunnels need no account.
- Revolut's scam-intervention flow cut investment-scam card losses ~30%; US FINRA Rule 2165 allows holds + trusted-contact notice for 65+ clients.
- Throwaway benchmark (1 thread): 5-class LightGBM, 100 rounds / depth 5 ≈ 2 ms p50 for probabilities + `pred_contrib`; 200 / depth 6 ≈ 22 ms.

## Final verdict
**Win-readiness ≈ 7.5 / 10 on paper** (rounds 3–4). Every member says no doc change will push it much higher: the rest depends on the build.

**What makes it a winner if executed**
- One hero flow, rehearsed three times. A judge's own phone cancels a held ₹85k. Allow, device and block are all in the same 3 minutes.
- Every claim is measured, labelled SIMULATED, or labelled a policy choice. The facts a judge can check are right: RBI's lag targets *first-time* A2A transfers, UPI blocks after 3 wrong PINs, and real fraud is about 7 per million.
- The close: scam recall with the call signal removed, on attack variants written blind, plus one honest failure case.

**What can still lose it**
1. The on-site statement differs, or pre-written code isn't allowed. Fix: ask the organisers on 29 Oct, and use the H0 gate (30 min) to map the statement onto the kernel.
2. The H8 skeleton or the H20 hero flow slips. Fix: hold the H24 and H30 checkpoints, follow the cut order, and keep one integrator.
3. A teammate can't defend a component live. Fix: Q&A owners (3 each), rehearsals at H36, H44 and H47, and flashcards.

**Skeptic's dissent (5/10), kept on record:** "no code, an unknown statement, and a 20-hour chain across Render, Supabase, Vercel and Telegram". We agree it is the main risk. The answer is the H8 skeleton and the laptop fallback, not more design.
