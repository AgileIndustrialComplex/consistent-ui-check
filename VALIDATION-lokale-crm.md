# lokale-crm UI-consistency audit — validation report

- **Date:** 2026-09-19
- **Harness:** `consistent-ui-check` (https://github.com/AgileIndustrialComplex/consistent-ui-check)
- **Target:** `lokale-crm` @ `master` `ffe9e04a`
- **Scope:** `src/` — 145 component/page files (`.tsx`/`.jsx`, tests excluded)

## Verdict

| Class | Count |
|---|---|
| INCONSISTENT | **124** |
| CONSISTENT | 19 |
| CANONICAL (exempt) | 2 |
| **Total scanned** | 145 |

**Exit code: 1** (violations present). Output is deterministic — re-running the
harness over the same tree yields byte-identical JSON.

## What "consistent" means here

A component is **consistent** when it conforms to the app's canonical design
vocabulary, judged by five exact, grep-computable rules. A component is
**inconsistent** when any rule fails.

## Violations by rule

| Rule | Count | Meaning |
|---|---|---|
| MAGIC_VALUE | 334 | hardcodes `text-sm`/`rounded-md`/`rounded-lg`/`px-3`/`h-8` instead of using a shared primitive |
| BUTTON_SHELL | 37 | renders a raw `<button>` without importing the shared `Button` |
| RAW_HEX | 35 | contains a hex literal not in the token allowlist |
| MODAL_SHELL | 16 | uses Headless `<Dialog>`/`<DialogPanel>` directly instead of the shared `Modal` |
| CN_ORIGIN | 4 | imports `cn` from `@/lib/utils` instead of canonical `@/utils/cn` |

## RAW_HEX — 35 (29 files, 3 with duplicates)

Worst offenders:

- `app/global-error.tsx` — 6 hexes
- `app/page.tsx` — 5
- `components/layout/Loader/LeaderLogo.tsx` — 5
- `components/forms/ManageFlatsModal.tsx` — 4
- `components/customCharts/GaugeChart.tsx`, `app/ustawienia/integracja/page.tsx`,
  `app/odbiory/lokale/[id]/page.tsx`,
  `app/odbiory/budowa/[id]/[phaseid]/page.tsx` — 2–3 each

Singles: `app/(legal)/layout.tsx`, `app/landing/page.tsx`,
`app/ustawienia/system-premiowy/page.tsx`, `DocumentSignModal.tsx`, `Nav.tsx`.

**Confirmed near-duplicates (same semantic color, multiple spellings):**

- Orange: `#f9b233` vs `#FBB03F`
- Near-black: `#1a202c` vs `#111827` vs `#1a1a2e`
- Amber-800: `#92400e`
- Emerald: `#10b981` / `#059669`

These are the strongest direct evidence that colors were copy-pasted per-file
instead of drawn from one palette.

## BUTTON_SHELL — 37 files with raw `<button>`

Includes all 3 dashboard tiles (`CalendarCard`, `HotLeadItem`, `NewCard`), 5
`clientTiles`, 4 `modals`, `Nav.tsx`, `Login.tsx`, `Pagination.tsx`,
`ActionsCalendar.tsx`, `InvestmentStagesTable.tsx`, and 9 `app/*/page.tsx`
routes.

> Note: a few listed are themselves primitives in their own right
> (e.g. `CloseButton.tsx`). The rule list is a **review checklist**, not an
> auto-merge list — each hit needs a human/agent confirmation before migration.

## MODAL_SHELL — 16 modals bypassing the shared shell

The highest-impact cluster. Most offenders live in `components/modals/` yet
don't use the shared `components/modals/Modal.tsx`:

`ConfirmDeleteModal`, `GenerateDocumentModal`, `GenerateOfferModal`,
`GenerateQuarterlyReportModal`, `InvoiceFormModal`, `ManageLeadsModal`,
`ImportErrorsModal`, `ImportExcelModal`, `ImportExportModal`,
`InvoiceTemplatePickerModal`, `QuickActions`, `SearchModal`,
`ActionDetailsModal`, `DocumentSignModal`, `InvoicePreviewDrawer`,
`ManageFlatsModal`.

One shared-modal migration covers most of the app's modal inconsistency.

## CN_ORIGIN — 4 files importing the duplicate `cn`

`FileUploader`, `Pagination`, `UrlPagination`, `NewCard` import `cn` from
`@/lib/utils`. This is a genuine duplicate: `src/lib/utils.ts` is byte-identical
to `src/utils/cn.ts`. Migrating these 4 then allows deleting `src/lib/utils.*`
entirely.

## MAGIC_VALUE — 334 hardcoded token classes

| Token | Count |
|---|---|
| `text-sm` | 112 |
| `rounded-lg` | 70 |
| `rounded-md` | 66 |
| `px-3` | 64 |
| `h-8` | 22 |

Widest but shallowest — after a token scale exists, this collapses. Until then
it is the copy-paste surface that feeds the other four classes of drift.

## Recommended remediation order

1. **MODAL_SHELL (16)** — one shared-shell migration; biggest visible win.
2. **CN_ORIGIN (4)** — trivial; deletes the duplicate `cn` helper entirely.
3. **RAW_HEX (35)** — tokenize the palette; kills the near-duplicate colors.
4. **BUTTON_SHELL (37)** — review checklist first (some are legit primitives).
5. **MAGIC_VALUE (334)** — meaningful only after the token scale exists.

## Reproduce

```bash
git clone https://github.com/AgileIndustrialComplex/consistent-ui-check.git
node consistent-ui-check.mjs <path-to-lokale-crm>/src --json-out report.json
```

See the harness `README.md` for full usage and rule configuration.