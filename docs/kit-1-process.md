# Kit 1 Lead to Close: Working Process

Kit 1 takes a business from finding the right prospects to a signed proposal. Four automations run in a chain, and the CRM pipeline stage of each lead decides which one acts next.

## End-to-end flow

Main path (top to bottom):

1. **Prospect data** (matched to the ICP) → **1 Prospect list builder** (find, research, score)
2. → *score above threshold* → **2 Outreach + replies** (sequence, sort replies)
3. → *interested reply* → **3 Qualify + book** (questions, score, brief). Inbound leads from the website form or chat also enter here.
4. → *qualified and booked* → **Sales call** (run by the rep — the only human step)
5. → *rep sets proposal needed* → **4 Proposal + follow-up** (draft, rep approves, e-sign)
6. → *signed* → **Won**: invoicing and onboarding start (Kits 2 and 3)

Exits (each logged in the CRM):

| From | Exit | When |
| --- | --- | --- |
| 1 Prospect list builder | Held for review | Score below threshold |
| 2 Outreach + replies | Nurture or do-not-contact | Not now, no reply, unsubscribe |
| 3 Qualify + book | Nurture or disqualified | Did not qualify |
| 4 Proposal + follow-up | Lost, reason logged | Declined or no response |

Only qualified, interested leads reach the rep. Nurtured leads re-enter outreach after 60–90 days, so nothing is lost for good.

## Foundation (set up once per client)

The CRM is the single source of truth: each automation watches for a pipeline stage, does its job, and moves the lead to the next stage.

**1. Ideal customer profile (ICP).** Written with the client before anything is built:

- Industry, company size (employees or revenue), location
- Job titles of the buyer and the decision maker
- Buying signals: hiring for a role, recent funding, new location, using a competitor's tool
- Exclusions: existing customers, competitors, partners, do-not-contact list

**2. Pipeline stages.** See [`data-tables/kit1-leads.md`](../data-tables/kit1-leads.md).

**3. CRM fields.** Fit score (0–100), score reason, research notes, outreach status, reply category, qualification answers, pre-call brief, proposal link, lost reason.

**4. Accounts and tools.** CRM, a data source for prospects, 2–3 separate sending inboxes on a secondary domain, a calendar booking tool, a call recorder, an e-signature tool, n8n, and a Claude API key. A Slack channel receives alerts and approvals.

## Automation 1 — Prospect list builder

Each week it adds a fresh batch of scored, researched prospects to the CRM, so outreach never runs dry.

**Trigger:** weekly schedule (e.g. Monday 7am), or manually when the ICP changes.

1. **Search.** Query the prospect database with the ICP filters and pull up to the week's target (e.g. 200 companies).
2. **Remove duplicates.** Drop anyone already in the CRM, on the exclusion list, or contacted in the last 90 days.
3. **Find contacts.** For each company, find 1–2 people with the target job titles and a verified work email.
4. **Research.** The agent reads the company website, recent news and the person's public profile, and notes buying signals.
5. **Score.** The agent scores fit from 0 to 100 against the ICP and writes a one-line reason.
6. **Write to CRM** at stage *Researched*.
7. **Promote.** Leads above the threshold (e.g. 70) move to *Ready for outreach*; the rest stay for review.
8. **Report.** Post a summary to Slack: how many found, how many qualified, top 5 prospects.

**Edge cases:** no verified email → skip the contact; website unreachable → score from database data only and mark "low confidence"; database credit limit reached → stop and alert.

## Automation 2 — Outreach and reply handler

Sends a personalized email sequence to each ready lead, reads every reply, and routes it.

**Part A: sending.** Write first email + 3 follow-ups from the research notes → first batch approved in Slack → send on day 1, 3, 7, 14, spread across inboxes at low volume → stage *Contacted*.

**Part B: replies.** Stop the sequence on any reply, then classify:

| Reply type | What happens |
| --- | --- |
| Interested | Stage *Interested* → Automation 3 |
| Question or objection | Agent drafts an answer; the rep approves it in Slack |
| Not now | Stage *Nurture*; restarts on the date they gave, or in 90 days |
| Referral | New contact created and added to outreach |
| Unsubscribe or angry | Marked do-not-contact everywhere |
| Out of office | Sequence pauses until their return date |

**Edge cases:** bounce → mark email invalid and try the second contact; unclear reply → send to the rep instead of guessing.

## Automation 3 — Inbound qualifier and meeting booker

Checks whether an interested lead is worth a sales call, books it, and briefs the rep.

1. **Respond fast** with 3–4 qualifying questions (what and why now, size, timing, who decides).
2. **Score** need, size, timing and authority, plus the fit score.
3. **Route:** qualified → booking link with 2–3 times; not yet → useful resource + *Nurture*; not a fit → polite close-out, *Disqualified*.
4. **Book** → *Meeting booked*.
5. **Remind** 24 h and 1 h before; one reschedule link on a no-show.
6. **Brief the rep** an hour before: snapshot, signals, answers, likely objections, 3 questions.

**Edge cases:** lead asks for a person → hand straight to the rep; no answers after 2 nudges → *Nurture*.

## Automation 4 — Proposal generator and follow-up

Turns the sales call into a proposal, gets it signed, closes the loop as won or lost.

1. **Read the call** (transcript or rep notes).
2. **Choose the package** from the pricing rules.
3. **Draft** the proposal and a cover email.
4. **Rep review** in Slack (*Approve* / *Edit*); discounts beyond the limit go to a manager.
5. **Send** → *Proposal sent*.
6. **Follow up** day 3, 7 (objections), 14.
7. **Close the loop:** signed → *Won* + Kits 2 and 3; declined or silent after 21 days → *Lost* with reason.

**Edge cases:** missing transcript → ask the rep for 5 bullets; custom scope → rep prices it; client asks for changes → revise and re-approve.

## Running rhythm

| When | What runs | Human time |
| --- | --- | --- |
| Monday morning | A1 builds the week's list | 5 min to scan top prospects |
| Every weekday | A2 sends sequences and sorts replies | Approve objection replies |
| Continuously | A3 answers new leads and books calls | Run the booked calls |
| After each call | A4 drafts the proposal | 5–10 min to review |
| Friday | Weekly report to Slack | 15 min review |
