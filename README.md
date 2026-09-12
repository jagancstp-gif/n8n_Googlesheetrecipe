# Google Sheet Expense Report Emailer

An n8n automation that reads expense rows from a Google Sheet, turns each row into a clean, readable email, and sends it via Gmail — automatically, with no manual copy-pasting.

This README explains, in plain steps, how it was built and how to run it yourself. No coding required.

---

## What you'll end up with

1. You add rows to a shared Google Sheet — each row is one expense, with its own recipient email address.
2. You click a button in n8n.
3. Each row gets turned into its own readable email (not a raw data dump) and sent to that row's address.
4. If the sheet is empty, nothing gets sent — no blank emails.

---

## What you need before starting

- A Google account with access to the shared spreadsheet.
- A Gmail account to send from.
- An n8n account (self-hosted or cloud) with Google Sheets and Gmail already connected as credentials. (If they're not connected yet, n8n will prompt you to sign in with Google the first time you open either node.)

---

## Part 1 — The spreadsheet

The sheet has one tab (`Sheet1`) with these columns:

| Date | Employee | Category | Description | Amount | Status | Email |
|------|----------|----------|--------------|--------|--------|-------|
| 2026-09-01 | Priya Nair | Travel | Flight to Chennai for client meeting | 8500 | Approved | priya@example.com |

- **Date, Employee, Category, Description, Amount, Status** — the actual expense details.
- **Email** — the address that *this specific row's* email should be sent to. Every row can go to a different person.


## Part 2 — The workflow
### 2. "Google Sheet Expense Report Emailer" (the real automation)

| # | Node | What it does |
|---|------|---------------|
| 1 | **Run Report** (Manual Trigger) | Starts the workflow when you click the button |
| 2 | **Read Expense Rows** (Google Sheets) | Reads every row currently in the sheet |
| 3 | **Build Email Per Row** (Code, runs once per row) | Turns each row's Date/Employee/Category/Description/Amount/Status into a friendly subject line and message body. Also reads that row's Email column |
| 4 | **Send Expense Report Email** (Gmail) | Sends the email to the address from that row |

Because step 3 runs **once per row**, five rows in the sheet means five separate, personalized emails go out — each to its own recipient, not one big combined email to everyone.

---

## Part 3 — How the tricky bits work

### Making the email readable, not a data dump
The Code node doesn't just paste the raw row into the email. It builds a proper sentence-based message:

```
Hi Priya Nair,

Here is your expense report entry from the sheet:

- Date: 2026-09-01
- Category: Travel
- Description: Flight to Chennai for client meeting
- Amount: Rs.8500
- Status: Approved

Regards,
Your Automation Bot
```

The subject line is also built dynamically, e.g. *"Your Expense Report Entry - Travel (2026-09-01)"*.

### Handling an empty sheet gracefully
If the sheet has zero rows, the **Read Expense Rows** node returns zero items. In n8n, when a node has nothing to hand off, every node after it simply doesn't run — so the Code node and the Gmail node never execute, and no email is sent. No special "if empty" logic was even needed for this part.

### Handling a row with no email address
If a row's **Email** column is blank, the Code node deliberately skips that specific row (returns nothing for it) instead of trying to send an email to nobody. Every other row still gets processed normally.

### Where the recipient address comes from
The Gmail node's "To" field isn't a fixed address — it reads `{{ $json.email }}`, which comes from whatever the Code node passed through for that row. So each of the (up to) five emails in a single run can go to a completely different person, automatically.

---

## Part 4 — Running it

1. Open the **Google Sheet Expense Report Emailer** workflow in n8n.
2. Make sure your sheet has at least one row with a valid Email address filled in.
3. Click **Execute workflow**.
4. Check the recipient inbox(es) — each should have received a separate, readable email for their row.

---

## Troubleshooting

**Nothing gets sent, no error shown**
Check the sheet actually has rows, and that at least one row has something in the Email column — rows without an email are silently skipped by design.

**"No approval received" or the workflow won't run from outside n8n**
n8n requires you to personally click **Execute workflow** inside the editor for any run that touches real Gmail/Sheets — this can't be triggered automatically from outside for safety reasons.

**Email goes to the wrong address**
Double check the exact column header in your sheet is spelled `Email` (capital E) — the Code node matches on that exact name.

**I want one combined summary email instead of one per row**
That's a different design — the Code node would need to run once for *all* rows together rather than once per row, and you'd need to decide which single address to send the combined report to. Ask and this can be switched back.

---

## Quick glossary (for non-technical readers)

- **n8n** — the automation tool running this behind the scenes.
- **Node** — one step in the automation, like "read the spreadsheet" or "send an email."
- **Trigger** — the node that starts the workflow (here, a manual button click).
- **Credential** — the saved Google/Gmail login that lets n8n act on your behalf.
- **Code node** — a small step that runs a bit of custom logic (here: formatting the email text) when the built-in nodes alone can't do exactly what's needed.
