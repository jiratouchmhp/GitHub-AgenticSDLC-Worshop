# Lab 3 — Coding Agent × 3: Create + Delegate Issues One-by-One via the GitHub MCP Server

> **Estimated time:** 35 minutes
> **Surface:** VS Code (Plan Mode + Copilot Chat with the GitHub MCP Server) + GitHub.com (PRs)

This is the lab where the cloud Coding Agent earns its keep. You'll use **Plan Mode** to draft *three independent task briefs*, then use the **GitHub MCP Server** inside Copilot Chat to **create each issue one at a time and delegate it to the Coding Agent** in a deliberate two-step rhythm. The three agents still run in **parallel in the cloud** once delegated — only the *create + delegate* loop is serial.

## 🎯 Learning objectives

1. Use **Plan Mode** to draft **three issue-ready briefs** with disjoint file scope (docs / tests / feature).
2. Use the **GitHub MCP Server** (`#create_issue`) to open each issue from Copilot Chat — one at a time — then **delegate** each to the **Coding Agent** via one of three surfaces: the `/delegate` slash command, the chat-mode picker's **Cloud** option, or github.com's **Assignees → Copilot**.
3. Watch the agent boot three ephemeral Actions-powered environments, open three draft PRs, and push commits in parallel.
4. Inspect the **three agent session views** side-by-side.
5. Iterate by **commenting on each PR** (the agent re-runs on `@copilot` mentions and PR reviews).
6. Add **Copilot Code Review** (PR-level) to all three PRs and read its comments.
7. Merge the three PRs in dependency order.

## ✅ Prerequisites

