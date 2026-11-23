# BBR Dashboard

A single-page dashboard for Bodies by Ryan. It is one HTML file with no build step and no dependencies.

Live site: https://bodiesbyryan-dashboard.vercel.app (Vercel project `bbr-dashboard` on the D2E team; every push to `main` redeploys). The old address bbr-dashboard-nu.vercel.app redirects here.

Until a `data.json` file is added, the page shows **sample data** (marked with a yellow "Sample data, not live" badge).

## Tabs

| Tab | Who | What it shows |
| --- | --- | --- |
| Overview | Admin | Sales collected vs. the monthly goal, recurring revenue, churn, collections due, money in vs. out, sign-ups, weekly sales by account, week-on-week and month-on-month, ROAS, and the **Reports** (revenue by account, fee income, monthly sales, payments not collected, upcoming collections, transfers, Slack and Stripe mismatches). The Week / Month / Year and account filters apply here only |
| Notifications | Admin | The Monday meeting banner with the Google Meet link, and every item that needs action. An **orange dot** on the tab means there is something to check, or the Monday meeting is today |
| Sales | Admin | **+ Sign-up** for Ryan and Amy. Total leads, calls booked, showed up, converted, revenue from calls; sales funnel; leads by source; calls booked by month (booked, showed, no-show, rescheduled, sign-ups); conversion rate by source; payment collected from new clients by start month (with Edit to confirm the amount received); and the **Offers** list and split-payment builder |
| Bookings | Outreach + admin | **+ New lead** and **+ Book a call**; who to remind (calls in the next 48 hours); calls that need an outcome (showed, no-show, reschedule, cancelled); upcoming calls by day; recent calls; new leads |
| Memberships | Admin | Onboarding (signed up, not started), Offboarding (termination notice running), Suspended (active and upcoming freezes, pauses and suspensions), Terminated, plus everything coming up in the next 2 months. Sort by date added or start date |
| Client Check-ins | Coaches + admin | Traffic-light health check: At risk / Watch / On track / Not set, updated by coaches; filters by coach, status and last update; check-ins due this week (weekly, on the weekday each client started) |
| Clients | Admin | Every client in one list; click a client for details, payments and notes |

The **USD rate** at the top (1 AUD = … USD) is the European Central Bank's daily reference rate, fetched from the free Frankfurter API (`api.frankfurter.dev`) and refreshed every 6 hours. All amounts in the dashboard are AUD.

## Manual entries

- **+ New lead** (Bookings): date, name, contact, source, interest, notes.
- **+ Book a call** (Bookings): lead name (a new name is also added as a lead), call date and time, contact, source, notes. Afterwards: **Mark reminded**, record the outcome, or **Reschedule** (counts how many times a call moved).
- **+ Sign-up** (Sales, Memberships): sign-up date, start date, client, email, Instagram, program, coach, type, account, contract value, first payment, payment type, number of instalments, closed by, lead source, notes. For instalments, Ryan or Amy enters the first payment; later payments will be matched from Stripe once it's connected.
- **Edit** on a new client's payment (Sales): amount received, date received, confirmed.
- **+ Termination / freeze / pause** (Memberships).
- **Check-in update** (Client Check-ins): click a client's light, choose On track / Watch / At risk and add a note.

**Demo limitation:** entries are saved in the browser where they were typed (localStorage). Other people and other devices won't see them until the shared database is set up. **Export → Data file (JSON)** includes them.

## Clients

The Clients tab joins every record for a person into one client: their gym membership (`members[]`), custom and online programs (custom offers named `[Person] – [Program] – n/total`, with instalment offers for the same person and program grouped into one contract) and sign-ups. Records are matched by email first, and by name only when one of them has no email, so two people with the same name stay separate. Add `email` to members and sign-ups for reliable matching.

Clicking a client opens their details:
- Status, type, how long they've been a client, email, Instagram, home gym, cross-gym access
- Membership, price, billing and member-since date; lost cash flow if suspended
- Sign-up date, who closed it and the lead source
- Terminations, freezes and pauses
- Programs, contract values and every payment (collected, not collected, scheduled)
- **Notes:** add a note (Update, Change, Payment or Other) for changes or updates. Each note saves who wrote it and when, newest first

## Monday meeting banner

