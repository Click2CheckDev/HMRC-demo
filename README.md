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

### Five test applicants, not an outcome switch

The demo used to ask which outcome a case should take. That was its biggest
remaining fiction: **nobody chooses.** What happens follows from what Equifax and
HMRC hold about the person.

So there is a list of test identities instead, each with their own records.
Picking one fills the form, and every field stays editable.

| Applicant | What their records produce |
|---|---|
| **Marcy Okonjo** | Equifax do not hold them, so the request goes to HMRC and waits on the applicant. **This is the usual case** — around four in five. |
| **Alan Pettifer** | An instant match in Equifax's own payroll data. No HMRC, no consent, report in one step. Equifax put this at 15–20%. |
| **Dev Ramanathan** | Self-employed. Consent is still needed, and the report that comes back has **no employment section at all**. |
| **Josie Hartnell** | No match at Equifax or HMRC. Closes as `FAILED`. Usually an NI number, date of birth or surname that does not match what is held. |
| **Ewan Blaylock** | The consent link is sent and nothing happens. After 24 hours of five-minute checks the case closes as `EXPIRED`, and can be resumed by hand. |

Ewan Blaylock reaches exactly the same screen as Marcy Okonjo, which is the
point: a broker cannot tell in advance which of the two they are dealing with.
That is why the 24-hour window exists.

### The report shows only what HMRC hold

Three different report shapes, because three different people:

- **Employment only** — payroll records and tax totals. No self-employment or
  other-income sections, because there is nothing to put in them.
- **Employment and a business** — the shape of Equifax's own sample report,
  which is where every figure in this demo comes from.
- **Self-employed** — no employment section, and no *Retrieved Personal
  Information* section either. A sole trader has no employer record, and that
  record is the only thing HMRC enrich the name and address from.

Empty sections are **absent, not blank**. A heading over an empty table reads as
missing data; a section that is not there reads as not applicable, which is what
it is. The live report does the same.

### Consent, step by step

The applicant signs in with their **Government Gateway ID, password and the
two-factor code** HMRC send them. Click2Check never sees any of it.

There is **no address step**, which is worth saying because it is easy to assume
otherwise: HMRC identify the person from the Government Gateway account itself.
The address on the order form is used for the Equifax lookup, not for consent.

Once they authorise it, the case moves `AWAITING CONSENT` → `POLLING` → `READY` →
`DELIVERED` on its own. `READY` is brief but real: one background task sees that
HMRC has released the record, and a second one fetches it.

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

Equifax also confirmed the National Insurance number is **not mandatory on their
side** — a request without one is accepted — but supplying it improves the chance
of an instant match.

The form requires it anyway, and that is **Click2Check's rule rather than
Equifax's**. The field says nothing beyond being required, which is the right
amount: an earlier version of this demo claimed "No verification is possible
without it", and that was simply untrue. If anyone asks on a call, the honest
answer is that we always collect it because it saves chasing the applicant for
consent, not because anything rejects an order without it.

### Data retention and deletion

The **Data retention** section in the sidebar. Personal data is deleted after
six months; the income figures are kept, so historic reporting still works.

The split runs through the middle of one record, so the demo shows what a case
looks like on the other side of it. One of the seeded cases is old enough to
have already been swept: open it and the report still renders, with the figures
intact and "-" where the name, date of birth and address were. The employer and
business names are gone too, because a sole trader's business name is routinely
their own name.

Three ways a deletion happens, all of which the section demonstrates:

| | |
|---|---|
| **One case** | From the case itself, or by picking it from the list |
| **A date range** | Every finished case opened between two dates, counted before anything is touched |
| **Automatically** | After six months, nightly |

Four things the demo deliberately shows failing, because a screen that only
shows the happy path implies the guards are not there:

- **No reason, no deletion.** Every deletion records who asked and why, so a
  blank reason is refused. Whitespace too.
- **An operator cannot delete.** Switch the role at the top of the section. The
  button greys out, and the deletion itself refuses as well, which is the half
  that matters.
- **A case still running cannot be deleted.** Clearing one mid-flight would be
  undone by the next status check, which fetches the report and writes the
  details straight back in.
- **The case survives.** Deleting the data does not remove the case, so a
  client's invoice history is never rewritten. The total on the portal does not
  move.

Every deletion lands in the **deletion record** at the bottom: when, which
case, who, why, how it was triggered, and a batch reference that groups a bulk
action into one decision. In the live service that table is append-only and
read-only even to an administrator, and it outlives the cases it describes.

In the live gateway the applicant's personal data is also encrypted at rest.

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
real case can take rather than only the one that succeeds instantly. In October
the outcome selector was replaced by a list of test applicants, and the retention
section was added.

Every figure in the report comes from Equifax's own sample report. Only the names
are substituted: applicants, employers and businesses are invented, the
National Insurance numbers use the `QQ` prefix HMRC never issues, and the mobile
numbers are in Ofcom's reserved drama range. None of it can belong to a real
person or company.

---

Click2Check Ltd — https://www.click2check.com
