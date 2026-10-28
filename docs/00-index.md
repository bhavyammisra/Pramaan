# Docs index

> One line per doc. Agents: pick from here and open only what you need. Keep this updated whenever docs change.

## Start here
- `../PLAN.md`: **the whole plan in one page** (problem, insight, solution, demo, evidence, scope). Wins over any older doc.
- `04-plan/roadmap.md`: execution order from 29 Oct to judging, with owners and checkpoints.

## 01 · Problem
- `01-problem/problem-statement.md`: the official statement (verbatim) and our reading of what judges will look for.
- `01-problem/scope.md`: hackathon logistics (dates, submission, judging) and our in/out-of-scope list.

## 02 · Research
- `02-research/README.md`: research hub; answers to the open questions and the one-line insight.
- `02-research/problem-evidence.md`: sourced fraud stats (MHA, RBI, NPCI), the APP-scam shift, friction cost, stakeholders, SDGs.
- `02-research/methods.md`: hybrid scoring, LightGBM + TreeSHAP, cost-based thresholds, latency, privacy (DPDP).
- `02-research/signals.md`: catalogue of behaviour, device and transaction signals we could use, with priority.
- `02-research/datasets.md`: candidate datasets and the synthetic-data plan.
- `02-research/prior-art.md`: static 2FA, commercial engines (Radar, Sift, BioCatch, 3DS), India's tools (NPCI FRM, FRI, MuleHunter), the gap statement, what we borrow.

## 03 · Design
- `03-design/architecture.md`: components, request flow, latency budget.
- `03-design/api-contract.md`: `/v1/authenticate` request/response schema and examples.
- `03-design/risk-engine.md`: how signals become a score, how the score becomes a decision, and how reasons are generated.
- `03-design/decisions.md`: decision log (ADR-lite). Why we chose what we chose. D-012…D-019 are proposals from council review 1.
- `03-design/threat-model.md`: how a scammer or a careless integration could beat Pramaan, and what stops it (demo vs prod).

## 04 · Plan
- `04-plan/team.md`: members, roll numbers, GitHub handles.
- `04-plan/timeline.md`: milestones up to the finals (the hour plan moved to `finals-playbook.md`).
- `04-plan/tasks.md`: task board (todo / doing / done).
- `04-plan/risks-and-metrics.md`: what we'll build in the hack, genuine risks + mitigations, success metrics (measured vs set by policy).
- `04-plan/finals-playbook.md`: **the plan for the finals**: rules check, H0 gate + decision kernel, MoSCoW tiers with owners, milestones, demo-day reliability, team ops.

## 05 · Pitch
- `05-pitch/README.md`: screening deck guide (slide-by-slide), video plan, finals demo script, judge Q&A with answers and owners.
- `05-pitch/Pramaan_BIOS_Screening_v4.pptx`: **final submitted** 5-slide screening deck (BIOS template) + `.pdf`. Built by `scripts/build_deck_v3.py`. `Pramaan_BIOS_Screening.pptx`, `_v2`, `_v3` are older drafts.
- `05-pitch/Pramaan_BIOS_Screening_v5.pptx`: revised deck (+ `.pdf`) after the v4 review: verified numbers, no invented scores, simpler diagram, 48-hour plan. Built by `scripts/build_deck_v5.py`. Superseded by v6.
- `05-pitch/Pramaan_BIOS_Screening_v6.pptx`: **latest** deck (+ `.pdf`) = v5 + review fixes (D-011): registered-phone passkey, family-approval wording, bot row, consistent dates. Built by `scripts/build_deck_v6.py`. Resubmit if the dashboard allows before 19 Oct.

## 06 · Review
- `06-review/council-review-1.md`: LLM-council review of the whole project (28 Oct): 12 weak spots, web fact-checks, where each is fixed, final verdict.
