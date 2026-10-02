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
| `PENDING` | created, not yet submitted to Payroll — seconds, in the live service |
| `AWAITING CONSENT` | waiting for the applicant to authorise at HMRC |
| `POLLING` | consent given, waiting for HMRC to release the record |
| `READY` | HMRC has released it; the report is being retrieved |
| `DELIVERED` | finished, report and PDF available |
| `FAILED` | no payroll match and no consent route |
| `EXPIRED` | the consent window closed with no response |

`PENDING` means one thing: created, and waiting for a worker to submit it to
Payroll. In the live service that is seconds, which is why only one is seeded.
A case appears in the list at `PENDING` the moment it is created, before Payroll
has been called at all — exactly what the real API does. Nothing is being polled
at that point, so no refresh is offered.

Opening a case shows what a broker can actually do about it at that point:

- **Pending** — what it is waiting on, and a way to move it past that
- **Awaiting consent** — the consent link, ready to resend, and a refresh
- **Polling or ready** — what is being waited on, and that it is followed
  automatically until the consent link expires
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

### Six test applicants, not an outcome switch

The demo used to ask which outcome a case should take. That was its biggest
remaining fiction: **nobody chooses.** What happens follows from what Payroll and
HMRC hold about the person.

So there is a list of test identities instead, each with their own records.
Picking one fills the form, and every field stays editable. The label says who they
are and the rest is left to the person demonstrating: a paragraph of commentary on
the order form is not what a customer is there to read.

| Applicant | What their records produce |
|---|---|
| **Marcy Okonjo** | Payroll does not hold them, so the request goes to HMRC and waits on the applicant. **This is the usual case** — around four in five. |
| **Alan Pettifer** | An instant match in Payroll. No HMRC, no consent, report in one step, and this happens for 15–20% of applicants. |
| **Dev Ramanathan** | Self-employed. Consent is still needed, and the report that comes back has **no employment section at all**. |
| **Josie Hartnell** | No match at Payroll or HMRC. Closes as `FAILED`. Usually an NI number, date of birth or surname that does not match what is held. |
| **Imran Chaudhary** | Commercial. A director paid in dividends rather than wages, so a payroll lookup finds nothing. Follows a **company link** of his own, and the report comes back with dividends and shareholdings instead of employment. |
| **Rosalind Whitaker** | Residential **and** commercial. Employed and a director, so both apply: she receives **two links** and has to follow both. |

An applicant who never gets round to it is not shown as a test identity: nothing
Click2Check does changes that outcome, and the `EXPIRED` status is already in the
seeded case list for anyone who wants to see it.

### The report shows only what HMRC hold

Five different report shapes, because the report only ever carries what HMRC hold
about that particular person:

- **Employment only** — payroll records and tax totals. No self-employment or
  other-income sections, because there is nothing to put in them.
- **Employment and a business** — the shape of the provider's own sample report,
  which is where the employment and self-assessment figures come from.
- **Self-employed** — no employment section, and no *Retrieved Personal
  Information* section either. A sole trader has no employer record, and that
  record is the only thing HMRC enrich the name and address from.
- **Commercial** — dividends and shareholdings, and no employment at all, because
  a director paid in dividends has no payroll record to find.
- **Residential and commercial** — employment, dividends and shareholdings
  together.

Empty sections are **absent, not blank**. A heading over an empty table reads as
missing data; a section that is not there reads as not applicable, which is what
it is. The live report does the same.

### The applicant's journey, shown rather than described

There used to be an "Applicant consent required" card explaining this step to a
broker in three bullet points. The step itself is more convincing, so the demo
shows it: **the applicant's inbox**, with the email landing in it.

The email is personalised, addressed to them by name, naming the client who asked
and why in the applicant's own terms rather than the broker's regulatory label,
and carrying their reference and the expiry date. It arrives a moment after the
inbox opens, unread, among mail that was already there, because that is the moment
the whole journey depends on.

