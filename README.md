# n8n Automations — Sales (Kit 1: Lead to Close)

Four connected [n8n](https://n8n.io) workflows, powered by Claude, that take a B2B business from finding prospects to a signed proposal. A human is only needed for the sales call and a few Slack approvals.

| Workflow | What it does | Runs |
| --- | --- | --- |
| [A1 Prospect List Builder](workflows/A1-prospect-list-builder.json) | Finds prospects in Apollo matching your ICP, researches them, scores fit 0–100 | Mondays 7am · weekly report Fridays 4pm |
| [A2 Outreach and Reply Handler](workflows/A2-outreach-and-reply-handler.json) | Writes and sends a 4-email sequence, classifies and routes replies | Drafts 8am · sends hourly 9–5 · replies every 5 min |
| [A3 Qualifier and Meeting Booker](workflows/A3-qualifier-and-meeting-booker.json) | Qualifying questions, booking, reminders, pre-call brief, call outcome | Every 5–15 min · website form |
| [A4 Proposal Generator and Follow-up](workflows/A4-proposal-generator-and-follow-up.json) | Drafts the proposal, rep approval, follow-ups, Won/Lost | Every 15 min · follow-ups 9am |

The workflows share one n8n **Data table** as the CRM. Each one acts on leads at a certain `stage` and moves them to the next — that column change is the hand-off.

```
Ready for outreach → Contacted → Interested → Meeting booked → Proposal needed → Proposal sent → Won / Lost
        A1/A2           A2            A3             A3           rep (Slack)          A4
```

## Repository layout

```
workflows/     n8n workflow exports (import these)
data-tables/   schema for the Kit1 Leads and Kit1 Exclusions tables
docs/          process spec and a beginner's guide
```

## Setup

1. **Create the data tables** listed in [`data-tables/kit1-leads.md`](data-tables/kit1-leads.md), with the exact names `Kit1 Leads` and `Kit1 Exclusions`.
2. **Import the workflows:** in n8n, *Create workflow → ⋯ → Import from file*, once per JSON file.
3. **Connect credentials** (nodes missing one show a warning):
   - Gmail — the sending inbox (ideally on a secondary domain)
   - Google Calendar — the rep's calendar
   - Slack bot — plus a `#kit1-sales` channel with the bot invited
   - Apollo API key — custom-auth credential sending the header `x-api-key`, used by A1's two Apollo nodes
   - Anthropic (Claude) and Firecrawl — or use n8n Cloud's built-in AI credits
4. **Replace placeholders:**
   - A1 **ICP Settings** — titles, sizes, locations, industries, score threshold
   - A2 **Outreach Settings** — your offer and sender name
   - A3 email nodes — `REPLACE-WITH-YOUR-BOOKING-PAGE`, `REPLACE-WITH-A-USEFUL-GUIDE`, `Your Name`
   - A4 **Packages and Pricing** — real packages, rules, acceptance step; `Your Name`
5. **Test** with your own email first (see the [beginner's guide](docs/beginners-guide.md#testing-safely)), then activate A1 → A4 one at a time.

## Security

- No credentials, API keys or tokens are stored in this repo. n8n keeps credentials in your instance, and exports only reference them by name.
- Don't commit real lead data. Export your tables locally if you need a backup.
- Cold email and AI-assisted outreach are regulated in many places (CAN-SPAM, GDPR, CASL). Keep the unsubscribe handling on and check local rules before sending.

## Docs

- [Kit 1 process spec](docs/kit-1-process.md)
- [Beginner's guide](docs/beginners-guide.md)
