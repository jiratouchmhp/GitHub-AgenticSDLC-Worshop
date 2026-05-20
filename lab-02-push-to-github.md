# Lab 2 — Push to GitHub

> **Estimated time:** 25 minutes
> **Surface:** GitHub.com + local Git + VS Code

## 🎯 Learning objectives

1. Create a GitHub repository in your workshop organization.
2. Push your local `store-app` repo to GitHub for the first time.
3. Stand up baseline **branch protection** via a ruleset: required PR review, signed commits, force-push block.

## ✅ Prerequisites

- Lab 0 + Lab 1 complete; `store-app` exists locally with committed code, custom agents under `.github/agents/`, and skills under `.github/skills/`.
- You have **write** access to a repository in the workshop GitHub organization (or can create one in your personal account).
- Commit signing configured (or you'll merge PRs via the GitHub web UI). See [PREREQUISITES.md §5](./PREREQUISITES.md#5-commit-signing-required-for-lab-2s-ruleset).

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| Creating a new repository | <https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories> |
| Adding a remote / pushing | <https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories> |
| **Rulesets** overview | <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets> |
| Creating a branch ruleset | <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository> |
| Required pull request reviews | <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-pull-request-reviews-before-merging> |
| Signed commit verification | <https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification> |
| CODEOWNERS | <https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners> |

---

## Step 1 — Create the GitHub repository (3 min)

> 📖 **Reference for this step:** [Quickstart for repositories](https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories) · [About repository visibility](https://docs.github.com/en/repositories/creating-and-managing-repositories/about-repositories#about-repository-visibility)

1. Go to your workshop organization (or your user account) and click **New repository**.
2. Name it **`store-app`**. Owner = workshop org.
3. Visibility = **Private** (or Internal).
4. **Do not** initialize with README, .gitignore, or LICENSE — we'll push the existing repo.
5. Click **Create repository**.

---

## Step 2 — Push your local repo to GitHub (3 min)

> 📖 **Reference for this step:** [Managing remote repositories](https://docs.github.com/en/get-started/getting-started-with-git/managing-remote-repositories) · [Pushing commits to a remote](https://docs.github.com/en/get-started/using-git/pushing-commits-to-a-remote-repository) · [`git push` (Pro Git book)](https://git-scm.com/docs/git-push)

In your local `store-app` working directory:

```bash
git remote add origin https://github.com/<ORG>/store-app.git
git push -u origin main
```

✅ **Checkpoint:** Visit your repo on GitHub — all commits from Lab 0 and Lab 1 are visible, including `.github/agents/*.agent.md` and `.github/skills/**/SKILL.md`.

> [!NOTE]
> Your custom agents and skills are now **part of the repo**. Anyone who clones this workspace gets them automatically. The Copilot **Coding Agent** also reads agents/skills from the repo when it runs in the cloud (Lab 3 enables that).

---

## Step 3 — Create a branch ruleset (7 min)

> 📖 **Reference for this step:** [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) · [Creating rulesets for a repository](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/creating-rulesets-for-a-repository) · [Available rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) · [Required pull request reviews](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-pull-request-reviews-before-merging) · [Signed commit verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)

1. Go to **Settings → Rules → Rulesets → New ruleset → New branch ruleset**.
2. **Ruleset name:** `protect-main`
3. **Enforcement status:** `Active`
4. **Bypass list:** leave empty.
5. **Target branches:** Add target → **Include default branch**.
6. Under **Branch protections**, enable:
   - ✅ **Restrict deletions**
   - ✅ **Block force pushes**
   - ✅ **Require a pull request before merging** — Required approvals: **1**, Dismiss stale approvals, Require code-owner review, Require approval of the most recent reviewable push
   - ✅ **Require signed commits**
   - ✅ **Require status checks to pass** *(we'll add the CodeQL check in Lab 4 — leave the list empty for now)*
   - ✅ **Require conversation resolution before merging**
7. Click **Create**.

> [!WARNING]
> "Require signed commits" means every commit on `main` must be signed. If you didn't configure local signing, merge PRs via the **GitHub web UI** or `gh pr merge --squash` — both produce signed commits via GitHub's web-flow key.

Confirm the ruleset is live by trying a direct push to `main`:

```bash
echo "" >> README-placeholder.txt    # tiny throwaway change
git add README-placeholder.txt
git commit -m "chore: test ruleset"
git push origin main                  # should be REJECTED by the ruleset
```

Then route the same change through a PR instead:

```bash
git checkout -b chore/ruleset-smoke-test
git push -u origin chore/ruleset-smoke-test
# Open a PR on GitHub.com, approve, and squash-merge via the web UI.
```

Delete the throwaway file in a follow-up commit (or just close the PR if you'd rather not merge a placeholder).

✅ **Checkpoint:** Direct `git push origin main` is rejected. A PR through the web UI merges successfully.

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| Push to `main` rejected (signed commits) | Merge PRs via the GitHub web UI (signed via `web-flow`), or set up GPG/SSH signing per [PREREQUISITES.md §5](./PREREQUISITES.md#5-commit-signing-required-for-lab-2s-ruleset). |
| Push to `main` rejected (no PR) | Expected — the ruleset is doing its job. Open a PR on a feature branch instead. |
| Custom agents/skills not visible to the Coding Agent later | Confirm `.github/agents/*.agent.md` and `.github/skills/**/SKILL.md` were pushed to the default branch. |

## 🎁 Stretch goals

- Create an **organization-level ruleset** that applies signed-commits and required reviews to all `store-*` repos.
- Add a **GitHub Actions CI** workflow that runs the test suite on PRs, then add it as a required status check.
- Add a `.github/dependabot.yml` for your ecosystem (you'll enable Dependabot in Lab 4).

---

➡️ **Next: [Lab 3 — Coding Agent for Docs & Unit Tests + Code Review](./lab-03-coding-agent-docs-tests.md)**
