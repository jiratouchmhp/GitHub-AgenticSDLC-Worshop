# Facilitator Guide — End-to-End Agentic SDLC on GitHub

This document is for the **workshop delivery lead**. It contains talking points, timing checkpoints, common attendee questions, and a "demo recovery" playbook.

## 👤 Audience profile (assumed)

- Working developers who already use Copilot for autocomplete and chat in VS Code.
- Mostly unfamiliar with: Copilot **Plan Mode**, **custom agents**, **agent skills**, **Coding Agent**, PR-level **Code Review**, **Autofix**, **rulesets**, and the full GHAS toolset.
- May span Node and Python — be ready to support both equally.

## ⏰ Master timing (180 min)

> Durations only — start whenever; pacing assumes ~180 min total.

| Duration | Activity | Facilitator focus |
|---|---|---|
| 10 min | Welcome, intros, prereq check | Verify everyone has a Copilot Business seat + IDE working; pair up |
| 15 min | **Lab 0** (custom agents + skills) | Demo `/create-agent` and `/create-skill` live |
| 35 min | Lab 1 (Plan Mode → full-stack MVP via `app-builder`) | The "wow" moment: `/plan`, then manually switch the dropdown to `app-builder`. Resist debugging individual stacks; rely on the custom-agent + skill combos |
| 25 min | Lab 2 (push to GitHub, ruleset) | Signed-commits is the #1 trip hazard — preempt with "merge via web UI" tip. Lab 2 is now a lean 3-step lab; lean discussion time on Rulesets vs branch protection |
| 15 min | Break | Reset the room |
| 35 min | Lab 3 (Coding Agent + Code Review) | Launch issues early — agent runs take 3–7 min |
| 35 min | Lab 4 (GHAS + Code Quality) | The vulnerability seeding is now agent-driven; eyeball each attendee's prompt outputs |
| 35 min | Lab 5 (remediation) | Wait for CodeQL re-runs together; use the time for discussion |
| 5 min  | Wrap-up + Q&A | Hit the agentic SDLC loop diagram, then open Q&A |

> [!TIP]
> Most cuttable segments if you're running tight: Lab 0 Step 6 (verification — quick demo only), Lab 2 stretch goals, Lab 5 Step 4b (alternative path).

## 🎤 Section-by-section talking points

### Opening (10 min)

- **Frame the day:** "Most of you have used Copilot like a smart autocomplete. Today we'll show you Copilot as an **asynchronous teammate** — with **Plan Mode** to think first, **custom agents** to specialize, **skills** to teach domain knowledge — all inside GitHub's governance: branch protections, code review, security scanning, audit logs."
- **Set the expectation:** "We won't write much code by hand. We'll write **plans**, **prompts**, and **issues**. The quality of those is the new craft. The labs give you **prompt templates** — you adapt the bracketed values to your stack."

### Lab 0 talking points

- **`/create-agent` is the moment custom agents click.** Demo it live with `app-builder`. Show the resulting `.agent.md` frontmatter and explain `tools` and `model`.
- **`/create-skill` is on-demand context.** Stress the three-level loading: name+description first, body when relevant, resources on reference. Skills keep context lean.
- **Why workspace-level** (`.github/agents`, `.github/skills`)? They travel with the repo; the Coding Agent picks them up in the cloud too (Lab 3 and 5 reuse them).

### Lab 1 talking points

- **Plan Mode is the starting point.** Walk through the `/plan` prompt line by line — the more constrained, the better the plan. Then demo the manual switch of the agents dropdown from **Plan → `app-builder`** — that's the handoff in this workshop (no handoff buttons configured).
- The **`backend`** skill loading when the agent edits routes, and the **`frontend`** skill loading when it edits the UI, is the most visible payoff of Lab 0. Call it out when it happens.
- Inline chat in **Agent Mode** is the right tool for tightening validation — don't switch to Edit mode for this.

### Lab 2 talking points

- Lab 2 is intentionally short — just push the local repo and stand up the `protect-main` ruleset. Hygiene-file generation + Coding Agent enablement happen up front in Lab 3.
- The **signed-commits rule** is the single biggest cause of confusion. Tell people upfront: "If you don't have GPG/SSH signing configured, merge PRs via the web UI."
- Spend time on **Rulesets vs branch protection** — new for many attendees.
- Highlight that the custom agents + skills are now **part of the repo** and visible to the Coding Agent once Lab 3 enables it.

### Lab 3 talking points

