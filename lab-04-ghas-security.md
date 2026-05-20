# Lab 4 — Enable & Assess Code Security + Code Quality

> **Estimated time:** 35 minutes
> **Surface:** GitHub.com (Security tab) + local Git + your terminal

## 🎯 Learning objectives

By the end of this lab you will:

1. Turn on the four GHAS pillars on the repo: **Dependabot** (alerts + security updates), **Secret Protection** (Secret Scanning + Push Protection), **Code Scanning (CodeQL)** via **Default setup**, and **GitHub Code Quality** (public preview).
2. Understand the **Default vs Advanced** CodeQL setup choice and the **Code Security vs Code Quality** distinction.
3. Seed a small set of intentional findings on a `vuln/demo-findings` branch **using paste-ready snippets** (no AI is used to generate vulnerable code).
4. Open a PR and watch CodeQL + Code Quality annotate it; triage every category of finding in the **Security** tab.
5. **Hand one alert off to the Copilot Coding Agent** as the bridge into Lab 5 — Lab 5 walks the remediation end-to-end.

> [!IMPORTANT]
> **Why paste snippets instead of asking an agent to write the bad code?**
> The cloud Copilot Coding Agent (and the IDE Agent Mode you used in Lab 1) are intentionally aligned to refuse intentionally-vulnerable code generation — that's the point of a secure-by-default coding assistant. To exercise the scanners we add the bad code by hand from canonical, well-known workshop snippets, then let the agent do what it's good at: **fixing** the findings (Lab 5).

## ✅ Prerequisites

- Labs 0–3 complete; `main` has the MVP, custom agents, skills, and `.github/copilot-instructions.md`.
- The repo is **Private** (or Internal) and **owned by an organization with a GitHub Advanced Security license** (Code Security + Secret Protection).
- For Code Quality: an **organization-owned** repo on **GitHub Team or Enterprise Cloud**. Code Quality does **not** require a GHAS or Copilot license.

> [!IMPORTANT]
> If your org has no GHAS license, you can still complete the security portion against a **public** fork — Code Scanning, Secret Scanning, and Dependabot are free on public repos.

> [!NOTE]
> **GitHub Code Quality is in public preview** and not billed during preview (it does consume GitHub Actions minutes).

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| GHAS overview | <https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security> |
| **Dependabot alerts** | <https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts> |
| **Dependabot security updates** | <https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates> |
| **Secret Scanning** | <https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning> |
| **Push Protection** | <https://docs.github.com/en/code-security/secret-scanning/push-protection-for-repositories-and-organizations> |
| **Code Scanning** with CodeQL — Default setup | <https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning> |
| **Code Scanning** — Advanced setup | <https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning> |
| Supported CodeQL languages | <https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages> |
| **GitHub Code Quality** (overview) | <https://docs.github.com/en/code-security/concepts/about-code-quality> |
| **GitHub Code Quality** (enablement) | <https://docs.github.com/en/code-security/how-tos/maintain-quality-code/enable-code-quality> |
| Security campaigns | <https://docs.github.com/en/code-security/security-campaigns/managing-security-campaigns/about-security-campaigns> |

---

## Part A — Enable the four scanners (10 min)

