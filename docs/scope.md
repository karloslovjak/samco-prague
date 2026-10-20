# Scope

All paths are relative to the repository root. Every file listed exists in this copy.

## Today's function: ORD101 with VAT300

Edit the lines of one customer order, with the VAT calculation.

- QRPGLESRC/ORD101.PGM.RPGLE: the program (288 lines). Reads ORDER1, updates DETORD1, reads ARTICLE1 and CUSTOMER1 through the copy members.
- QDDSSRC/ORD101D.DSPF: the screen (subfile of lines, detail format FMT02).
- functionsVAT/vat300.rpgle: GetVATRate, GetVATDesc, ClcVAT, ExistVATRate (81 lines).
- functionsVAT/vatdef.pf: the VATDEF file. functionsVAT/fvat.bnd: its exports.
- QPROTOSRC/VAT.RPGLEINC, ARTICLE.RPGLEINC, CUSTOMER.RPGLEINC: prototypes the program copies in.
- QRPGLESRC/ART300-Function_Article.RPGLE and CUS300.RPGLE: the article and customer lookups the program calls.
- QDDSSRC/ORDER.PF, ORDER1.LF, DETORD.PF, DETORD1.LF, ARTICLE-Article_File.PF, ARTICLE1-Article_File.LF, CUSTOMER.PF, CUSTOME1.LF; common/SAMREF.PF (field definitions).
- QMSGSRC/SAMMSGF.MSGF: messages ERR1001 and ERR1002.
- QTESTSRC/vat300.test.sqlrpgle: the RPGUnit test of VAT300.

## Then: ORD700, the stock trigger

- QRPGLESRC/ORD700.PGM.RPGLE (107 lines): trigger program on DETORD. Keeps ARCUSQTY on the article in step with insert, delete and update of a line.
- QTRGSRC/ORD700A.SYSTRG, ORD700D.SYSTRG, ORD700U.SYSTRG: the insert, delete and update trigger definitions.
- QRPGLESRC/LOG300.RPGLE, QPROTOSRC/LOG_functions.RPGLEINC, APICALL-Prototypes_for_Ibm_API.RPGLEINC: the log call it makes.
- It reads the trigger buffer by byte offsets. It is the hardest file to read.

## Follow-on functions

PAR200, parameter maintenance.
- QRPGLESRC/PAR200.RPGLE, QDDSSRC/PAR200D.DSPF, QDDSSRC/PARAMETER.PF, QRPGLESRC/PAR300.RPGLE, QPROTOSRC/PARAMETER.RPGLEINC.
- Keyed create, read, update and delete of the PARAMETER file through a subfile. Needs only that file.

PRO202, reorder proposal.
- QRPGLESRC/PRO202.SQLRPGLE, QDDSSRC/PRO202D.DSPF, QRPGLESRC/PRO200.RPGLE (the caller), QRPGLESRC/XML001.RPGLE, QRPGLESRC/TXT001.RPGLE, QPROTOSRC/XML.RPGLEINC, txt.rpgleinc, QSRVSRC/XML.BND, TXT.BND.
- Lists the articles of one provider whose stock is below the reorder level, proposes a quantity, and can write an XML file. Needs ARTICLE, ARTIPROV, PROVIDER, PARAMETER (the PATH parameter) and the XML001 writer, so XML001 stays with it.

ORD200 and ORD201, order list and lifecycle.
- QRPGLESRC/ORD200.PGM.SQLRPGLE, ORD201.PGM.SQLRPGLE, ORD202.PGM.RPGLE, QDDSSRC/ORD200D.DSPF, ORD201D.DSPF, ORD202D.DSPF, QSQLSRC/ORDERCUS.VIEW, QSQLSRC/ISOTODATE4.SQLUDF, QRPGLESRC/DAT002.RPGLE.
- List the orders of customers and offer edit, delete, display, close and deliver. Need ORDER, DETORD, CUSTOMER and the ORDERCUS view. ORD201 is the version to read; compare with ORD200.

ART200, article maintenance.
- QRPGLESRC/ART200-Work_with_article.PGM.SQLRPGLE, QRPGLESRC/ART201-Work_with_article.PGM.RPGLE, QRPGLESRC/ART202-Function_Article.PGM.RPGLE, QRPGLESRC/ART301.SQLRPGLE, QRPGLESRC/ART302.SQLRPGLE, QDDSSRC/ART200D-Work_with_Article.DSPF, QDDSSRC/ART201D-Work_with_Article.DSPF, QDDSSRC/ART202D-Work_with_Article.DSPF, QDDSSRC/ART301D-Function_Select_an_article.DSPF, QDDSSRC/FAMILLY.PF, QDDSSRC/FAMILL1.LF, QSQLSRC/ARTIINF.TABLE.
- Create, change and soft-delete articles, with next id, family lookup, stamps and free text. Needs ARTICLE, FAMILLY, ARTIPROV and PROVIDER.

## Also kept because the functions depend on them

- QRPGLESRC: ORD100.PGM.RPGLE (order creation, which uses the LASTORDNO data area), ORD900, ORD901, FAM300, FAM301, COU300, COU301, CUS300, CUS301, PRO300, LOG100, DAT001.
- QDTASRC/LASTORDNO.DTAARA, SQL objects: QSQLSRC/ORD701.SQLTRG, ART801.SQLPRC, ARTLSTDAT.VIEW, ISOTODATE.SQLUDF.
- QCLSRC, QCMDSRC, QILESRC, QILESRVSRC, QSRVSRC, QBNDSRC: build-time source for the service programs and CL helpers.
- QTESTSRC/cus300.test.sqlrpgle and testing.json.

## Out of scope

- Printing: ORD500 is called by ORD100, ORD101, ORD200 and ORD201 but its source is not in the upstream. Its printer file and CL wrapper were removed.
- CUS200: a display file exists (QDDSSRC/CUS200D.DSPF) but there is no program.
- Removed upstream material: C and C++ sources, the sources in other languages, RPG/400 cycle programs, globalization samples, menus and help panels.
