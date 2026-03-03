---
name: executive-digest
description: Generate the daily executive digest with stalled scheduling threads, pending intros, follow-ups, open Todoist tasks, and upcoming calendar events. Use when running the daily digest cron or when user asks for a status digest.
---
# Daily Executive Digest

## Config — read before starting
Read `../config/user.json` (resolves to `~/hypergrowth-skills/config/user.json`).
Extract and use throughout:
- `primary_email`, `work_email` — both Gmail accounts to check
- `whatsapp` — for delivery
- `workspace` — absolute path to OpenClaw workspace

Do not proceed until you have these values.

## Steps

### 1. Read rules and state
- Read `style/DIGEST_RULES.md` for format and rules
- Read state files:
  - `state/scheduling-threads.json` — stalled threads (>3 days since proposed)
  - `state/decisions-memory.json` — context on people/companies
  - `state/digest-state.json` — avoid repeating items

### 2. Check Todoist
```bash
source {user.workspace}/.env
todoist-cli review
```
Include: overdue tasks, today's tasks, inbox (needs triage).
Format as "📋 Open Tasks" section: task name, due date, priority.
Highlight overdue first. Skip no-due-date tasks unless in inbox.

### 3. Check calendar (next 7 days) — BOTH accounts MANDATORY
```bash
gog --account {user.primary_email} --no-input calendar events --from today --to '+7 days' --json
gog --account {user.work_email} --no-input calendar events --from today --to '+7 days' --json
```
**CRITICAL: You MUST run BOTH commands and merge ALL events from both calendars into a single timeline.** Events live on different calendars — showing only one gives an incomplete picture. Deduplicate by time+title if the same event appears on both. Look for OOO, travel, vacation blocks, back-to-back conflicts, and double-bookings across calendars.

### 4. Check Gmail for pending items
- Recent intros not actioned: `subject:(intro OR introduction OR connecting) newer_than:7d`
- Follow-ups from others: `(following up OR checking in OR circling back) newer_than:7d`
- Drafts awaiting send

**Draft hygiene (MANDATORY):**
- For each draft candidate, check if a matching message (same thread/subject intent) was already sent from that account.
- If already sent, delete the stale draft and exclude it from digest output.
- Only report drafts that still require a send/edit decision.

### 4b. Unanswered emails from known contacts
Search BOTH Gmail accounts for recent inbound emails (last 7 days) from real people (not newsletters, automated, or system notifications) that have NO reply.
- Check SENT mail on the same account for a reply in the same thread
- If no reply exists after 24-48h, surface it as needing a decision (reply, ignore, or delegate)
- Prioritize: known contacts > first-time senders, VIPs always surface
- Frame as: "[Name] emailed about [topic] — reply, ignore, or delegate?"

### 4c. Promised follow-ups not yet executed
Scan for commitments made but not yet completed:
- **From SENT mail (last 14 days):** Look for promises like "I'll intro you", "I'll send the deck", "let me connect you with", "I'll follow up with" — then check if the intro/email was actually sent
- **From Todoist:** Check open tasks tagged with follow-up intent (intros, send deck, ping someone) that are overdue or due today
- Frame as: "Promised [action] to [person] on [date] — still pending"

### 5. Compile and send
- Format per `style/DIGEST_RULES.md`
- If items exist → send via WhatsApp to {user.whatsapp}
- Update `state/digest-state.json` with items surfaced
- Nothing needs attention → NO_REPLY
