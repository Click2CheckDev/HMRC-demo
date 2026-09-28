# Employment Income Verification — demo

A clickable walkthrough of Click2Check's HMRC-backed income and employment
verification, for showing to prospective customers.

**Live: https://click2checkdev.github.io/HMRC-demo/**

## What it shows

The journey a broker would follow, end to end:

1. A case list for an example customer, filterable by Residential/Commercial
2. Raising a verification request
3. The HMRC record lookup, which takes one of three paths (below)
4. Any case, at any stage, opened from the list — not just finished ones
5. The returned report — order details, the personal information collected and
   retrieved, PAYE payroll data, self-employment income, and other income
   sources
6. Exporting the report as a PDF

### Every state a case can be in

The case list seeds one of each, so all of them can be shown:

| Status | What it means |
|---|---|
| `PENDING` | created, not yet submitted to Equifax — seconds, in the live service |
| `AWAITING CONSENT` | waiting for the applicant to authorise at HMRC |
| `POLLING` | consent given, waiting for HMRC to release the record |
| `READY` | HMRC has released it; the report is being retrieved |
| `DELIVERED` | finished, report and PDF available |
| `FAILED` | no payroll match and no consent route |
| `EXPIRED` | 24 hours passed with no response |

`PENDING` means one thing: created, and waiting for a worker to submit it to
Equifax. In the live service that is seconds, which is why only one is seeded.
A case appears in the list at `PENDING` the moment it is created, before Equifax
has been called at all — exactly what the real API does. Nothing is being polled
at that point, so no refresh is offered.

Opening a case shows what a broker can actually do about it at that point:

- **Pending** — what it is waiting on, and a way to move it past that
- **Awaiting consent** — the consent link, ready to resend, and a refresh
- **Polling or ready** — what is being waited on, and that it is re-checked
  every five minutes for up to 24 hours
- **Expired** — a *Resume checking* button, which is the real service's
  `POST /orders/<id>/refresh/`. That endpoint only accepts expired orders, which
  is why the button appears nowhere else
- **Failed** — the reason, and a *Start a corrected case* button that carries the
  details over, because most failures are a mistyped NI number or date of birth
- **Delivered** — the report and the PDF export

Refreshing honestly reports no change. It reads state; it does not invent it —
and on a case that has not been submitted yet there is nothing to refresh, so the
button is not offered at all.

A case can be followed all the way through from the list: pending, submitted,
consent, polling, ready, delivered report.

### The three outcomes

A real case ends up on one of three paths, and nobody chooses which:

- **Found at Equifax** — Equifax already holds the person in its own payroll
  data, and the report comes back in seconds with no involvement from the
  applicant.
- **Forwarded to HMRC** — Equifax does not hold them, so the request goes to
  HMRC instead and the person is identified through the **Government Gateway**.
  They are sent a link, sign in with their own credentials, and authorise the
  data share. Until they act, the case sits at `AWAITING CONSENT` and is
  re-checked every five minutes for up to 24 hours, after which it is closed as
  `EXPIRED`.
- **Not found either way** — neither Equifax's payroll data nor the HMRC route
  produces a match, so the case is closed as `FAILED`. Usually an NI number,
  date of birth or surname that does not match what is held.

### Residential and Commercial are the same thing

Worth stating plainly, because an earlier version of this demo said otherwise.
`query_category` is a reporting label. It is **never sent to Equifax**, nothing in
the gateway branches on it, and a commercial case is processed exactly like a
residential one. Its only jobs are filtering the case list and splitting the
counts on the usage endpoint for billing.

There is **no manual review**. An earlier version of this demo held commercial
cases at "pending review" and never produced a report for them, which was
invented rather than observed. It has been removed, and commercial cases now
appear across the same statuses as residential ones.

The **demo control** on the new-case form picks which one to show. It is
labelled as a demo control on screen and is deliberately not styled to look
like part of the product — it exists so all three can be shown on demand
rather than waiting for luck.

**Equifax put the instant match rate at 15&ndash;20%** (confirmed 25 September
2026). So roughly **four cases in five reach the consent step**, which is why the
demo defaults to it. Worth saying out loud when showing this: results are usually
not instant, and the applicant has to be reachable and willing.

Equifax also confirmed the National Insurance number is **not mandatory**, but
supplying it improves the chance of an instant match. Worth collecting for that
reason rather than because anything rejects an order without it.

## What it is not

**Every figure, name and employer in this demo is invented.** There is no real
person's data here, and none has ever been in it. The NI numbers use the `QQ`
prefix, which HMRC never issues, and the mobile numbers are in Ofcom's reserved
`07700 900xxx` drama range.

It is a single self-contained HTML page with no backend, no API calls and no
database. Nothing you click sends anything anywhere. That is deliberate: it can
be shown from a laptop on a train, and it cannot leak anything or bill anything.

It follows the shape of the real product but is not wired to it, so treat the
figures as illustrative rather than as a specification.

Two things in particular are simplifications:

- **"Simulate the applicant consenting"** stands in for something that happens
  on GOV.UK, in the applicant's own time — minutes or hours after the case is
  opened, not seconds.
- **Timings are compressed.** The real service re-checks a pending case every
  five minutes for up to 24 hours. Here the steps take about a second each, so
  the sequence can be shown rather than waited out.

## Data handling in the real service

Not shown on screen, but the question comes up: in the live gateway the
applicant's personal data is encrypted at rest, and cleared on a retention
schedule once it is no longer needed.

## Running it without the internet

Download `index.html` and `logo.png` into the same folder and open the HTML file
in a browser. The PDF export needs a connection, as it pulls jsPDF from a CDN;
everything else works offline.

## Provenance

Originally written by Yusuf Yigit in July 2026. Copied here from the earlier
`Click2Check-dev` organisation so it sits with the rest of the project, with the
original commit history intact.

The consent journey, the failure outcomes and the National Insurance, email and
mobile fields were added in September 2026, so that the demo shows the paths a
real case can take rather than only the one that succeeds instantly.

---

Click2Check Ltd — https://www.click2check.com
