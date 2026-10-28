# STATUS — live project state

> Read this every session. Update it after meaningful work (keep it short and overwrite stale lines).

**Last updated:** 2026-10-28 · by Bhavyam (+ Claude)

**Selected for the BIOS finals.** Hack runs 30 Oct to 1 Nov (48 h); new problem statements are picked on-site. **Plan for the finals: `docs/04-plan/finals-playbook.md`.**

## Phase
setup → research → design → plan → pitch (screening, done) → review (council, done) → **`PREP (29 Oct)`** → build (hack) → polish

## Done
- **Research** (`docs/02-research/`): sourced evidence, prior art and gap, signals v2, datasets, methods. Key insight: RBI says ATO is "now negligible" and most fraud is authorised push payment, so OTP can't stop it.
- **Design v1** (`docs/03-design/`): architecture, risk engine (hybrid rules + LightGBM + TreeSHAP; amount-aware thresholds; challenge type by risk type), API contract v1, decisions D-002…D-008. Working name **Pramaan**.
- **Plan** (`docs/04-plan/`): team, hack timeline, tasks, risks & metrics.
- **Promo video** (60 s, 1080p60, silent) + timed VO script: `promo/` (see `promo/SCRIPT.md`).
- **Screening deck submitted on 15 Oct = `docs/05-pitch/Pramaan_BIOS_Screening_v4.pptx`** (now superseded by v6, below) (Abhijay, built by `scripts/build_deck_v3.py`): 4 levels (allow · biometric · 30-min hold + scam warning + trusted contact via n8n/Telegram · block), ₹0 stack (D-009), all roll nos + Frontier PS-04. Earlier versions (`Pramaan_BIOS_Screening.pptx`, `_v2`, `_v3`) are superseded.
- **v5 deck** (`Pramaan_BIOS_Screening_v5.pptx`, `scripts/build_deck_v5.py`): review fixes to v4. Screening submissions close **19 Oct 2026**; resubmit v5 if the dashboard allows. Results 22 Oct; hack 30 Oct to 1 Nov (48 h) with **new** statements picked on-site.
- **Screening round** deck + promo video submitted (15 Oct).
- **v6 deck** (`Pramaan_BIOS_Screening_v6.pptx` + `.pdf`, `scripts/build_deck_v6.py`): review fixes to v5, D-011 (passkey on registered phone, family-approval wording, bot row, dates). Docs brought in line: DPDP dates, Render sleep risk, threshold-vs-target risk, stale Cloud Run/Redis and 24 h plan removed.
- **Repo renamed** to `github.com/bhavyammisra/Pramaan` (28 Oct). Old URL redirects; run `git remote set-url origin https://github.com/bhavyammisra/Pramaan.git` once.
- **Council review 1 (28 Oct)** (`docs/06-review/council-review-1.md`): the whole project went through the LLM council (3 rounds + web research). 12 weak spots found and fixed in **design v2 (proposed D-012…D-019)**: risk rank + bank-set friction budget, only hard rules block, multiclass model, threat model, one release rule for scam holds, corrected RBI/Google claims, finals playbook with MoSCoW tiers and owners.

## In progress
| Who | What |
|-----|------|
| All | 29 Oct prep (no project code until organisers allow it): see `docs/04-plan/tasks.md` |

## Next up
1. **29 Oct:** ask organisers the 5 questions (statement, pre-written code, rubric/demo length, phones, AI coding tools) → `docs/04-plan/finals-playbook.md` §0.
2. Accept or change proposals D-012…D-019 (`docs/03-design/decisions.md`); confirm track + Q&A owners.
3. Non-code prep: accounts + Telegram bot, phone checks (passkey, `adb shell dumpsys audio` during a WhatsApp call, Telegram in-app browser), record the warning mp3, generator spec, deck v7 with corrected claims, README skeleton, IEEE-CIS download.
4. **H0:** 30-min gate mapping the on-site statement onto the decision kernel, then the playbook milestones (hero flow by H20).
5. Promo video (already submitted) still says "fingerprint" and "son can cancel": answer ready in `docs/05-pitch/README.md` Q&A.

## Blockers / open questions
- Organisers: what "new statement on-site" means; whether pre-written code is allowed; judging rubric and demo length.
- Challenge: Frontier mode · PS-04.
