# End-to-End Agentic SDLC on GitHub

A hands-on, 3-hour developer workshop that walks you through the complete agentic software development lifecycle using **GitHub Copilot in VS Code** (Plan Mode, Agent Mode, custom agents, skills), **GitHub Copilot Coding Agent**, **GitHub Copilot Code Review**, and **GitHub Advanced Security (GHAS)** — entirely on GitHub.

By the end of this workshop, you will have planned, scaffolded, documented, tested, secured, and remediated a small full-stack web app using AI agents at every stage of the SDLC.

> [!NOTE]
> This workshop assumes you are already comfortable using GitHub Copilot for inline completions and Copilot Chat in VS Code. We focus on the **newer agentic capabilities** (Plan Mode, custom agents, skills, Coding Agent, Code Review, Autofix) and how they connect into a cohesive workflow.

---

## 🧰 Prerequisites

> [!IMPORTANT]
> Before the workshop starts, complete every step in the **[Prerequisites & install guide](./PREREQUISITES.md)**. It contains the exact install commands for VS Code, Node.js 20 LTS, Git, the GitHub CLI, the Copilot extensions, and how to configure commit signing.
> **If any of these are missing, do it now — the labs build on each other.**

### Quick summary (full details in PREREQUISITES.md)

**Accounts & licenses**

- A **GitHub Enterprise Cloud** user account that is a member of the workshop organization
- An assigned **GitHub Copilot Business** (or Enterprise) seat — confirm at <https://github.com/settings/copilot>
- Org-level policies enabled by your admin: **Copilot Coding Agent**, **Copilot Code Review**, **MCP servers on GitHub.com**

**Local tooling**

- **VS Code** 1.121 or newer
- **GitHub Copilot** + **GitHub Copilot Chat** extensions (signed in)
- **GitHub Pull Requests** extension (`GitHub.vscode-pull-request-github`)
- **Git** 2.40+ configured with your name/email
- **Node.js 20 LTS** + **npm 10+**
- **GitHub CLI** (`gh`) 2.50+ authenticated to your org
- A code-signing setup for commits (GPG / SSH / GitHub web flow)

**Knowledge assumed**

- Comfortable cloning, branching, committing, and pushing with Git
- Have written or read REST API code before (Express, Fastify, Koa, or similar)
- Have built a small React component before (hooks + JSX); Tailwind familiarity is helpful but not required
- Have used GitHub Copilot inline suggestions and Copilot Chat in VS Code

> [!TIP]
> Run **`GitHub Copilot: Sign In Status`** from the command palette to confirm a Business/Enterprise seat. Then open the agents dropdown in Chat and confirm **Plan**, **Agent**, and **Edit** are all visible.

---

## ⏱️ Agenda & labs

The full ~180-minute flow at a glance. Each lab row links directly to its instructions.

| # | Duration | Lab | What you'll do |
|---|---|---|---|
| — | 10 min | *Welcome* | Intros, prereq check, stack walkthrough |
| 0 | 15 min | [**Lab 0 — Bootstrap: Custom Agents + Skills**](./lab-00-bootstrap-plan-agents.md) | Scaffold reusable agents and skills |
| 1 | 35 min | [**Lab 1 — Build the Full-Stack MVP with Custom Agents**](./lab-01-mvp-copilot-vscode.md) | Plan Mode → Agent Mode → local Code Review |
| 2 | 25 min | [**Lab 2 — Push to GitHub**](./lab-02-push-to-github.md) | Repo protection, signed commits, required reviews |
| — | 15 min | ☕ *Break* | — |
| 3 | 35 min | [**Lab 3 — Coding Agent for Docs & Unit Tests + Code Review**](./lab-03-coding-agent-docs-tests.md) | Cloud agent fans out 3 parallel PRs |
| 4 | 35 min | [**Lab 4 — Enable & Assess Code Security + Code Quality**](./lab-04-ghas-security.md) | Turn on GHAS and triage findings |
| 5 | 35 min | [**Lab 5 — Remediate with Copilot Autofix + Coding Agent**](./lab-05-remediate-autofix.md) | Fix Code Scanning, Code Quality & Dependabot alerts |
| — | 5 min  | *Wrap-up* | Full agentic SDLC loop discussion, Q&A |

