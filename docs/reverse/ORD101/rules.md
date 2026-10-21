# ORD101 — Edit the lines of one customer order, with VAT

## Purpose

ORD101 is a green-screen program to edit the lines of one customer order. Called with
the order id (`id`, `QRPGLESRC/ORD101.PGM.RPGLE:20-22`), it shows a subfile of the
order's lines (format `FMT01`/`CTL01`/`SFL01`) and, per line, a detail edit screen
(`FMT02`), both defined in `QDDSSRC/ORD101D.DSPF`.

- **Reads:** `ORDER1` (the order header, chained once at start), `DETORD1` (the
  order's lines, keyed on order id + line), `ARTICLE1` (via `ART300`, for the
  article description and VAT code), `CUSTOMER1` (via `CUS300`, for the customer
  name) and `VATDEF` (via `VAT300`, for the VAT rate).
- **Updates:** `DETORD1` — deletes a line (option 4) or rewrites a line's quantity,
  delivered quantity and price (option 2, after edit). It never updates `ORDER1`,
  `ARTICLE1` or `CUSTOMER1`.
- **Calls:** `GetArtDesc`, `GetArtVatCode` (`ART300`), `GetCusName` (`CUS300`),
  `GetVATRate`, `ClcVAT` (`VAT300`/`functionsVAT/vat300.rpgle`). It also declares a
  prototype for `ORD500` (`Prtord`, line 24) but never calls it — a leftover
  declaration with no effect.

## Data dictionary

| Field | Meaning | Length/decimals | Source |
|---|---|---|---|
| `ORID` | Order number | Packed 6.0 | `common/SAMREF.PF`, `QDDSSRC/ORDER.PF:3` |
| `ORYEAR` | Order year | Packed 4.0 | `common/SAMREF.PF`, `QDDSSRC/ORDER.PF:4` |
| `ORCUID` | Customer id of the order | Packed 5.0 | `common/SAMREF.PF`, `QDDSSRC/ORDER.PF:5` |
| `ODLINE` | Order line number | Packed 5.0 | `common/SAMREF.PF`, `QDDSSRC/DETORD.PF:5` |
| `ODARID` | Article id of the line | Char 6 | `common/SAMREF.PF`, `QDDSSRC/DETORD.PF:6` |
| `ODQTY` | Ordered quantity | Zoned 5.0 | `common/SAMREF.PF` (`QUANTITY`), `QDDSSRC/DETORD.PF:7-9` |
| `ODQTYLIV` | Delivered quantity | Zoned 5.0 | `common/SAMREF.PF` (`QUANTITY`), `QDDSSRC/DETORD.PF:10-12` |
| `ODPRICE` | Unit price of the line | Packed 7.2 | `common/SAMREF.PF` (`UNITPRICE`), `QDDSSRC/DETORD.PF:13` |
| `ODTOT` | Line total without VAT | Packed 9.2 | `common/SAMREF.PF` (`TOTPRICE`), `QDDSSRC/DETORD.PF:14` |
| `ODTOTVAT` | Line total with VAT | Packed 9.2 | `common/SAMREF.PF` (`TOTPRICE`), `QDDSSRC/DETORD.PF:15-17` |
| `DSQTY`, `DSQTYLIV`, `DSPRICE` | Editable copies of `ODQTY`/`ODQTYLIV`/`ODPRICE` on the detail screen | same as above | `QDDSSRC/ORD101D.DSPF:116,118,134` |
| `VAT` | Computed VAT amount of the current line, shown on `FMT02` | Packed 9.2 (REFFLD `TOTPRICE`) | `QDDSSRC/ORD101D.DSPF:121`, implicit program var |
| `VATRATE` | VAT rate % for the line's article, shown on `FMT02` | Zoned 4.2 | `common/SAMREF.PF`, `QDDSSRC/ORD101D.DSPF:122`, `functionsVAT/vatdef.pf:4` |
| `ARVATCD` | VAT code on the article | Char 1 (REFFLD `VATCODE`) | `QDDSSRC/ARTICLE-Article_File.PF` |
| `VATCODE` | Key of `VATDEF` | Char 1 | `common/SAMREF.PF`, `functionsVAT/vatdef.pf:3` |
| `VATDEL` | Delete flag of a VAT code, `X` = deleted | Char 1 (REFFLD `DLCODE`) | `functionsVAT/vatdef.pf:12` |
| `ARTDESC` | Article description shown on screen | Char 30 (program-described, **not** the full 50-char `ARDESC`) | `QDDSSRC/ORD101D.DSPF:18,124` |
| `CUSTNAME` | Customer name shown on screen | Char 30 | `QDDSSRC/ORD101D.DSPF:54` |
| `TOT`, `TOTVAT` | Order totals without/with VAT, summed over all lines | Packed 9.2 (REFFLD `TOTPRICE`) | `QDDSSRC/ORD101D.DSPF:84,86` |

