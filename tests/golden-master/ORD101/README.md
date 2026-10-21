# Golden master — ORD101 (edit the lines of a customer order, with VAT)

## Where these outputs come from

There is no IBM i available in this environment, so none of these cases are recorded
runs. Every expected value in `cases.json` is worked out by hand from the source:
`QRPGLESRC/ORD101.PGM.RPGLE` and `functionsVAT/vat300.rpgle`, applying the exact
packed-decimal arithmetic the program uses — truncation into intermediate fields,
`%DECH` half-adjust into the final result — and citing the line(s) responsible. On a
project where a live system exists, these same cases would instead come from
recorded runs of the real program and the `derivation` fields would be unnecessary.

The business rules this golden master implements are in
`docs/reverse/ORD101/rules.md` (R1 through R14, plus 11 named quirks); every case
below cites which rule(s) and/or quirk(s) it exercises.

## How the cases are organised

Each entry in `cases.json` has:
- `id`, `title`, `rules` (the R-numbers from rules.md it exercises), `quirks` (the
  quirk(s) from rules.md it exercises, where applicable), and `citations`
  (file:line references).
- `inputs`: the VAT definitions, article, customer, order and order-line records
  involved, in the layouts of `QDDSSRC/DETORD.PF`, `ORDER.PF`, `ARTICLE-Article_File.PF`,
  `CUSTOMER.PF` and `functionsVAT/vatdef.pf`, plus the user input (option typed,
  quantities, price) where relevant.
- `steps`: one or more actions. Multi-step cases model a short user session (e.g.
  "edit a line, then press Enter again unchanged") where a later step's outcome
  depends on state the earlier step left behind. Each step has its own `expected`
  output and `derivation` showing the arithmetic.

Most cases reuse rows straight out of `data/*.csv`, cited as `"source"` in the
case's `inputs`. Four rules/quirks have no seed-data combination that exercises
them (the exact half-cent VAT rounding, R11/quirk 1; the >30-character article
description, quirk 9), so those cases (`C12`, `C17`) add their own fixture rows
inline in the case — these are **not** written back into `data/*.csv`, to avoid
changing shared seed data other slices and agents also rely on.

## Rule and quirk coverage

| Case | Rules | Quirks | What it shows |
|---|---|---|---|
| C01 | R1, R2 | — | List load sums stored ODTOT/ODTOTVAT verbatim; ARTDESC looked up fresh |
| C02 | R3 | — | Invalid option on two rows: both flagged, only the first cursor-positioned |
| C03 | R4 | option 4 refused when delivered > 0 | Delete blocked, delivered 8 of 20 |
| C04 | R6 | — | Option 4 deletes the line, adjusts order totals, zeroes the row |
| C05 | R5, R7 | list totals not re-derived | Opening a line prices it fresh; unsaved edit does not change the list total |
| C06 | R7 | quantity 0 | Qty-0 line prices to 0.00 VAT |
| C07 | R8, R9 | — | ERR1001 (delivered > old ordered); repriced anyway |
| C08 | R8, R9 | — | ERR1002 (ordered < old delivered); repriced anyway |
| C09 | — | error indicators never turned off | A later, valid line still shows ERR1001 from an earlier failure |
| C10 | R8, R9 | stale-value cross check | Checks compare against old file values, not each other's new value |
| C11 | R10 | not saved on first Enter | Two clean Enters needed to actually save a changed line |
| C12 | R11 | half-up rounding (0.065 -> 0.07) | Exact half-cent VAT, rounds up, not banker's-rounds down |
| C13 | R12 | unknown VAT code | Code '9' (not in VATDEF) prices at 0%, no error |
| C14 | R13 | deleted VAT code stays priceable | Code '3' (VATDEL='X') still prices at its rate |
| C15 | R14 | VATDEF cache cleared on miss | Same-code calls are cached; a miss is never cached |
| C16 | — | option 6 does nothing | Option 6 accepted, no error, no effect |
| C17 | R2 | description truncated 50->30 | A 39-char ARDESC is cut to 30 chars on screen |
| C18 | R4, R8 | delivered == ordered | Boundary case: delete still blocked; edit checks use strict `>`/`<` |
| C19 | R11, R12, R13 | — | The four literal values from `QTESTSRC/vat300.test.sqlrpgle`, with its own isolated VATDEF fixture (codes '1' and '3' only — distinct from the seed data's VATDEF) |
| C20 | — | Prtord/ORD500 never called | Static fact: declared prototype, no call site anywhere in the source |

All 14 rules (R1-R14) and all 11 quirks from `docs/reverse/ORD101/rules.md` are
covered by at least one case above.
