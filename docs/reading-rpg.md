# Reading this RPG

Short and factual. Each point has a line from a file in this repository.

## Fixed form and free form

Most programs are fixed form: each line has a specification letter in column 6.
H is control, F is files, D is definitions, C is calculations, P marks a procedure.
ORD101 line 2 is an H line (`h dftactgrp(*no) bnddir('SAMPLE')`), lines 5 to 7 are F lines, lines 16 to 59 are D lines.
Column positions matter in these lines.

The logic sits in a free-form block. In ORD101 it starts at `/free` (line 65) and ends at `/end-free` (line 287), and there the lines are ordinary statements ending in a semicolon.
ORD700 has no free block. Its logic is in C lines such as `c eval pn = %addr(parm1) + newoff` (line 71).
functionsVAT/vat300.rpgle is fully free: it starts with `**FREE` on line 1 and uses `Dcl-Proc`, `Dcl-Pi` and `Ctl-Opt`.

## Files and keys

The F lines declare files. In ORD101, `forder1 uf e k disk` (line 5) means update-capable, externally described, keyed, a disk file. `ford101d cf e workstn` (line 7) is the display file.
The record layout comes from the DDS, not from the program. The fields of DETORD are named in QDDSSRC/DETORD.PF and appear in the program as variables without a declaration.
Keys are defined in the DDS. DETORD1.LF is keyed on ODORID and ODLINE. ORD101 reads with `setll id detord1` and `reade(n) id detord1` (lines 105 and 106) which returns the lines of one order.
`chain (id:odline) detord1` (line 216) reads one line by the full key. `%eof` and `%found` test the outcome (lines 107 and 151).

## Record-level access and SQL

Most programs read and write records by key as above. Some also embed SQL, in `exec sql` blocks, and are named .SQLRPGLE. ORD200 declares a cursor on the ORDERCUS view at line 102.
PRO202 selects with SQL (line 82 onward) and writes nothing.

## Indicators

An indicator is a boolean switch used to talk to the display file. ORD101 gives them names in the `indds` data structure at lines 30 to 50, for example `sflclr 30 30n`.
Setting `sflclr = *on` (line 94) turns indicator 30 on, which the display file uses to clear the subfile (`A 30 SFLCLR` in QDDSSRC/ORD101D.DSPF).
`*inlr = *on` (line 285) ends the program.

## Packed decimals and truncation

Numbers have a declared number of digits and decimals. `Packed(11:4)` in vat300.rpgle line 46 holds 11 digits with 4 decimals.
Assigning a value with more decimals to such a field drops the extra digits, it does not round. Line 48 assigns `(net * vatrate) / 100` to that field.
Rounding is explicit: `%dech(tot : 9 : 2)` on line 49 rounds half up to 2 decimals.
Field sizes come from DDS. QUANTITY in common/SAMREF.PF is 5 digits, so ARCUSQTY cannot hold more than 99999.

## Display files and subfiles

A DSPF defines record formats, each one a screen part. A subfile record format has the keyword SFL and its control format SFLCTL (QDDSSRC/ORD101D.DSPF, `R SFL01 SFL` and `R CTL01 SFLCTL(SFL01)`).
The program clears the subfile (`write ctl01`, line 95), writes one row per line (`write sfl01`, line 112), shows it with `exfmt ctl01` (line 124), and reads the rows the user changed with `readc(e) sfl01` (line 150).
The relative row number is `rrn01`. The option the user typed is `opt01`.
The program is a small state machine: `step01` holds prp, lod, dsp, key, chk or act (lines 53 to 64 and 75 to 89), and each step is a subroutine.

## Triggers

A trigger is attached to a file outside the program. QTRGSRC/ORD700U.SYSTRG attaches ORD700 to DETORD after an update.
The database passes a buffer. ORD700 maps it with a data structure (lines 11 to 45) and finds the old and new record by offsets: `eval pn = %addr(parm1) + newoff` (line 71).
`teven` tells the event: 1 insert, 2 delete, 3 update (lines 70, 73, 80).

## Procedures and prototypes

A callable procedure is declared in a prototype (PR) in a copy member and brought in with `/copy`. ORD101 copies `vat` (line 14), which holds the prototype of ClcVAT (QPROTOSRC/VAT.RPGLEINC).
The body is in a different source, here vat300.rpgle line 40.