> 📖 **Reference for this part:** [GHAS overview](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security) · [Configuring security features for your repository](https://docs.github.com/en/code-security/getting-started/securing-your-repository)

All four toggles live under **Settings → Security**. Knock them out in one pass so the first scans are already warming up while you seed the demo findings in Part B.

### A1. Dependabot alerts + security updates

> 📖 **Reference for this step:** [About Dependabot alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts) · [About Dependabot security updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates) · [About grouped security updates](https://docs.github.com/en/code-security/dependabot/dependabot-security-updates/about-dependabot-security-updates#grouping-dependabot-security-updates-into-one-pull-request)

1. **Settings → Security → Advanced Security**.
2. Under **Dependabot**, click **Enable** for:
   - **Dependabot alerts**
   - **Dependabot security updates**
   - **Grouped security updates** (optional)

✅ **Checkpoint:** Both Dependabot toggles show **Enabled**.

### A2. Secret Protection (Secret Scanning + Push Protection)

> 📖 **Reference for this step:** [About Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning) · [About Push Protection](https://docs.github.com/en/code-security/secret-scanning/push-protection-for-repositories-and-organizations) · [Bypassing push protection](https://docs.github.com/en/code-security/secret-scanning/working-with-push-protection/bypassing-push-protection-for-a-secret)

On the same Advanced Security page, click **Enable** next to **Secret Protection**. This turns on Secret Scanning **and** Push Protection in one move.

✅ **Checkpoint:** Both Secret Scanning and Push Protection show **Enabled**.

### A3. Code Scanning with CodeQL (Default setup)

> 📖 **Reference for this step:** [Configuring default setup for code scanning](https://docs.github.com/en/code-security/code-scanning/enabling-code-scanning/configuring-default-setup-for-code-scanning) · [Configuring advanced setup](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning) · [CodeQL query suites](https://docs.github.com/en/code-security/code-scanning/managing-your-code-scanning-configuration/codeql-query-suites) · [Supported CodeQL languages](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages)

1. Still in Advanced Security: find **CodeQL analysis** → **Set up → Default**.
2. Languages: auto-detected (expect `javascript-typescript`).
3. Query suite: **Default** (you can switch to `security-extended` later for more findings).
4. Events: leave defaults.
5. **Save changes**.

GitHub manages the CodeQL configuration behind the scenes — no workflow file lands in the repo. The first analysis kicks off immediately.

> [!NOTE]
> **Default vs Advanced setup:**
>
> | | Default setup | Advanced setup |
> |---|---|---|
> | Config | Managed by GitHub | `.github/workflows/codeql.yml` you own |
> | Best for | Most repos | Custom build steps, query packs, monorepos |
> | Switching | To advanced any time | Must disable default first |
> | Start here | ✅ | Adopt when you need customization |

✅ **Checkpoint:** A CodeQL run is in progress in **Actions**.

### A4. GitHub Code Quality (public preview)

> 📖 **Reference for this step:** [About Code Quality](https://docs.github.com/en/code-security/concepts/about-code-quality) · [Enable Code Quality](https://docs.github.com/en/code-security/how-tos/maintain-quality-code/enable-code-quality) · [Reviewing Code Quality findings](https://docs.github.com/en/code-security/how-tos/maintain-quality-code/review-code-quality-findings)

**Code Quality** complements Code Security: CodeQL hunts vulnerabilities (CWEs), Code Quality flags **reliability and maintainability** issues.

1. **Settings → Security → Code quality → Enable code quality**.
2. Review configuration (languages, runner type) → **Save**.

> [!IMPORTANT]
> If **Enable code quality** is missing, an enterprise owner must enable it under **Enterprise → Policies → Code security → GitHub Code Quality**.

| Surface | What you'll see |
|---|---|
| **Pull requests** | Inline comments from `github-code-quality[bot]` |
| **Security → Code quality → Standard findings** | Rule-based CodeQL quality findings |
| **Security → Code quality → AI findings** | AI-powered analysis of recent pushes |
| **Actions tab** | Each scan is a "Code Quality" workflow run |

**Code Security vs Code Quality at a glance:**

| | **Code Security** (CodeQL) | **Code Quality** (CodeQL + AI) |
|---|---|---|
| Goal | Find vulnerabilities | Find code-health issues |
| Licensing | GHAS / Code Security | Free during preview |
| Bot on PRs | `github-actions[bot]` | `github-code-quality[bot]` |
| Default-branch surface | Security → Code scanning | Security → Code quality |

✅ **Checkpoint:** All four scanners are enabled. The **Code quality** entry is visible in the Security sidebar; a "Code Quality" workflow has started in **Actions**.

---

## Part B — Seed intentional findings on `vuln/demo-findings` (10 min)

> 📖 **Reference for this part:** [`js/sql-injection` CodeQL query](https://codeql.github.com/codeql-query-help/javascript/js-sql-injection/) · [CWE-89: SQL Injection](https://cwe.mitre.org/data/definitions/89.html) · [Push Protection — supported secret patterns](https://docs.github.com/en/code-security/secret-scanning/introduction/supported-secret-scanning-patterns)

Now we add four findings to exercise each scanner: a vulnerable dependency, a SQL injection, a fake hardcoded secret, and a code-quality smell. **Every snippet below is paste-ready** — copy it into the indicated file, no Copilot needed.

Create and switch to the demo branch:

```bash
git checkout -b vuln/demo-findings
```

### B1. Seed a vulnerable dependency (Dependabot)

Open `server/package.json` and pin `lodash` to a known-vulnerable version under `dependencies`. Add the entry (or update it if `lodash` is already present):

```jsonc
// server/package.json — under "dependencies"
{
  "dependencies": {
    // NOTE: lodash 4.17.20 has a known prototype-pollution CVE.
    // Intentionally pinned for the workshop scanners — DO NOT use in production.
    "lodash": "4.17.20"
  }
}
```

Regenerate the lockfile from the repo root:

```bash
npm install
```

### B2. Seed two SQL injections (CodeQL) — one per Lab 5 option

Lab 5 walks **two supported remediation paths** for Code Scanning alerts — **Copilot Autofix** (one-click on the alert) and the **Copilot Coding Agent** (assign alert → draft PR). Seed **two separate** injectable routes so each path has its own alert and the two flows don't race on the same file.

#### B2a — `server/src/routes/search.ts` (will be fixed by the **Coding Agent** in Lab 5)

```ts
// server/src/routes/search.ts
// ⚠️ INTENTIONALLY VULNERABLE — DO NOT USE IN PRODUCTION ⚠️
// Used in the workshop to demonstrate the js/sql-injection CodeQL rule.
// Lab 5 (Option B) will remediate this file using the Copilot Coding Agent.

import { Router, Request, Response } from 'express';
import db from '../db.js';

const router = Router();

router.get('/', (req: Request, res: Response) => {
  const q = String(req.query.q ?? '');

  // BAD: user input concatenated directly into SQL — classic CWE-89.
  // The repo convention (see .github/skills/backend/SKILL.md) is to use
  // parameterized prepared statements; this route deliberately violates it.
  const sql = `SELECT id, name, price_cents, category, stock
               FROM products
               WHERE name LIKE '%${q}%'`;
  const rows = db.prepare(sql).all();

  res.json({ items: rows });
});

export default router;
```

#### B2b — `server/src/routes/lookup.ts` (will be fixed by **Copilot Autofix** in Lab 5)

```ts
// server/src/routes/lookup.ts
// ⚠️ INTENTIONALLY VULNERABLE — DO NOT USE IN PRODUCTION ⚠️
// Second seeded js/sql-injection. Lab 5 (Option A) will remediate this
// file using Copilot Autofix directly from the Code Scanning alert.

import { Router, Request, Response } from 'express';
import db from '../db.js';

const router = Router();

router.get('/:category', (req: Request, res: Response) => {
  const category = req.params.category ?? '';

  // BAD: route param concatenated directly into SQL — classic CWE-89.
  // Same rule (js/sql-injection) as search.ts but a different sink, so
  // CodeQL files this as a distinct alert that can be fixed independently.
  const sql = `SELECT id, name, price_cents, stock
               FROM products
               WHERE category = '${category}'`;
  const rows = db.prepare(sql).all();

  res.json({ items: rows });
});

export default router;
```

Wire **both** routes into `server/src/app.ts`. Find the section that registers `/api/products` and `/api/cart` and add the two lines:

```ts
// server/src/app.ts — add alongside the existing /api routers
import searchRouter from './routes/search.js';
import lookupRouter from './routes/lookup.js';
// ...
app.use('/api/search', searchRouter);
app.use('/api/lookup', lookupRouter);
```

### B3. Seed a fake hardcoded secret (Secret Scanning + Push Protection)

Create `.env.example` at the **repo root** (not under `server/` or `client/`) with exactly these canonical test values:

```dotenv
# .env.example
# ⚠️ INTENTIONALLY FAKE CREDENTIALS — for workshop Secret Scanning demo only.
# These are GitHub/AWS canonical test values. They are NOT real and grant no access.
# Push Protection will block the first push and require an explicit bypass.

AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
GITHUB_TOKEN=ghp_0123456789abcdefghijklmnopqrstuvwxyz
```

> [!WARNING]
> **Never paste real credentials.** Use only the canonical test values above. Push Protection blocks real keys — that's good — but it's best to never have them locally in the first place.

### B4. Seed a code-quality smell (Code Quality)

Create `server/src/utils/quality-smell.ts` with this paste-ready snippet — its mere presence on a scanned branch is enough; it does not need to be imported anywhere:

```ts
// server/src/utils/quality-smell.ts
// ⚠️ WORKSHOP-ONLY SMELL — intentional low-quality code for Code Quality scanners.
// Lab 5 will remediate this file using the Copilot Coding Agent.

export function smell(x: number): number {
  const unused = 'this variable is never read'; // unused local

  // Duplicate / useless condition
  if (x >= 0) {
    if (x >= 0) {
      return x * 2;
    }
  }

  return x;
  // Unreachable branch
  if (x < 0) {
    return -1;
  }
}
```

### B5. Commit and push — watch push protection fire

```bash
git add -A
git commit -m "chore: intentionally vulnerable demo code (workshop)"
git push -u origin vuln/demo-findings
```

You will see a push rejection because `.env.example` contains token patterns. Follow the link from the error message, choose a **bypass reason** (e.g., "Used in tests"), and push again:

```bash
git push
```

✅ **Checkpoint:** Push protection blocked the push, then accepted it after bypass. A Secret Scanning alert was created. The branch `vuln/demo-findings` is now on GitHub with all four seeded findings.

---

## Part C — Open the PR and let the scans complete (3 min)

> 📖 **Reference for this part:** [Approving workflow runs from public forks / first-time contributors](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks) · [Triaging code scanning alerts in pull requests](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests)

1. Open a PR from `vuln/demo-findings` → `main`.
2. Approve the CodeQL and Code Quality workflows when prompted (workflows on first-time branches require an explicit click).
3. Wait 2–3 minutes for both scans to complete.

✅ **Checkpoint:** Checks tab shows CodeQL + Code Quality complete; the PR's Conversation tab has annotations from both `github-actions[bot]` (Code Scanning) and `github-code-quality[bot]` (Code Quality).

---

## Part D — Assess the findings in the Security tab (8 min)

> 📖 **Reference for this part:** [About the Security tab](https://docs.github.com/en/code-security/getting-started/quickstart-for-securing-your-repository) · [Severity levels for security alerts](https://docs.github.com/en/code-security/security-advisories/working-with-repository-security-advisories/about-repository-security-advisories#about-cvss-levels)

Open the **Security** tab on the repo and walk all four panels.

### D1. Dependabot

> 📖 [Viewing and updating Dependabot alerts](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/viewing-and-updating-dependabot-alerts) · [About the CVE program](https://www.cve.org/) · [About GHSA IDs](https://docs.github.com/en/code-security/security-advisories/working-with-global-security-advisories-from-the-github-advisory-database/about-the-github-advisory-database)

**Security → Vulnerabilities** (under Dependabot). For your lodash alert read: **severity**, **CVE/GHSA**, **affected versions**, **patched version**, **dependency scope**, **manifest**. Dependabot may have already opened a fix PR — leave it open for now; Lab 5 walks the remediation.

### D2. Code Scanning

> 📖 [Managing code scanning alerts](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts) · [Showing data-flow paths in CodeQL](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-code-scanning-alerts#about-paths-in-code-scanning) · [`js/sql-injection` query help](https://codeql.github.com/codeql-query-help/javascript/js-sql-injection/)

**Security → Code scanning** → you should see **two** `js/sql-injection` alerts — one for `server/src/routes/search.ts` and one for `server/src/routes/lookup.ts`. Click each to confirm the **security severity**, the **rule id** (`js/sql-injection`), the **CWE-89** mapping, the **data-flow** view via **Show paths** (from `req.query.q` / `req.params.category` to the SQL sink), the **affected branches** list, and the rule's **help text**.

> [!NOTE]
> Two alerts is intentional. Lab 5 resolves one with **Copilot Autofix** (Option A) and the other with the **Copilot Coding Agent** (Option B) so you can compare both remediation paths side by side.

### D3. Secret Scanning

> 📖 [Managing Secret Scanning alerts](https://docs.github.com/en/code-security/secret-scanning/managing-alerts-from-secret-scanning) · [Audit log for push-protection bypasses](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/auditing-security-alerts)

**Security → Secret scanning** → see the AWS Access Key, AWS Secret, and PAT alerts. Your **push-protection bypass reason** is recorded for audit on each.

### D4. Code Quality

> 📖 [Reviewing Code Quality findings](https://docs.github.com/en/code-security/how-tos/maintain-quality-code/review-code-quality-findings) · [About Code Quality](https://docs.github.com/en/code-security/concepts/about-code-quality)

**Security → Code quality** has two sub-views:
- **Standard findings** — rule-based CodeQL quality results (the smell file should appear here).
- **AI findings** — AI-powered analysis of recent default-branch pushes.

Click into one finding and observe: rule, severity, location, and the suggested-fix preview.

### D5. PR view

Scroll the PR's **Conversation** tab — alerts appear inline. Two distinct bots post:
- `github-actions[bot]` — Code Scanning (security)
- `github-code-quality[bot]` — Code Quality

✅ **Checkpoint:** In each of the four panels (Dependabot, Code Scanning, Secret Scanning, Code Quality), you can identify at least one alert and explain its severity/category and location.

---

## Part E — Hand the `search.ts` alert off to the Coding Agent (4 min)

> 📖 **Reference for this part:** [Fixing a code-scanning alert with Copilot (assign-to-Copilot)](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests#fixing-a-code-scanning-alert-with-copilot) · [About Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent)

This is the bridge into Lab 5. You will assign **only the `search.ts` alert** to Copilot now (that becomes Lab 5's **Option B** demo). **Leave the `lookup.ts` alert open** — Lab 5's **Option A** uses it to demonstrate Copilot Autofix.

1. **Security → Code scanning** → click the `js/sql-injection` alert whose location is **`server/src/routes/search.ts`**.
2. In the right sidebar, **Assignees → Copilot**.
3. Within ~30 seconds you should see a 👀 reaction on the alert and a new draft PR titled something like *"Fix js/sql-injection in server/src/routes/search.ts"* appear in the **Pull requests** tab.

That draft PR is the Coding Agent's proposed fix. **Do not merge it yet** — Lab 5 walks the full review/iterate/merge loop.

> [!IMPORTANT]
> **Do not assign the `lookup.ts` alert to Copilot.** It must stay **Open** so Lab 5 Option A can demo Copilot Autofix on it from a clean state.

> [!NOTE]
> The agent reads the CodeQL alert (rule, message, location, data-flow paths, query help), the surrounding code, `.github/copilot-instructions.md`, and any matching custom agents/skills (so the `backend` skill's parameterized-query convention is in scope). The cloud Coding Agent is driven entirely by the alert — no extra prompt needed.

✅ **Checkpoint:** A draft PR from `copilot/fix-<n>` is open against `main`, linked to the **search.ts** SQL-injection alert. The **lookup.ts** SQL-injection alert is still **Open** with no assignee. You are ready for Lab 5.

---

## 🧠 Triage workflow refresher

```
                  ┌─────────────────────────────────────┐
                  │ Alert appears                       │
                  └────────────────┬────────────────────┘
                                   │
            ┌──────────────────────┼──────────────────────┐
            ▼                      ▼                      ▼
       True positive          False positive          Risk accepted
            │                      │                      │
    ┌───────▼────────┐    ┌────────▼─────────┐  ┌─────────▼────────┐
    │ Fix it:        │    │ Dismiss with     │  │ Dismiss with     │
    │ • Coding Agent │    │ reason "False    │  │ reason "Used in  │
    │ • Human PR     │    │  positive"       │  │  tests" / "Won't │
    │                │    │                  │  │  fix"            │
    └───────┬────────┘    └──────────────────┘  └──────────────────┘
            │
            └──► Verify rescan shows alert resolved
```

> [!TIP]
> For organizations: use **Security campaigns** to bulk-assign findings to teams (or to Copilot) against a deadline.

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| "CodeQL analysis" option missing | Repo language unsupported, or GHAS license not attached. |
| CodeQL workflow runs but reports no findings | Confirm `server/src/routes/search.ts` is on the pushed branch and wired into `app.ts`. Check the **Actions** log. |
| Push protection didn't block | Token format must match a known pattern — use the canonical test values in B3 verbatim. |
| Dependabot didn't open a PR | **Security → Dependabot → Configuration**: confirm "Security updates" is on. Check **Insights → Dependency graph → Dependabot** for errors. |
| Code Scanning workflow stuck on approval | **Settings → Actions → Fork pull request workflows** → click "Approve and run". |
| "Enable code quality" button missing | Enterprise owner must enable Code Quality. |
| No `github-code-quality[bot]` comments | Language unsupported, or PR doesn't target the default branch. |
| "Copilot" missing from the alert's Assignees menu | Org/enterprise policy for Coding Agent is off, or repo is not in the allowlist (see Lab 3 Step 1a). |

## 🎁 Stretch goals

- Switch CodeQL to **Advanced setup** + `security-extended` query suite. Compare findings count.
- Add a **Code scanning required status check** *and* a **Code Quality required status check** to the `protect-main` ruleset.
- Configure **delegated bypass** for push protection so an approver — not the pusher — decides.
- Add an **auto-triage rule** for Dependabot to auto-dismiss low dev-dep alerts.
- Explore the **org-level Code Quality dashboard** to compare scores across repos.

---

➡️ **Next: [Lab 5 — Remediate with Copilot Autofix + the Coding Agent](./lab-05-remediate-autofix.md)**
