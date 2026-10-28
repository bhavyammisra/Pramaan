# Roadmap: how we execute, in order

> The *what* and *why* is in `../../PLAN.md`. This file is the *when* and *who*. **Roles are not decided yet**: owner cells say TBD until the team assigns them. Tick boxes as you go and update `STATUS.md`.

## Phase 0 — 29 Oct (day before): decide, prepare, no project code
**Morning: decisions (whole team, 1 h)**
- [ ] Send the organisers 5 questions: what "new statement on-site" means · is pre-written code allowed · judging rubric + demo length · own phones/hotspot OK · AI coding tools allowed.
- [ ] Team: read `PLAN.md`; accept or change D-012…D-019 (`docs/03-design/decisions.md`); **decide roles** (who builds what, who integrates, who presents).

**Day: checks and prep (in parallel)**
| Who | Task | Done when |
|---|---|---|
| TBD | Real-call test: WhatsApp call + `adb shell dumpsys audio` → does the mode change to "in communication"? | Yes/No written in STATUS (decides whether call detection becomes REAL) |
| TBD | Passkey checks: webauthn.io on each phone, and opened from inside Telegram | Notes in STATUS |
| TBD | Accounts: Render, Vercel, Supabase, Cloudflare, cron-job.org; Telegram bot via BotFather (token in a password manager) | All logins work |
| TBD | Write the **blind attack variants** (someone who won't build the model) | A list in a private note, not in git until H36 |
| TBD | Generator spec (≈300 users × 60 days, honest look-alikes, 4 attack types) | Doc ready |
| TBD | Deck v7 with corrected claims + the one-page **depth card** + README skeleton (REAL/SIMULATED table) | Files ready |
| TBD | Rules-scorecard spec on paper (features + points + hard-coded thresholds for the hero flow) | Doc ready |
| TBD | Record the warning mp3 (Hindi + English, ~10 s) · download IEEE-CIS · write Q&A flashcards | Files ready |

**Evening (team, 30 min):** paper drill: if the on-site statement were "mule detection" or "KYC fraud", what is our hero flow? (`finals-playbook.md` §1)

## Phase 1 — H0 to H2 (30 Oct, start): lock the target
- [ ] **H0 gate (30 min, all):** read the statement → map it to the pattern → commit to one hero flow. If it's PS-04, everything below stands.
- [ ] Mock `/v1/authenticate` returning canned answers, so everyone can build against it.
- [ ] Repo skeleton, `demo` branch, deploy targets, keep-alive pings on.

## Phase 2 — H2 to H8: the skeleton (stubs are fine)
Goal at **H8**: app → bank route → rules → hold saved in Supabase → Rohan page → PIN unlocked, end to end on Render.
| Who | Builds |
|---|---|
| TBD | API + rules scorecard (hard-coded thresholds) + fraud-list block + time a dummy model on Render |
| TBD | Demo payment app + Reset (seeds Asha) |
| TBD | Hold screen + recorded warning + **one Rohan component** (judge QR page / in-app fallback / Telegram link) |
| TBD | Generator v1 producing data |

## Phase 3 — H8 to H20: the hero flow
Goal at **H20**: the full 3-minute demo works on the `demo` deployment.
| Who | Builds |
|---|---|
| TBD | Real features (paste, hesitation, app switches, payee history, amount vs usual) into the rules; reasons text |
| TBD | Split screen · device scene (new phone → "Not me") · dashboard with "bank call-back" button |
| TBD | Telegram bot (contact Approve/Cancel), holds saved in Supabase, laptop-fallback runbook |
| TBD | Replay harness (OTP-only vs RBI rule vs Pramaan) · demo script v1 |
**Checkpoint H24:** every MUST item green, or stop and fix.

## Phase 4 — H20 to H36: the evidence (SHOULD, in this order)
1. Blind-test results + "call signal off" result.
2. LightGBM (small) + server-only model; show rules vs model side by side.
3. Probation features + 20-s investment-scam scene.
4. Real passkey approval on the registered phone.
- COULD (only if all green): real call detection app (if the adb test passed) · IEEE-CIS number · hall user test with 10–15 people · cross-victim payee flags.
**Checkpoint H30:** cut any SHOULD item still red.
**H36:** measured numbers frozen · **rehearsal 1** · laptop-fallback switch rehearsed.

## Phase 5 — H36 to H44: polish and protect
- [ ] H40: backup video of the full demo + a 60-s recorded run on a second phone.
- [ ] H40: leave-behind ready: QR card + README with the live link, diagram, REAL/SIMULATED table, the 3 numbers, commit timeline.
- [ ] H42: **code freeze**, fixes only.
- [ ] H44: **rehearsal 2** with Q&A drill (each owner answers their rows).

## Phase 6 — H44 to H48 and judging
- [ ] Warm the server; check keep-alive, Telegram poller, Supabase.
- [ ] H47: **rehearsal 3.**
- [ ] On stage: story → split screen → allow → hold (call off, then on) → judge cancels → device → block → one-line close + one failure case.
- [ ] Hand the depth card to any judge who digs deeper.

## If something goes wrong
| Problem | Do this |
|---|---|
| Behind at H24 | Drop SHOULD items in reverse order; the hero flow comes first |
| Render asleep / down | Switch to the laptop + hotspot + quick tunnel (runbook in `finals-playbook.md` §4) |
| Telegram fails on stage | Use the in-app Rohan panel |
| Totally different statement | Re-skin using the H0 gate table; keep the same demo shape: story → typed intervention → a human approves |
