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
| `PENDING` | not yet submitted to HMRC — see below, it means two different things |
| `AWAITING CONSENT` | waiting for the applicant to authorise at HMRC |
| `POLLING` | consent given, waiting for HMRC to release the record |
| `READY` | HMRC has released it; the report is being retrieved |
| `DELIVERED` | finished, report and PDF available |
| `FAILED` | no payroll match and no consent route |
| `EXPIRED` | 24 hours passed with no response |

`PENDING` covers two unrelated situations, and the demo now says which:

- **A commercial case** is held for manual review before anything is sent to
  HMRC. Nothing is being polled, because nothing has been submitted. The review
  is a step outside the system, so a button stands in for someone completing it
  — the same arrangement as the applicant consenting on GOV.UK.
- **A residential case** is simply waiting for a worker to pick it up, which in
  the live service takes seconds. A case appears in the list at `PENDING` the
  moment it is created, before Equifax has been called at all, which is exactly
  what the real API does.

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

A commercial case can now be followed all the way through: held for review, then
submitted, then consent, then polling, then ready, then a delivered report. It
used to stop at the review and have no way forward.

### The three outcomes

A real case ends up on one of three paths, and nobody chooses which:

- **Instant match** — HMRC already holds a payroll match, and the report comes
  back in seconds with no involvement from the applicant.
- **Consent required** — the usual alternative. The applicant is sent a link,
  signs in to HMRC through GOV.UK with their own Government Gateway
  credentials, and authorises the data share. Only then is the record released.
  Until they act, the case sits at `AWAITING CONSENT` and is re-checked every
  five minutes for up to 24 hours, after which it is closed as `EXPIRED`.
- **No record found** — no payroll match and no consent route, so the case is
  closed as `FAILED`. Usually an NI number, date of birth or surname that does
  not match what HMRC holds.

The **demo control** on the new-case form picks which one to show. It is
labelled as a demo control on screen and is deliberately not styled to look
like part of the product — it exists so all three can be shown on demand
rather than waiting for luck.

**We do not yet know the real split between these three paths.** Until Equifax
confirms it, treat the demo as showing that all three exist, not how often each
happens.

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

Three things in particular are simplifications:

- **"Simulate the applicant consenting"** stands in for something that happens
  on GOV.UK, in the applicant's own time — minutes or hours after the case is
  opened, not seconds.
- **Commercial cases stop at `PENDING`** with a note about manual review. That
  rule came with the original demo and has not been confirmed against the
  product — the gateway has no manual-review path, and `query_category` changes
  nothing about how a case is processed there. Worth settling before a customer
  asks how commercial cases get reviewed.
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
