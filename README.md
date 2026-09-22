# Employment Income Verification — demo

A clickable walkthrough of Click2Check's HMRC-backed income and employment
verification, for showing to prospective customers.

**Live: https://click2checkdev.github.io/HMRC-demo/**

## What it shows

The journey a broker would follow, end to end:

1. A case list for an example customer
2. Raising a verification request
3. The HMRC record lookup
4. The returned report — order details, the personal information collected and
   retrieved, PAYE payroll data, self-employment income, and other income
   sources
5. Exporting the report as a PDF

## What it is not

**Every figure, name and employer in this demo is invented.** There is no real
person's data here, and none has ever been in it.

It is a single self-contained HTML page with no backend, no API calls and no
database. Nothing you click sends anything anywhere. That is deliberate: it can
be shown from a laptop on a train, and it cannot leak anything or bill anything.

It follows the shape of the real product but is not wired to it, so treat the
figures as illustrative rather than as a specification.

## Running it without the internet

Download `index.html` and `logo.png` into the same folder and open the HTML file
in a browser. The PDF export needs a connection, as it pulls jsPDF from a CDN;
everything else works offline.

## Provenance

Originally written by Yusuf Yigit in July 2026. Copied here from the earlier
`Click2Check-dev` organisation so it sits with the rest of the project, with the
original commit history intact.

---

Click2Check Ltd — https://www.click2check.com
