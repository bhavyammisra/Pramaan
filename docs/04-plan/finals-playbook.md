# Finals playbook (30 Oct → 1 Nov, 48 h)

> **Big picture: `../../PLAN.md`. Order of work: `roadmap.md`.** This file holds the detail (organiser questions, H0 gate table, tiers, milestones, reliability). From council review 1 (28 Oct; full review in `../06-review/council-review-1.md`). Principle: **one hero flow built perfectly; every claim is measured, labelled SIMULATED, or labelled a policy choice.** Read with `STATUS.md`.

## 0. Before the hack (29 Oct): rules first
**Ask the organisers today** (Discord / email / coordinator):
1. Does "new problem statement on-site" mean a twist on PS-04 or a new problem?
2. Can we bring code written before 30 Oct? If yes, must we disclose it?
3. What's the judging rubric, the demo slot length, and will judges walk the floor mid-hack?
4. Can we use our own phones + a mobile hotspot? Is venue Wi-Fi reliable?
5. Are AI coding assistants allowed (and must we disclose them)?

**Default if no answer: no pre-written project code** (typical rules, e.g. MLH: no work on the project before the event, no reuse of earlier project code). Prepare only non-code:
- [ ] Accounts: Render, Vercel, Supabase (touch it daily; free projects pause when idle), Cloudflare, cron-job.org. Telegram bot via @BotFather (token in a password manager, never in git).
- [ ] **Real-call test (no code):** enable USB debugging on a team Android phone, start a WhatsApp call, run `adb shell dumpsys audio` and check whether the audio mode shows "in communication". Yes → the Android micro-app gets its COULD slot. No → drop it and never claim it.
- [ ] **Passkey from Telegram:** open a webauthn.io link from inside Telegram on each phone; if Telegram's in-app browser can't do passkeys, the approval flow tells users to open the Pramaan page in Chrome (the registered phone's page polls for pending approvals anyway).
- [ ] **Record the hold warning** (a teammate's voice, Hindi + English, ~10 s): "CBI, police, ED or judges never arrest anyone on a video call. We've paused this payment and asked Rohan to confirm."
- [ ] **Deck v7** from deck v6 with the corrected claims (RBI "first-time A2A", passkey not fingerprint, card-testing bot, budgets as policy) + one "what we built" line; README skeleton (what's REAL vs SIMULATED); Q&A flashcards.
- [ ] **Generator spec** (a doc, Abhijay): ≈300 users × 60 days (30 warm-up + 30 eval), honest look-alikes, the 4 attack types.
- [ ] **Held-out attack variants written blind** by someone who won't build the model (suggested: Shefali): victim told to hang up first, payee typed not pasted, ₹9,999 splits, repeat investment payments after a bait profit, a spoofed "clean" client. This is our answer to "you graded your own generator".
- [ ] Phone checks: on each Android phone create a passkey on webauthn.io (synced to Google Password Manager or not?); Android version; in Chrome, does `speechSynthesis.getVoices()` list a hi-IN voice?; test the Telegram `request_contact` button in any chat; who has Android Studio working.
- [ ] Build cards (one per component): libraries, file layout, test command, so typing starts at H0.
- [ ] Paper re-skin drill (30 min, §1). Q&A owners confirmed (`../05-pitch/README.md`).

If Q2 = yes: pre-build only the decision kernel and list it in the README as pre-existing work.

## 1. H0 gate (30 min, whole team): map the statement, commit to a hero flow
Kernel: **events → features vs the user's own baseline → risk rank + type → intervention typed to the risk → human approval → reasons + audit → replay vs a blanket rule.**

| If the statement is… | Hero flow | Public data |
|---|---|---|
| PS-04 or a twist | Pramaan scam hold (below) | synthetic |
| Mule / money laundering | Mule receives from many victims → hold the credit → bank reviewer decides | PaySim |
| Account opening / KYC / loan app | Device farm signs up → video-KYC step instead of a reject | Feedzai BAF |
| Scam link / SMS / call | Lookalike link → warn + family alert, with reasons | phishing URL lists |
| Insurance / expense claims | Duplicate/inflated claim → document request → investigator approves | synthetic |
| Non-fintech (health, campus, content) | Score → typed action → a human approves → reasons | — |

## 2. Scope (MoSCoW). Owners are suggestions: confirm in the team chat
| Tier | What | Suggested owner |
|---|---|---|
| **MUST (hero flow, by H20)** | **H8 skeleton first** (stubs are fine): app → `/bank/*` → rules → hold saved in Supabase → Rohan page → PIN step unlocked. Then fill in: | |
| | `/v1/authenticate` + `/bank/*` route on Render, API key, simple per-user/device rate limit · rules scorecard with **hard-coded thresholds** for the hero flow (no dependency on generator data) · **mock-FRI Very-High payee → block** (the demo's block beat) · `ENGINE_DEADLINE_MS` env var (~2 s in the demo) | Bhavyam |
| | **Mini generator + replay harness**: ≈300 users × **60 days** (30 warm-up for baselines/probation + 30 eval), 4 attack types + honest look-alikes; card testing lives here | Abhijay |
| | Demo UPI app + **Reset demo** (seeds Asha) · split screen (same ₹85k, OTP-only vs Pramaan) · "kill API" switch | Gunagye |
| | Hold + spell-breaker screen with the recorded warning · **one "Rohan" component** used three ways (judge QR web page with a one-time token per hold, in-app fallback panel, link inside the Telegram message) · Telegram bot (long polling) | Shefali (also **integrator**) |
| | Reasons panel + "bank call-back (simulated)" button on the dashboard · keep-alive + laptop fallback runbook | Gunagye (dashboard) · Shefali (runbook) |
| | **Device scene (15 s, SIMULATED):** new phone + SIM changed yesterday → "Approve on your registered phone" page (same component as Rohan's) with Approve / **Not me** → Not me blocks. Real WebAuthn stays SHOULD (4) | Gunagye |
| | Demo script + presenting (the presenter is not the integrator and doesn't deploy in the last hour) | Abhijay (suggested presenter) |
| **SHOULD (H20–H36), in this order** | (1) recall-vs-budget curve + `on_call` ablation on held-out variants → (2) multiclass LightGBM + server-only model (risk = max) → (3) probation features + a 20-s investment-scam scene (a "profit" credit from a stranger, then a bigger payment → held) → (4) impostor path: transaction-bound passkey on the registered phone + "Not me" | (1) Abhijay · (2) Bhavyam · (3) Abhijay · (4) Gunagye |
| **COULD (only when MUST + SHOULD are green)** | Hall user test: 10–15 people play "Asha" while a teammate coaches them on a call; count cancels at the warning screen · One non-synthetic check: transaction + device sub-model AUC on IEEE-CIS (real data; download it on 29 Oct; 1-hour box) · live spoken warning via Chrome speech · bait-profit · collective-immunity counter · Android micro-app (native, no WebView) posting only the call audio mode — **only if the 29 Oct `adb shell dumpsys audio` test showed the mode change** | whoever is free |
| **TALK-ONLY** | SDK signing + Play Integrity · bank call-back desk · second independent generator · DPIP/FRI feeds · appeals workflow | — |

**Rules:** never start a COULD while a MUST/SHOULD is red. The Android micro-app starts only if the hero flow is green by H24, with a 4-hour box.

## 3. Milestones and checkpoints
| Hour | Must be true |
|---|---|
| H2 | Mock `/v1/authenticate` from the API contract live; repo skeleton; generator spec agreed; all tracks unblocked |
| H8 | **Skeleton end to end** (app → `/bank` → rules → Supabase hold → Rohan page → PIN unlock); generator producing data; **dummy model timed on Render (predict and `pred_contrib` separately) + cold start** |
| H20 | **Hero flow deployed** on the `demo` branch: app → API → hold → Rohan Approve/Cancel (judge QR web page + Telegram) → PIN unlocked / cancelled; FRI-payee block works |
| H24 | **Checkpoint:** every MUST green, or stop SHOULD work and fix it |
| H30 | **Checkpoint:** cut any SHOULD item still red; LightGBM behind the same API if green |
| H36 | Measured numbers in · laptop-fallback switch rehearsed · **rehearsal 1** |
| H40 | Backup video of the full demo; 60-s recorded run on a second device |
| H42 | **Code freeze**, fixes only |
| H44 / H47 | **Rehearsals 2 and 3** |

**Cut in this order if late:** load test → threshold sliders → mule depth → realtime feed (poll) → interactive replay (static chart) → impostor passkey (show the design slide) → Android micro-app (toggles labelled SIMULATED).

**Leave-behind (H40):** QR card + README with the live link, architecture diagram, REAL/SIMULATED table, the three numbers, commit timeline.

**Closing numbers (measured on held-out variants):** SCAM recall at the 10% budget **with `on_call` removed**, the model's lift over rules, and honest payments held %. Plus one honest failure case.

## 4. Demo-day reliability
- A `demo` branch deploys to Render + Vercel; `main` moves freely, so a bad merge never goes live. **On Vercel, make `demo` the production branch**: preview hostnames (`-git-demo-…`) change, and passkeys are tied to the hostname.
- Render runtime: FastAPI + LightGBM Booster (+ numpy) only; no pandas/shap/sklearn in the runtime image (512 MB RAM); the model is trained offline and shipped as `model.txt`. Time build + cold start at H8.
- The app's API base URL is runtime-configurable (settings / `?api=`).
- Passkey RP ID = the **full production hostname** of the frontend (e.g. `pramaan-demo.vercel.app`; `vercel.app` itself is a public suffix and can't be an RP ID), so the API can move to a laptop without breaking passkeys.
- Telegram buttons can only open http(s)/tg:// links, so "Call Asha" is her number in the message text (Telegram makes it tappable), not a `tel:` button.
- Telegram uses long polling everywhere (no webhook URL to change). **Only one poller per bot token** (a second gets 409 Conflict): an env flag `BOT_POLLER=on` on exactly one instance. Laptop switch runbook: set it off on Render → on on the laptop → change the app's API URL → test one hold. Rehearse the switch at H36.
- Holds, approvals and contacts live in Supabase, not memory, so a restart or a switch loses nothing.
- Keep-alive: cron-job.org pings `/v1/health` every 10 min from H0 (Render free: 0.1 CPU, 512 MB, sleeps after 15 min idle, ~1 min to wake).
- Fallback: laptop + phone hotspot + Cloudflare quick tunnel (`cloudflared tunnel --url http://localhost:8000`; no account, random URL, no SSE).
- The demo SDK owns the timeout (3 s network timeout in the demo; the server's internal deadline stays 150 ms); the "kill API" switch shows the standard-2FA fall-back on purpose.
- Backup video by H40.

## 5. Team ops
- Sleep: **one person asleep at a time**, 4–5 h blocks between H8 and H40, so at least 3 are awake at H20 and H30.
- One integrator merges to `main` and promotes to `demo`; everyone else uses small PRs.
- The presenter doesn't deploy in the last hour.
- Each member owns the Q&A rows with their name (`../05-pitch/README.md`).
