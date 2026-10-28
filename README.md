# ☕ Caffeinated Nerds: BIOS Hackathon

> ## 🚀 New teammate? Paste this into Claude (opened in the folder where you want the project):
> ```
> Read https://raw.githubusercontent.com/bhavyammisra/Pramaan/main/SETUP.md and follow it to set me up on this repo.
> ```
> **🤖 Claude:** if you were asked to set someone up on this repo, fetch and follow [`SETUP.md`](SETUP.md) step by step.

**Problem: Fast and Secure Online Transaction Authentication** (Frontier track, fixed statement)

> Build a cloud authentication service that uses behaviour, device information and transaction patterns to **allow, challenge or block** an online transaction.
> **Expected outcome:** a real-time risk service that returns a decision and explains the signals behind it.

## What we're building
A low-latency risk API. Send it a transaction (plus device and behaviour context) and it returns:

```json
{ "decision": "challenge", "risk_score": 0.62,
  "reasons": ["New device for this user", "Amount 8x above user's median", "Login → pay in 4s"] }
```

There's also a dashboard that shows decisions live and the "why" behind each one.

## Where things are
- **Project state:** [`STATUS.md`](STATUS.md)
- **All docs, summarised:** [`docs/00-index.md`](docs/00-index.md)
- **How we work with git:** [`CONTRIBUTING.md`](CONTRIBUTING.md)
- **Code:** [`src/`](src/README.md)

## Quickstart
_To be filled once the stack is decided (see `docs/03-design/decisions.md`)._

## Team
See [`docs/04-plan/team.md`](docs/04-plan/team.md).
