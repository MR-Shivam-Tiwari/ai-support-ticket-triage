# 02 — AI Support Ticket Triage & Draft Reply

**Problem:** A shared support inbox mixes billing questions, bugs and genuine outages. Urgent issues wait behind password resets, and agents rewrite the same first reply all day.

**Solution:** Each ticket is classified by AI into category, sentiment and priority the moment it arrives. Urgent tickets trigger an immediate Slack escalation; every ticket is logged, and a Gmail draft reply is prepared for an agent to review and send — a human always approves the final message.

## Flow
`Ticket Webhook → AI Classify Ticket (OpenAI) → Parse Triage → Urgent? → Slack escalation / Ticket log → Gmail draft reply`

The AI returns strict JSON: `{category, sentiment, priority, summary, draft_reply}`. Priority rules are stated in the prompt: `urgent` is reserved for outages, data loss, security and payment failures.

## Setup
1. Import `workflow.json` into n8n.
2. Create a Header Auth credential for OpenAI (`Authorization: Bearer <your key>`) and select it in the *AI Classify Ticket* node.
3. Connect Google Sheets (columns: from, subject, message, category, sentiment, priority, summary, draft_reply, received_at), Slack (`#support-urgent`), and Gmail (draft operation) credentials.
4. POST tickets to the webhook — in production this same flow can be triggered by a Gmail/IMAP trigger or a helpdesk webhook (Zendesk/Freshdesk) instead.

## Sample input
```json
POST /webhook/support-ticket
{ "from": "ops@shopkart.in", "subject": "Payments failing at checkout",
  "message": "Since 9am all UPI payments are failing. We are losing orders every hour, please treat this as critical." }
```
Expected: category `billing`/`technical`, priority `urgent`, Slack alert fires, draft reply created acknowledging the outage.

## Resume bullet
Built an AI support-triage workflow in n8n + OpenAI that classifies tickets by category, sentiment and priority, auto-escalates urgent issues to Slack, and prepares human-reviewed Gmail draft replies.

## Interview talking points
- Human-in-the-loop by design: the AI drafts, a person sends — explain why fully automatic replies are risky for angry customers.
- Prompt guardrails: priority inflation is the classic failure (everything becomes 'urgent'), so the prompt defines urgent narrowly.
- Extension ideas: confidence threshold with a fallback queue, auto-tagging in the helpdesk, CSAT follow-up after resolution.