Worth showing because we ask for the link rather than having the data provider
send it, so this is Click2Check's email in Click2Check's words. It is the first
thing an applicant ever sees of us, and whether they act on it decides whether the
case completes at all.

The mobile number stays on the form as an optional field, because the API accepts
one, but the demo makes no claim about texting the link. **Nothing in the service
sends SMS**, and a demo that implies otherwise is promising a channel that does not
exist.

The email **lists what they will need before they start**: the Government Gateway
user ID, the password, and the phone or authenticator app HMRC send the access
code to. A director doing both legs is told the company's credentials are separate
from their own. An applicant who gets three screens in and then cannot find their
user ID abandons it, and an abandoned consent is a case that expires.

It also says what to do if they do not know their details. **HMRC can recover a
user ID or reset a password; Click2Check cannot.** And if they have never had a
Government Gateway account at all, they have to create one, which can take HMRC a
few days to confirm — so the email says to start early rather than close to the
deadline.

> Worth saying out loud when showing this: **the consent path assumes the
> applicant has a Government Gateway account.** Self-employed people generally do.
> A PAYE-only employee often does not, and for them the consent window is not the
> binding constraint — HMRC's identity check is. That is a real risk to the four
> cases in five that need consent, and it is a conversation to have with a
> prospective customer rather than a surprise later.

Pressing the link opens the page it goes to, **inside a browser frame with the
address bar visible**, because "where does that link actually take them" is the
question clients ask and the address answers it better than the page does.

**The consent window is 120 hours, not 24.** Measured rather than assumed: the UAT
order of 2 October was created at 12:05 and the applicant's email said "Expires on:
2026-10-07 12:05 PM", five days to the minute. The demo said 24 hours everywhere
before that, which was *our own polling window* rather than theirs — so the demo
and the applicant's own email contradicted each other. The case detail now shows
the real date and says the applicant has been given it.

**HMRC's status values are shown on the case**, in the casing their API actually
returns, with what each means to us:

| Value | What it means |
|---|---|
| `In Progress` | waiting on the applicant, or on HMRC releasing the record |
| `Ready` | released; the report is fetched next |
| `Downloaded` | **also fetchable** — not a sign the report has gone |
| `Delivered` | fetchable too, and **absent from their written list** |
| `Expired` | the window closed with no authorisation |

Two of those are worth a client's attention. `Downloaded` reads as though the
report has already gone somewhere; treating it as unavailable would abandon a case
whose report is sitting there ready. And `Delivered` was not in the list the
provider sent on 25 September — their API returned it the same afternoon.

That page is styled as a government service and asks for the three things HMRC
ask for: the Government Gateway user ID, the password, and the access code HMRC
send, and states what approving actually shares.

The fields arrive **pre-populated**, so it reads as a form somebody has filled in
rather than empty boxes, and the recovery routes HMRC offer are shown — *forgotten
your user ID*, *forgotten your password*, *not received your access code*.

It is deliberately **not a replica.** No crown, no GOV.UK wordmark, a simulation
banner above it, every field **readonly**, no form element, nothing to submit, and
**none of those recovery routes is a working link.** Operable recovery flows are
what make a copy of a credential form convincing, which is the reason they are
text. The guidance an applicant actually needs is in the email, which tells them
HMRC can recover a user ID or reset a password and that a new account takes days. A working copy
of a government sign-in page asking for a user ID, password and access code is a
phishing kit whatever it was built for, and this repository is public.
Recognisable is the useful part; convincing is not. There are checks for the
absence of those things, not only the presence of the right ones.

Once authorised, the case moves `AWAITING CONSENT` → `POLLING` → `READY` →
`DELIVERED` on its own. `READY` is brief but real: one background task sees that
HMRC has released the record, and a second fetches it.

The broker's own need did not disappear with the card. The consent link, and a
button to copy it, are on the case in the list — which is where a broker goes when
an applicant says they never received it.

