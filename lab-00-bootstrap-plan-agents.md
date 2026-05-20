# Lab 0 — Bootstrap: Custom Agents + Skills

> **Estimated time:** 15 minutes
> **Surface:** VS Code + Copilot Chat (**Agent Mode**)

## 🎯 Learning objectives

By the end of this lab you will:

1. Generate three **custom agents** with `/create-agent`: `app-builder`, `doc-agent`, and `qa-agent`.
2. Generate three **Agent Skills** with `/create-skill`: `frontend`, `backend`, and `docs` — each loads on demand.
3. Verify all custom agents and skills are loaded and committed locally, ready for Lab 1 to use.

## ✅ Prerequisites

- All items in [PREREQUISITES.md](./PREREQUISITES.md) complete (VS Code 1.95+, Copilot extensions signed in, Node 20, Git, `gh` CLI)
- An empty folder for the workshop

```bash
mkdir store-app && cd store-app
git init -b main
code .
```

Open Copilot Chat (`Ctrl/Cmd + Alt + I`) and set the agents dropdown to **Agent**.

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| Plan / Agent / Edit modes in VS Code Chat | <https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode> |
| Authoring **custom agents** (`.agent.md`) | <https://code.visualstudio.com/docs/copilot/customization/custom-agents> |
| Authoring **Agent Skills** (`SKILL.md`) | <https://code.visualstudio.com/docs/copilot/customization/agent-skills> |
| Custom instructions overview | <https://code.visualstudio.com/docs/copilot/customization/custom-instructions> |
| `/create-agent` / `/create-skill` reference | <https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features> |

---

## Step 1 — Create the `app-builder` custom agent (3 min)

