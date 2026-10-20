# Seed data

Invented data for the training. One CSV per table. No real people, companies or addresses.
The header of each file is the DDS field list of the table, in DDS order.
The programs compute their own outputs. Nothing here states what ClcVAT or ORD101 return.

Conventions
- Dates in date fields (VATCREA, ARCREA, FACREA, CUCREA, PRCREA, APCREA) are yyyy-mm-dd. Timestamps (the *MOD fields) are yyyy-mm-dd hh:mm:ss.nnnnnn.
- Order dates (ORDATE, ORDATDEL, ORDATCLO) and CULASTORD are numbers of 8 digits, yyyymmdd, as in the DDS. 0 means not set.
- Ids with leading zeros (ARID, FAID) are text. Keep the zeros.
- An empty cell is a blank field. The delete flag is blank or X.
- Numeric widths: see the DDS. QUANTITY fields are 5 digits, prices 7.2, totals 9.2, VATRATE 4.2.

## VATDEF (key VATCODE), 4 rows
VATCODE, VATRATE, VATDESC, VATCREA, VATMOD, VATMODID, VATDEL.
Codes 1 and 4 are ordinary. Code 2 carries the rate 7.70. Code 3 is flagged deleted (VATDEL X). Code 9 does not exist on purpose.
Note: the RPGUnit test in QTESTSRC loads its own rows (codes 1 and 3) and treats code 2 as unknown. The seed here is separate.

## FAMILLY (key FAID), 4 rows
FAID, FADESC, FAVATCD (default VAT code), FACREA, FAMOD, FAMODID, FADEL.

## COUNTRY (key COID), 4 rows
COID, COUNTR, COISO, COISO5, COISO1. The meaning of COISO5 and COISO1 is not stated in the code, so they hold plausible codes.

## ARTICLE (key ARID), 10 rows
ARID, ARDESC, ARSALEPR, ARWHSPR, ARTIFA, ARSTOCK, ARMINQTY, ARCUSQTY, ARPURQTY, ARVATCD, ARCREA, ARMOD, ARMODID, ARDEL.
- 000006 costs 12.35 and is on VAT code 2 (rate 7.70).
- 000008 is on VAT code 3, the deleted code.
- 000009 is on VAT code 9, which is not in VATDEF.
- 000010 is flagged deleted (ARDEL X) and has no order lines.
- ARCUSQTY equals the sum of ODQTY minus ODQTYLIV over the lines of the article, which is the invariant ORD700 keeps.
- Stock, minimum and purchase quantities are set so that some articles fall below their reorder level and some do not (for PRO202).

## CUSTOMER (key CUID), 4 rows
CUID 1001 to 1004, with names, contact fields (placeholders), CUCOUN referencing COUNTRY, credit fields, CULASTORD equal to the latest order date of the customer.

## PROVIDER (key PRID), 3 rows
PRID 1 to 3, with placeholder contact and address fields.

## ARTIPROV (key APPRID, APARID), 8 rows
Which provider supplies which article, with buy price and the provider reference. The row for article 000008 is flagged deleted.

## PARAMETER (key PACODE, PASUBCODE), 4 rows
PATH (blank subcode) holds an export directory in PARM2, used by PRO202. CURRENCY, DELIVERY/DAYS and DELIVERY/PARTIAL are invented examples of the other columns. A blank subcode is an empty cell.

## ORDER (key ORID), 6 rows
ORID 60714 to 60719 (the data area LASTORDNO in QDTASRC holds 60719), all year 2026.
- 60714: closed (ORDATCLO set), all lines delivered.
- 60715: delivery date set, not closed, mixed delivery.
- 60716: open, ORDATDEL 0. The best order to open in ORD101: it has the 12.35 line and a quantity 0 line.
- 60717: open, ORDATDEL 0, nothing delivered.
- 60718: delivery date set, partly delivered.
- 60719: open, ORDATDEL 0, lines on the deleted and the unknown VAT code.

## DETORD (key ODORID, ODLINE), 20 rows
ODORID, ODYEAR, ODLINE, ODARID, ODQTY, ODQTYLIV, ODPRICE, ODTOT, ODTOTVAT.
- ODTOT equals ODQTY times ODPRICE on every row.
- ODTOTVAT equals ODTOT on every row. That is a placeholder: VAT has not been applied to the seed lines. The programs compute the real value.
- Quantity 0: 60716 line 2.
- Delivered equal to ordered: 60714 lines 1 to 3, 60715 lines 2 and 4, 60718 line 1.
- Delivered above 0 but below ordered: 60715 line 1, 60718 line 2. Option 4 (delete) is refused on every line with delivered above 0.
- Delivered 0: all other lines.
- Article on the VAT code that does not exist: 60719 line 2. On the deleted VAT code: 60719 line 1.
- VAT 7.70 on a net of 12.35: 60716 line 1 (quantity 1, price 12.35).
- A price different from the article price: 60717 line 3.
Agents are expected to add records of their own so that each rule changes an output.
