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
2. **Search email history**: Find the latest thread with this person across both Gmail accounts to get context, their email address, and the right account to reply from
3. **Determine email type**: follow-up, ping/check-in, intro, send-doc, etc.
4. **Draft the email**: Create a Gmail draft on the correct account (the one with the existing thread). If it's a reply, use `--thread-id` to keep it in the same thread.
5. **Use context**: Pull from the task description (which should have meeting/Granola links) and email history to write a contextual, non-generic draft

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