The weekly team meeting is **every Monday at 8:00 am Perth time** (UTC+8, no daylight saving). The banner and its **Join Google Meet** button live at the top of the **Notifications** tab. On Monday before and during the meeting, the Notifications tab shows the orange dot; admins still land on Overview when they sign in. To change the link, time or length, edit `MEETING` in `index.html`.

## Notifications

The Notifications tab lists everything that needs action. Each group has its own colour, and every heading and line can be clicked:

| Notification | Colour | Click goes to |
| --- | --- | --- |
| Payments not yet collected | Red | The client's Instagram chat (`ig.me/m/<handle>`, new tab). No handle: the "Payments not yet collected" table in Overview → Reports |
| Clients at risk | Orange | Client Check-ins, At risk section |
| Slack and Stripe mismatches | Magenta | The Slack post (`slackPosts[].permalink`) or the Stripe payment. No link: the mismatches table in Overview → Reports |
| Transfers between accounts | Green | "Transfers waiting" in Overview → Reports |
| Upcoming collections | Blue | "Upcoming collections" in Overview → Reports |
| Terminations, freezes and pauses starting in the next 14 days | Purple | Memberships (Offboarding or Suspended) |
| Offers needing details | Amber | Sales → Offers, filtered to "Needs details" |

Slack messages will appear here once Slack is connected.

The client's Instagram handle is looked up by name from `payments[].instagram`, `members[].instagram`, sign-ups (the form has an Instagram field) or a lead whose contact starts with `@`. With sample data the Instagram links are turned off and show where they would go.

## Deploy on GitHub and Vercel

1. The repository is `DelegatetoElevate/BBR.Dashboard`. It is **public** (Vercel's free plan can't deploy private organisation repositories), so only code and sample data go in. `data.json` is in `.gitignore` and must never be committed; real data will come from Supabase.
2. `index.html`, `README.md`, `vercel.json` and `.gitignore` sit at the root of the repository. `vercel.json` stops search engines listing the site and stops browsers caching `data.json`.
3. In Vercel, click **Add New → Project**, import the repository and click **Deploy**. Leave the framework preset as **Other**; no build command or output directory is needed.
4. Every push to the main branch redeploys the site automatically.

To preview locally, run `python3 -m http.server` in the folder and open `http://localhost:8000`. Opening the file directly also works, but it can't load `data.json` that way.

## Access (access codes)

Everyone signs in with **their name and an access code**. The code decides what they see:

| Role | Sees | Can add |
| --- | --- | --- |
| Admin (Ryan, Amy, Sheila, D2E) | All tabs | Everything |
| Outreach | Bookings only | Leads, booked calls, reminders, outcomes, reschedules |
| Coach | Client Check-ins only | Check-in status updates and notes. A coach who signs in with their own name sees their clients first |

The codes are shared with the team directly. **Never write them in this repository**: it is public.

- Codes ignore spaces, dashes and upper/lower case.
- The name is saved with every lead, call, sign-up and note that person adds, and is remembered on that device for next time.
- After 5 wrong codes, sign-in pauses for 30 seconds.
- People stay signed in on that device until they click **Log out**.

**Changing a code:** open the dashboard, open the browser console (F12), run `await hashCode("NEW-CODE")`, and paste the result into the matching `hash` in the `ACCESS` list near the top of the script in `index.html`. Only the hash is stored, so the codes can't be read from the page source. Give the new code to the team in person or by direct message, not in a shared channel.

**Important:** this is a simple lock, not real security. The check runs in the browser, so someone who edits the page or reads `data.json` directly can still see everything, including revenue and member names. Before sharing real data, add real logins (e.g. Supabase Auth with a database for the entries) or at least protect the site:

- **Vercel password protection or Vercel Authentication** (Project → Settings → Deployment Protection). This is the simplest option, but it protects the whole site, not individual tabs.
- For true per-role access (outreach sees sales only), deploy two projects: an admin one with full `data.json`, and an outreach one whose `data.json` holds only the `funnel` data. Each gets its own password.

## Connecting live data

The page loads `data.json` from the same folder on every visit. Generate that file from the sources below. A Zapier or Make flow, or a small scheduled script (e.g. a GitHub Action that runs every morning and commits `data.json`), can produce it.

Admins can choose **Export → Data file (JSON)** to download a complete example in the exact format (`bbr-data-YYYY-MM-DD.json`).