## Business rules

**R1 — Loading the line list.** On entry to the subfile panel, the program positions
on the order id and reads every `DETORD1` line of that order with `SETLL`/`READE` on
key `id`, writing one subfile row per line and accumulating the order totals `tot`
(sum of `ODTOT`) and `totvat` (sum of `ODTOTVAT`) from the stored values already on
each line — it does not recompute VAT while building the list.
`QRPGLESRC/ORD101.PGM.RPGLE:102-117`.

**R2 — Article description on the list and the detail screen.** Each row's
description is looked up fresh from the article master by `GetArtDesc(ODARID)` —
it is not stored on the order line. `QRPGLESRC/ORD101.PGM.RPGLE:111,220`.

**R3 — Option validation.** The option typed in a subfile row must be 0 (blank), 2
(edit), 4 (delete) or 6 (deliver); any other value turns the row's reverse image on
and shows "Invalid Option" (indicator 35), and only the first invalid row of the
batch is positioned to. `QRPGLESRC/ORD101.PGM.RPGLE:146-160`, message text
`QDDSSRC/ORD101D.DSPF:36`.

**R4 — Delete is blocked once a line has a delivery.** Option 4 on a line whose
`ODQTYLIV` (delivered quantity) is greater than 0 is rejected with "Line with
delivery can not be deleted." (indicator 36); the line is not deleted.
`QRPGLESRC/ORD101.PGM.RPGLE:161-169`, message text `QDDSSRC/ORD101D.DSPF:37-38`.

**R5 — Option 2 opens the line for edit.** It switches to the detail panel
(`FMT02`) for the chosen line. `QRPGLESRC/ORD101.PGM.RPGLE:182-184`.

**R6 — Option 4 deletes the line.** It subtracts the line's `ODTOT`/`ODTOTVAT` from
the order totals, deletes the `DETORD1` record by key (`id`, `ODLINE`), and
redisplays the row with quantity, price and total zeroed and the description
replaced by `**** Delete ***`. `QRPGLESRC/ORD101.PGM.RPGLE:185-195`.

**R7 — Starting an edit reads the line and prices it.** `CHAIN (id:odline)
DETORD1` loads the current record; the editable fields `DSQTY`, `DSQTYLIV`,
`DSPRICE` are seeded from `ODQTY`, `ODQTYLIV`, `ODPRICE`; the VAT amount shown
(`VAT`) is `ClcVAT` of the article's *current* VAT code (`GetArtVatCode(ODARID)`,
looked up fresh, not stored on the line) against the stored `ODTOT`; `ODTOTVAT` is
set to `ODTOT + VAT`; and the displayed rate (`VATRATE`) is `GetVATRate` of that
same current article VAT code. `QRPGLESRC/ORD101.PGM.RPGLE:215-224`.

