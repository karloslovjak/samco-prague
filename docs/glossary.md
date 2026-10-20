# Glossary

Only terms that occur in the code of this repository. File and line references are to this copy.

## Names in the data

- ODQTYLIV: delivered quantity of an order line (French livre, delivered). Field of DETORD, QDDSSRC/DETORD.PF.
- ODQTY: ordered quantity of the line. ODPRICE: unit price. ODTOT: line total without VAT. ODTOTVAT: line total with VAT.
- ORDATDEL: delivery date of an order. 0 means no delivery date yet. ORDATCLO: close date. 0 means the order is open. Both are 8 digit numbers written yyyymmdd (QDDSSRC/ORDER.PF).
- FAMILLY: the product family file (the misspelling is upstream). FAID is the family id. QDDSSRC/FAMILLY.PF.
- ARTIFA: the family id on an article (QDDSSRC/ARTICLE-Article_File.PF).
- ARCUSQTY: quantity ordered by customers and not yet delivered, kept on the article. Maintained by ORD700.
- ARPURQTY: quantity on purchase orders to providers. ARMINQTY: minimum stock. ARSTOCK: stock.
- ARVATCD: the VAT code of an article. VATCODE is the key of VATDEF. VATRATE is the percentage.
- VATDEL, ARDEL, FADEL, CUDEL, PRDEL, APDEL: delete flag. X means deleted. Rows are flagged, not removed (ART200 line 207).
- DLCODE: the reference field all delete flags are defined from (common/SAMREF.PF).
- LASTORDNO: data area holding the last order number (QDTASRC/LASTORDNO.DTAARA).
- LIV (in ODQTYLIV) and CLO (in ORDATCLO): short for livre (delivered) and clos or closed.
- OPT01: the option field of the subfile row. The user types 2, 4 or 6 in it (QRPGLESRC/ORD101.PGM.RPGLE line 152).
- DSQTY, DSQTYLIV, DSPRICE: the editable copies on the detail screen FMT02 of ORD101.
- ORYEAR and ODYEAR: the year part of an order key.

## Indicators as used here

Indicators are named switches. In ORD101 they are mapped to names in the indds data structure (lines 30 to 50).
- 30 sflclr: clear the subfile. 31 sfldsp: display the subfile. 32 sfldspctl: display the subfile control.
- 33 sflnxtchg: mark the row as changed so the next read of changed rows returns it.
- 34 dspatr_ri: reverse image on the row, for an error.
- 35 sflmsg: message "Invalid Option". 36 sflmsg2: message "Line with delivery can not be deleted."
- 37 ErrQtyLiv: shows ERR1001. 38 ErrQty: shows ERR1002. ORD101 sets both on and never sets them off.
- 80 sflend: end of the subfile data, also controls PAGEDOWN in the display file.

## RPG and IBM i terms

- Packed decimal: digits stored two per byte with a sign, for example 9P 2 is 9 digits with 2 decimals. Zoned decimal: one digit per byte, written 9S 2 or just 9 2 in DDS.
- Subfile: a list on a green screen, loaded row by row (WRITE), displayed with EXFMT, read back with READC for changed rows.
- DDS: data description specifications. The source language for files and screens.
- PF: physical file, the table. LF: logical file, a view over a PF with a different key. DSPF: display file, a screen.
- Key: the fields a keyed file is ordered by. ORDER1 is keyed on ORID (QDDSSRC/ORDER1.LF).
- CHAIN: read one record by full key. SETLL: position at a key. READE: read the next record with an equal key. Used in ORD101 lines 105 to 113.
- %EOF and %FOUND: tell whether the last read hit the end or found a record.
- %DECH: convert to a decimal with half-adjust (rounding). Used in VAT300 line 49.
- Trigger: a program the database calls on insert, delete or update of a file. ORD700 on DETORD (QTRGSRC/ORD700*.SYSTRG).
- Data area (dtaara): a small named storage object. ORD100 line 53 declares one, and reads and writes it with IN and OUT.
- Service program and binding directory: a callable module and the list that finds it. ORD101 line 2 names the directory SAMPLE. vat300.rpgle is a nomain module (line 10), so it has no entry point.
- Prototype (PR): the declaration of a procedure so another source can call it. The copy members in QPROTOSRC hold them.
- INFDS: file information data structure. ORD101 line 28 reads the subfile row number from it.
- RPGUnit: the unit test tool used in QTESTSRC.
- ILE: the module and binding model the newer sources use (service programs, binder source in QSRVSRC).
