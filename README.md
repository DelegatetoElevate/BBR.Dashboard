# BBR Dashboard

A single-page dashboard for Bodies by Ryan. It is one HTML file with no build step and no dependencies.

Until a `data.json` file is added, the page shows **sample data** (marked with a yellow "Sample data, not live" badge).

## Tabs

| Tab | What it shows |
| --- | --- |
| Overview and Notifications | Monday meeting banner, sales collected vs. the monthly goal, recurring revenue, churn (lost cash flow), collections due, money in vs. money out, sign-ups, weekly sales by account, week-on-week and month-on-month comparisons, notifications |
| Sales | New leads, calls booked, show-up rate, sign-ups, closing rate, no-shows, cancellations, funnel, calls and leads lists, sign-ups list (admin), programs sold, ROAS (admin) |
| Clients | Every client in one list (gym members, custom and online program clients, sign-ups), terminations, freezes and pauses starting soon. Click a client for their details and notes |
| Offers | Every offer with an active/inactive toggle, naming and missing-detail checks, standard prices, offer builder for split payments |
| Reports | Revenue by account (collected, outstanding, scheduled), Momence fee income net of waived fees, monthly sales, payments not yet collected, upcoming collections, transfers waiting, Slack and Stripe mismatches |

Filters: week, month or year, plus the account (BBR New Sales, BBR Claremont, BBR Jolimont; admin only).

**Export** (pill at the top right): exports the current tab with the filters applied.
- **PDF** opens the print window; choose "Save as PDF". Buttons, filters and the "Show as table" sections are left out.
- **CSV** downloads the tab's summary tiles and every table on it, ready for Excel or Google Sheets.

## Manual entries

- **+ New lead** (admin and outreach): date, name, contact, source, interest, notes.
- **+ Call booked** (admin and outreach): lead name (a new name is also added as a lead), call date and time, contact, notes. Update the call's status (Booked, Showed, No-show, Cancelled, Closed) in the Calls table after the call.
- **+ Sign-up** (admin only): every sign-up is entered by hand. Date, client, email (unique identifier), program, type, account, contract value, cash collected, payment type, closed by, lead source, notes.

Sign-ups drive the Sign-ups count, closing rate and Programs sold. Revenue still comes from Stripe.

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

Admins see a banner at the top of the dashboard for the weekly team meeting, **every Monday at 8:00 am Perth time** (Perth has no daylight saving, so it's always UTC+8). **Join Google Meet** opens the meeting link in a new tab.

| When | What admins see |
| --- | --- |
| Monday, before 8:00 am | Black banner on every tab, "today · in 25 min". Signing in opens Overview so it's the first thing on screen |
| Monday, 8:00–9:00 am | Black banner on every tab, "on now" |
| Rest of the week | Outlined banner on Overview only, with the countdown to next Monday |

Outreach accounts don't see it. To change the link, time or length, edit `MEETING` near the top of the script in `index.html`.

## Notifications

Each group has its own colour, and every heading and line can be clicked:

| Notification | Colour | Click goes to |
| --- | --- | --- |
| Payments not yet collected | Red | The client's Instagram chat (`ig.me/m/<handle>`, new tab). No handle: the "Payments not yet collected" table on Reports |
| Slack and Stripe mismatches | Magenta | The Slack post (`slackPosts[].permalink`) or the Stripe payment. No link: the mismatches table on Reports |
| Transfers between accounts | Green | The "Transfers waiting" table on Reports |
| Upcoming collections | Blue | The "Upcoming collections" table on Reports |
| Offers needing details | Amber | Offers, filtered to "Needs details" |
| Terminations, freezes and pauses starting in the next 14 days | Purple | "Starting soon" list on Clients |

**Terminations, freezes and pauses.** The Clients tab lists everything starting in the next 2 months, grouped by month: start date, member, type, length (or "open" with no end date), home gym, how long they've been a member, estimated lost cash flow, reason and an Instagram link, so Ryan can contact long-term members before they leave. Admins log new ones with **+ Termination / freeze / pause** (member, email, type, start date, end date, reason, notes).

The client's Instagram handle is looked up by name from `payments[].instagram`, `members[].instagram`, sign-ups (the form has an Instagram field) or a lead whose contact starts with `@`. With sample data the Instagram links are turned off and show where they would go.

**Demo limitation:** entries are saved in the browser where they were typed (localStorage). Other people and other devices won't see them. "Download data.json template" includes them, so they can be copied into `data.json`.

## Deploy on GitHub and Vercel

1. The repository is `DelegatetoElevate/BBR.Dashboard`. It is **public** (Vercel's free plan can't deploy private organisation repositories), so only code and sample data go in. `data.json` is in `.gitignore` and must never be committed; real data will come from Supabase.
2. `index.html`, `README.md`, `vercel.json` and `.gitignore` sit at the root of the repository. `vercel.json` stops search engines listing the site and stops browsers caching `data.json`.
3. In Vercel, click **Add New → Project**, import the repository and click **Deploy**. Leave the framework preset as **Other**; no build command or output directory is needed.
4. Every push to the main branch redeploys the site automatically.

To preview locally, run `python3 -m http.server` in the folder and open `http://localhost:8000`. Opening the file directly also works, but it can't load `data.json` that way.

## Access: admin vs. outreach (access codes)

Everyone signs in with **their name and an access code**. The code decides what they see:

| Role | Sees | Can add |
| --- | --- | --- |
| Admin (Ryan, Amy, Sheila, D2E) | All tabs, all accounts, revenue | Leads, calls, sign-ups, terminations/freezes/pauses, client notes |
| Outreach | Sales tab only, without revenue | Leads and calls |

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

Click **Download data.json template** in the page footer to get a complete example in the exact format.

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
- `notes[]`: client notes. `clientKey` is `e:` + the client's email (or `n:` + their name in lower case when there's no email); `kind` is `update`, `change`, `payment` or `other`.
- `members[].status`: `active`, `suspended`, `terminated` or `free`. Lost cash flow = `suspensionWeeks` ÷ 2 × fortnightly `price`; upfront payers count as no loss.
- `fees[].type`: `late_cancel` ($15, within 8 hours of class) or `no_show` ($30). Set `waived: true` for waived or refunded fees.
- `calls[].status`: `booked`, `showed`, `no_show`, `cancelled` or `closed`. Calls booked count on `bookedOn`; show-ups, no-shows and cancellations on `callDate`.
- `funnel[]` is weekly history from the outreach KPI tracker, for weeks before manual entry started. Don't keep both for the same week, or the counts double. Closes are no longer read from `funnel`; they come from `signups`.
- Dates are `YYYY-MM-DD`. Weeks start on Monday.
- Custom offer names must follow `[Person] – [Program] – [instalment]/[total]`, e.g. `Jordan H. – Body Recomp – 1/6`. The Sales tab reads the program name from this.

Excluded by design: guest passes and season-pass vouchers (no revenue impact), and the retired Google sales sheet.

## Still to confirm before going live

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
