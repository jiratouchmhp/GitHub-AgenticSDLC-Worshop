# Lab 5 — Remediate with Copilot Autofix + the Coding Agent

> **Estimated time:** 35 minutes
> **Surface:** GitHub.com (Security tab + PRs) — entirely browser-driven

In Lab 4 you enabled all four scanners, seeded the findings on `vuln/demo-findings`, and pre-assigned **one** of the two SQL-injection alerts (the one in `server/src/routes/search.ts`) to the Coding Agent. The sibling alert in `server/src/routes/lookup.ts` was deliberately left **Open**. This lab walks the **full remediation loop** through **two officially-supported AI options** — once for each — then applies the same two-option lens to Dependabot and Code Quality.

## 🎯 Learning objectives

By the end of this lab you will:

1. Resolve a Code Scanning alert with **Option A — Copilot Autofix** (one-click "Generate fix" inside the alert).
2. Resolve a sibling Code Scanning alert with **Option B — Copilot Coding Agent** (delegated draft PR, iterable via `@copilot` comments).
3. Apply the same two-option pattern to a **Dependabot alert** (Dependabot auto-PR vs. Coding Agent escalation).
4. Resolve a **Code Quality finding** with the Coding Agent and understand why Autofix doesn't apply there today.
5. Re-run scans, verify alerts close, and reflect on the full agentic SDLC loop.

## ✅ Prerequisites

- Lab 4 complete: `vuln/demo-findings` pushed to GitHub, four scanners enabled, **two** `js/sql-injection` alerts visible (search.ts + lookup.ts), and the **search.ts alert already assigned to Copilot** with a draft PR open (Lab 4 Part E).

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| **Copilot Autofix** for Code Scanning | <https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning> |
| **Responsible use** of Autofix | <https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning> |
| Autofix — supported languages | <https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#supported-languages-for-codeql-code-scanning> |
| Autofix — limitations of dependency suggestions | <https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#limitations-of-dependency-suggestions> |
| Working with Autofix suggestions | <https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/working-with-copilot-autofix> |
| Copilot **Coding Agent** — about | <https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent> |
| Coding Agent — configuring agent settings (validation tools) | <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings> |
| Coding Agent — auto-approve workflow runs | <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings#allowing-github-actions-workflows-to-run-automatically-when-copilot-pushes> |
| Assigning a code-scanning alert to Copilot | <https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests#fixing-a-code-scanning-alert-with-copilot> |
| **Dependabot** — pull requests | <https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates> |
| Security campaigns | <https://docs.github.com/en/code-security/security-campaigns/managing-security-campaigns/about-security-campaigns> |

## 🧭 The two supported remediation options

Both options are first-party GitHub features and both end at the same outcome (alert closes on the next scan). Pick per situation.