> 📖 **Reference for this step:** [Custom agents — authoring guide](https://code.visualstudio.com/docs/copilot/customization/custom-agents) · [Custom agent file format (`.agent.md`)](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-format) · [Choosing a model for an agent](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_choose-a-model)

You'll use the **`/create-agent`** slash command to scaffold a workspace-level custom agent. The command opens a guided dialog and writes the file under `.github/agents/`.

> **Prompt template (Agent Mode):**
> ```
> /create-agent
>
> Name: app-builder
> Description: Full-stack scaffolder for the Store app (a small e-commerce
> MVP: browse products, add to cart, update quantity, remove). Use to
> generate or modify the Express + better-sqlite3 backend under server/
> and the Vite + React 18 + TailwindCSS frontend under client/, wire
> SQLite in-memory persistence (products + cart_items tables), seed the
> products catalog on import, and add new endpoints end-to-end. The repo
> is an npm-workspaces monorepo with root scripts (dev/build/start/test).
> Tools: file editing, terminal, search, fetch.
> Model: leave default.
> Behaviors I want in the body:
> - Follow Conventional Commits when suggesting commit messages.
> - Always use parameterized SQL via better-sqlite3 prepared statements
>   — never string-concatenate user input.
> - All backend endpoints live under the /api prefix so they don't collide
>   with the SPA fallback. Product endpoints under /api/products, cart
>   endpoints under /api/cart.
> - Money is stored and transmitted as integer cents (price_cents). Never
>   use floats for money.
> - The cart is a single anonymous in-memory cart per process (no auth,
>   no user concept). On every server start the cart starts empty and
>   the products table is re-seeded from a fixed catalog in db.ts.
> - When working on files under client/**, load the "frontend" skill on
>   demand.
> - When working on files under server/**, load the "backend" skill on
>   demand.
> - In dev, expect Express on :3001 and Vite on :5173 with /api/* proxied
>   from Vite to Express. In prod, Express serves client/dist as static +
>   SPA fallback (registered AFTER /api routes, BEFORE error middleware).
> - Keep responses short and lead with file changes.
> Save as a workspace agent at .github/agents/app-builder.agent.md.
> ```

Review the file Copilot writes. **Keep** it.

✅ **Checkpoint:** `.github/agents/app-builder.agent.md` exists. Open the agents dropdown — **app-builder** appears in the list.

---

## Step 2 — Create the `doc-agent` custom agent (3 min)

> 📖 **Reference for this step:** [Custom agents — defining tool permissions](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_define-tool-permissions) · [Conventional Commits spec](https://www.conventionalcommits.org/)

> **Prompt template (Agent Mode):**
> ```
> /create-agent
>
> Name: doc-agent
> Description: Read-mostly documentation specialist for the Store app.
> One agent covering ALL written docs: README, architecture overview
> (with mermaid diagrams), endpoint reference tables, OpenAPI-style
> narratives, curl examples, ADRs under architecture/decisions/, and
> CONTRIBUTING. Use whenever docs need to be authored, updated, or
> kept in sync after a code change.
> Tools: search, fetch, editing restricted to README.md, CONTRIBUTING.md,
> docs/**, architecture/** (no source-code edits under client/ or server/).
> Behaviors in the body:
> - Mirror the existing README's writing style — concise, no marketing.
> - Use real example values (e.g., a product named "Wireless Headphones"
>   with id "sku_headphones_001" and price_cents 12900), never
>   "foo"/"bar".
> - All API examples use the /api prefix and the dev port 3001 (Express)
>   or 5173 (Vite, when calling through the dev proxy). Product endpoints
>   under /api/products, cart endpoints under /api/cart.
> - For request/response examples, use collapsible <details> blocks.
> - Always show prices as integer cents in JSON and call out the cents
>   convention in the API docs.
> - Cross-reference docs/api.md from README, and link architecture/
>   overview.md (with its mermaid client/server/DB diagram) from README.
> - When the topic warrants it, load the "docs" skill on demand for
>   section ordering, diagram templates, and ADR formatting.
> Save as a workspace agent at .github/agents/doc-agent.agent.md.
> ```

✅ **Checkpoint:** `.github/agents/doc-agent.agent.md` exists and shows up in the agents dropdown.

---

## Step 3 — Create the `qa-agent` custom agent (3 min)

> 📖 **Reference for this step:** [Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents) · [Jest — getting started](https://jestjs.io/docs/getting-started) · [supertest](https://github.com/ladjs/supertest) · [Vitest](https://vitest.dev/guide/) · [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)

> **Prompt template (Agent Mode):**
> ```
> /create-agent
>
> Name: qa-agent
> Description: Test author and reviewer for the Store app. Use whenever
> tests need to be written, extended, or evaluated for coverage. Also use
> for reviewing diffs for missing assertions or flaky patterns. The repo
> has two test runners that the agent must keep happy independently:
> Jest + supertest under server/, and Vitest + React Testing Library
> under client/.
> Tools: file editing, terminal (to run tests), search.
> Behaviors in the body:
> - Server tests: Jest + supertest hitting the exported Express `app`
>   (the app must be importable without starting a listener). Re-seed
>   the in-memory SQLite store between tests via beforeEach (clear the
>   cart_items table, re-run the products seed) so tests are order-
>   independent.
> - Client tests: Vitest + React Testing Library. Mock fetch (or use MSW)
>   for endpoint calls. Test optimistic-UI rollback paths explicitly
>   (Add-to-Cart rejected by server → line item disappears + StatusRegion
>   announces the {error} body).
> - Cover happy-path AND negative cases: product not found (404), invalid
>   quantity (0, negative, > stock), unknown body fields rejected with 400,
>   missing required fields, oversized name/description.
> - Confirm the SPA fallback returns 200 on "/" in production-mode tests
>   and serves an HTML document with the expected <title>.
> - Coverage target ≥80% lines and ≥75% branches; enforce in both
>   client/ and server/ configs. Root `npm test` runs both.
> Save as a workspace agent at .github/agents/qa-agent.agent.md.
> ```

✅ **Checkpoint:** Three custom agents are now visible in the agents dropdown: `app-builder`, `doc-agent`, `qa-agent`.

---

## Step 4 — Create the `frontend` skill (on-demand loaded) (2 min)

> 📖 **Reference for this step:** [Agent Skills — overview](https://code.visualstudio.com/docs/copilot/customization/agent-skills) · [Skill file format & progressive loading](https://code.visualstudio.com/docs/copilot/customization/agent-skills#_skill-structure) · [Writing effective skill descriptions](https://code.visualstudio.com/docs/copilot/customization/agent-skills#_authoring-tips) · [Vite — server proxy](https://vitejs.dev/config/server-options.html#server-proxy) · [Tailwind CSS install (PostCSS)](https://tailwindcss.com/docs/installation/using-postcss)

Skills package specialized knowledge that Copilot **loads only when relevant** (the three-level progressive loading: name+description → body → resources). They are reusable across agents and chat modes.

> **Prompt template (Agent Mode):**
> ```
> /create-skill
>
> Name: frontend
> Description: Use when adding or modifying the Store app web UI under
> client/**. Covers the Vite + React 18 + TypeScript + TailwindCSS 3
> conventions for this repo: component layout, a tiny typed fetch wrapper
> against /api/* paths, optimistic UI with rollback on error, accessible
> forms, and Tailwind utility-class grouping. Only load when the user is
> working on files under client/** or referencing React / Vite / Tailwind
> explicitly.
> Body should include:
> - The exact directory layout: client/src/components/{ProductList,
>   ProductCard,Cart,CartItem,StatusRegion}.tsx, client/src/lib/api.ts,
>   client/src/lib/format.ts (formatPriceCents helper), client/src/App.tsx,
>   client/src/main.tsx, client/index.html, client/tailwind.config.js,
>   client/postcss.config.js, client/vite.config.ts.
> - The Vite proxy config (server.proxy: { '/api': 'http://localhost:3001' })
>   so fetch('/api/products') works in dev.
> - The api.ts wrapper: typed request helper that sets Content-Type:
>   application/json on POST/PATCH, throws on non-2xx, and surfaces
>   the parsed { error: string } body to the caller. Typed methods:
>   listProducts(), getCart(), addToCart(productId, quantity),
>   updateCartItem(productId, quantity), removeCartItem(productId).
> - Money formatting: never display raw cents. Always format with
>   formatPriceCents(cents) → "$129.00" (USD, 2-decimal). Centralize in
>   client/src/lib/format.ts.
> - Accessibility checklist: <label> for every input (including the
>   quantity stepper), aria-live="polite" StatusRegion, focus returns to
>   the "Add to cart" button (or the qty input) after a successful action.
> - Optimistic UI pattern with rollback: append (or update qty / remove)
>   in local cart state immediately; on fetch rejection, revert state AND
>   surface the error message via StatusRegion (e.g., "Only 3 in stock").
> - ProductCard layout: image placeholder (solid color block), name,
>   category badge, formatted price, "Add to cart" button disabled when
>   stock === 0.
> - Cart layout: list of CartItem rows (product name, qty stepper,
>   line subtotal, remove button) + a total at the bottom.
> - Tailwind conventions: classes ordered layout → spacing → typography →
>   color → state; no @apply for one-offs; reusable patterns go in
>   client/src/styles/components.css behind @layer components.
> Save at .github/skills/frontend/SKILL.md.
> ```

✅ **Checkpoint:** `.github/skills/frontend/SKILL.md` exists. Type `/skills` in chat to see it listed.

---

## Step 5 — Create the `backend` skill (on-demand loaded) (2 min)

> 📖 **Reference for this step:** [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) · [Express 4 routing](https://expressjs.com/en/guide/routing.html) · [better-sqlite3 — prepared statements](https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md#preparesource---statement) · [SQLite `CREATE TABLE`](https://www.sqlite.org/lang_createtable.html)

> **Prompt template (Agent Mode):**
> ```
> /create-skill
>
> Name: backend
> Description: Use when adding routes or the DB layer to the Store API.
> Covers the Express 4 (ESM + TypeScript) layout under server/**, the
> in-memory better-sqlite3 schema (products + cart_items), the seed-on-
> import convention, parameterized-statement conventions, the centralized
> error middleware shape, the /api prefix convention, and the SPA-fallback
> ordering. Only load when the user is working on files under server/**
> or referencing the API explicitly.
> Body should include:
> - Directory layout: server/src/app.ts (exports the Express app, no
>   listener), server/src/server.ts (entrypoint that starts the listener),
>   server/src/db.ts (better-sqlite3 :memory: + schema + prepared
>   statements + product seed), server/src/seed.ts (PRODUCT_SEED array
>   of 6–8 catalog items), server/src/routes/products.ts,
>   server/src/routes/cart.ts, server/src/middleware/error.ts,
>   server/src/middleware/spa.ts (prod-only static + fallback).
> - The "products" table DDL:
>     id TEXT PRIMARY KEY,
>     name TEXT NOT NULL,
>     description TEXT,
>     price_cents INTEGER NOT NULL CHECK(price_cents >= 0),
>     category TEXT NOT NULL,
>     stock INTEGER NOT NULL DEFAULT 0 CHECK(stock >= 0),
>     created_at TEXT NOT NULL.
> - The "cart_items" table DDL:
>     product_id TEXT PRIMARY KEY REFERENCES products(id),
>     quantity INTEGER NOT NULL CHECK(quantity >= 1).
> - Seed-on-import: db.ts MUST re-seed the products table on every import
>   (DELETE FROM products; then INSERT each item from PRODUCT_SEED).
>   Cart starts empty per process. 6–8 realistic items spread across
>   categories "electronics", "groceries", "books", "home" (example
>   names: "Wireless Headphones", "Organic Bananas (1 bunch)",
>   "Espresso Beans 1kg", "Mechanical Keyboard", "Atomic Habits",
>   "Cotton Bath Towel"). Stock between 0 and 50; price_cents between
>   199 and 24999.
> - Money convention: prices stored and returned as integer cents
>   (price_cents). Never use floats. Cart responses include line_total =
>   price_cents * quantity and total = sum of line_total.
> - Endpoints (all parameterized SQL, all under /api):
>     GET /api/health
>     GET /api/products
>     GET /api/products/:id
>     GET /api/cart                              — returns { items: [...], total }
>     POST /api/cart/items                       — { productId, quantity }
>     PATCH /api/cart/items/:productId           — { quantity }
>     DELETE /api/cart/items/:productId
> - The parameterized-statement convention: use db.prepare(...).run /
>   .get / .all with positional or named parameters; NEVER template-literal
>   user input into SQL.
> - The error response shape { "error": "<message>" } and status code
>   conventions (200/201/204/400/404/409 for out-of-stock).
> - Validation rules:
>     productId: must reference an existing product (404 if not).
>     quantity: integer, >= 1, <= product.stock (409 with a descriptive
>       error if quantity exceeds stock).
>     name (admin/seed only): trim, 1–200 chars.
>     description: optional, <= 2000 chars.
>     category: must be one of "electronics", "groceries", "books", "home".
>     price_cents: integer >= 0.
>     Reject unknown body fields with 400 on all POST/PATCH.
> - Route registration order in app.ts: JSON body parser → /api/products
>   routes → /api/cart routes → SPA fallback (prod only) → error
>   middleware (last).
> Save at .github/skills/backend/SKILL.md.
> ```

✅ **Checkpoint:** `.github/skills/backend/SKILL.md` exists and appears under `/skills`.

---

## Step 5b — Create the `docs` skill (on-demand loaded) (2 min)

> 📖 **Reference for this step:** [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) · [GitHub-flavored Mermaid in markdown](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams) · [Architecture Decision Records (ADR) template](https://adr.github.io/) · [GitHub Alerts (NOTE/TIP/WARNING/IMPORTANT)](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#alerts)

This is the skill the `doc-agent` from Step 2 reaches for. Keeping it as a skill (rather than baking everything into the agent body) means future agents that touch docs can reuse the same conventions.

> **Prompt template (Agent Mode):**
> ```
> /create-skill
>
> Name: docs
> Description: Use when authoring or updating any documentation in this
> repo: README, docs/**, architecture/**, ADRs, or CONTRIBUTING. Only
> load when the user is editing files under those paths or asking for
> docs/diagrams/ADRs explicitly.
> Body should include:
> - README section order: Title → one-paragraph summary → badges →
>   Features → Quickstart → API reference → Shopping flow walkthrough
>   (browse → add to cart → adjust quantity → remove) → Architecture
>   (with mermaid diagram) → Error model → Project layout → Contributing
>   → License.
> - The architecture diagram template — a mermaid `flowchart LR` with
>   three nodes (Browser/React UI → Express API /api/* → SQLite
>   :memory: with products + cart_items) plus a note for the Vite dev
>   proxy AND a note that the products catalog is re-seeded on every
>   server start.
> - The API reference table columns: Method, Path, Description,
>   Success status, Error statuses. One row per endpoint, all under
>   /api/*. Group rows by resource (Products, Cart). Each row links
>   to a <details> block with a curl example AND a sample JSON response.
> - The curl-example style: always include `-H 'Content-Type:
>   application/json'` on POST/PATCH; default port 3001; use real values.
>   Canonical examples:
>     curl http://localhost:3001/api/products
>     curl -X POST http://localhost:3001/api/cart/items \
>       -H 'Content-Type: application/json' \
>       -d '{"productId":"sku_headphones_001","quantity":1}'
>     curl -X PATCH http://localhost:3001/api/cart/items/sku_headphones_001 \
>       -H 'Content-Type: application/json' \
>       -d '{"quantity":2}'
> - Money convention note: every response shows prices as integer cents
>   (e.g., 12900 = $129.00). Document this at the top of the API reference.
> - The ADR template at architecture/decisions/NNNN-title.md (Status,
>   Context, Decision, Consequences, References) and the convention to
>   number them sequentially from 0001.
> - The "keep docs in sync" rule: whenever an endpoint signature or a
>   client component prop changes, the matching README row AND any
>   referenced docs/api.md section must be updated in the same PR.
> Save at .github/skills/docs/SKILL.md.
> ```

✅ **Checkpoint:** `.github/skills/docs/SKILL.md` exists and appears under `/skills`.

> [!NOTE]
> **On-demand loading.** All three skills set their descriptions narrowly so Copilot only loads them when the conversation matches. In Lab 1 you'll *see* this in action — when you ask `app-builder` to scaffold the API, the `backend` skill is pulled in; when you switch to the React UI, the `frontend` skill is pulled in; when `doc-agent` writes the README in Lab 3, the `docs` skill is pulled in.

---

## Step 6 — Verify and commit (2 min)

> 📖 **Reference for this step:** [Chat: Diagnostics command](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_chat-diagnostics) · [Source Control — Generate Commit Message](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_source-control) · [Git basics — `git add` & `git commit`](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

Run the **`Chat: Diagnostics`** command (right-click in the Chat view → Diagnostics). You should see:

- 3 custom agents loaded: `app-builder`, `doc-agent`, `qa-agent`.
- 3 skills loaded: `frontend`, `backend`, `docs`.
- No errors.

Also commit what you've built so far (we'll add a remote in Lab 2):

```bash
git add .github/
# Open the Source Control view, click the ✨ "Generate Commit Message" icon,
# then commit. Suggested message: chore: scaffold custom agents and skills
```

✅ **Checkpoint:** Diagnostics shows 3 agents + 3 skills, no errors. One local commit exists. **Lab 1 starts from this state.**

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| `/create-agent` or `/create-skill` not recognized | Update VS Code (1.95+) and the Copilot Chat extension. |
| Agent dropdown doesn't show your new agent | Run `Developer: Reload Window`. Run `Chat: Diagnostics` and check for YAML frontmatter errors. |
| Skill never loads | The skill's `description` is too vague. Edit it to mention concrete file paths or task triggers (e.g., "use when modifying src/routes/**"). |

## 🎁 Stretch goals

- Add a fourth custom agent called `security` (read-only tools, instructions focused on reviewing diffs for OWASP Top 10) — you'll use it in Lab 4/5 stretch.
- Add a fourth skill called `db` that documents how to migrate from `:memory:` to a file-backed SQLite or Postgres.
- Move any of these agents/skills to your **user profile** (`~/.copilot/agents` / `~/.copilot/skills`) so they're available across all workspaces.

---

➡️ **Next: [Lab 1 — Build the Full-Stack MVP with Custom Agents](./lab-01-mvp-copilot-vscode.md)**
