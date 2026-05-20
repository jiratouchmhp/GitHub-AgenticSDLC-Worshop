# Lab 1 — Build the Full-Stack MVP with Custom Agents

> **Estimated time:** 35 minutes
> **Surface:** VS Code + Copilot Chat (**Plan Mode** + **Agent Mode** with the `app-builder` / `qa-agent` custom agents from Lab 0)

## 🎯 Learning objectives

By the end of this lab you will:

1. Use **Copilot Plan Mode** (`/plan`) to draft a structured implementation plan for the full-stack MVP, then manually hand the plan to the `app-builder` agent created in Lab 0.
2. Use the **`app-builder`** custom agent to scaffold the backend + SQLite layer **and** seed the products catalog on import.
3. Trigger the on-demand **`backend`** and **`frontend`** skills by working in their respective files.
4. Use **inline chat in Agent Mode** to harden input validation.
5. Switch to the **`qa-agent`** custom agent to author smoke tests.
6. Use the **Generate Commit Message** and **Code Review – Uncommitted Changes** features in VS Code before committing.

> [!IMPORTANT]
> Most prompts in this lab are written for **Agent Mode** with a custom agent active. Step 1 is the exception — it uses **Plan Mode** to draft the plan first. After Step 1, switch the agents dropdown to `app-builder` and stay in Agent Mode for the rest of the lab. Adapt the bracketed values in each prompt for your stack. We do not show example output — Copilot will produce the code from your prompt, and you'll review what it writes.

## ✅ Prerequisites

- Lab 0 complete; the agents dropdown lists `app-builder`, `doc-agent`, `qa-agent`.
- Skills `frontend`, `backend`, and `docs` appear under `/skills`.
- One local commit exists with `.github/agents/` and `.github/skills/`.
- A fresh Copilot Chat session is open with no active custom agent yet.

## 📖 Official documentation

| Feature you'll use | Doc link |
|---|---|
| **Plan Mode** in VS Code | <https://code.visualstudio.com/docs/copilot/agents/planning> |
| Agent Mode reference | <https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode> |
| **Inline chat** in the editor | <https://code.visualstudio.com/docs/copilot/chat/inline-chat> |
| **Generate commit message** (Source Control) | <https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_source-control> |
| **Code Review – Uncommitted Changes** | <https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_code-review> |
| npm workspaces | <https://docs.npmjs.com/cli/v10/using-npm/workspaces> |
| Vite — server proxy | <https://vitejs.dev/config/server-options.html#server-proxy> |
| better-sqlite3 — prepared statements | <https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md#preparesource---statement> |
| Tailwind CSS install (PostCSS) | <https://tailwindcss.com/docs/installation/using-postcss> |

---

## Step 1 — Plan the MVP in Plan Mode, then hand off to `app-builder` (5 min)

