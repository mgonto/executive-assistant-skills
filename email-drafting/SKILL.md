---
name: email-drafting
description: Auto-draft and manually-requested email drafts for Gonto's accounts. Use when drafting, replying, or sending emails, or when Gmail hooks detect draft triggers.
---
# Email Drafting Skill

## Config — read before starting
Read `../config/user.json` (resolves to `~/hypergrowth-skills/config/user.json`).
Extract and use throughout:
- `primary_email`, `work_email` — Gmail accounts
- `scheduling_cc` — scheduling assistant email (CC on all scheduling emails, mention in body)
- `scheduling_silent_cc` — silent CC for scheduling visibility (do NOT mention in email body)
- `signature` — sign-off for all drafts (e.g. "--gonto")
- `name` — short name for context

Do not proceed until you have these values.

## Overview
Auto-draft and manually-requested email drafts for {user.primary_email} and {user.work_email}.

## When to Use
- Gmail hook detects a trigger (intro, scheduling, thanks/ack, positive reply)
- User asks to draft/reply/send an email
- Action items cron identifies email follow-ups needed

## Architecture
- Detects triggers and creates Gmail drafts
- Handles all scheduling (slot-finding, conflict checking, calendar ops)
- NEVER proposes specific dates/times or creates calendar events

## Execution
- **"Reply" always means Reply All** — include all original To + CC recipients. Only exclude if user explicitly says to reply to one person.
- **After sending any email/draft**, check if it fulfills an open Todoist task (send deck, intro, follow-up, etc.). If yes → complete the task immediately and confirm.
- Always run via isolated sub-agent (not main session context)
- Sub-agent reads this SKILL.md + style files below, NOT MEMORY.md or daily memory files

## Rules (non-negotiable)

### Core
1. **Draft-only mode** — never send automatically
2. **Mirror inbound language** — draft in the same language as the thread (Spanish/English)
3. **Always sign** — end every draft with `{user.signature}`
4. **Low confidence** — don't draft; ask user for guidance
5. **No dash punctuation** — no em-dash/en-dash in bodies. Use commas/periods.
6. **Humanize** — apply `~/hypergrowth-skills/humanizer/SKILL.md` to every draft before finalizing

### Intro Handling (required sequence)
1. Thank introducer first
2. Move introducer to BCC
3. Reply to introduced contact directly
4. CC `{user.scheduling_cc}` and `{user.scheduling_silent_cc}` for scheduling
5. Include one line like: "Connecting {user.scheduling_cc_name} to find a time."
6. **Do NOT mention `{user.scheduling_silent_cc}` in the email body** — silent CC only

### Scheduling Drafts
- **ALWAYS CC {user.scheduling_cc} AND {user.scheduling_silent_cc}**
- **NEVER propose specific dates or times**
- Just confirm willingness to meet + mention scheduling assistant will coordinate
- Example: "Connecting Alfred to find a time that works"
- **Do NOT mention {user.scheduling_silent_cc} in the email body** — she's CC'd silently for visibility

### Allowed Auto-Draft Classes
- Thanks/ack
- Scheduling intent
- Positive short replies
- Intro acceptance

### Notification Format
- `account + one-line intent + draft link`
- Example: `📧 Draft ({user.primary_email} → John): intro acceptance. https://mail.google.com/...`

## Trigger Detection

### A) Intro
- Cues: `intro`, `introduction`, `meet`, `connecting you`, `looping in`, `cc'ing`
- At least 2 external participants + clear handoff language
- Apply intro sequence exactly

### B) Scheduling Intent
- Cues: `find a time`, `schedule`, `availability`, `next week`, `calendar`
- Spanish cues: `agendar`, `agenda una`, `tenés unos minutos`
- CC scheduling contacts, don't propose times

### C) Thanks/Ack
- Cues: `thanks`, `got it`, `appreciate it`, status updates
- Short acknowledgment + optional one-line next step

### D) Positive Short Reply
- Cues: `works for me`, `sounds good`, `perfect`, `great`
- Short affirmative + close

