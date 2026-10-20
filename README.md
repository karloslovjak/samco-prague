# SAMCO training copy

A pruned copy of SAMCO, an order-management sample application written in RPG for IBM i.
It is used as the working repository for the Agent Studio training day in Prague.

## The application in five lines

SAMCO keeps articles, families, customers, providers, VAT rates and parameters in physical files.
Customer orders are an order header (ORDER) and its lines (DETORD).
Green-screen programs (display files with subfiles) maintain these files.
VAT is calculated by one service program, VAT300.
A trigger on DETORD keeps the quantity ordered by customers on the article in step.

## Licence

Apache-2.0, from the upstream project. See LICENSE and NOTICE.
The ARCAD copyright and trial-licence comment lines in the sources are kept on purpose.

## How the training day uses it

Each team works on its own private copy of this repository.
The day follows five functions of the code, one after the other.
Today's function is the order-line edit ORD101 with the VAT calculation VAT300, then the stock trigger ORD700.
The follow-on functions are PAR200, PRO202, ORD200 and ORD201, and ART200.
The agents read the RPG, write the business rules and a golden master, then rebuild the logic in Java and a screen in React.

The functions, their files and what is out of scope are in docs/scope.md.
Read docs/reading-rpg.md first if the code is new to you, and docs/glossary.md for the names.
Invented seed data is in data/ (see data/README.md).

Nothing here can be compiled or run in the training room. The code is read, not executed.
