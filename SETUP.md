# SETUP.md — onboarding a new teammate (written for Claude to execute)

> **Claude:** a teammate on **Caffeinated Nerds** asked you to set them up on this repo. Follow these steps in order. Run the commands yourself wherever you can. Only stop to ask the user for things you can't know or do (their name, email, a browser sign-in). Keep your messages short and friendly.
>
> Repo: `https://github.com/bhavyammisra/Pramaan` (HTTPS only, no SSH).

---

## Step 0 · Ask the user (once, all together)
1. Their **name** and **GitHub username**
2. The **email** they use on GitHub (for commits)
3. **Where to put the project.** Default: the folder Claude is currently open in.
4. Have they **accepted the collaborator invite**? If not, they should check their email or go to https://github.com/bhavyammisra/Pramaan/invitations. If no invite has arrived, they should ask Bhavyam to add their GitHub username under Settings → Collaborators. Reading and cloning work without it; only pushing needs it.

## Step 1 · Check git
```bash
git --version
```
- If git is missing on **Windows**, run `winget install --id Git.Git -e --source winget`, or ask them to install it from https://git-scm.com. Git for Windows includes **Git Credential Manager**, which handles the GitHub sign-in through the browser. No SSH keys or tokens are needed.
- On **macOS**: `xcode-select --install` or `brew install git`.

## Step 2 · Clone (HTTPS)
- If the chosen folder is **empty**, clone into it:
  ```bash
  git clone https://github.com/bhavyammisra/Pramaan.git .
  ```
- Otherwise, clone into a subfolder:
  ```bash
  git clone https://github.com/bhavyammisra/Pramaan.git
  cd Pramaan
  ```
- If you're **already inside a clone** of this repo (`git remote -v` shows it), skip the clone and run `git pull --rebase --autostash`.

## Step 3 · Configure identity (repo-local)
```bash
git config user.name  "<Their Name>"
git config user.email "<their-github-email>"
```
If git later says **"detected dubious ownership"** (this happens on Windows), run:
```bash
git config --global --add safe.directory "<absolute path of the repo, using forward slashes>"
```

## Step 4 · Add themselves to the team and test push access
1. In `docs/04-plan/team.md`, fill in their roll number and `@github-username` on their row (add a row if missing).
2. Commit and push:
   ```bash
   git add docs/04-plan/team.md
   git commit -m "chore: onboard <Name>"
   git pull --rebase
   git push
   ```
3. **The first push opens a browser window to sign in to GitHub.** Tell the user to complete it.
4. If the push fails:
   - **403 / "Permission denied" / "Repository not found"**: they aren't a collaborator yet. Go back to Step 0.4, then retry the push.
   - **Rejected (non-fast-forward)**: run `git pull --rebase`, then `git push`.

## Step 5 · Confirm the Claude setup works
- Check that `.claude/settings.json` exists. It holds a `SessionStart` hook that runs `git pull` automatically every time Claude starts in this folder.
- Tell the user: **"From now on, open Claude inside this folder. It will auto-sync and read `CLAUDE.md` first."** If Claude was opened somewhere else, they should reopen it in the repo folder so the hook and `CLAUDE.md` load.
- If the user is in a Claude app that **can't run commands on their computer**, give them Steps 1–4 as copy-paste commands instead, and ask them to paste back any errors.

## Step 6 · Hand off
1. Read `STATUS.md` and `docs/04-plan/tasks.md`.
2. Give the user a 4–5 line summary: what the project is, the current phase, and 2–3 unowned tasks they could pick up.
3. Remind them of the daily loop: **pull → work → commit → pull → push** (details in `CONTRIBUTING.md`).

✅ Setup is done once their onboarding commit is visible on GitHub.