| Source | Feeds | Refresh |
| --- | --- | --- |
| Stripe: BBR New Sales, BBR Claremont, BBR Jolimont (source of truth) | `offers`, `payments` | Automatic; new offers appear when created |
| BBR data drops | daily Claremont and Jolimont revenue (`payments`) | Every morning |
| Momence Total Sales report (filtered by product type) | `fees` | Mondays 5am |
| Slack sale posts | `slackPosts` (matched against Stripe) | As posted |
| Suspension and termination pop-up forms | `members[].status`, `suspensionWeeks` | When submitted |
| Meta Ads (login from Gene) | `adSpend` | Daily or weekly |
| Outreach KPI tracker (history before manual entry) | `funnel` | Once |
| Dashboard forms | `leads`, `calls`, `signups` | As entered |

Note: Zapier's free plan (100 tasks) won't cover every Stripe transaction. Use a paid plan, or pull from the Stripe API directly in a scheduled script.

### data.json format

```json
{
  "meta": { "business": "Bodies by Ryan", "currency": "AUD", "generatedAt": "2026-11-23T05:00:00+08:00", "sample": false },
  "goals": { "monthlyRevenue": { "2026-11": 40000, "2026-12": 45000 } },
  "offers": [
    {
      "id": "price_123", "title": "Jordan H. – Body Recomp – 1/6", "created": "2026-06-10", "price": 3100,
      "account": "claremont", "paymentType": "instalments", "duration": "12 weeks", "billing": "Monthly",
      "homeGym": "claremont", "crossGym": false, "active": true, "confirmed": true, "kind": "custom",
      "transfer": { "to": "claremont", "confirmed": false, "onList": true }
    }
  ],
  "payments": [
    {
      "id": "pi_123", "date": "2026-06-10", "account": "claremont", "client": "Jordan H.", "offerId": "price_123",
      "amount": 1100, "status": "collected", "kind": "custom", "instalment": 1, "of": 6,
      "instagram": "jordanh.fit", "stripeUrl": "https://dashboard.stripe.com/payments/pi_123", "transfer": null
    }
  ],
  "slackPosts": [ { "date": "2026-06-10", "client": "Jordan H.", "amount": 1100, "account": "claremont", "permalink": "https://bbr.slack.com/archives/C0123/p1718000000000000" } ],
  "members": [
    {
      "id": "cus_123", "name": "Sam R.", "homeGym": "jolimont", "offerId": "gfm-jolimont", "status": "suspended",
      "price": 149.98, "billing": "fortnight", "upfront": false, "since": "2026-02-13", "suspensionWeeks": 4, "crossGym": false
    }
  ],
  "fees": [ { "date": "2026-11-20", "gym": "claremont", "type": "late_cancel", "amount": 15, "waived": false } ],
  "adSpend": [ { "weekStart": "2026-11-16", "channel": "gyms", "amount": 380 } ],
  "changes": [ { "id": "x1", "member": "Sam R.", "email": "sam@example.com", "type": "freeze", "start": "2026-12-01", "end": "2026-12-29", "reason": "Travel", "loggedBy": "Sheila", "loggedOn": "2026-11-23" } ],
  "notes": [ { "id": "n1", "clientKey": "e:sam@example.com", "client": "Sam R.", "kind": "change", "text": "Asked to freeze from 1 Dec for travel. Actioned in Momence.", "by": "Sheila", "at": "2026-11-23T09:30:00+08:00" } ],
  "funnel": [ { "weekStart": "2026-11-16", "leads": 30, "booked": 14, "showed": 11, "noShows": 2, "cancellations": 1 } ],
  "leads": [ { "id": "l1", "date": "2026-11-20", "name": "Chris P.", "contact": "@chrisp", "source": "Instagram DM", "interest": "Gym membership", "addedBy": "Outreach team", "notes": "" } ],
  "calls": [ { "id": "c1", "leadName": "Chris P.", "contact": "@chrisp", "bookedOn": "2026-11-20", "callDate": "2026-11-22", "callTime": "10:00", "status": "showed", "bookedBy": "Outreach team", "notes": "" } ],
  "signups": [ { "id": "s1", "date": "2026-11-22", "client": "Chris P.", "email": "chris@example.com", "program": "12-Week Transformation", "offerType": "Custom program", "account": "claremont", "total": 3000, "cash": 1000, "paymentType": "instalments", "closedBy": "Ryan", "source": "Instagram DM", "notes": "" } ]
}
```

