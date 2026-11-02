# Kit 1 in n8n: A Beginner's Guide

This guide explains what each workflow does, the n8n ideas you need to read them, and how to set up and test Kit 1 safely.

## n8n in 5 minutes

n8n builds automations by connecting boxes on a canvas. Each box does one job, and data flows left to right along the lines.

| Term | What it means | Example in Kit 1 |
| --- | --- | --- |
| Workflow | One automation: a canvas of connected boxes | *A2 Outreach and Reply Handler* |
| Node | One box with a single job | *Send First Email* (a Gmail node) |
| Trigger | The first node; decides **when** the workflow starts | *Every Monday 7am*, *New Reply in Inbox* |
| Item | One record moving through the workflow; each node runs once per item | Each lead read from the table |
| Credential | Your saved login for an outside app | *Gmail (sending inbox)*, *Slack Bot* |
| Expression | A formula inside `{{ }}` that fills a field from earlier data | `{{ $json.contact_email }}` |
| Data table | A built-in table inside n8n | *Kit1 Leads*, *Kit1 Exclusions* |
| AI Agent + Chat Model | Sends instructions to Claude and returns its answer | *Research and Score Prospect* |
| Structured Output Parser | Forces Claude to answer in fixed fields | *Score Format* |
| IF / Switch | Sends each item down a different path | *Route by Reply Type* |
| Send and Wait | Posts to Slack and pauses until someone approves or fills a form | *Ask Client to Approve Tone* |
| Execution | One run; open it to see what every node received and produced | Executions tab |

**How to read a canvas:** start at the trigger on the left and follow the lines. Where a line splits, each branch is labelled with its condition. Double-click any node to see its settings.

## How the pieces connect

The four workflows never call each other. Each one reads **Kit1 Leads**, acts on leads at a certain `stage`, then changes the stage — and the next workflow picks it up on its next run.

```
Ready for outreach ──▶ Contacted ──▶ Interested ──▶ Meeting booked
   (A1 sets, A2 emails)  (A2 sends 4)  (A3 qualifies)  (A3 reminds, briefs)
                                                            │
        Won ◀── Proposal sent ◀── Proposal needed ◀─────────┘
   (Kits 2 & 3)  (A4 follows up)   (rep sets, A4 drafts)
                       │
                       ▼
                     Lost        Nurture ──▶ back to Ready for outreach in 60–90 days
```

## The four workflows

### A1 — Prospect list builder (`workflows/A1-prospect-list-builder.json`)

Every Monday: **ICP Settings** → **Search Apollo for Prospects** (if Apollo errors, e.g. out of credits, **Alert: Apollo Search Failed** posts to Slack and the run stops) → drop companies already in the CRM or on the exclusion list → **Get Verified Work Email** from Apollo → **Read Company Website** (Firecrawl) → **Research and Score Prospect** (Claude, can search recent news) → stage *Ready for outreach* if score ≥ 70, else *Researched* → **Save Lead to CRM** → top 5 posted to Slack.

Every Friday a second trigger counts the week across the pipeline and posts the Kit 1 weekly report.


### A2 — Outreach and reply handler (`workflows/A2-outreach-and-reply-handler.json`)

Four triggers on one canvas:

1. **Weekdays 8am:** Claude writes a subject, first email and 3 follow-ups per new lead. While `require_approval` is on, samples go to Slack for tone approval.
2. **Hourly 9–5 weekdays:** sends up to 6 due emails; follow-ups go as replies in the same thread. Cadence: day 1, 3, 7, 14, then 90 days of nurture.
3. **Every 5 min:** reads new replies, matches them to a lead, Claude classifies them:

| Reply type | What happens |
| --- | --- |
| Interested | Stage *Interested*; A3 takes over |
| Question | Claude drafts an answer; rep approves in Slack before sending |
| Not now | *Nurture* until their date or 90 days |
| Referral | Referred person added as a new lead |
| Unsubscribe | Do-not-contact + exclusion list |
| Out of office | Paused until their return date |
| Bounce | Marked invalid; Slack note |
| Unclear | Sent to the rep in Slack |

4. **Daily 6am:** leads whose nurture date has passed go back to *Ready for outreach*.

### A3 — Qualifier and meeting booker (`workflows/A3-qualifier-and-meeting-booker.json`)

- **Website Lead Form** — inbound leads are saved at *Interested*.
- **Every 5 min** — new *Interested* leads get 4 qualifying questions.
- **Answers** — Claude scores need, size, timing and authority, then: qualified → booking link with 3 free times from the rep's calendar; not yet → resource + *Nurture*; not a fit → polite close-out; wants a person → Slack.
- **New calendar event** with the lead as guest → *Meeting booked* + confirmation.
- **Every 15 min** — 24 h reminder; 1 h reminder + Claude pre-call brief to Slack; 45 min after the call the rep logs the outcome in a Slack form (*Proposal needed* hands off to A4).
- **Weekdays 10am** — two gentle nudges for quiet leads, then *Nurture*.

### A4 — Proposal generator and follow-up (`workflows/A4-proposal-generator-and-follow-up.json`)

- **Every 15 min** — Claude drafts the proposal from the call notes using **Packages and Pricing**; the rep approves or requests edits in Slack. Custom scope or a discount over 10 % is flagged for a manager.
- **Weekdays 9am** — follow-ups on day 3 (check-in), 7 (Claude-written objection handling), 14 (final note); day 21 → *Lost*.
- **Every 5 min** — Claude reads the client's reply: accepted → *Won* + Slack; declined → *Lost* with reason; changes → redraft and re-approve; other → rep.

## Design choices

| Choice | Why |
| --- | --- |
| One table as the single source of truth | Workflows never depend on each other directly; pause one without breaking the rest |
| Schedules that poll the table | Simpler and sturdier than webhooks between workflows |
| Status flags (`drafting_proposal`, `brief_sent`…) | Stop a lead being emailed or processed twice |
| Claude answers in fixed fields | Later nodes rely on `score`, `category`, `decision` |
| Humans approve risky steps in Slack | First-batch tone, answers to questions, every proposal, big discounts |
| Replies matched by thread **or** sender | Works when a reply lands in a new thread |
| Emails marked read only after handling | A failed step retries instead of losing the email |

**Differences from the spec:** clients accept by replying "I accept" (no e-signature tool yet); Gmail can't detect opens, so the day-2 "not opened" nudge is merged into day 3; Kits 2 and 3 are announced in Slack rather than started automatically.

## Testing safely

1. Add one row to **Kit1 Leads** by hand: your own name, a second inbox you control as `contact_email`, `stage` = `Ready for outreach`, a sentence in `research_notes`, `fit_score` = 80, `do_not_contact` = false.
2. In A2 set `new_leads_per_day` to 1 and run **Weekdays 8am: Draft Sequences** manually. Approve in Slack.
3. Run **Hourly 9-5 Weekdays: Send** and check the email arrives.
4. Reply "Sounds interesting", run **New Reply in Inbox** — the row moves to *Interested*.
5. Continue through A3 and A4 the same way. Check follow-ups reach the lead, not your own inbox.

To test A1 cheaply, set `weekly_target` to 5 first — Apollo charges credits per email revealed.

## Troubleshooting

| What you see | What to check |
| --- | --- |
| Red node | Open the execution, click the node, read the message |
| "Credentials not found" / 401 | Reconnect that node's credential |
| Ran but did nothing | No rows matched — check `stage` and `outreach_status` |
| No Slack message | Bot not invited to `#kit1-sales` |
| Claude step failed | Usually a temporary limit; run again |
| Lead stuck | Its `stage` + `outreach_status` say which workflow acts next; edit by hand if needed |
