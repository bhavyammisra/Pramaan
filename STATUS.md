# STATUS — live project state

> Read this every session. Update it after meaningful work (keep it short and overwrite stale lines).

**Last updated:** 2026-10-18 · by Bhavyam (+ Claude)

## Phase
setup → research → design → plan → pitch (screening, done) → **`BUILD (hack)`** → polish

## Done
- **Research** (`docs/02-research/`): sourced evidence, prior art and gap, signals v2, datasets, methods. Key insight: RBI says ATO is "now negligible" and most fraud is authorised push payment, so OTP can't stop it.
- **Design v1** (`docs/03-design/`): architecture, risk engine (hybrid rules + LightGBM + TreeSHAP; amount-aware thresholds; challenge type by risk type), API contract v1, decisions D-002…D-008. Working name **Pramaan**.
- **Plan** (`docs/04-plan/`): team, hack timeline, tasks, risks & metrics.
- **Promo video** (60 s, 1080p60, silent) + timed VO script: `promo/` (see `promo/SCRIPT.md`).
- **Screening deck: final = `docs/05-pitch/Pramaan_BIOS_Screening_v4.pptx`** (Abhijay, built by `scripts/build_deck_v3.py`): 4 levels (allow · biometric · 30-min hold + scam warning + trusted contact via n8n/Telegram · block), ₹0 stack (D-009), all roll nos + Frontier PS-04. Earlier versions (`Pramaan_BIOS_Screening.pptx`, `_v2`, `_v3`) are superseded.
- **v5 deck** (`Pramaan_BIOS_Screening_v5.pptx`, `scripts/build_deck_v5.py`): review fixes to v4. Screening submissions close **19 Oct 2026**; resubmit v5 if the dashboard allows. Results 22 Oct; hack 30 Oct to 1 Nov (48 h) with **new** statements picked on-site.
- **Screening round** deck + promo video submitted (15 Oct).

## In progress
| Who | What |
|-----|------|
| – | Waiting for screening results |

## Next up
1. Start the hack build: `docs/04-plan/timeline.md` (H0–H22 plan) and `docs/04-plan/tasks.md`.
2. First code: synthetic data generator → risk engine (`docs/03-design/risk-engine.md`) → `/v1/authenticate` (`api-contract.md`).
3. Then the 4-level demo: passkeys, n8n + Telegram guardian loop, replay simulator (D-009).
4. Confirm product name D-008; Shefali + Abhijay add GitHub handles in `team.md`.

## Blockers / open questions
- Hack dates/duration and judging criteria not published yet.
- Challenge: Frontier mode · PS-04.
