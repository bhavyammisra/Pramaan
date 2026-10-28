# CLAUDE.md — Caffeinated Nerds · BIOS Hackathon

Problem: **Fast and Secure Online Transaction Authentication**. We're building a real-time risk service that returns **allow / challenge / block** for a transaction, with reasons.

New teammate or no git identity → follow [`SETUP.md`](SETUP.md).

## Sync
- The `SessionStart` hook runs `git pull --rebase --autostash`. If it failed or shows a conflict, tell the user before anything else. If it didn't run, pull yourself.
- Push with: `git add -A && git commit -m "<type>: <what>" && git pull --rebase && git push` (prefixes in `CONTRIBUTING.md`).

## When to push
- **Docs** (`docs/`, `*.md`, `src/*/README.md`): push right after every edit.
- **Task complete** (works, tests pass): push automatically.
- **Task incomplete**: don't push yet; still push any docs edits.
- **Needs approval** — ask first: deleting/rewriting a teammate's work, changing a decision, big code change (use a branch), secrets/config, or the user said wait.
- Never force-push. Report failures immediately; after pushing, say what went up in one line.

## Where to look
- Always read `STATUS.md` first, then `docs/00-index.md` to pick which docs to open.
- Problem/scope → `docs/01-problem/` · research → `docs/02-research/` · design, API, risk engine, decisions → `docs/03-design/` · plan, tasks → `docs/04-plan/` · pitch → `docs/05-pitch/` · code → `src/README.md`.
- Skills: `.claude/skills/llm-council/` (Karpathy's LLM Council on Claude subagents, free). Say "council this: <question>".

## Rules
- Keep docs short and concise; no filler.
- Docs are the source of truth. If code and docs disagree, flag it.
- Adding/renaming a doc → update `docs/00-index.md` in the same commit.
- Non-trivial choices → append to `docs/03-design/decisions.md`.
- Update `STATUS.md` after meaningful work.
- No secrets in the repo (`.env` is git-ignored; list keys in `.env.example`).