- **Step 1 is doing double duty:** enable the Coding Agent AND bootstrap README + CONTRIBUTING + architecture + `.github/copilot-instructions.md` in one PR via `doc-agent`. Don't skip it — the parallel agents in Step 3 rely on `copilot-instructions.md`.
- The **prompts are the lesson, not the output.** Walk through Issue #2 line by line: negative constraints, acceptance criteria as checkboxes, custom-agent reference.
- Fill agent wait time with: agent **session-view tour**, comparison of **Agent Mode in IDE** (sync) vs **Coding Agent** (async, in the cloud).
- Clarify: the agent reads the issue **at start time**. Future edits to the issue body are ignored — iterate on the PR.

### Lab 4 talking points

- Frame the lab as **four pillars**: dependencies, secrets, security, and quality.
- Make the **Code Security vs Code Quality** distinction explicit — same engine, different rule sets, different bots, different dashboards. Code Quality needs **no GHAS or Copilot license**.
- Note Code Quality is **public preview** — re-run Step 4 on the morning of in case UI shifted.
- **Vulnerability seeding is prompt-driven now.** Attendees ask `app-builder` to generate the vulnerable code — emphasize *why* this is preferable to copy-pasting examples (you learn what a "deliberately bad prompt" looks like).
- When push protection blocks the test secret, **slow down**. Show the error, show the bypass UX, explain the audit trail.

### Lab 5 talking points

- Autofix is **available to all GHAS customers, no Copilot license required**.
- Stress **"Always review the diff"** — the governance gate going forward.
- The **assigning-an-alert-to-Copilot** flow is the newest piece. Demo it.
- End on the **agentic SDLC loop** diagram and the framing "AI is a force multiplier on good engineering hygiene."

## ❓ Frequently asked questions

**Q: What's the difference between Plan Mode, Agent Mode, and a custom agent?**
A: **Plan Mode** is a built-in agent that *researches and plans* without modifying code — uses read-only tools by default. **Agent Mode** is the built-in implementation agent — full tool access. A **custom agent** is your own persona (`.agent.md`) layered on top of Agent Mode with a specific role, tool list, model, instructions, and optional handoffs (this workshop uses manual dropdown switches instead).

**Q: Why create skills vs custom agents?**
A: **Custom agents** are personas (a role with tools and instructions). **Skills** are reusable capabilities (knowledge + scripts + examples) loaded on demand by *any* agent. Use custom agents for "who" should act; use skills for "what they should know when relevant." Skills also work across VS Code, Copilot CLI, and the cloud Coding Agent.

**Q: Does Copilot Coding Agent train on my code?**
A: No. Business and Enterprise customer data is contractually excluded from training. Autofix and Code Review data is also not used for training.