| | **Option A — Copilot Autofix** | **Option B — Copilot Coding Agent** |
|---|---|---|
| **Where it lives** | The alert page itself (**Generate fix** button) | Alert sidebar (**Assignees → Copilot**) |
| **What you get** | A suggested diff committed to a new branch + auto-opened PR | A full draft PR opened by the agent with a session you can replay |
| **Best for** | Single localized CodeQL alert in a supported language (JS/TS, Python, Java/Kotlin, Go, C#, C/C++, Swift, Ruby, Rust) | Cross-file changes, new tests required, Dependabot bumps that need call-site updates, Code Quality findings |
| **Iteration model** | Edit the suggestion before committing; re-run **Generate fix** for a new attempt | Comment `@copilot …` on the PR; agent re-runs and pushes a follow-up commit |
| **Cost / latency** | Free on public repos; included with **GitHub Code Security** for private repos; suggestion appears in seconds | Consumes Copilot premium requests + Actions minutes; draft PR appears in ~30 s, full run takes minutes |
| **Trace / auditability** | The committed diff + the suggestion's explanation | Full **agent session** view (plan + tool calls) plus the PR diff |
| **Limitations to remember** | LLM-generated, non-deterministic, may be partial / introduce syntax or semantic errors / suggest fabricated dependencies — you must verify CI and behavior ([responsible use](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning)) | Agent PR may need iteration; Actions workflows on agent PRs require **Approve and run** by default ([cloud agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings)) |

> [!NOTE]
> **Coding Agent is secure-by-default.** By design, the Coding Agent runs its own output through built-in **validation tools** (a security scan + Copilot Code Review) before it finishes the PR, attempting to resolve issues it introduces. Repo admins can toggle these at **Settings → Copilot → Cloud agent → Validation tools**. See [Configuring agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings#enabling-or-disabling-built-in-code-quality-and-security-validation-tools).

> [!IMPORTANT]
> Both options are **AI suggestions** — you stay accountable for the merge. Always read the diff, run the tests, and confirm the alert closes on the post-merge re-scan.

---

## Step 1 — Resolve the `lookup.ts` SQL injection with **Option A: Copilot Autofix** (8 min)

> 📖 **Reference for this step:** [Copilot Autofix for code scanning](https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning) · [Working with Autofix suggestions](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/working-with-copilot-autofix) · [Responsible use of Autofix](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning) · [Limitations of dependency suggestions](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#limitations-of-dependency-suggestions) · [Supported languages for Autofix](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#supported-languages-for-codeql-code-scanning)

This is the alert you intentionally left **Open** in Lab 4 Part E. We'll close it with one click, review the suggestion, and ship.

1. **Security → Code scanning** → click the `js/sql-injection` alert whose location is **`server/src/routes/lookup.ts`**.
2. In the alert page, click the **Generate fix** button (top right of the alert details, next to *Show paths*).
   - If the button is missing, see Troubleshooting below — Autofix requires the repo to be public or covered by **GitHub Code Security**, and the alert must come from a CodeQL query that supports Autofix.
3. After a few seconds, Autofix renders a suggested diff inline with a natural-language explanation of the fix (typically: switch to a parameterized prepared statement, bind `category` as a parameter).
4. **Read the diff before you trust it.** Per the [responsible-use guidance](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning), Autofix suggestions are LLM-generated and may:
   - be syntactically wrong or applied at the wrong location,
   - partially fix the alert (e.g., parameterize the binding but leave a `LIKE` wildcard issue),
   - change semantics in subtle ways,
   - or even add a fabricated dependency.
   - Verify the fix uses `better-sqlite3`'s positional/named parameter API and does **not** introduce a new dependency.
5. Click **Commit to a new branch and open a pull request** (the exact label is *Create PR with fix*).
6. On the new PR:
   - Wait for CI (Jest + Vitest + CodeQL) to pass. If your `protect-main` ruleset requires workflow approval on first-run branches, click **Approve and run**.
   - Approve (pair up if a second human reviewer is required) and **squash-merge via the web UI**.
7. Within ~2 minutes after merge, the next CodeQL scan on `main` closes the alert automatically.

> [!TIP]
> Don't love the suggestion? Either **edit the diff in-place** before committing, or close the suggested PR and click **Generate fix** again — Autofix is non-deterministic and a second attempt may produce a cleaner fix.

✅ **Checkpoint:** The `lookup.ts` `js/sql-injection` alert transitions from **Open** to **Fixed** in **Security → Code scanning**. The merge commit is signed by `web-flow`.

---

## Step 2 — Resolve the `search.ts` SQL injection with **Option B: Copilot Coding Agent** (10 min)

> 📖 **Reference for this step:** [About Copilot Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent) · [Configuring agent settings (validation tools)](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings) · [Iterating with `@copilot` on PRs](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/iterate-on-coding-agent-pull-requests) · [Track Coding Agent sessions](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/track-coding-agent-sessions) · [better-sqlite3 — prepared statements](https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md#preparesource---statement)

The draft PR opened at the end of Lab 4 (titled something like *"Fix js/sql-injection in server/src/routes/search.ts"*) is the Coding Agent's first attempt. Treat it like any other PR from a junior engineer — read the diff before you trust it.

1. Open the draft PR. Click **View session** (or **Agents → Recent sessions**) to see the agent's plan and the steps it took.
2. Open the **Files changed** tab. Check that:
   - `server/src/routes/search.ts` uses a **parameterized prepared statement** (no string concatenation of `req.query.q`).
   - The `backend` skill's convention from `.github/skills/backend/SKILL.md` is honored (named or positional parameters, no inline SQL in handlers if a prepared statement helper exists).
   - The agent added or updated a server test for the route.
   - The built-in **validation tools** (security scan + Copilot Code Review) ran and did not flag the agent's own diff. If they did, the agent should have already attempted a follow-up commit — confirm via the session view.

If the fix is too narrow (e.g., it parameterizes but leaves the `LIKE` wildcards unescaped, so a search for `100%` matches everything), iterate via a PR comment. The agent re-runs on `@copilot` mentions and PR reviews.

> **Prompt template (PR comment to the Coding Agent):**
> ```
> @copilot the change is correct for /api/search, but also escape the
> LIKE wildcard characters (% and _) from user input — otherwise a
> search for "100%" matches every product. Add a server test
> (Jest + supertest) that hits /api/search?q=100%25 against the seeded
> catalog and asserts the response returns zero rows, not the full
> products table.
> ```

Wait for the agent's follow-up commit. When you're satisfied:

1. Approve the PR (pair up if your `protect-main` ruleset requires a second human reviewer).
2. **Squash-merge via the web UI** so the merge commit is signed by `web-flow`.
3. The linked Code Scanning alert auto-closes on the next CodeQL run (~2 min after merge).

✅ **Checkpoint:** The `search.ts` SQL-injection alert transitions from **Open** to **Fixed**. You have now closed **two sibling CodeQL alerts with two different AI options** and can compare the experience side by side.

---

## Step 3 — Remediate the Dependabot alert (6 min)

> 📖 **Reference for this step:** [Configuring Dependabot security updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/configuring-dependabot-security-updates) · [Reviewing & merging Dependabot PRs](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates) · [Assigning a Dependabot alert to Copilot](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts) · [Approving workflow runs from first-time contributors](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks)

The lodash 4.17.20 pin in Lab 4 fired a Dependabot alert. Dependabot has its own auto-PR flow that is the supply-chain equivalent of Autofix — fast, scoped, deterministic. The Coding Agent is the escalation when the bump isn't trivially compatible.

### Option A — Take Dependabot's auto-PR (the supply-chain "Autofix" equivalent)

1. Open the PR Dependabot already opened (titled e.g. *"Bump lodash from 4.17.20 to 4.17.21"*).
2. Skim the changelog link in the PR body.
3. Wait for CI to pass (workflows on Dependabot PRs may require you to click **Approve and run**).
4. **Squash-merge** via the web UI.

✅ **Checkpoint:** The lodash Dependabot alert flips to **Fixed**.

### Option B — Escalate to the Coding Agent (when Dependabot can't auto-fix)

For breaking-change bumps Dependabot can't apply alone (call sites need updates, types changed, etc.):

1. **Security → Dependabot** → open the alert → **Assignees → Copilot**.
2. Within ~30 seconds a 👀 reaction appears; the agent opens a draft PR that bumps the dependency, updates call sites, and re-runs the test suite.
3. Review the diff as in Step 2, iterate via PR comments if needed, then merge.

✅ **Checkpoint:** You've seen both remediation paths for a Dependabot alert and know when to use each.

---

## Step 4 — Remediate the Code Quality finding with the Coding Agent (6 min)

> 📖 **Reference for this step:** [Reviewing Code Quality findings](https://docs.github.com/en/code-security/how-tos/maintain-quality-code/review-code-quality-findings) · [About Code Quality](https://docs.github.com/en/code-security/concepts/about-code-quality) · [About Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent) · [`.github/copilot-instructions.md`](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)

> [!NOTE]
> **Why no Option A here?** Copilot Autofix today is scoped to **CodeQL code scanning** queries on supported languages. Code Quality findings (CodeQL maintainability rules + the AI-findings stream) are remediated through the Coding Agent surface — same `Assignees → Copilot` flow you used in Step 2, just on a different bot (`github-code-quality[bot]`).

1. **Security → Code quality → Standard findings** → click the finding for `server/src/utils/quality-smell.ts`.
2. In the right sidebar: **Assignees → Copilot**.
3. The Coding Agent opens a draft PR that cleans up the unused variable, collapses the duplicate condition, and removes the unreachable branch (or deletes the file if it determines it's dead code — confirm via the session view).

Review and iterate via PR comments if needed:

> **Prompt template (PR comment to the Coding Agent):**
> ```
> @copilot the smell file is intentionally unused — please delete
> server/src/utils/quality-smell.ts outright rather than rewriting it,
> and confirm no production code imports it.
> ```

Approve and squash-merge.

> [!TIP]
> The agent honors `.github/copilot-instructions.md` *and* every repo-defined custom agent/skill. Maintainability rules you add to those files (e.g., "prefer early returns", "no nested ternaries") are respected on every remediation PR going forward.

✅ **Checkpoint:** The Code Quality finding has a merged fix (or a draft PR ready to merge from the agent).

---

## Step 5 — Re-run scans & verify (3 min)

> 📖 **Reference for this step:** [About code scanning alert states](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts#alert-status) · [Dismissing Secret Scanning alerts (Revoked / Used in tests / False positive)](https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning#dismissing-alerts) · [Closing Dependabot alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts#dismissing-dependabot-alerts)

After all four fix PRs are merged into `main` (Autofix on lookup.ts, Coding Agent on search.ts, Dependabot on lodash, Coding Agent on quality-smell):

1. **Actions** → watch CodeQL + Code Quality re-run on `main`.
2. **Security** sidebar:
   - **Code scanning** — **both** `js/sql-injection` alerts: **Fixed** (one by Autofix, one by Coding Agent).
   - **Dependabot** — lodash: **Fixed**.
   - **Code quality** — smell finding: **Resolved** on Standard findings.
   - **Secret scanning** — the fake AWS keys + PAT remain **Open** (intentionally). In real life: rotate at the provider, then dismiss with reason **"Revoked"**.

✅ **Checkpoint:** Four alerts (two security + dependency + quality) have transitioned Open → Fixed/Resolved using **two different AI options** — all driven from the browser.

---

## Step 6 — Wrap-up: the full agentic SDLC loop (3 min)

> 📖 **Reference for this step:** [Plan Mode](https://code.visualstudio.com/docs/copilot/agents/planning) · [Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents) · [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) · [Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent) · [Copilot Code Review](https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review) · [Copilot Autofix](https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning) · [GHAS overview](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security)

Across all six labs, you've used:

| Stage | What you used | What the AI did |
|---|---|---|
| **Plan** | Copilot **Plan Mode** | Drafted the MVP plan |
| **Scaffold agents** | `/create-agent` × 3, `/create-skill` × 3 | Generated reusable agent personas and on-demand skills |
| **Code** | Custom agents (`app-builder`) + on-demand skills | Built backend + frontend + SQLite layer |
| **Commit** | Generate Commit Message | Summarized diffs into Conventional Commits |
| **Pre-commit review** | Code Review – Uncommitted Changes (VS Code) | Caught issues before pushing |
| **Async coding** | Coding Agent (issue → PR) | Wrote docs, tests, and a feature in three parallel PRs |
| **PR review** | Copilot Code Review (PR-level) | Reviewed every PR with consistent rigor |
| **Secure** | CodeQL, Secret Scanning, Push Protection, Dependabot | Found vulnerabilities continuously |
| **Quality** | GitHub Code Quality (CodeQL + AI findings) | Flagged reliability/maintainability issues |
| **Remediate — Option A** | Copilot **Autofix** + Dependabot auto-PR | One-click suggested fix scoped to a single alert |
| **Remediate — Option B** | Copilot **Coding Agent** (alert → draft PR) | Produced fix PRs for security, dependency, and quality findings, iterable via `@copilot` comments |

### Discussion prompts

- **Where did Autofix beat the Coding Agent, and vice versa?** Localized, single-file CodeQL alerts vs. cross-file / new-test / Dependabot-escalation work.
- **Where does the human stay in the loop?** (Review and merge gates, prompt authoring, dismiss decisions, signed commits, branch protections, **Approve and run** on agent PRs.)
- **What was the most leveraged use of the Coding Agent?** (Parallel docs/tests/feature in Lab 3, or alert-driven remediation here in Lab 5?)
- **Where did you push back on the AI?** Those signals tell you what to put in `copilot-instructions.md` or a custom agent next.
- **What governance is non-negotiable?** Signed commits, required reviews, CODEOWNERS, push protection, GHAS, ruleset bypass logging, **Cloud agent → Validation tools** left ON.
- **Cost awareness:** Autofix is essentially free; the Coding Agent consumes Copilot premium requests *and* GitHub Actions minutes — multiply by the number of alerts in a real triage.

> [!IMPORTANT]
> Agents aren't replacements for engineers — they're force multipliers on **good engineering hygiene**. Strong issues, strong branch protections, strong tests, strong reviews → strong AI leverage. Weak hygiene + AI = faster mess.

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| **Generate fix** button missing on a Code Scanning alert | Autofix requires (a) a public repo **or** a private repo covered by **GitHub Code Security**, and (b) the alert must come from a CodeQL query that Autofix supports. See the [supported languages list](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#supported-languages-for-codeql-code-scanning). |
| Autofix shows "No suggestion available" | Autofix is best-effort and non-deterministic. Try again later, switch to **Option B** (assign the alert to Copilot), or fix by hand. |
| Autofix's suggestion is wrong or partial | Edit the diff before committing, **or** discard and click **Generate fix** again, **or** escalate to **Option B** (Coding Agent) for a fuller change with tests. |
| Autofix added a new dependency you don't recognize | **Do not merge.** Per the [responsible-use guidance](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning#limitations-of-dependency-suggestions), the model can fabricate dependency names. Verify the package exists on the registry and is trustworthy, or remove the dependency change before merging. |
| "Copilot" missing from an alert's Assignees | Org/enterprise policy for Coding Agent is off, or repo is not in the allowlist. |
| Coding Agent's PR closes the alert but breaks tests | Comment on the PR: `@copilot the fix broke <test>; please update the test or adjust the fix without weakening the security guarantee.` |
| Coding Agent's draft PR never appears | Workflows on agent PRs require human approval — click **Approve and run** on the queued workflow. To remove this gate per repo, see [Allowing GitHub Actions workflows to run automatically when Copilot pushes](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings#allowing-github-actions-workflows-to-run-automatically-when-copilot-pushes) (read the warning first). |
| Coding Agent's PR contains a security or quality issue of its own | The built-in **validation tools** should have caught and re-fixed it. If you've disabled them (**Settings → Copilot → Cloud agent → Validation tools**), re-enable per [the docs](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings#enabling-or-disabling-built-in-code-quality-and-security-validation-tools), then `@copilot` to retry. |
| Dependabot PR fails CI | Bump is incompatible. Cancel the Dependabot PR and assign the **alert** (not the PR) to Copilot instead so it can update call sites. |
| Secret Scanning alerts won't go away after delete | Secrets stay in history. Rewrite history (`git filter-repo`) or dismiss with reason "Revoked" after rotation. |
| Code Quality finding stays Open after merge | Code Quality re-runs on the next push to the default branch. Wait for the next scan, then refresh the Security tab. |
| The agent fixed the symptom but missed the root cause | Iterate on the PR with a precise `@copilot` comment that points to the root cause; the agent re-runs on PR comments. |

> [!TIP]
> Always re-run the test suite locally (`npm test` at the repo root) before merging an AI fix — whether Autofix or Coding Agent. A green CodeQL re-scan confirms the analyzer no longer flags the location — not that behavior is preserved.

## 🎁 Stretch goals

- **Diff the two options.** Pick a fresh CodeQL alert on a sandbox repo and resolve it **twice** — once with Autofix, once with the Coding Agent — in two separate PRs. Compare the diffs, test coverage, and time-to-merge. Which one would you ship?
- Build a **Security Campaign** from any remaining alerts and bulk-assign them to Copilot.
- Create a `security` custom agent (`/create-agent`) specializing in OWASP Top 10 fixes and reference it from future remediation issues; the Coding Agent will pick it up automatically when it runs.
- Run a "quality sprint": filter Code Quality findings older than 30 days and bulk-assign to Copilot.
- Try **Copilot Autofix** as a side-by-side comparison on a fresh Code Scanning alert — same fix, different surface; note which is faster and which produces a better diff for your codebase.

---

🎉 **Congratulations — you finished the workshop!** Return to the [README](./README.md) for the overall summary, or open the [Facilitator Guide](./FACILITATOR.md) if you're running it for others.
