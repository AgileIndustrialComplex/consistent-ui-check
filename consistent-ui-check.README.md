# consistent-ui-check

Deterministic UI-consistency harness for the Next.js/Tailwind CRM frontend.
Zero dependencies — runs with plain `node`, no install.

A component is **CONSISTENT** when it conforms to the app's canonical design
vocabulary, or **INCONSISTENT** when it fails any exact rule below.

## Usage

```bash
node consistent-ui-check.mjs [srcDir]          # human summary (exit 0 = consistent)
node consistent-ui-check.mjs [srcDir] --json    # full JSON report to stdout
node consistent-ui-check.mjs [srcDir] --json-out report.json   # write report file
```

`srcDir` defaults to `src`. **Exit code is the CI gate:** `0` = fully
consistent, `1` = violations found. Determinism: same source tree → identical
verdict (byte-identical JSON across runs).

## Rules

| Rule         | Fails when |
|--------------|-----------|
| BUTTON_SHELL | file renders a raw `<button>` but doesn't import the shared `Button` |
| MODAL_SHELL  | uses `<Dialog>/<DialogPanel>` directly but doesn't import shared `Modal` |
| CN_ORIGIN    | imports `cn` from a non-canonical path (`@/lib/utils` vs `@/utils/cn`) |
| RAW_HEX      | contains a hex literal not in the token allowlist |
| MAGIC_VALUE  | hardcodes `rounded-md/lg`, `h-8`, `px-3`, `text-sm` (should come from a shared primitive) |

Canonical sources (`Button.tsx`, `Modal.tsx`, `cn` files) are exempt — they
define the rules, not violate them. Files matching `ignoreHeaders` are skipped.

## Config (top of the file)

- `sharedButton` / `sharedModal` — import specifier regexes for canonical primitives
- `canonicalCn` — allowed `cn` import paths
- `colorTokens` — hexes allowed (extend with the real brand palette; every hex
  NOT listed is flagged as hardcoded)
- `canonicalSources` — files exempt as the design-system reference
- `MAGIC_VALUES` — tokenized utility classes

To make the RAW_HEX rule fully accurate, move the real brand colors into
`colorTokens` (and ideally `tailwind.config.ts`). Until then it flags all raw
hex as hardcoded — including the near-duplicate oranges/blacks that are the
same semantic color spelled differently.

## Verified reference (2026-09, lokale-crm/src)

145 files scanned → 126 INCONSISTENT, 19 consistent. Most-failing files:
`app/ustawienia/integracja/page.tsx`, `components/forms/ManageFlatsModal.tsx`,
`app/global-error.tsx`, `app/page.tsx`. Identified structural duplicates:
`src/lib/utils.ts` ≡ `src/utils/cn.ts` (identical `cn()`), two `Pagination`
implementations, two modal shells.