### The commercial journey is a separate link

This is the part most worth understanding, and the newest.

A director typically takes income as **dividends rather than wages**, so a payroll
lookup finds nothing at all. The company's records sit behind the **company's own
Government Gateway account**, not the director's personal one. So it is a separate
authorisation on a separate link, and the report comes back with different
sections:

| | Residential | Commercial |
|---|---|---|
| Link | `/individual/…` | `/company/…` |
| Account label | *Gateway Account — Personal* | *Gateway Account — Commercial* |
| Signs in as | themselves | the organisation |
| Report shows | employment, self-employment, other income | **dividends, shareholdings** |

Somebody who is **both** employed and a director gets **two links in one email**
and has to follow both. The demo holds this: authorising one marks it done and
returns them to the inbox for the other, and the case does not complete until both
are in. That is the behaviour the separate link exists to represent, and it is the
thing a client will plan around.

### What Residential and Commercial do and do not share

Two different things get called the same name here, and it is worth separating
them.

**`query_category` is a reporting label.** It is never sent to the data provider,
nothing in the gateway branches on it, and its only jobs are filtering the case
list and splitting the counts on the usage endpoint for billing. That has not
changed.

**The commercial journey is genuinely different**, as above: its own link, the
organisation's Government Gateway account, and dividends and shareholdings in
place of employment. An earlier version of this demo said commercial and
residential were the same in every respect. That was true of the label and wrong
about the journey.

There is still **no manual review**. An earlier version held commercial cases at
"pending review" and never produced a report for them, which was invented rather
than observed. Nothing at Click2Check sits between the applicant and their report.

> **For whoever maintains this:** the commercial journey is **not implemented in
> the HMRC gateway yet.** The gateway sends an individual's NI number, date of
> birth, name and address, and has no company path. The data provider has not
> documented one either — their onboarding document and sample report contain no
> reference to organisation accounts, self-assessment or scope selection. The
> journey shown here is C2C's product direction, specified on 2 October 2026, and
> the dividend and shareholding figures are the only invented numbers in this
> demo. Everything else comes from the provider's own sample.

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

- **"Simulate the applicant consenting"** stands in for something the applicant
  does in their own time — minutes or hours after the case is opened, not
  seconds. Opening the consent link shows their side of it.
- **Timings are compressed.** The real service follows a pending case for up to
  five days. Here the steps take about a second each, so the sequence can be shown
  rather than waited out.

### On a phone

The page had no media queries at all, which with a viewport meta tag set is the
worse of the two failures: the browser renders at device width and everything
overflows, rather than zooming out to something legible.

Two breakpoints now. 820px for tablets and narrow windows, 560px for phones.

| | |
|---|---|
| Sidebar | becomes a scrollable strip above the content. 300px of navigation on a 390px screen leaves nothing for the page |
| Form, stats, detail grid | drop to one column |
| Inbox | the reading pane goes under the message list rather than beside it |
| **Case list** | **stacks** into one card per case, each cell labelled |
| **Report tables** | **scroll** sideways inside the card |

The last two are deliberately different. The case list is the landing view and a
horizontally scrolling list is a list nobody reads, so it stacks and each cell
carries its own label in place of the hidden header row. A year-by-year income
table does not stack into anything comparable — the columns have to stay side by
side — so those scroll instead.

Checked as far as it can be: the rules parse, sit in the right media query, and
the markup the stacked layout depends on is emitted. **None of that proves it
looks right** — jsdom does no layout, so the only real test is opening it on a
phone.

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

Every figure in the report comes from the data provider's own sample report,
except the dividends and shareholdings, which are invented because they gave us
no commercial example. Only the names
are substituted: applicants, employers and businesses are invented, the
National Insurance numbers use the `QQ` prefix HMRC never issues, and the mobile
numbers are in Ofcom's reserved drama range. None of it can belong to a real
person or company.

---

Click2Check Ltd — https://www.click2check.com
