# hypergrowth-skills

OpenClaw agent skills for the Hypergrowth Partners team.

## Skills

| Skill | Description |
|-------|-------------|
| `humanizer` | Remove AI writing patterns from text |
| `meeting-prep` | Daily meeting briefs: email context, Granola history, LinkedIn research |
| `action-items-todoist` | Extract action items from meetings → Todoist tasks + email drafts |
| `email-drafting` | Auto-draft and manually-requested email drafts |
| `executive-digest` | Daily status digest: stalled threads, pending intros, tasks, calendar |

---

## Quick Setup

### 1. Clone the repo

```bash
git clone <repo-url> ~/hypergrowth-skills
```

> Must clone to `~/hypergrowth-skills/` — skills reference config at a fixed relative path.

### 2. Create your user config

```bash
cp ~/hypergrowth-skills/config/user.example.json ~/hypergrowth-skills/config/user.json
# Edit user.json with your personal values — it's gitignored, never committed
```

### 3. Configure OpenClaw to load these skills

Edit `~/.openclaw/openclaw.json`. Add `"load"` inside the `"skills"` key:

```json
{
  "skills": {
    "load": {
      "extraDirs": ["~/hypergrowth-skills"]
    },
    "install": {
      "nodeManager": "npm"
    }
  }
}
```

> If you don't have a `"skills"` key yet, add the whole block. The `install` section is optional.

### 4. Restart the gateway

```bash
openclaw gateway restart
```

Skills from `~/hypergrowth-skills/` will now load alongside your workspace skills.

---

## How skills use your config

Each skill reads `../config/user.json` at startup — an explicit file read, not passive context injection. This makes config usage reliable even in long or isolated sessions.

The `user.json` supplies: email accounts, WhatsApp number, timezone, scheduling contacts, signature, and workspace path.

Pattern used at the top of every skill:

```markdown
## Config — read before starting
Read `../config/user.json` (resolves to `~/hypergrowth-skills/config/user.json`).
Extract: name, primary_email, work_email, whatsapp, timezone, scheduling_cc ...
Do not proceed until you have these values.
```

---

## User config fields

See `config/user.example.json` for the full template. Key fields:

| Field | Example | Used for |
|-------|---------|----------|
| `name` | `"Gonto"` | Granola queries, task attribution |
| `primary_email` | `"you@gmail.com"` | Gmail account 1 |
| `work_email` | `"you@company.com"` | Gmail account 2 |
| `whatsapp` | `"+5491234567890"` | Digest/alert delivery |
| `timezone` | `"America/Argentina/Buenos_Aires"` | Meeting times, cron scheduling |
| `scheduling_cc` | `"assistant@company.com"` | CC on scheduling emails (mentioned in body) |
| `scheduling_silent_cc` | `"colleague@company.com"` | Silent CC (never mentioned in body) |
| `slack_username` | `"yourname"` | Slack DM for meeting briefs |
| `signature` | `"--yourname"` | Email sign-off |
| `workspace` | `"/home/user/.openclaw/workspace"` | Absolute path for scripts/state |

---

## Full setup & cron configuration

- `docs/setup.md` — complete setup guide (mcporter, OAuth, gog, Todoist)
- `docs/crons.md` — cron job templates for all skills

---

## Repository conventions

- `config/user.json` is **gitignored** — each person creates their own
- `config/user.example.json` is the **committed template**
- `state/` and `logs/` are gitignored (machine-local)
- Skills reference workspace files (`style/`, `state/`, `scripts/`) via relative paths — these work because OpenClaw runs with the workspace as the working directory
