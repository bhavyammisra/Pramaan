# Timeline

| Date | Milestone | Status |
|---|---|---|
| 2026-10-14 | Problem finalised, repo set up | ✅ |
| 2026-10-15 | Research + design + build plan (docs) | ✅ |
| 2026-10-15 (night) | Screening submission: 5-slide deck (BIOS template, v4) + video | ✅ |
| **2026-10-19** | Screening closes: resubmit **v6** deck if the dashboard allows | ⏳ |
| 2026-10-22 | Screening results | |
| **2026-10-30 → 11-01** | BIOS hack, **48 h**, new statements picked on-site (plan below) | |

## Build plan for the hack (48 h; matches deck v6 slide 5)
Stack per D-009: FastAPI on Render free tier, in-memory profiles backed by Supabase, React (Vite) on Vercel, n8n + Telegram, WebAuthn.

| Hours | Milestone |
|---|---|
| H0–H8 | Synthetic generator (users, normal traffic, 4 attack types, hard negatives); FastAPI skeleton + `/v1/health`; Render deploy + Supabase tables; keep-alive ping |
| H8–H20 | Features + LightGBM + calibration + TreeSHAP + reason templates + policy (`config.yaml`); `/v1/authenticate` end-to-end with profiles, event log, 150 ms fail-safe |
| H20–H34 | Demo UPI app + JS SDK + scenario toggles (on-call, screen-share, SIM change); passkey on the registered phone; n8n → Telegram family approval (no reply in 30 min = cancel) |
| H34–H44 | Dashboard (live feed, "why" panel, latency, threshold sliders); replay vs RBI blanket delay; scenario tests; load test; tune `k`/`τ_min` to the metrics |
| H44–H48 | Final deploy, warm the server, rehearse the demo |
