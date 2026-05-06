# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

Single-file financial control web app for **JM Transportes** (a Brazilian
transport/logistics business). The entire application — markup, styles, state,
rendering, and chart drawing — lives in `index.html` (~944 lines). There is **no
build step, no package manager, no tests, no linter**. The page is opened
directly in a browser; CSS and runtime libraries are loaded from CDNs (Tailwind
2.2.19, plus React/ReactDOM/Recharts/Lucide that are referenced but not
actually used by the runtime code).

UI strings, categories, and comments are in Brazilian Portuguese. Preserve
pt-BR text and the BRL currency formatting when editing.

## Running and "building"

- **Open locally:** open `index.html` in any browser, or serve the directory:
  `python3 -m http.server 8000` then visit `http://localhost:8000/`.
- **No build, no install, no tests.** Don't add a `package.json`, bundler, or
  framework unless explicitly requested — the "single-file, zero-build"
  property is intentional (see comment at line ~78: *"Arquivo único HTML puro
  (sem build, sem framework)"*).
- **CI:** `.github/workflows/jekyll-docker.yml` runs a Jekyll Docker build, but
  its `branches:` filter is a placeholder string
  (`git-branch--m-main-<BRANCH>-...`) that will never match a real branch, so
  the workflow effectively never runs. Don't assume CI gates anything; if
  Jekyll publishing is wanted, the branch filter needs to be fixed first.

## Architecture

The app uses a hand-rolled vanilla-JS render loop — **not** React, despite
React being included in `<script>` tags. The Babel `<script type="text/babel">`
block (line ~69) is empty; all real code lives in the plain `<script>` block
below it.

**Render cycle (`index.html`):**
1. `state` (line ~123) is a single mutable object holding the active tab,
   four data arrays, modal state, and search text.
2. `setState(patch)` mutates `state` via `Object.assign` and calls `render()`.
3. `render()` rebuilds `#app-root.innerHTML` from `buildApp()`, then calls
   `bindEvents()` to re-attach DOM listeners and `renderCharts()` to redraw
   canvases. **Every state change re-renders the entire page** — there is no
   diffing. New interactive elements must be wired up inside `bindEvents()` or
   they will be dead after the next render.
4. `persist(key, arr)` saves to `localStorage` *and* calls `setState`.

**Templating:** UI is built by string-concatenating helpers (`card`, `btn`,
`inputField`, `selectField`, `badge`) into HTML strings. **Always run
user-supplied values through `escHtml()`** before interpolating into these
templates — the `inputField` helper already does this for `value`, but
hand-written templates often don't.

**Charts:** Drawn directly on `<canvas>` with the 2D context in
`renderBarChart()` and `renderPieChart()`. Recharts is loaded but unused;
prefer extending the existing canvas code over pulling in Recharts.

**Icons:** Inline SVG strings in the `ICONS` object (line ~179). Lucide is
loaded but unused.

## Domain model

Four collections, all persisted to `localStorage` under these exact keys
(short-form, do not rename without a migration):

| `state` field | localStorage key | Item shape (key fields)                                                                                          |
| ------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------- |
| `lancamentos` | `jm:lanc`        | `{id, data, descricao, categoria, tipo: "Receita"\|"Despesa", valor}`                                            |
| `boletos`     | `jm:bol`         | `{id, fornecedor, valor, vencimento, categoria, status: "A Pagar"\|"Pago"}`                                      |
| `notas`       | `jm:nota`        | `{id, cliente, numNota, valor, vencimento, status: "A Receber"\|"Recebido"}`                                     |
| `dividas`     | `jm:div`         | `{id, credor, tipo, valorOriginal, saldo, parcelaTotal, parcelaPaga, valorParcela, proxVenc, status: "Ativo"\|"Quitado"}` |

Conventions to keep consistent:

- `id` = `uid()` (timestamp + random). Never reuse or reorder by id.
- Dates are stored as ISO `YYYY-MM-DD` strings; month grouping uses
  `iso.slice(0,7)` to compare against `todayISO().slice(0,7)`. Don't introduce
  `Date` objects into stored records.
- Money is stored as JS `Number` and rendered via `fmtBRL` / `fmtCompact`. All
  display goes through these helpers.
- Category lists `CATEGORIAS_RECEITA` / `CATEGORIAS_DESPESA` (line ~90) are the
  single source of truth for receita/despesa selects.
- Status strings are user-facing pt-BR labels and are compared directly
  (`status==="Pago"`, etc.). Don't translate them or change casing.

Tabs (`state.tab`): `dashboard`, `lancamentos`, `boletos`, `notas`, `dividas`
— keep these literal strings in sync between `buildApp()`, `buildBottomNav()`,
and any new section.

## Color palette

Single `C` object near the top (line ~81) defines the navy/orange brand
palette. Reuse `C.navy`, `C.primary`, `C.green`, `C.red`, `C.amber`,
`C.purple`, etc. instead of hard-coding hex values.

## Note on ARCHITECTURE.md

`ARCHITECTURE.md` contains a generic Client/Server/Database Mermaid diagram
that **does not describe this codebase** (this app has no server, no API, no
database — only `localStorage`). Treat it as a leftover template, not as
authoritative documentation. Prefer this file.