> [!IMPORTANT]
> Timings are tight. The facilitator may move the break, shorten stretch goals, or run labs in parallel small groups. Stay on the happy path during the lab and explore stretch goals later.

For instructors: see the [Facilitator Guide](./FACILITATOR.md).

---

## 🧭 The agentic SDLC loop

The mental model the whole workshop is built on. Every capability you'll meet maps to one of these five phases.

```
        ┌──────────────────────────────────────────────┐
        │            PLAN                              │
        │  Copilot Plan Mode  +  GitHub Issues         │
        └───────────────────┬──────────────────────────┘
                            │ handoff
        ┌───────────────────▼──────────────────────────┐
        │            CODE                              │
        │  Agents (app-builder/doc-agent/qa-agent)     │
        │  Skills on demand (frontend/backend/docs)    │
        │  Copilot Coding Agent — N parallel PRs       │
        └───────────────────┬──────────────────────────┘
                            │
        ┌───────────────────▼──────────────────────────┐
        │            REVIEW                            │
        │  Copilot Code Review (local + on PRs)        │
        │  CODEOWNERS + human reviewers                │
        └───────────────────┬──────────────────────────┘
                            │
        ┌───────────────────▼──────────────────────────┐
        │         SECURE & QUALITY                     │
        │  CodeQL  •  Secret Scanning  •  Dependabot   │
        │  Push Protection  •  GitHub Code Quality     │
        └───────────────────┬──────────────────────────┘
                            │
        ┌───────────────────▼──────────────────────────┐
        │            REMEDIATE                         │
        │  Copilot Autofix on security & quality       │
        │  Coding Agent on complex/Dependabot fixes    │
        └───────────────────┬──────────────────────────┘
                            │
                            └──► back to PLAN
```

---

## 🎯 What you will learn

Grouped by the five SDLC phases from the loop above. Each item links to the lab that covers it end-to-end.

### 🗺️ Plan

- **Custom agents** — scaffold reusable roles (`app-builder`, `doc-agent`, `qa-agent`) via `/create-agent` · [Lab 0](./lab-00-bootstrap-plan-agents.md)
- **Agent Skills** — package specialized knowledge (`frontend`, `backend`, `docs`) via `/create-skill` · [Lab 0](./lab-00-bootstrap-plan-agents.md)
- **Plan Mode** — turn a one-line idea into a structured implementation plan via `/plan` · [Lab 1](./lab-01-mvp-copilot-vscode.md)

### ⚙️ Code

- **Agent Mode** — drive multi-file edits with custom agents in the loop · [Lab 1](./lab-01-mvp-copilot-vscode.md)
- **Generate Commit Message** — author conventional commits from staged changes · [Lab 1](./lab-01-mvp-copilot-vscode.md)
- **Copilot Coding Agent** — enable the cloud agent and seed `.github/copilot-instructions.md` · [Lab 3](./lab-03-coding-agent-docs-tests.md)
- **Coding Agent — parallel issues** — fan out 3 issues (docs + tests + feature) into 3 PRs · [Lab 3](./lab-03-coding-agent-docs-tests.md)

### 👀 Review

- **Copilot Code Review — Uncommitted Changes** — local pre-commit review pass · [Lab 1](./lab-01-mvp-copilot-vscode.md)
- **Branch protection & rulesets** — required reviews, signed commits, status checks · [Lab 2](./lab-02-push-to-github.md)
- **Copilot Code Review (PR-level)** — automated reviewer on the agent's PRs · [Lab 3](./lab-03-coding-agent-docs-tests.md)

### 🛡️ Secure & assess