- Labs 0–2 complete (custom agents + skills committed, repo pushed, `protect-main` ruleset live).
- The three custom agents (`app-builder`, `doc-agent`, `qa-agent`) and the three skills (`frontend`, `backend`, `docs`) are all on `main`.
- **GitHub MCP Server (remote)** configured per [PREREQUISITES.md §9](./PREREQUISITES.md#9-github-mcp-server-remote--used-in-lab-3). Verify in Copilot Chat that typing `#create_issue` autocompletes.

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| Copilot **Coding Agent** — about | <https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent> |
| Coding Agent — quickstart | <https://docs.github.com/en/copilot/get-started/coding-agent> |
| Coding Agent — customize the environment | <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment> |
| Coding Agent — configuring agent settings (validation tools, firewall, MCP) | <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings> |
| Coding Agent — best practices | <https://docs.github.com/en/copilot/get-started/best-practices-for-using-copilot-to-work-on-tasks> |
| `.github/copilot-instructions.md` | <https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions> |
| Copilot **Code Review** (PR-level) | <https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review> |
| Configuring Copilot Code Review | <https://docs.github.com/en/copilot/how-tos/use-copilot-for-common-tasks/use-copilot-code-review/configuring-copilot-code-review> |
| **GitHub MCP Server** README | <https://github.com/github/github-mcp-server> |
| GitHub MCP Server — remote server docs | <https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md> |
| Using MCP servers in VS Code | <https://code.visualstudio.com/docs/copilot/chat/mcp-servers> |
| Plan Mode (VS Code) | <https://code.visualstudio.com/docs/copilot/agents/planning> |

> [!NOTE]
> Lab 3 starts by enabling the cloud Coding Agent for your repo and bootstrapping `.github/copilot-instructions.md` + the README/CONTRIBUTING/architecture baseline that the agent will rely on. **Do Step 1 first** — every subsequent step assumes those files are merged into `main`.

> [!IMPORTANT]
> **What the Coding Agent can and cannot do** (May 2026):
> - ✅ Works asynchronously in a GitHub Actions runner; you can close your laptop.
> - ✅ Opens a draft PR per issue on a branch named `copilot/fix-<issue#>`.
> - ✅ Reads the issue title + body + comments **at start time**, plus `.github/copilot-instructions.md`, plus any repository custom agents/skills it deems relevant.
> - ✅ **Multiple issues assigned to Copilot run in parallel** — each gets its own runner and its own PR.
> - ✅ Iterates when you `@copilot` a PR comment or leave a PR review.
> - ✅ Branch protections still apply to its PRs.
> - ⚠️ Workflows on agent PRs require human approval before they run.
> - ⚠️ Best for low-to-medium complexity tasks in well-tested codebases.
> - ⚠️ Each run consumes Copilot premium requests and GitHub Actions minutes — multiply by 3 for this lab.

---

## Step 1 — Configure the Coding Agent and bootstrap the docs baseline (8 min)

> 📖 **Reference for this step:** [About Copilot Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent) · [Grant Coding Agent access to your repository](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/grant-coding-agent-access-to-your-repository) · [Configuring agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings) · [Customize the agent environment](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment) · [`.github/copilot-instructions.md`](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)

The Copilot Coding Agent runs in a GitHub-hosted ephemeral environment when you assign it an issue. Before fanning out work in Step 3, enable it and seed the docs/instructions it will read.

### 1a. Verify enablement

Go to **Settings → Code & automation → Copilot → Coding agent**. Confirm this repo is enabled.

> [!IMPORTANT]
> If the section is missing or controls are greyed out, the org policy is off. An org owner must enable **Copilot coding agent** in org policies and add this repo to the allowlist. See [Granting access to the coding agent](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/grant-coding-agent-access-to-your-repository).

### 1b. Bootstrap docs baseline + `copilot-instructions.md` with `doc-agent`

Switch to the **`doc-agent`** custom agent. The `docs` skill should auto-load as soon as the agent starts writing under `docs/**`, `architecture/**`, or `README.md`.

> **Prompt template (Agent Mode → `doc-agent`):**
> ```
> Create the docs baseline + Coding Agent instructions for the Store
> app. Do NOT modify application source under client/ or server/.
>
> 1. README.md — follow the section order from the `docs` skill:
>    title ("store-app"), one-paragraph summary (small e-commerce MVP:
>    browse products, add to cart, update quantity, remove; SQLite
>    in-memory; single-page React UI calling an Express JSON API),
>    badge placeholders, Features, Quickstart (`npm install` then
>    `npm run dev` → open http://localhost:5173), API reference table
>    (Method, Path, Description, Success status, Error statuses) grouped
>    by Products and Cart resources, Shopping flow walkthrough
>    (ProductList → ProductCard → Add to cart → Cart → CartItem qty
>    stepper → Remove), Architecture section with the mermaid diagram
>    from #2 inlined, Error model ({"error":"..."}), Project layout
>    (file tree showing client/ and server/ workspaces), Contributing
>    link, License link. Use real example values such as the product
>    "Wireless Headphones" (id "sku_headphones_001", price_cents 12900).
>    Document the integer-cents money convention at the top of the API
>    reference.
> 2. architecture/overview.md — prose overview + a mermaid `flowchart LR`
>    diagram with three nodes (React UI in browser → Express /api/* on
>    :3001 → SQLite :memory: with products + cart_items tables) plus a
>    note about the Vite dev proxy on :5173 AND a note that the products
>    catalog is re-seeded on every server start.
> 3. architecture/decisions/0001-stack-choice.md — ADR per the template
>    in the `docs` skill (Status: Accepted, Context: workshop scope and
>    e-commerce MVP demo, Decision: Vite+React+Tailwind+Express+better-
>    sqlite3 with in-memory products + cart_items, Consequences,
>    References).
> 4. CONTRIBUTING.md — trunk-based branching, Conventional Commits,
>    1 reviewer + Copilot Code Review expectation, how to run both test
>    suites (`npm test` at the root runs server then client; or
>    `npm -w server test` / `npm -w client test` individually).
> 5. .github/copilot-instructions.md — repository-wide instructions
>    for the Copilot Coding Agent and the local Copilot Code Review.
>    Cover:
>    - Project intent: full-stack Store app (e-commerce MVP), SQLite
>      in-memory with products + cart_items tables, single-page React
>      UI calling an Express JSON API.
>    - Stack: Node.js 20 LTS, npm workspaces (client/, server/),
>      Vite 5 + React 18 + TypeScript + TailwindCSS 3 in client/,
>      Express 4 (ESM) + TypeScript strict + better-sqlite3 in server/.
>    - All API endpoints live under /api/* (Products + Cart routers).
>      Never break the prefix — the SPA fallback assumes it.
>    - Money is stored and transmitted as integer cents (price_cents).
>      Never use floats for money. Display formatting lives in
>      client/src/lib/format.ts.
>    - Always use parameterized better-sqlite3 prepared statements —
>      never string-concatenate user input into SQL.
>    - Always write tests for new endpoints (Jest + supertest under
>      server/) AND for new client behavior (Vitest + React Testing
>      Library under client/). Coverage target ≥80% lines / ≥75%
>      branches in BOTH packages.
>    - Use Conventional Commits. Keep PRs small and focused; one issue
>      → one PR. Where reasonable, keep server-only and client-only
>      changes in separate commits.
>    - Run BOTH `npm -w server test` AND `npm -w client test` (or the
>      root `npm test`) before opening a PR.
>    - Match the existing code style; do not reformat unrelated files.
>    - When fixing a security alert, link it in the PR description.
>    - The repo defines custom agents (.github/agents/) and skills
>      (.github/skills/) that the Coding Agent should respect when
>      relevant — docs work → doc-agent + docs skill; UI work →
>      frontend skill; API work → backend skill; tests → qa-agent.
>    Keep it under 80 lines.
>
> Use real, idiomatic content. Every curl example in the API reference
> must use `http://localhost:3001/api/...` and include `-H 'Content-Type:
> application/json'` on POST/PATCH.
> ```

Commit on a feature branch, open a PR, and merge via the web UI:

```bash
git checkout -b chore/docs-and-copilot-instructions
git add README.md CONTRIBUTING.md architecture/ .github/copilot-instructions.md
git commit -m "docs: bootstrap README, architecture, and copilot-instructions"
git push -u origin chore/docs-and-copilot-instructions
# Open + approve + squash-merge the PR via GitHub.com.
```

### 1c. (Optional) Pre-install dependencies for the agent

Ask `app-builder` to generate `.github/workflows/copilot-setup-steps.yml` that sets up Node 20, runs `npm ci` at the repo root (which installs both workspaces), and optionally pre-builds the client. Only the `copilot-setup-steps` job in that file is honored; everything else is ignored. Reference: [Customizing the agent environment](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment).

### 1d. (Optional) MCP servers / firewall

In **Settings → Copilot → Coding agent** you can register MCP servers (JSON config) and adjust the agent's firewall allowlist. Skip unless your environment requires it. Reference: [Configuring agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings).

> [!TIP]
> Lab 3 references a `copilot` label on issues. We did not create labels in Lab 2 — either create a purple `copilot` label now under **Issues → Labels**, or drop `copilot` from the `--label` flags in Step 3.

✅ **Checkpoint:** `.github/copilot-instructions.md`, `README.md`, `CONTRIBUTING.md`, and `architecture/` are all merged into `main`. The Coding Agent settings page shows this repo enabled.

---

## Step 2 — Use Plan Mode to draft three parallel issue briefs (6 min)

> 📖 **Reference for this step:** [Plan Mode in VS Code](https://code.visualstudio.com/docs/copilot/agents/planning) · [Best practices for using Copilot to work on tasks](https://docs.github.com/en/copilot/get-started/best-practices-for-using-copilot-to-work-on-tasks) · [About issues](https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues)

1. In VS Code, open Copilot Chat and switch the agents dropdown to **Plan**.
2. Submit the prompt below. Plan Mode will think through the work and emit three issue-ready briefs into session memory.

> **Prompt template (Plan Mode):**
> ```
> /plan I want to fan out THREE parallel issues to the Copilot Coding
> Agent. They must touch DISJOINT files so the resulting PRs do not
> conflict on merge. Produce three issue-ready briefs.
>
> Repo state: store-app monorepo (npm workspaces), client/ = Vite +
> React 18 + TS + TailwindCSS, server/ = Express 4 (ESM) + TS +
> better-sqlite3 (:memory:) with products + cart_items tables, products
> re-seeded on every server start, single anonymous in-memory cart per
> process, all endpoints under /api/* (Products + Cart routers).
> README + CONTRIBUTING + architecture/overview.md already exist
> (bootstrapped in Lab 3 Step 1) but are deliberately thin.
>
> The three issues:
> 1. DOCS — Expand README + author docs/api.md + enrich
>    architecture/overview.md so a new dev is productive in 10 min.
>    Cover both Products and Cart resources, the integer-cents money
>    convention, the products-seed-on-import behavior, and the
>    shopping flow walkthrough.
>    Honor the `doc-agent` agent persona and load the `docs` skill.
>    Touches: README.md, docs/**, architecture/**, CONTRIBUTING.md.
>    NEVER touches client/ or server/.
> 2. TESTS — Raise coverage to ≥80% lines / ≥75% branches in BOTH
>    server/ (Jest + supertest) AND client/ (Vitest + React Testing
>    Library). Must cover product list/get, cart add/update/remove
>    happy-path AND validation edges (unknown productId → 404,
>    quantity 0/negative/non-integer → 400, quantity > stock → 409,
>    unknown body fields → 400, merging duplicate productId).
>    Honor the `qa-agent` agent persona. Touches: server/**/*.test.ts,
>    client/**/*.test.tsx, jest/vitest config files, and the root
>    test-runner script ONLY. NEVER touches production source.
> 3. FEATURE — Add a category filter tab strip (All / Electronics /
>    Groceries / Books / Home) above the product grid in the React UI.
>    Pure client-side filtering of the existing /api/products response;
>    NO server change, NO schema change, NO new endpoint. Default tab
>    is "All". Selected tab persists across reloads via localStorage.
>    Tabs use proper ARIA roles (role="tablist" / role="tab" /
>    aria-selected). Honor the `app-builder` agent persona and load
>    the `frontend` skill. Touches: client/src/components/** (a new
>    CategoryFilter.tsx + minimal edit to ProductList.tsx or App.tsx
>    to slot it in) and the matching new *.test.tsx file ONLY.
>    NEVER touches server/, docs/, README, or architecture/.
>
> For EACH issue produce, in this exact structure:
>   ### Issue N — <title>
>   Title: <conventional-commit-prefixed title>
>   Labels: <comma-separated>
>   Body: a markdown block with `## Goal`, `## Deliverables`,
>         `## Constraints` (including "Do NOT touch <other 2 issues'
>         file scopes>"), and `## Acceptance criteria` as `- [ ]`
>         checkboxes. Reference the relevant custom agent + skill by
>         path. Use real example values (e.g., the product "Wireless
>         Headphones", id "sku_headphones_001", price_cents 12900).
>
> Save the three briefs to session memory so I can copy them straight
> into GitHub. Defer code generation — these are issue specs only.
> ```

3. Iterate with Plan Mode until each brief is sharp, file scopes are clearly disjoint, and the acceptance criteria are testable.

> [!TIP]
> Plan Mode writes its plan to `/memories/session/plan.md`. Run **Chat: Show Memory Files** to view it. Keep this Chat session open — you'll feed each brief into the MCP server in Step 3.

> [!NOTE]
> **All three briefs are drafted upfront, but you'll create + delegate them one at a time in Step 3.** The cloud runs the three agents in parallel once they're delegated — only the create-and-delegate loop is serial. This keeps you in control of each issue's title/body/labels and gives you a moment to double-check disjointness before assigning Copilot.

✅ **Checkpoint:** Session memory contains three issue briefs (docs / tests / feature) with no overlapping file scopes.

---

## Step 3 — Create + delegate each issue one at a time via the GitHub MCP Server (5 min)

> 📖 **Reference for this step:** [github/github-mcp-server README](https://github.com/github/github-mcp-server) · [Remote server documentation](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md) · [Use Coding Agent from VS Code (`Delegate to coding agent`)](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/use-coding-agent-in-org#using-the-coding-agent-from-vs-code) · [Quickstart for Coding Agent](https://docs.github.com/en/copilot/get-started/coding-agent)

You'll do the **same two-action loop three times** — once per brief from Step 2. The two actions are:

1. **Create the issue** by asking Copilot Chat to invoke the GitHub MCP tool `#create_issue`. You stay in chat; no terminal.
2. **Delegate to the Coding Agent.** Pick **any one** of the three surfaces below — they all produce the same draft PR. We list them in order of "stay-in-flow" first:
   - **Option A — `/delegate` slash command** in VS Code Copilot Chat (provided by the GitHub Pull Requests extension): `/delegate #<issue>` — the most ergonomic when you've just created the issue in the same chat.
   - **Option B — Switch the chat mode picker to *Cloud*** in the agents dropdown, then prompt: *"Pick up issue #<issue> and open a PR."* The Coding Agent handles it in the cloud.
   - **Option C — On github.com**, open the issue → **Assignees → Copilot**. The web-UI fallback; useful when you want to demonstrate the delegation flow without VS Code.

> [!TIP]
> Doing it one issue at a time (instead of fanning out in a single batch) gives you a deliberate review beat between each `#create_issue` call: skim the rendered issue, confirm title/body/labels look right, *then* delegate. If a brief looks off, you can edit the issue body **before** delegating — once Copilot is assigned, the agent reads the body at start time and ignores later edits.

> [!IMPORTANT]
> Make sure the **github** MCP server is selected in the chat tools picker (`🛠` icon above the input) **and** the `issues` toolset is enabled. If `#create_issue` doesn't autocomplete, go back to [PREREQUISITES §9](./PREREQUISITES.md#9-github-mcp-server-remote--used-in-lab-3).

### Step 3a — Issue 1: DOCS

**① Create the issue.** With your Step 2 brief #1 visible in chat (or in `/memories/session/plan.md`), submit:

> **Prompt template (Agent Mode, GitHub MCP enabled):**
> ```
> Use #create_issue on this repository (<ORG>/store-app) to open a new issue with:
>
> Title: docs: comprehensive README + docs/api.md + architecture refresh
> Labels: documentation, copilot
> Body: <paste Brief #1 from Step 2 verbatim — ## Goal / ## Deliverables /
>        ## Constraints / ## Acceptance criteria>
>
> Do NOT assign anyone yet. After creating, print the issue number and URL.
> ```

The MCP tool returns the new issue number (e.g., `#42`) and URL. Open it in the browser and skim it for 15 seconds.

**② Delegate to the Coding Agent.** Pick one:

> **Option A — `/delegate` slash command (VS Code Copilot Chat):**
> ```
> /delegate #42
> ```

> **Option B — Cloud mode (VS Code Copilot Chat):**
> Click the agents dropdown above the chat input → **Cloud**. Then submit:
> ```
> Pick up issue #42 in <ORG>/store-app and open a PR.
> ```

> **Option C — github.com web UI:**
> Open issue #42 → right sidebar → **Assignees → Copilot**.

Within ~30 seconds a 👀 reaction appears on the issue and a draft PR titled something like *"docs: comprehensive README + docs/api.md + architecture refresh"* opens on branch `copilot/fix-42`.

✅ **Checkpoint 3a:** Issue 1 has a 👀 reaction and a `copilot/fix-<n>` draft PR.

### Step 3b — Issue 2: TESTS

Repeat the same two-action loop with brief #2. You do **not** need to wait for PR-1 to finish — once it's delegated, that run is in the cloud.

> **Prompt template (Agent Mode, GitHub MCP enabled):**
> ```
> Use #create_issue on <ORG>/store-app:
>
> Title: test: raise coverage to ≥80%/≥75% in client and server
> Labels: enhancement, copilot
> Body: <paste Brief #2 from Step 2 verbatim>
>
> Do NOT assign anyone. Print the issue number and URL.
> ```

Then delegate via **any one** of Option A (`/delegate #<issue>`), Option B (switch to **Cloud** mode and prompt the agent), or Option C (github.com → Assignees → Copilot).

✅ **Checkpoint 3b:** Issue 2 has a 👀 reaction and a second `copilot/fix-<n>` draft PR.

### Step 3c — Issue 3: FEATURE

One more time with brief #3.

> **Prompt template (Agent Mode, GitHub MCP enabled):**
> ```
> Use #create_issue on <ORG>/store-app:
>
> Title: feat: add category filter tabs above the product grid
> Labels: enhancement, copilot
> Body: <paste Brief #3 from Step 2 verbatim>
>
> Do NOT assign anyone. Print the issue number and URL.
> ```

Then delegate via Option A, B, or C from above.

✅ **Checkpoint 3c:** Issue 3 has a 👀 reaction and a third `copilot/fix-<n>` draft PR.

> [!WARNING]
> **Do not modify any issue body after assigning Copilot** — the agent reads it at start time only. Use PR comments for any follow-up direction (Step 5).

> [!NOTE]
> **Outcome:** three issues exist, each assigned to Copilot, each with a 👀 reaction, each spawning a draft PR. Same end-state as a parallel fan-out — just authored one at a time with a deliberate review beat between each delegation.

✅ **Checkpoint:** Three issues exist, each assigned to Copilot. Within ~60 seconds of the last delegation, three draft PRs are open.

---

## Step 4 — Inspect the three agent sessions side-by-side (5 min)

> 📖 **Reference for this step:** [Viewing a Coding Agent session](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/track-coding-agent-sessions) · [About Coding Agent](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent)

In your browser, open all three draft PRs in adjacent tabs. On each one, click **"View session"** (or go to **Agents → Recent sessions** for a single overview).

For each session you'll see:

- **Plan** — the agent's breakdown of how it tackled the issue.
- **Actions** — file reads/writes, terminal commands, test runs.
- **Final summary** — what changed and any caveats.

> [!TIP]
> The plan is the moment of truth. If any of the three plans misreads your intent (e.g., the feature agent decides to touch `server/`), request a re-plan via a PR comment on that PR *before* letting the agent finish.

✅ **Checkpoint:** All three sessions show a plan + several executed steps. File scopes look disjoint — the docs PR touches `README.md` / `docs/**` / `architecture/**`, the tests PR touches only `*.test.*` files + configs, the feature PR touches only `client/src/components/**` + its new test file.

---

## Step 5 — Iterate on each PR (8 min)

> 📖 **Reference for this step:** [Iterating on a Coding Agent PR with `@copilot`](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/iterate-on-coding-agent-pull-requests) · [Reviewing proposed changes in a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-proposed-changes-in-a-pull-request)

When an agent flags its PR **Ready for review**, leave feedback **on the PR** (the agent does not see new issue comments after start).

Three equivalent feedback channels per PR:

**Option A — Inline `@copilot` comment** on a specific diff line.
**Option B — Pull request review** (Files changed → Review changes → Request changes).
**Option C — Conversation comment** that starts with `@copilot` and lists numbered asks.

Example follow-ups, one per PR:

> **Docs PR — PR comment:**
> ```
> @copilot
> 1. Every curl example must include `-H 'Content-Type: application/json'`
>    on POST/PATCH and use the /api prefix.
> 2. The Architecture section must inline the mermaid diagram (not just
>    link to architecture/overview.md).
> 3. Add a "Project layout" tree that shows BOTH client/ and server/
>    workspaces.
> 4. Document the integer-cents money convention prominently at the top
>    of the API reference (e.g., "price_cents: 12900 = $129.00").
> ```

> **Tests PR — PR comment:**
> ```
> @copilot
> 1. The optimistic-rollback path in the Cart component is missing a
>    test for the 409 "out of stock" case where the rejected request
>    has no parseable JSON body.
> 2. Add an explicit test that adding the same productId twice MERGES
>    quantities rather than creating a duplicate line.
> 3. Paste the final coverage % from BOTH packages in the PR description.
> ```

> **Feature PR — PR comment:**
> ```
> @copilot
> 1. The filter tabs need `role="tablist"` + `role="tab"` + `aria-selected`
>    for screen-reader users, plus keyboard arrow-key navigation between
>    tabs.
> 2. Persist the selected category in localStorage under the key
>    `store-app.categoryFilter` so it survives a reload. Default to "All"
>    if the stored value is unknown.
> 3. Add a Vitest test that switches from "All" to "Electronics" and
>    asserts only electronics products render.
> ```

Each agent reacts with 👀 and pushes additional commits to its branch.

> [!WARNING]
> Don't pile on new feature requests at this stage. For new features, open a new issue. Keep PR iterations focused on **correcting** what was delivered.

✅ **Checkpoint:** Each of the three agents has pushed at least one follow-up commit.

---

## Step 6 — Add Copilot Code Review as a reviewer on all three PRs (5 min)

> 📖 **Reference for this step:** [Using Copilot Code Review (PR-level)](https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review) · [Configuring Copilot Code Review](https://docs.github.com/en/copilot/how-tos/use-copilot-for-common-tasks/use-copilot-code-review/configuring-copilot-code-review) · [Code review limitations](https://docs.github.com/en/copilot/responsible-use-of-github-copilot-features/responsible-use-of-copilot-code-review)

On each PR page → **Reviewers → Copilot**. Wait ~30 seconds — Copilot Code Review posts inline comments on each.

What you'll typically see:
- Clearer error messages
- Missing edge-case handling
- Best-practice nits
- "Implement suggestion" buttons that can hand a change off to the Coding Agent

> [!NOTE]
> Copilot Code Review is a separate, purpose-built reviewer — not the same as the IDE "Code Review – Uncommitted Changes" you used in Lab 1. It also reads `.github/copilot-instructions.md`. Reference: [Configuring Copilot Code Review](https://docs.github.com/en/copilot/how-tos/use-copilot-for-common-tasks/use-copilot-code-review/configuring-copilot-code-review).

✅ **Checkpoint:** Inline review comments from Copilot are visible on all three PRs.

---

## Step 7 — Approve & merge in dependency order (5 min)

> 📖 **Reference for this step:** [Merging a pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request) · [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) · [Keeping a PR in sync with its base branch (`Update branch`)](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork)

Merge order matters when three PRs touch a shared repo:

1. **Tests PR first** — it adds the safety net the other two PRs benefit from.
2. **Feature PR second** — the docs PR can then describe the new category filter without re-rebasing.
3. **Docs PR last** — picks up the feature's existence in README + `docs/api.md`.

For each PR:

1. Approve (pair up if your ruleset needs a second human reviewer).
2. **Squash-merge via the web UI** (signed by `web-flow`).
3. The linked issue auto-closes.
4. Delete the agent's branch.

After each merge, the next PR may need to rebase — click **"Update branch"** on the GitHub UI; the Coding Agent re-runs its tests on the rebased branch.

Verify locally after all three are merged:

```bash
git pull origin main
npm test
npm run dev   # confirm the category filter works at http://localhost:5173
```

✅ **Checkpoint:** All three PRs merged. `main` now has rich docs, ≥80% test coverage across both packages, and the new category-filter UI above the product grid.

---

## 🧠 Prompt-engineering takeaways

The issue prompts in this lab share these traits:

| Pattern | Example |
|---|---|
| **Clear goal** | "Raise coverage to ≥80%" |
| **Explicit deliverables list** | Numbered list of files/sections |
| **Constraints (what NOT to do)** | "Do NOT modify client/" |
| **Acceptance criteria as checkboxes** | `- [ ] ...` — the agent self-verifies |
| **Reference the custom agent + skill** | "Honor the `qa-agent` agent persona; load the `docs` skill" |
| **Real example values** | "Wireless Headphones" / `sku_headphones_001`, not "foo" |
| **Parallelizable scope** | Each issue lists the file paths it owns *and* the paths it must not touch |

> [!TIP]
> Save your best prompts as **issue templates** in `.github/ISSUE_TEMPLATE/` so teammates can reuse them. The three briefs from this lab make excellent templates. Reference: [Configuring issue templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository).

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| `#create_issue` doesn't autocomplete in chat | GitHub MCP Server isn't loaded. Re-verify [PREREQUISITES §9](./PREREQUISITES.md#9-github-mcp-server-remote--used-in-lab-3) and confirm the `issues` toolset is enabled in the chat tools picker. |
| `/delegate` slash command not recognized | The **GitHub Pull Requests** extension provides it — install/update it (Lab prereq). Reload the VS Code window if the command still doesn't appear. |
| **Cloud** missing from the chat mode picker | Update VS Code and the GitHub Copilot Chat extension to the latest. Confirm the org policy for **Coding Agent** is enabled (Step 1a). |
| MCP created the issue but no agent picked it up | The two actions are intentionally separate. Pick **any one** of Option A (`/delegate`), Option B (Cloud mode), or Option C (github.com → Assignees → Copilot) to delegate. |
| "Copilot" missing from Assignees on github.com | Org/enterprise policy disabled, or repo not in the Coding Agent allowlist. |
| Only 1 of 3 agents starts | Confirm you actually delegated all three issues (look for 👀 reactions on each). Some orgs throttle parallel Coding Agent runs — check Actions concurrency limits. |
| Agent PR fails its first workflow | Workflows on agent PRs require human approval. Click **Approve and run**. |
| Two of the three PRs conflict on merge | The briefs' file scopes overlapped. Cancel one, refine the brief in Plan Mode to be disjoint, reopen via Step 3's two-action loop. |
| Agent ignored part of your instructions | Move them into `.github/copilot-instructions.md` or your custom agent body. |
| You want to redirect mid-flight | The agent re-reads only PR comments, not new issue comments. Comment on the PR. |
| Coding Agent didn't use the `qa-agent` / `doc-agent` custom agent | Reference the agent file by path in the issue body, or move the relevant guidance into `copilot-instructions.md`. |

## 🎁 Stretch goals

- Open a **fourth** parallel issue: *"refactor: split server routes into controllers + services"* and assign to Copilot. Notice it's more aggressive than docs/tests/feature — review carefully and merge last of all.
- Wire up a CI workflow that runs both test suites + reports coverage and make it a required status check for `main`.
- Add a `security` custom agent and explicitly invoke it in a future security-remediation issue (Lab 5 stretch).
- Convert the three Plan-Mode briefs into reusable `.github/ISSUE_TEMPLATE/parallel-{docs,tests,feature}.yml` files so a teammate can fan out a new round of parallel work with three clicks.

---

➡️ **Next: [Lab 4 — Enable & Assess Code Security + Code Quality](./lab-04-ghas-security.md)**