Field notes:

- `account`: `online` (BBR New Sales), `claremont` or `jolimont`. Totals count toward the account the money came from.
- `payments[].status`: `collected`, `outstanding` (due, not paid) or `scheduled` (future instalment).
- `offers[].paymentType`: `full`, `instalments` or `recurring`. Leave fields empty for new Stripe offers that still need details; they are flagged on the Offers tab.
- `offers[].transfer`: for gym sales taken through the online account (e.g. Black Friday). Set `confirmed: true` after accounting confirms; `onList` means it is on Sheila's transfer list.
- `instagram`: the client's handle, with or without `@` (on payments, members or sign-ups). Used for the Instagram links on overdue payments.
- `stripeUrl` (optional): link to the payment in Stripe. If missing, a `pi_`, `ch_` or `in_` id is turned into a Stripe dashboard link. `slackPosts[].permalink` links to the Slack post.
- `changes[].type`: `termination`, `freeze` or `pause`. `end` is blank for terminations and for open freezes or pauses. Matched to `members[]` by `email`, then by name; add `email` to members for a reliable match.
- `calls[]`: also `source`, `rescheduled` (times moved), `reminded` (true once the lead was reminded), `remindedBy`, `remindedAt`.
- `signups[]`: also `start` (start date), `coach`, `instalments` (number), `confirmed` (first payment checked), `received` (date received).
- `pulse[]`: check-in updates, `{ "clientKey", "client", "status": "on_track" | "watch" | "at_risk", "note", "at", "by" }`. The newest per client counts.
- `members[]`: also `coach`, `suspendedFrom`, `suspendedUntil`, `endDate`. Client Check-ins includes clients who have a coach and an active membership, or a program that started in the last 6 months (until program end dates come from Coach Vault).
- `notes[]`: client notes. `clientKey` is `e:` + the client's email (or `n:` + their name in lower case when there's no email); `kind` is `update`, `change`, `payment` or `other`.
- `members[].status`: `active`, `suspended`, `terminated` or `free`. Lost cash flow = `suspensionWeeks` ÷ 2 × fortnightly `price`; upfront payers count as no loss.
- `fees[].type`: `late_cancel` ($15, within 8 hours of class) or `no_show` ($30). Set `waived: true` for waived or refunded fees.
- `calls[].status`: `booked`, `showed`, `no_show`, `cancelled` or `closed`. Calls booked count on `bookedOn`; show-ups, no-shows and cancellations on `callDate`.
- `funnel[]` is weekly history from the outreach KPI tracker, for weeks before manual entry started. Don't keep both for the same week, or the counts double. Closes are no longer read from `funnel`; they come from `signups`.
- Dates are `YYYY-MM-DD`. Weeks start on Monday.
- Custom offer names must follow `[Person] – [Program] – [instalment]/[total]`, e.g. `Jordan H. – Body Recomp – 1/6`. The Sales tab reads the program name from this.

Excluded by design: guest passes and season-pass vouchers (no revenue impact), and the retired Google sales sheet.

## Still to confirm before going live

- Booking calendar to connect (GoHighLevel, Calendly or Google Calendar) for upcoming calls
- Stripe access, then Zapier to fill the sales sheet; instalments after the first matched from Stripe
- Coach Vault connection for check-in dates and program end dates
- Coach names (coaches use the coach access code)

- Student GFM price and family pricing
- Online offer catalogue (Ryan and Gene)
- Ryan's monthly revenue goals (`goals.monthlyRevenue`)
- Meta login and KPI tracker access
- How every Stripe transaction will be pulled (Zapier paid plan or script)
- Ryan's agreement to the naming convention
- Real logins and a shared database for entries (replacing the demo login)
- Week start, timezone and month cut-off (assumed Monday, gym local time, calendar month)

## Browser-only settings

Leads, calls, sign-ups, terminations/freezes/pauses and client notes entered in the demo, call status changes, product active/inactive toggles, the offer builder, the login session, filters and theme are saved in the viewer's browser (localStorage). They are not shared between people. Make permanent changes in Stripe or the data source.
