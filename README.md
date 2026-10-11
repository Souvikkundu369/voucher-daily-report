# Daily Voucher Report — Automating a Manager's Spreadsheet, Rule by Rule

> **Before automating a report someone builds by hand every day, find out what it actually means. The coloured arrows in this one weren't comparing anything.**

<p>
  <img src="https://img.shields.io/badge/role-Sole%20Builder-orange" alt="Sole Builder">
  <img src="https://img.shields.io/badge/Node.js-339933?style=flat&logo=nodedotjs&logoColor=white" alt="Node.js">
  <img src="https://img.shields.io/badge/Render-46E3B7?style=flat&logo=render&logoColor=black" alt="Render">
  <img src="https://img.shields.io/badge/Supabase-3ECF8E?style=flat&logo=supabase&logoColor=white" alt="Supabase">
</p>

A live daily report of prepaid voucher sales across 26 stores — store counts, unique customers, new vs repeat %, trend arrows, and plain-language remarks — that replaced a report compiled by hand in Excel every day and sent to management over WhatsApp.

---

## The problem

Every day, someone pulled voucher sales from the POS, pasted them into a workbook, and produced a formatted report: per-store counts, customer numbers, coloured arrows, remarks in red, a highlighted top store. It took real time every single day, and because it was manual, its accuracy depended on the person making it.

---

## Decode first, automate second

The obvious approach is to look at the finished report and rebuild what it appears to show. That would have produced something that *looked* right and *meant* something different. Instead, every rule was reverse-engineered from the workbook itself, then checked against **two full weeks of the real hand-made sheets — every store, every day matched**.

What decoding turned up:

**The arrows weren't a comparison.** They looked like "up vs yesterday." They were actually Excel's *3 Arrows* conditional-format icon set applied to the count column — green for the top third of that day's min→max range, yellow for the middle third, red for the bottom. A store with green had a strong day *relative to the other stores*, not relative to its own past. Rebuilding it as a day-over-day comparison would have silently changed the meaning of every arrow.

**The comparison baseline is the same weekday last week, not yesterday.** Comparing a Saturday to a Friday is mostly noise in a weekend-driven business; Saturday vs last Saturday is a real signal.

**Weekends get their own mode.** On Mondays the report switches to a two-day view (Saturday + Sunday), compares the two days with each other, and compares the weekend total with last weekend.

**"Highest performing store" is only shown when exactly one store holds the top count.** A tie means no store gets the title.

---

## The automation caught the manual report's own mistakes

Rebuilding the logic precisely meant the automated version could be compared with the hand-made one, and the differences pointed at errors in the manual version, not the code:

- A store table that missed a sale its own salesperson table had recorded
- A headline "+300%" that was actually +295%
- A percentage that no longer held once late-arriving data settled

---

## Remarks that follow a rule

Red remarks used to be a judgment call. Now they're logical: in the daily view a store is flagged red when it sold nothing, or sold less than on the same day last week (*No voucher sale*, *Performance down*, *Better performance on previous Saturday*); equal or better stays black. In the weekend view, red means a total of 2 or fewer.

---

## Same person, two spellings

Salespeople are recorded as free text in the POS, so one person can appear under two spellings and look like two weak performers instead of one strong one. Look-alike names get a **Map** button — *Same person* or *Different people* — and a panel to review or remove saved mappings.

Two design decisions matter here:

- **Mappings are stored in a table nobody can query directly.** Row-level security is on with no policies; all reads and writes go through a small set of security-definer database functions. The page can do exactly three things — list, save, delete — and nothing else.
- **The report never writes to payroll tables.** It *reads* the incentive system's own name remap so merges already made there apply here too, but a mapping made in the voucher report can never change anyone's pay.

---

## Consistent with payroll by construction

A voucher is counted using the **same rule the incentive payout uses**: activity contains "voucher", deduplicated by voucher number keeping the highest amount, minimum value applied. Management's daily report and the month-end incentive calculation can't disagree about how many vouchers a store sold.

---

## Built for the way it's actually shared

The report is read on phones and forwarded on WhatsApp, so it renders as an image card with **Copy / Share / Download**, plus a plain-text version, with previous/next-day navigation.

---

## Architecture

```
POS booking API ──► voucher counts per store/day (one call per day)
Supabase RPC   ──► unique customers, new vs repeat
Supabase RPCs  ──► name mappings (security-definer functions only)
        │
        ▼
Node server (node:http, zero dependencies) on Render
        │
        ▼
Report page: KPI cards · store table · arrows · remarks · image/text export
```

**Zero dependencies on purpose.** The server uses Node's built-in HTTP module only: nothing to update, nothing to break on an install, a deploy that's just the code. Render auto-deploys on every push.

---

## Tech stack

`Node.js (node:http)` · `Render` · `Supabase (PostgreSQL RPC, RLS)` · `POS REST API` · `HTML/CSS`

---

### Author

**Souvik Kundu** — Business Intelligence & Automation Engineer.

📫 [LinkedIn](https://linkedin.com/in/souvik-kundu-bi) · [GitHub](https://github.com/Souvikkundu369)