- **CodeQL** — default-setup code scanning for vulnerabilities · [Lab 4](./lab-04-ghas-security.md)
- **Secret Scanning + Push Protection** — block secrets before they land · [Lab 4](./lab-04-ghas-security.md)
- **Dependabot** — vulnerable dependency alerts and updates · [Lab 4](./lab-04-ghas-security.md)
- **GitHub Code Quality** (public preview) — reliability & maintainability scanning · [Lab 4](./lab-04-ghas-security.md)

### 🔧 Remediate

- **Copilot Autofix** — one-click suggested fixes on Code Scanning & Code Quality alerts · [Lab 5](./lab-05-remediate-autofix.md)
- **Coding Agent remediation** — delegate complex fixes and Dependabot upgrades to the cloud agent · [Lab 5](./lab-05-remediate-autofix.md)

---

## 🛣️ The workshop stack

Everyone uses the same full-stack monorepo. No stack picking — every prompt template is concrete.

| Layer | Choice |
|---|---|
| **Backend** | Express 4 (ESM) + TypeScript (strict), in `server/` |
| **Database** | SQLite in-memory (`better-sqlite3`) — lives only for the process lifetime |
| **Frontend** | Vite 5 + React 18 + TypeScript + TailwindCSS 3, in `client/` |
| **Dev orchestration** | `concurrently` runs Express on `:3001` and Vite on `:5173`; Vite proxies `/api/*` to Express |
| **Production** | `npm run build` emits `client/dist`; Express serves it as static + SPA fallback |
| **API prefix** | All endpoints live under `/api/*` so the SPA fallback never collides with them |
| **Backend tests** | Jest + supertest |
| **Frontend tests** | Vitest + React Testing Library |
| **Workspaces** | npm workspaces at the repo root with `client/` and `server/` packages |

> [!NOTE]
> Yes, this stack is heavier than a single static page — that's intentional. A realistic React + Tailwind frontend and a separate Express backend let the Lab 3 demo show the Coding Agent fanning out **three parallel PRs across `client/` and `server/`** without merge conflicts.

---

## 📖 Official documentation index

Bookmark these — every lab links to a focused subset, but this is the master list:

**Copilot in VS Code**

- [Planning with agents (Plan Mode)](https://code.visualstudio.com/docs/copilot/agents/planning)
- [Agent Mode in VS Code](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode)
- [Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Copilot Code Review in the editor](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_code-review)

**Copilot on GitHub.com**

- [Copilot Coding Agent — about](https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent)
- [Coding Agent — customize the environment](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment)
- [Coding Agent — configuring agent settings](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings)
- [Copilot Code Review (PR-level)](https://docs.github.com/copilot/using-github-copilot/code-review/using-copilot-code-review)
- [Copilot Autofix for Code Scanning](https://docs.github.com/en/code-security/concepts/code-scanning/copilot-autofix-for-code-scanning)
- [Responsible use of Autofix](https://docs.github.com/en/code-security/responsible-use/responsible-use-autofix-code-scanning)

**GitHub Advanced Security & governance**

- [GHAS overview](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security)
- [Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [CodeQL Default vs Advanced setup](https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning)
- [Secret Scanning + Push Protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning)
- [Dependabot alerts & security updates](https://docs.github.com/en/code-security/dependabot/dependabot-alerts/about-dependabot-alerts)
- [GitHub Code Quality (overview)](https://docs.github.com/en/code-security/concepts/about-code-quality)
- [GitHub Code Quality (enablement)](https://docs.github.com/en/code-security/how-tos/maintain-quality-code/enable-code-quality)

---

## 📝 Conventions used in this workshop

- `> [!NOTE]`, `> [!TIP]`, `> [!WARNING]`, `> [!IMPORTANT]` are GitHub-flavored alert callouts.
- Blocks labeled **Prompt template** are meant to be typed into Copilot Chat (or pasted into an issue body) — **never into source files**. You adapt the placeholder values to your stack and project before submitting.
- ✅ **Checkpoint** lines tell you the exact state you should be in. If you don't see what's described, stop and ask the facilitator.
- We provide **prompt templates**, not example source code. The whole point is to let Copilot generate the code from a well-crafted prompt — and to learn what makes a good prompt.

Happy hacking! 🚀