**R8 — Validating an edited line.** Two checks run on every Enter from the detail
screen, both comparing the *newly typed* value on one side against the line's
*old, on-file* value on the other (not against the other newly typed value):
- if `DSQTYLIV` (new delivered qty) > `ODQTY` (old ordered qty on file), indicator
  37 is turned on and message ERR1001 ("Delivered quantity must be lower or equal
  to ordered quantity.") is shown on `DSQTYLIV`;
- if `DSQTY` (new ordered qty) < `ODQTYLIV` (old delivered qty on file), indicator
  38 is turned on and message ERR1002 ("Ordered quantity can not be lower that the
  quantity already delivered.") is shown on `DSQTY`.
`QRPGLESRC/ORD101.PGM.RPGLE:245-254`, messages `QMSGSRC/SAMMSGF.MSGF`
(`ERR1001`, `ERR1002`), indicators wired to the fields at
`QDDSSRC/ORD101D.DSPF:117,135`.

**R9 — The line is repriced on every Enter, even when validation fails.** Right
after the two checks above, regardless of whether an error was raised, the program
recomputes `ODTOT = DSQTY * DSPRICE`, then `VAT = ClcVAT(GetArtVatCode(ODARID),
ODTOT)` and `ODTOTVAT = ODTOT + VAT`, so the screen always reflects the typed
quantity/price even on an error redisplay. `QRPGLESRC/ORD101.PGM.RPGLE:255-257`.

**R10 — A changed line needs two unmodified Enters to save.** After validation,
if the DDS `CHANGE` indicator (27, record-level on `FMT02`) is on — meaning the
user altered a field since the line was last shown — the program redisplays the
line (showing the recalculated total) instead of saving. Only when the user
presses Enter again *without* changing anything further does it commit: `ODQTY`,
`ODQTYLIV`, `ODPRICE` are set from `DSQTY`/`DSQTYLIV`/`DSPRICE`, the `DETORD1`
record is rewritten with `UPDATE`, and control returns to the line list.
`QRPGLESRC/ORD101.PGM.RPGLE:260-273`, `QDDSSRC/ORD101D.DSPF:92` (`CHANGE(27)`).

**R11 — VAT amount truncation then rounding (`ClcVAT`).** The rate is applied as
`(net * vatrate) / 100` into an intermediate `Packed(11:4)` field, which truncates
(does not round) to 4 decimals; the result is then half-adjusted into the
`Packed(9:2)` return value with `%DECH`. For positive amounts this intermediate
step does not change the final cent versus rounding the raw product directly,
because the extra two decimals are simply dropped before the half-adjust.
`functionsVAT/vat300.rpgle:46-49`.

**R12 — Unknown VAT code gives rate 0.** `chainVATDEF` clears the whole `VATDEF`
record buffer before chaining; if the code is not found in `VATDEF`, the cleared
values remain, so `GetVATRate` returns 0, `GetVATDesc` returns blanks and `ClcVAT`
returns 0 for that line. `functionsVAT/vat300.rpgle:61-72`.

**R13 — A deleted VAT code still prices normally.** `GetVATRate`, `GetVATDesc` and
`ClcVAT` do not look at `VATDEL`; only `ExistVATRate` checks
`%found(VATDEF) and VATDEL <> 'X'`. A line on a VAT code flagged deleted
(`VATDEL = 'X'`) is priced with that code's rate exactly as if it were active.
`functionsVAT/vat300.rpgle:53-59`, confirmed by `QTESTSRC/vat300.test.sqlrpgle`
(`test_GetVATRate`, `test_ClcVAT`, `test_ExistVATRate`).

**R14 — VAT lookups are cached by code, not by article.** `chainVATDEF` only
re-chains `VATDEF` when the requested code differs from the code already in the
record buffer (`P_VATCODE <> VATCODE`); repeated calls for the same code reuse the
buffer. Because the buffer is cleared before every re-chain, an invalid code never
gets cached — each call for the same unknown/invalid code clears and re-chains
again, since the cleared `VATCODE` never matches the requested one.
`functionsVAT/vat300.rpgle:61-72`.

## Quirks

A rebuild could easily get these wrong.

1. **VAT rounding path.** `ClcVAT` computes `(net * vatrate) / 100` into a
   `Packed(11:4)` field (truncating to 4 decimals, not rounding), then
   half-adjusts to 2 decimals with `%DECH`. For a positive net amount this is
   equivalent to half-up rounding of the raw product — e.g. a VAT of 0.065
   rounds to 0.07 — whereas naive half-even (banker's) rounding, as Java's
   default `BigDecimal` rounding or similar, would give 0.06 for the same input.
   `functionsVAT/vat300.rpgle:46-49`.
2. **Unknown VAT code silently prices at 0%.** There is no error raised anywhere
   in ORD101 or VAT300 when `GetArtVatCode` returns a code absent from `VATDEF`
   (seed data: article `000009`, code `9`) — the line simply shows 0 VAT.
   `functionsVAT/vat300.rpgle:61-72`.
3. **Deleted VAT codes stay priceable.** `VATDEL = 'X'` only affects
   `ExistVATRate`; `GetVATRate`/`ClcVAT`/`GetVATDesc` ignore it, so a line on a
   deleted code (seed data: article `000008`, code `3`) continues to price at
   that code's rate with no warning. `functionsVAT/vat300.rpgle:53-59`.
4. **The VATDEF lookup caches the last chained code** in a static buffer
   (`chainVATDEF`), and the buffer is reset to blanks on every miss — worth
   knowing if a rebuild is tempted to add its own cache with different miss
   semantics. `functionsVAT/vat300.rpgle:61-72`.
5. **Error indicators 37 (`ErrQtyLiv`) and 38 (`ErrQty`) are never turned off** in
   ORD101's source — once either quantity check fails once in a run, the
   indicator stays on. Since the DDS shows `ERRMSGID` for a field whenever its
   indicator is on, it only matters while that record format is current (the
   indicator still is tested combined with the live comparison), but there is no
   corresponding `*OFF` anywhere, which is inconsistent with the rest of the
   program's style of explicitly resetting indicators like `dspatr_ri`.
   `QRPGLESRC/ORD101.PGM.RPGLE` (indicators declared lines 48-49, set lines
   248,252, never cleared).
6. **The quantity checks compare against stale, on-file values, not against each
   other's newly typed value.** `DSQTYLIV` is checked against `ODQTY` (the old
   ordered quantity still on file) and `DSQTY` is checked against `ODQTYLIV` (the
   old delivered quantity still on file) — not `DSQTYLIV` against `DSQTY`. A user
   who lowers `DSQTY` and raises `DSQTYLIV` in the same edit so that the new
   delivered quantity exceeds the new ordered quantity, while both stay on the
   correct side of the *old* values, passes validation.
   `QRPGLESRC/ORD101.PGM.RPGLE:245-254`.
7. **Option 6 ("Deliver") is offered and accepted but never implemented.** The
   screen documents it (`QDDSSRC/ORD101D.DSPF:76`) and the option-range check
   accepts it (`QRPGLESRC/ORD101.PGM.RPGLE:152`), but the action subroutine
   `s01act` only branches on options 2 and 4 — typing 6 does nothing (no error,
   no delivery). `QRPGLESRC/ORD101.PGM.RPGLE:177-197`.
8. **A changed line is not saved on the first Enter.** Because of the `CHANGE(27)`
   check in `S02act`, a user who edits a value and presses Enter sees the line
   redisplayed with the recalculated total but **not yet saved**; the update to
   `DETORD1` only happens on a second Enter with no further change. A rebuild
   that saves on the first Enter would diverge from this behaviour.
   `QRPGLESRC/ORD101.PGM.RPGLE:260-273`.
9. **Article description is truncated from 50 to 30 characters on screen.**
   `GetArtDesc` returns the full `ARDESC` (char 50), but `ARTDESC` on both the
   subfile row and the detail screen is declared char 30, so the last 20
   characters of a long description are silently dropped.
   `QDDSSRC/ORD101D.DSPF:18,124`, `QPROTOSRC/ARTICLE.RPGLEINC`.
10. **List totals are not re-derived from current VAT rates.** The order-level
    `tot`/`totvat` shown on the subfile panel are the sums of each line's stored
    `ODTOT`/`ODTOTVAT`, which only get refreshed when that specific line is
    opened and saved through the detail screen — editing one line does not
    recompute the VAT of the others even if they share a VAT code whose rate has
    since changed. `QRPGLESRC/ORD101.PGM.RPGLE:102-117`.
11. **`Prtord` (`ORD500`) is declared but never called** in this program — a
    leftover prototype with no runtime effect. `QRPGLESRC/ORD101.PGM.RPGLE:24`.

## What I could not determine

- The exact precision semantics of the intermediate `Packed(11:4)` truncation in
  `ClcVAT` for **negative** net amounts (credit lines) were not checked against a
  live example; the note above about equivalence to half-up rounding is stated
  for positive amounts only, as instructed.
- Whether `VATCODE 9` (unknown, per seed data notes) or deleted code `3` ever
  reach ORD101 in practice depends on data in `ORDER`/`DETORD` outside this
  slice's normal entry paths (order `60719` per `data/README.md`) — not traced
  further here since order creation (`ORD100`) is out of this slice's scope.
