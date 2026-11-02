# Data tables

Create these two tables in n8n (**Overview → Data tables → Create**) **before importing the workflows**. The workflows find them by name, so the names must match exactly.

## Kit1 Leads

The CRM. One row per contact. The `stage` column decides which workflow acts on the lead next.

| Column | Type | Set by | Meaning |
| --- | --- | --- | --- |
| company_name | string | A1 | Company name |
| domain | string | A1 | Company website domain, used for de-duplication |
| industry | string | A1 | Industry |
| employees | number | A1 | Estimated headcount |
| location | string | A1 | City, state, country |
| contact_name | string | A1 | Person's name |
| contact_title | string | A1 | Job title |
| contact_email | string | A1 | Verified work email |
| linkedin_url | string | A1 | Public profile |
| stage | string | all | Pipeline stage (see below) |
| fit_score | number | A1 | 0–100 fit against the ICP |
| score_reason | string | A1 | One-line reason, starts with the score |
| research_notes | string | A1 | What they do, buying signals, an opening observation |
| confidence | string | A1 | `high` / `medium` / `low` |
| outreach_status | string | A2–A4 | Sub-status inside a stage (e.g. `active`, `qualifying_sent`, `proposal_review`) |
| email_drafts | string | A2 | JSON: subject + 4 emails |
| sequence_step | number | A2, A3 | Emails or nudges sent so far |
| next_send_at | date | A2 | When the next email is due |
| last_contacted_at | date | A2–A4 | Last email sent |
| gmail_thread_id | string | A2–A4 | Gmail thread of the conversation |
| gmail_message_id | string | A2–A4 | Latest message to reply to |
| reply_category | string | A2 | interested, question, not_now, referral, unsubscribe, out_of_office, bounce, unclear |
| qualification_answers | string | A3 | Summary of the 4 answers |
| qual_score | number | A3 | 0–100 qualification score |
| meeting_time | date | A3 | Booked call start |
| calendar_event_id | string | A3 | Google Calendar event |
| reminder_24h_sent | boolean | A3 | Flag so the reminder is sent once |
| reminder_1h_sent | boolean | A3 | Flag so the reminder is sent once |
| brief_sent | boolean | A3 | Flag so the brief is posted once |
| pre_call_brief | string | A3 | Brief posted to the rep |
| call_notes | string | A3, A4 | Rep notes / transcript / revision requests |
| proposal_text | string | A4 | Latest proposal draft |
| proposal_link | string | A4 | Link to the email thread |
| proposal_sent_at | date | A4 | When the proposal went out |
| proposal_followup_step | number | A4 | Follow-ups sent after the proposal |
| lost_reason | string | A2–A4 | Why the lead was lost or disqualified |
| nurture_until | date | A2–A4 | When the lead re-enters outreach |
| do_not_contact | boolean | A2 | Never email again |
| source | string | A1–A3 | apollo, referral, inbound_form |

Do **not** add an `id` column — n8n creates one automatically.

## Pipeline stages

| Stage | Set by | What happens next |
| --- | --- | --- |
| Researched | A1 | Below the score threshold; waits for you to review |
| Ready for outreach | A1 | A2 drafts and sends the email sequence |
| Contacted | A2 | Follow-ups run until a reply or the sequence ends |
| Interested | A2 / form | A3 asks qualifying questions and books |
| Nurture | A2 / A3 | Re-enters outreach after 60–90 days |
| Meeting booked | A3 | Rep runs the call |
| Proposal needed | Rep (Slack form) | A4 drafts the proposal |
| Proposal sent | A4 | Follow-ups until signed or declined |
| Won / Lost / Disqualified | A3 / A4 | Final; reason logged |

## Kit1 Exclusions

Companies and people the kit must never contact.

| Column | Type | Meaning |
| --- | --- | --- |
| domain | string | Company domain to skip (existing customers, competitors, partners) |
| email | string | Person to skip (added automatically on unsubscribe) |
| reason | string | Why |