## Skip Conditions (do NOT auto-draft)
- Confidence low / intent ambiguous
- User already replied in the thread (SENT message exists)
- Legal, financial, security, hiring-final, sensitive conflict topics
- Multi-question strategic asks
- Automated/system/calendar notifications
- Messages requiring attachments, deep verification, or policy commitments
- Language unclear or unmirrorable

## Confidence Gate
Only auto-draft when ALL are true:
- Trigger class is one of the 4 allowed
- Language confidently detected and mirrorable
- Clear recipient intent and next step
- No skip condition present

Otherwise: ask user.

## Drafting Principles
- **Keep it SHORT** — drafts are always brief. 2-3 short paragraphs max.
- **No over-explaining** — state the point, don't elaborate unless necessary
- **When promising intros**: before drafting the intro, search sent emails for previous intros to that person/company, copy the format and tone, and use the same email address
- **When recommending a person/company**: use your own words from past emails about them rather than inventing new descriptions
- **Deck/one-pager**: say "Hypergrowth Partners deck" (not "our one-pager" or "our deck"). When attaching, frame WHY it's useful (e.g. "where we explain what Hypergrowth is and how we help companies")
- **Future availability**: frame as an opportunity, not a brush-off. Position it warmly: "I'd love to reconnect then to explore working together if the timing still makes sense" rather than blunt "let's connect closer to June"
- **Offers of help should use meeting context**: read Granola notes from the call and reference specific things discussed. The draft should feel like it came from someone who was in the meeting.
- **No generic "Great meeting today"** unless it's explicitly a first meeting (first VC call or first dealflow call).
- **Proposal-first rule**: if the commitment is to build/provide a proposal first, do not draft outbound email yet; create TODO only.

## Use Grain as primary source for meeting-based drafts
When drafting follow-up emails from meetings, **Grain transcript is the primary source** (not Granola):
1. Find the meeting in Grain: `mcporter call grain.list_attended_meetings --args '{"limit": 5}'`
2. Fetch transcript: `mcporter call grain.fetch_meeting_transcript --args '{"meeting_id": "<id>"}'`
3. Search transcript for email commitments: "I'll send", "I'll email", "I'll share", "let me intro", "I'll follow up", "I'll connect you", etc.
4. **Draft from the transcript** — use your actual words and the real conversation context, not Granola's summary.
5. Fall back to Granola only if Grain has no recording for that meeting.

## Style
Read `style/EMAIL_STYLE.md` for the full writing style guide (derived from 200+ real sent emails).
Read `style/FEEDBACK_LOG.md` for user corrections — latest overrides win.

Key points:
- Friendly, concise, action-oriented. Warm but not fluffy.
- 1–4 short paragraphs, ~6–7 word sentences
- Context-first openings, straight to point
- Common opens: "Hey <Name>,", "Thanks…", "Perfect…", "Great…"
- Sign off: `{user.signature}`
- No dash punctuation (no em-dash/en-dash)
- **Do:** be brief, clear, warm, decisive, include draft link in notification
- **Don't:** over-explain, corporate fluff, long formal prose

## Templates
See `style/EMAIL_TEMPLATES.md` for pattern templates (intros, follow-ups, VC, etc.)
See `style/email-templates.md` for HGP business templates (v1).

## Feedback Log
Already referenced in Style section above.

## Audit Logging (MANDATORY)
After every external action, log it:
- **Draft created**: `python3 scripts/audit_log.py log email_drafted "<recipient>" success '{"account": "<account>", "subject": "<subject>", "type": "<trigger_class>"}'`
- **Email sent**: `python3 scripts/audit_log.py log email_sent "<recipient>" success '{"account": "<account>", "subject": "<subject>"}'`
- **Draft skipped** (low confidence): `python3 scripts/audit_log.py log email_draft_skipped "<recipient>" skipped '{"reason": "<reason>"}'`

## Auto-Draft Constraints
- **NEVER create calendar events** — only the scheduling assistant handles that
- Only create email drafts
- Include `--to <sender>` explicitly when creating drafts

## Notification Policy
- No routine "no change" notifications
- Alert on: meaningful changes, breakages, time-sensitive items
- Time-sensitive: approvals, meeting changes, 2FA codes, security, travel changes
- Evaluate Promotions, suppress Spam/Junk/Trash