**Q: How much does an agent run cost?**
A: Copilot premium requests (your seat's monthly allowance, then per-request after) **and** GitHub Actions minutes.

**Q: Does Copilot Code Review replace human reviewers?**
A: No. Treat it as a tireless first-pass reviewer. CODEOWNERS humans still required by the ruleset.

**Q: Why doesn't the Coding Agent see my new issue comment?**
A: It reads the issue at start time only. Iterate on the PR.

**Q: How is "Copilot Code Review" different from "Code Review – Uncommitted Changes"?**
A: Two surfaces, same product family. Uncommitted Changes runs in VS Code on the local diff before push. PR-level Code Review runs on github.com after push. Both read `.github/copilot-instructions.md`.

**Q: What if the org has no GHAS license?**
A: All security features are free on **public** repos. Demo on a public fork or skip Labs 4–5 hands-on.

**Q: Do I need GHAS or Copilot to use GitHub Code Quality?**
A: No. Public preview, free, consumes Actions minutes. Available on GitHub Team and Enterprise Cloud for organization-owned repos.

**Q: Will Code Quality block our PRs?**
A: Only if you make it a required status check via a ruleset. Default is advisory.

**Q: Can custom agents be shared at the org level?**
A: Yes — see GitHub's docs on org-level custom agents. Enable `github.copilot.chat.organizationCustomAgents.enabled` in VS Code.

## 🚨 Demo recovery playbook

### "Plan Mode isn't in the agents dropdown"

1. Update VS Code (1.95+) and the Copilot Chat extension.
2. Sign out + back in to refresh the agent catalog.
3. Backup: skip the Plan-Mode demo, write the plan into Agent Mode as a prose preamble, then jump straight to `/create-agent`.

### "`/create-agent` or `/create-skill` is unknown"

1. Update VS Code + Copilot Chat.
2. Confirm in command palette: **Chat: New Custom Agent** and **Chat: New Skill** are present.
3. Backup: create the files by hand from the structure described in the Lab 0 prompt templates.

### "My Copilot Chat won't respond"

1. Click the Copilot icon in the status bar → sign-in status.
2. Reload window: `Developer: Reload Window`.
3. Check <https://www.githubstatus.com/>. If Copilot is degraded, switch to a backup recording.

### "The Coding Agent never starts"

Most common causes, in order:
1. **Org policy not enabled.** Have the host org-owner enable **Copilot coding agent**.
2. **Repo not in the allowlist.** Set **Org → Copilot → Coding agent → All repositories**.
3. **Actions disabled.** Repo → **Settings → Actions → General**.
4. **No premium requests left** on the assigning user's account.

Backup plan: pre-recorded screencast of an agent run (5–7 min).

### "Push protection didn't fire on the fake secret"

Use exactly `AKIAIOSFODNN7EXAMPLE` + `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` and a `ghp_…` 40-char PAT. Plain "password" strings won't trigger.

### "CodeQL workflow won't finish"

1. **Actions** → check for pending approval.
2. Confirm Actions billing limits aren't exhausted.
3. Fallback: `gh codeql upload-results --sarif=...` from a pre-built SARIF.

### "Autofix isn't generating a suggestion"

- Confirm CodeQL Default setup is the source.
- Confirm the rule has the "autofix available" indicator.
- Backup: assign the alert to the Coding Agent.

### "Code Quality won't enable / no findings appear"

- Confirm Team/Enterprise Cloud and enterprise hasn't disabled Code Quality.
- Confirm Actions enabled on the repo.
- Confirm a supported language is present.
- If queued waiting for approval, click **Approve and run**.

### "Signed-commits rule is blocking the whole room"

- Quickest unblock: merge via the GitHub web UI → **Squash and merge**.
- Alternative: add yourself as a **bypass actor** for the duration, remove afterwards.

### "Internet is flaky"

- Pre-clone a "complete" reference repo.
- All labs have screenshot-ready states you can narrate.

## 🔑 Pre-workshop checklist (24 hours before)

- [ ] Confirm a shared workshop GitHub org exists; you are an owner.
- [ ] Org-level Copilot policies set: **Coding Agent**, **Code Review**, **MCP servers** — all Enabled.
- [ ] **Plan Mode** visible in the agents dropdown on your build of VS Code.
- [ ] **`/create-agent`** and **`/create-skill`** slash commands work.
- [ ] Coding-agent repo allowlist: **All repositories** for the workshop org.
- [ ] GHAS licenses available for at least one private repo per attendee (or plan to use public repos).
- [ ] GitHub Code Quality confirmed allowed at the enterprise level. Pre-enable on a reference repo for fallback demos.
- [ ] Workshop attendees added as members with **write** access.
- [ ] **Run all 6 labs yourself end-to-end on the morning of** — feature flags change.
- [ ] Fallback recording of Lab 3's Coding Agent run ready.
- [ ] Fallback reference repo at the end-of-Lab-2 state ready (includes the three custom agents + two skills already on `main`).
- [ ] Confirm a public-repo option for any attendee without GHAS.
- [ ] Pair attendees so everyone has a reviewer for the required-reviews ruleset.

## 🧰 Reference links to keep open in tabs

- Planning with agents — <https://code.visualstudio.com/docs/copilot/agents/planning>
- Custom agents — <https://code.visualstudio.com/docs/copilot/customization/custom-agents>
- Agent Skills — <https://code.visualstudio.com/docs/copilot/customization/agent-skills>
- Coding Agent — <https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent>
- Coding Agent setup — <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment>
- Copilot Code Review — <https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review>
- Copilot Autofix — <https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning>
- Rulesets — <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets>
- GHAS overview — <https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security>
- CodeQL Default vs Advanced — <https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning>
- Code Quality overview — <https://docs.github.com/en/code-security/concepts/about-code-quality>
- Code Quality enablement — <https://docs.github.com/en/code-security/how-tos/maintain-quality-code/enable-code-quality>

## 💬 Closing words to attendees

> "What you saw today wasn't a demo of a futuristic AI — it was a demo of how the SDLC is changing **right now**. Notice how often the answer was 'write a better plan,' 'write a better prompt,' 'set better guardrails,' 'review the diff.' The discipline didn't go away — it moved up the stack. Your craft is now: **defining the work, defining the agents that do the work, and reviewing the output**. The labs you ran today are good starting templates. Bring them to your team, tune them to your conventions, and start with one low-risk repo."

🚀 *Have a great workshop!*
