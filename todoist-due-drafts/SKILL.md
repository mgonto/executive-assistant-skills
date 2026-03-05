---
name: todoist-due-drafts
description: Check Todoist for tasks due today that involve pinging, emailing, or following up with someone. Auto-draft the emails and notify via WhatsApp. Runs daily at 7 AM ART.
---
# Todoist Due-Today Email Drafts

## Config — read before starting
Read `~/executive-assistant-skills/config/user.json`.
Extract and use throughout:
- `primary_email`, `work_email` — Gmail accounts
- `whatsapp` — for notification delivery
- `workspace` — absolute path to OpenClaw workspace
- `signature` — email signature

## Steps

### 1. Get today's due tasks from Todoist
```bash
source {user.workspace}/.env
todoist-cli today --json
```

Also check overdue tasks:
```bash
todoist-cli list --filter "overdue" --json
```

### 2. Identify email/ping tasks
Filter for tasks whose content matches outreach intent:
- Keywords: `ping`, `email`, `follow up`, `follow-up`, `send`, `reach out`, `text`, `message`, `intro`, `connect`, `check in`, `nudge`, `reply`, `respond`, `draft`
- Pattern: any task that implies sending a communication to a specific person

Skip tasks that are clearly NOT outreach (e.g. "review doc", "read report", "build proposal").

### 3. For each outreach task, draft the email
Read and follow `~/executive-assistant-skills/email-drafting/SKILL.md` for all drafting rules.

For each task:
1. **Identify the recipient**: Extract person/company name from task content + description
2. **Pull meeting context from Granola/Grain**: The task description should contain the meeting name/date. Use this to retrieve the actual conversation:
   ```bash
   # Find the meeting in Granola
   mcporter call granola list_meetings --args '{"time_range": "custom", "custom_start": "<meeting-date>", "custom_end": "<meeting-date+1>"}'
   # Query for what was discussed with this person
   mcporter call granola query_granola_meetings --args '{"query": "What did {user.name} discuss with <recipient> and what did he promise or commit to do?", "document_ids": ["<meeting_id>"]}'
   ```
   Then cross-check with Grain for the full transcript:
   ```bash
   mcporter call grain.list_attended_meetings --args '{"filters": {"start_date": "<meeting-date>", "end_date": "<meeting-date+1>"}}'
   mcporter call grain.fetch_meeting_transcript --args '{"meeting_id": "<grain_meeting_id>"}'
   ```
   **This is the most important step** — the email draft must reflect what was actually said in the meeting, not just the task title. Look for: specific commitments, timelines discussed, names/projects mentioned, tone of the conversation, and any docs/links promised.
3. **Search email history**: Find the latest thread with this person across both Gmail accounts to get context, their email address, and the right account to reply from
4. **Determine email type**: follow-up, ping/check-in, intro, send-doc, etc.
5. **Draft the email**: Create a Gmail draft on the correct account (the one with the existing thread). If it's a reply, use `--thread-id` to keep it in the same thread.
6. **Use all context together**: Combine the meeting transcript (what was actually discussed), task description, and email history to write a specific, contextual draft. Never write generic follow-ups — reference concrete topics from the conversation.

#### Draft rules
- Mirror the language of the existing thread (English or Spanish)
- Keep it short — these are follow-ups and pings, not essays
- Sign with `{user.signature}`
- If the task says "ping" or "check in" — write a brief, friendly nudge
- If the task says "send [thing]" — draft the email with the attachment reference (or attach if path is known)
- If the task says "intro" — follow intro format from email-drafting skill
- If recipient email can't be found — report it, don't skip silently

### 4. Notify via WhatsApp
Send a single WhatsApp message to {user.whatsapp} with:

```
📬 *Due-today drafts*

<N> email drafts created from today's Todoist tasks:

1. **<recipient>** — <one-line intent> (draft in <account>)
2. ...

Review and send when ready.
```

If tasks exist but none are outreach-type, report:
```
📋 *Due-today tasks (no drafts needed)*

<list of today's tasks — quick reference>
```

If no tasks due today → NO_REPLY (skip notification entirely).

### 5. Infrastructure
```bash
python3 {user.workspace}/scripts/cron_canary.py ping todoist-due-drafts
```

## Rules
- Don't complete the tasks — just draft and notify. User decides when to send.
- Don't draft if user already replied in the thread (check SENT mail first).
- If a draft already exists for the same thread, don't create a duplicate.
- Overdue outreach tasks get a ⚠️ prefix in the notification.
