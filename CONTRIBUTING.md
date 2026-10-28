# Contributing (HTTPS only, no SSH needed)

## One-time setup
**Easiest:** ask Claude to follow [`SETUP.md`](SETUP.md) (see the prompt at the top of the README). The manual steps are below.

1. Accept the collaborator invite from **bhavyammisra**. Check your email, or go to github.com/notifications.
2. Install **Git for Windows** (https://git-scm.com). It includes Git Credential Manager, so the first push opens a browser window to sign in to GitHub. No SSH keys or tokens are needed.
   - Not comfortable with the CLI? **GitHub Desktop** works just as well.
3. Clone with HTTPS into your hackathon folder:
   ```bash
   git clone https://github.com/bhavyammisra/Pramaan.git
   cd Pramaan
   git config user.name  "Your Name"
   git config user.email "you@example.com"
   ```
4. Open Claude in this folder. `CLAUDE.md` and `.claude/settings.json` load automatically, and the session-start hook runs `git pull` for you.

## Every session
```bash
git pull --rebase --autostash     # Claude's hook does this automatically
# ... work ...
git add -A
git commit -m "docs: add device-signal research"
git pull --rebase                 # pick up teammates' work
git push
```

## Commit message prefixes
`docs:` documentation · `feat:` new code · `fix:` bug fix · `refactor:` · `test:` · `chore:` setup/tooling · `data:` datasets/scripts

## Rules
- Commit small and often, and **push before you close your laptop**.
- Working on a big code change? Use a branch (`git switch -c feat/risk-engine`) and open a PR. Docs edits can go straight to `main`.
- If you get a conflict, don't panic. Ask Claude: "help me resolve this merge conflict".
- Never commit `.env`, API keys or large raw datasets.