> 📖 **Reference for this step:** [Plan Mode in VS Code](https://code.visualstudio.com/docs/copilot/agents/planning) · [Chat agents dropdown (Plan / Agent / Edit)](https://code.visualstudio.com/docs/copilot/chat/chat-agent-mode) · [Custom agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents)

1. In the Chat view, open the **agents dropdown** at the top and select **Plan**.
2. Submit the following prompt template. The stack is fixed for the whole workshop — nothing to substitute.

> **Prompt template (Plan Mode):**
> ```
> /plan Build a tiny full-stack e-commerce "Store" app I can demo in under
> 30 minutes. Two resources: products (read-only catalog seeded on every
> server start) and cart (a single anonymous in-memory cart per process —
> no auth, no checkout, no payment).
>
> Stack (fixed):
> - Backend: Express 4 (ESM) + TypeScript (strict), in server/.
>   Uses better-sqlite3 in-memory (database lives only for the process
>   lifetime). All endpoints under /api/*. The Express `app` is exported
>   from server/src/app.ts so tests can import it without starting a
>   listener; server/src/server.ts is the entrypoint that listens on
>   PORT (default 3001).
> - Frontend: Vite 5 + React 18 + TypeScript + TailwindCSS 3, in client/.
>   Single-page UI with five components: ProductList, ProductCard, Cart,
>   CartItem, StatusRegion. Uses fetch() through a typed
>   client/src/lib/api.ts wrapper against /api/* (Vite proxy forwards
>   /api to Express in dev). Money is formatted via
>   client/src/lib/format.ts (formatPriceCents).
> - Monorepo: npm workspaces at the repo root with client/ and server/
>   packages. Root scripts: `dev` (runs both with concurrently), `build`
>   (builds client then server), `start` (prod: serve client/dist via
>   Express + listen), `test` (runs server + client test suites).
> - Schema (both tables in server/src/db.ts; re-seeded on every process
>   start):
>     products(id TEXT PK, name TEXT NOT NULL, description TEXT,
>              price_cents INTEGER NOT NULL CHECK(>=0),
>              category TEXT NOT NULL CHECK in
>              ('electronics','groceries','books','home'),
>              stock INTEGER NOT NULL CHECK(>=0),
>              created_at TEXT NOT NULL)
>     cart_items(product_id TEXT PK REFERENCES products(id),
>                quantity INTEGER NOT NULL CHECK(quantity >= 1))
>   Seed 6–8 realistic products across the four categories on import.
>   Cart starts empty per process.
> - Endpoints:
>     GET /api/health
>     GET /api/products
>     GET /api/products/:id            — 404 if not found
>     GET /api/cart                    — { items: [...], total }
>     POST /api/cart/items             — { productId, quantity }
>     PATCH /api/cart/items/:productId — { quantity }
>     DELETE /api/cart/items/:productId
>   Validation: productId must exist (404), quantity int >= 1 and
>   <= product.stock (409 if exceeds), reject unknown body fields (400),
>   parameterized SQL only. Money as integer cents end-to-end.
> - Tests: server uses Jest + supertest; client uses Vitest + React
>   Testing Library. Smoke tests covering health, product list/get,
>   cart add/update/remove happy-path, validation edges (unknown
>   product, qty 0, qty > stock, unknown fields), and Cart UI
>   optimistic-rollback. Coverage target ≥80% lines / ≥75% branches
>   in both packages.
> - Prod: Express serves client/dist as static + SPA fallback,
>   registered AFTER /api routes and BEFORE the error middleware.
>
> Produce:
> - A short summary of the design.
> - An implementation plan as an ordered todo list.
> - A verification plan (how to confirm each step is done).
>
> Defer code generation. I will hand this plan off to the `app-builder`
> custom agent next.
> ```

3. Answer any clarifying questions the Plan agent asks. Iterate until the plan looks right.

> [!TIP]
> The Plan agent saves its plan to `/memories/session/plan.md`. Run **Chat: Show Memory Files** from the command palette to see it. Session memory is wiped when the conversation ends, so keep this conversation open through the rest of the lab.

4. When you're happy with the plan, **manually switch the agents dropdown from Plan → `app-builder`**. The plan stays visible as conversation context.

✅ **Checkpoint:** Plan Mode has produced a plan; the chat is now running in **`app-builder`** with the plan in context.

---

## Step 2 — Scaffold the backend with `app-builder` (8 min)

> 📖 **Reference for this step:** [Express 4 — hello world](https://expressjs.com/en/starter/hello-world.html) · [Express 4 — routing](https://expressjs.com/en/guide/routing.html) · [better-sqlite3 API](https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md) · [npm workspaces](https://docs.npmjs.com/cli/v10/using-npm/workspaces) · [TypeScript `tsconfig` options](https://www.typescriptlang.org/tsconfig) · [`concurrently`](https://github.com/open-cli-tools/concurrently)

The agent should automatically load the `backend` skill when it starts touching server source files.

> **Prompt template (Agent Mode → `app-builder`):**
> ```
> Scaffold the Store app backend into this empty workspace.
>
> Stack: Express 4 (ESM) + TypeScript strict + better-sqlite3.
> Place EVERYTHING server-related under server/. Use npm workspaces at
> the repo root: root package.json declares { "workspaces": ["client",
> "server"] } and adds dev dependency `concurrently`.
>
> Requirements:
> - server/src/db.ts opens better-sqlite3 ':memory:' on import and creates
>   BOTH tables per the conventions in the `backend` skill:
>     products(id TEXT PK, name TEXT NOT NULL, description TEXT,
>              price_cents INTEGER NOT NULL CHECK(price_cents >= 0),
>              category TEXT NOT NULL,
>              stock INTEGER NOT NULL DEFAULT 0 CHECK(stock >= 0),
>              created_at TEXT NOT NULL)
>     cart_items(product_id TEXT PK REFERENCES products(id),
>                quantity INTEGER NOT NULL CHECK(quantity >= 1))
>   Export prepared statements (listProducts, getProduct, getCartRows,
>   addCartItem, updateCartItem, removeCartItem, clearCart) — no inline
>   SQL in route handlers. Leave the actual product seeding for the next
>   step (placeholder PRODUCT_SEED = [] is fine for now).
> - server/src/app.ts builds and exports the Express app (JSON parser →
>   /api/products router → /api/cart router → SPA-fallback middleware
>   (prod only) → error middleware). Do NOT call app.listen here.
> - server/src/server.ts imports `app` and listens on PORT (default 3001).
> - Endpoints (all parameterized SQL, all under /api):
>     GET /api/health
>     GET /api/products
>     GET /api/products/:id            — 404 if not found
>     GET /api/cart                    — returns { items: [...], total }
>     POST /api/cart/items             — { productId, quantity }
>     PATCH /api/cart/items/:productId — { quantity }
>     DELETE /api/cart/items/:productId
> - Validation:
>     productId must reference an existing product (404 otherwise).
>     quantity must be an integer, >= 1, <= product.stock (409 with
>       descriptive {error} if quantity exceeds stock).
>     Reject unknown body fields with 400 on POST/PATCH.
> - GET /api/cart returns: { items: [{ productId, name, price_cents,
>     quantity, line_total }], total } where line_total = price_cents
>   * quantity and total = sum of line_total. Join cart_items ⋈ products.
> - Centralized error middleware returning { "error": "<message>" }.
> - SPA fallback (server/src/middleware/spa.ts): in production only,
>   serve client/dist as static and serve client/dist/index.html for any
>   GET that isn't under /api/*.
> - Add a root .gitignore (node_modules, dist, *.log, .DS_Store, .env),
>   server/tsconfig.json (strict, NodeNext), and root scripts:
>     "dev":   "concurrently -n srv,web -c blue,magenta \"npm -w server run dev\" \"npm -w client run dev\"",
>     "build": "npm -w client run build && npm -w server run build",
>     "start": "NODE_ENV=production npm -w server run start",
>     "test":  "npm -w server test && npm -w client test".
>   server scripts: dev (tsx watch src/server.ts), build (tsc),
>   start (node dist/server.js), test (jest --coverage).
> - Do NOT add a README yet (the doc-agent handles that later).
> - Do NOT scaffold the client/ workspace in this step — step 3 does.
> - Do NOT seed product data in this step — step 2b does that next.
>
> Apply changes and tell me when ready to smoke-test.
> ```

When `app-builder` completes:

```bash
npm install
npm -w server run dev
```

Smoke-test from a second terminal:

```bash
curl http://localhost:3001/api/health
curl http://localhost:3001/api/products   # expect an empty list for now
```

✅ **Checkpoint:** `/api/health` returns `{"status":"ok"}` and `/api/products` returns `[]`. Dev server stays running.

---

## Step 2b — Seed the products catalog with `app-builder` (3 min)

> 📖 **Reference for this step:** [better-sqlite3 — transactions](https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md#transactionfunction---function) · [SQLite `INSERT`](https://www.sqlite.org/lang_insert.html) · [`Date.toISOString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)

The schema exists but the catalog is empty. This step adds a small,
realistic seed so the UI in Step 3 has something to render.

> **Prompt template (Agent Mode → `app-builder`):**
> ```
> Seed the products table with a realistic catalog. The seed must run
> automatically on every server start (re-seed pattern — the database is
> in-memory and resets per process). Cart starts empty.
>
> Requirements:
> - Create server/src/seed.ts that exports PRODUCT_SEED: a typed array
>   of 6–8 products spread across the four categories "electronics",
>   "groceries", "books", "home". Include at minimum these items so
>   downstream docs/tests have stable example data:
>     { id: "sku_headphones_001", name: "Wireless Headphones",
>       description: "Over-ear, 30-hour battery, USB-C.",
>       price_cents: 12900, category: "electronics", stock: 12 }
>     { id: "sku_keyboard_001", name: "Mechanical Keyboard",
>       description: "75% layout, hot-swappable switches.",
>       price_cents: 8900, category: "electronics", stock: 7 }
>     { id: "sku_bananas_001", name: "Organic Bananas (1 bunch)",
>       description: "Fair-trade, ~6 bananas per bunch.",
>       price_cents: 299, category: "groceries", stock: 50 }
>     { id: "sku_espresso_001", name: "Espresso Beans 1kg",
>       description: "Medium-dark roast, single origin.",
>       price_cents: 2499, category: "groceries", stock: 20 }
>     { id: "sku_book_001", name: "Atomic Habits",
>       description: "Paperback, James Clear.",
>       price_cents: 1599, category: "books", stock: 0 }
>     { id: "sku_towel_001", name: "Cotton Bath Towel",
>       description: "600 GSM, charcoal grey.",
>       price_cents: 1899, category: "home", stock: 15 }
>   At least one item must have stock: 0 so the "sold out" UI path is
>   exercised.
> - Update server/src/db.ts to call seedProducts() at the END of its
>   schema setup, AFTER the CREATE TABLE statements. The seed function:
>     1. DELETE FROM products  (idempotent re-seed)
>     2. DELETE FROM cart_items (cart starts empty per process)
>     3. For each item in PRODUCT_SEED, run an INSERT via a prepared
>        statement with named parameters. Set created_at = new Date()
>        .toISOString().
>   Wrap the inserts in a transaction.
> - Export seedProducts from db.ts so server tests can call it from
>   beforeEach to restore the catalog between tests.
> - Do NOT add any /api endpoint to manage seed data — this is a
>   server-internal concern.
>
> When done, restart the dev server and verify with:
>   curl http://localhost:3001/api/products | jq 'length'
> Expect 6 or more.
> ```

Restart the server and verify:

```bash
curl http://localhost:3001/api/products | jq '.[0]'
curl http://localhost:3001/api/products/sku_headphones_001
```

✅ **Checkpoint:** `/api/products` returns 6+ items, each with
`price_cents` as an integer. `/api/products/sku_headphones_001` returns
the headphones object. `/api/products/does-not-exist` returns 404.

---

## Step 3 — Scaffold the frontend (still `app-builder`, with the `frontend` skill) (7 min)

> 📖 **Reference for this step:** [Vite — scaffolding `react-ts`](https://vitejs.dev/guide/#scaffolding-your-first-vite-project) · [Vite — server proxy](https://vitejs.dev/config/server-options.html#server-proxy) · [Tailwind CSS install (PostCSS)](https://tailwindcss.com/docs/installation/using-postcss) · [React 18 docs](https://react.dev/learn) · [`fetch()` API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) · [MDN — ARIA live regions](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)

When the agent opens files in `client/`, the `frontend` skill should load automatically. You can also hint it with `/frontend` to be explicit.

> **Prompt template (Agent Mode → `app-builder`):**
> ```
> /frontend
>
> Scaffold the Store app UI as a Vite + React 18 + TypeScript + Tailwind
> workspace under client/.
>
> Steps to take:
> 1. Initialize Vite with the react-ts template inside client/ (no extra
>    prompts — use the non-interactive form). Add it to the root npm
>    workspaces; do `npm install` at the repo root.
> 2. Install Tailwind 3 + PostCSS + autoprefixer in client/, run
>    `npx tailwindcss init -p`, configure tailwind.config.js content
>    globs to ["./index.html", "./src/**/*.{ts,tsx}"], and add the three
>    @tailwind directives to client/src/index.css.
> 3. Configure the dev proxy in client/vite.config.ts:
>    server: { port: 5173, proxy: { '/api': 'http://localhost:3001' } }.
> 4. Create the components per the `frontend` skill layout:
>    client/src/lib/api.ts (typed fetch wrapper for /api/* with
>    Content-Type JSON on POST/PATCH, throws on non-2xx surfacing
>    {error} body; methods: listProducts, getCart, addToCart,
>    updateCartItem, removeCartItem),
>    client/src/lib/format.ts (formatPriceCents helper),
>    client/src/components/{ProductList,ProductCard,Cart,CartItem,
>    StatusRegion}.tsx, client/src/App.tsx, client/src/main.tsx,
>    client/index.html.
>
> Behavior:
> - On load, fetch GET /api/products AND GET /api/cart in parallel and
>   render the product grid (ProductList → ProductCard per item) on the
>   left/top and the cart (Cart → CartItem per row + total) on the
>   right/bottom.
> - ProductCard: image placeholder block + name + category badge +
>   formatPriceCents(price_cents) + "Add to cart" button. Disable the
>   button when stock === 0 and label it "Sold out" instead.
> - Add to cart: POST /api/cart/items with { productId, quantity: 1 },
>   optimistically append (or merge into existing line) in local cart
>   state; on non-2xx, roll back and surface the {error} via
>   StatusRegion (aria-live="polite"). 409 "out of stock" must produce
>   a clear human-readable status message.
> - CartItem: quantity stepper (− / + buttons + numeric label),
>   line subtotal via formatPriceCents(price_cents * quantity), "Remove"
>   button. −/+ call PATCH /api/cart/items/:productId optimistically;
>   Remove calls DELETE optimistically. Same rollback + StatusRegion
>   pattern on failure.
> - Cart total: sum of line subtotals, rendered via formatPriceCents.
>   Show an empty-cart message when the cart has no items.
> - Accessibility: <label> for every input (including the qty stepper);
>   focus returns to the "Add to cart" button (or the qty input) after a
>   successful action; StatusRegion announces add/update/remove/errors.
> - Styling: Tailwind utility classes only — system font stack via the
>   default Tailwind preset, a centered max-w-5xl container, two-column
>   layout at md+ (products left, cart right), clear hover:/focus:
>   states, soft shadow on cards. No CSS framework beyond Tailwind.
>
> Update the root scripts so `npm run dev` runs server (3001) + client
> (5173) via concurrently. Do not change the server workspace.
> ```

Start the full dev stack:

```bash
npm run dev
```

Open `http://localhost:5173` in your browser. (The Vite proxy forwards `/api/*` to Express on `:3001`.)

✅ **Checkpoint:** The page loads and shows the seeded products. You can add an item to the cart, change its quantity, and remove it. Carts and catalogs reset if you restart the server (in-memory SQLite — expected).

---

## Step 4 — Strengthen input validation with inline Agent Mode chat (4 min)

> 📖 **Reference for this step:** [Inline chat in VS Code](https://code.visualstudio.com/docs/copilot/chat/inline-chat) · [HTTP status codes (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) · [Express — error handling](https://expressjs.com/en/guide/error-handling.html)

Open the route file for `POST /api/cart/items` (under `server/src/routes/cart.ts`). Position your cursor inside the handler and press `Ctrl/Cmd + I` for inline chat — confirm the agents dropdown shows **`app-builder`** (or **Agent**).

> **Inline prompt template (Agent Mode):**
> ```
> Strengthen this endpoint:
> 1. Reject requests where productId is missing, not a string, or does
>    not reference an existing product (404 with a descriptive {error}).
> 2. Reject quantity that is not an integer, is < 1, or exceeds
>    product.stock (400 for type/range errors, 409 for out-of-stock).
> 3. If the cart already has a line for this productId, MERGE: the new
>    quantity = existing + requested (still validated against stock).
> 4. Reject unknown body fields with 400.
> 5. On success, return 201 with the created/merged cart line and a
>    Location header pointing to /api/cart/items/<productId>.
> Apply only to this function.
> ```

Repeat for **PATCH `/api/cart/items/:productId`**:

> **Inline prompt template (Agent Mode):**
> ```
> Make this a quantity-only update: only the `quantity` field is
> accepted. Re-apply the same integer + range + stock validation rules
> (quantity must be an integer >= 1 and <= product.stock). Return 200
> with the updated cart line. Return 404 if the productId is not in the
> cart. Return 400 on unknown fields. If quantity is set to the current
> value, still return 200 (idempotent).
> ```

✅ **Checkpoint:**
```bash
curl -X POST http://localhost:3001/api/cart/items \
  -H 'Content-Type: application/json' \
  -d '{"productId":"does-not-exist","quantity":1}'
```
returns HTTP 404, and
```bash
curl -X POST http://localhost:3001/api/cart/items \
  -H 'Content-Type: application/json' \
  -d '{"productId":"sku_headphones_001","quantity":9999}'
```
returns HTTP 409.

---

## Step 5 — Switch to `qa-agent` for smoke tests (6 min)

> 📖 **Reference for this step:** [Jest — configuration](https://jestjs.io/docs/configuration) · [Jest — coverage thresholds](https://jestjs.io/docs/configuration#coveragethreshold-object) · [supertest](https://github.com/ladjs/supertest) · [Vitest — configuration](https://vitest.dev/config/) · [Vitest — coverage](https://vitest.dev/guide/coverage.html) · [React Testing Library queries](https://testing-library.com/docs/queries/about/) · [`vi.mock`](https://vitest.dev/api/vi.html#vi-mock)

Switch the **agents dropdown** from `app-builder` to **`qa-agent`**. The conversation context stays — the new agent inherits it.

> **Prompt template (Agent Mode → `qa-agent`):**
> ```
> Author smoke tests for the current code base. Two runners, two suites:
>
> SERVER (Jest + supertest, importing the exported Express `app` from
> server/src/app.ts — do NOT spawn a listener):
> - GET /api/health returns 200 and {"status":"ok"}.
> - GET /api/products returns 200 with the seeded array (>= 6 items),
>   each with integer price_cents.
> - GET /api/products/sku_headphones_001 returns 200 with the headphones
>   object; GET /api/products/does-not-exist returns 404 with an {error}
>   body.
> - POST /api/cart/items:
>     valid create returns 201 with a Location header and the cart line;
>     unknown productId returns 404;
>     quantity 0 / negative / non-integer returns 400;
>     quantity > product.stock returns 409;
>     unknown body field returns 400;
>     adding the same productId twice MERGES quantities.
> - GET /api/cart returns { items, total } with total = sum of
>   price_cents * quantity (integer cents).
> - PATCH /api/cart/items/:productId:
>     valid quantity update returns 200 with the new line;
>     productId not in cart returns 404;
>     same validation rules (integer, 1..stock) re-apply;
>     unknown body field returns 400.
> - DELETE /api/cart/items/:productId returns 204; subsequent GET
>   /api/cart no longer lists that line.
> - Re-seed the better-sqlite3 :memory: store between tests via
>   beforeEach: import seedProducts from db.ts and call it (it also
>   clears cart_items) so suites are order-independent.
>
> CLIENT (Vitest + React Testing Library, jsdom env):
> - Render <ProductList products={...} /> with two seeded products
>   (one in-stock, one stock=0). Verify both render with their name,
>   formatted price (e.g., "$129.00"), and that the sold-out product's
>   "Add to cart" button is disabled and reads "Sold out".
> - Render <Cart /> with a mocked api.ts (vi.mock the module) showing
>   two cart lines. Verify the rendered total equals the sum of line
>   subtotals (formatPriceCents).
> - For the optimistic-rollback path: simulate clicking "Add to cart"
>   on an in-stock product; mock api.addToCart to reject with an
>   {error: "Only 3 in stock"} body. Verify the optimistic line is
>   removed AND the StatusRegion announces that error message.
>
> Coverage: configure jest --coverage with thresholds 80/75 in
> server/jest.config.ts; configure vitest with the same thresholds in
> client/vitest.config.ts. Both must fail the build under threshold.
> The root `npm test` must run server then client.
>
> When done, run `npm test` and report the pass/fail summary and the
> coverage numbers from both packages.
> ```

Watch the agent edit files, run the tests, and report back.

✅ **Checkpoint:** All tests pass and coverage is at or above the target in **both** packages.

✅ **Checkpoint:** All tests pass and coverage is at or above the target.

---

## Step 6 — Review uncommitted changes with Copilot (2 min)

> 📖 **Reference for this step:** [Copilot Code Review in VS Code](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_code-review) · [`.github/copilot-instructions.md`](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)

1. Click the **Source Control** icon in the Activity Bar.
2. Hover the **CHANGES** header → click the ✨ **"Copilot Code Review – Uncommitted Changes"** button.
3. Read 2–3 of the suggested comments and apply any that make sense.

> [!NOTE]
> Code Review – Uncommitted Changes reads `.github/copilot-instructions.md` for repo-wide guidance. You don't have one yet — we'll add it in Lab 2.

✅ **Checkpoint:** You've applied at least one Copilot review suggestion.

---

## Step 7 — Generate a commit message and commit locally (2 min)

> 📖 **Reference for this step:** [Generate commit message](https://code.visualstudio.com/docs/copilot/reference/copilot-vscode-features#_source-control) · [Customizing commit-message generation](https://code.visualstudio.com/docs/copilot/customization/custom-instructions#_settingsspecific-instructions-files) · [Conventional Commits](https://www.conventionalcommits.org/)

1. Stage all changes (`+` next to **Changes**, or `git add -A`).
2. Click the ✨ **"Generate Commit Message with Copilot"** icon in the commit-message box.
3. Edit if needed, then **Commit**.

> [!TIP]
> You can pin a Conventional Commits style by adding `github.copilot.chat.commitMessageGeneration.instructions` entries to `.vscode/settings.json`. Ask `app-builder` to do that for you with a one-line prompt.

✅ **Checkpoint:** Your commit log now has a feat-style commit covering the MVP. No remote is configured yet — Lab 2 takes care of that.

---

## 🩹 Troubleshooting

| Symptom | Fix |
|---|---|
| `app-builder` agent missing from dropdown | Re-run Lab 0 Step 2, then `Developer: Reload Window`. |
| The `backend` / `frontend` skill never loads | The skill's `description` is too vague. Edit it to mention concrete file paths (e.g., "use when editing client/**" or "use when editing server/**"). |
| Tailwind classes don't apply | Check `client/postcss.config.js` exists, the three `@tailwind` directives are in `client/src/index.css`, and `tailwind.config.js` content globs include `./src/**/*.{ts,tsx}`. Restart the Vite dev server. |
| Browser hits `/api/products` and gets HTML / 404 | The Vite proxy isn't configured. Confirm `server.proxy: { '/api': 'http://localhost:3001' }` in `client/vite.config.ts`, and that Express is actually running on `:3001`. |
| `__dirname is not defined` in ESM Express code | In ESM you must compute it: `const __dirname = path.dirname(fileURLToPath(import.meta.url))`. |
| "Generate Commit Message" ✨ missing | Update Copilot Chat extension. Confirm `github.copilot.enable.scminput` is `true`. |
| Inline chat doesn't honor the active custom agent | Some inline-chat surfaces use the default Agent mode. Move the prompt to the main Chat view if behavior drifts. |
| Tests can't import the app | Tell `app-builder`: *"Ensure the Express `app` is exported from server/src/app.ts so tests can import it without starting the server."* |

## 🎁 Stretch goals

- Tell `doc-agent` to draft an initial `README.md` and a `docs/api.md` for the MVP — but commit it as a separate commit so Lab 3 starts from a clean docs baseline.
- Add `dotenv` and ask `app-builder` to load `PORT` and `NODE_ENV` from `.env` in `server/src/server.ts`.
- Ask `app-builder` to write a multi-stage `Dockerfile` that builds both workspaces (client → dist, server → dist) and runs the production server on a distroless base.

---

➡️ **Next: [Lab 2 — Push to GitHub](./lab-02-push-to-github.md)**
