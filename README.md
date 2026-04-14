# 📬 Email Inbox Manager

**An adaptive AI skill that learns YOUR email system — not the other way around.**

Built as a [Claude Skill](https://docs.claude.com) for use with Claude Code, Claude Cowork, or claude.ai.

## The Problem

Everyone organizes email differently. Some people have 5 folders, some have 500. Some reply instantly, some batch-process once a day. Every "inbox zero" system prescribes ONE way to manage email — and it rarely sticks because it ignores how you *actually* work.

## The Solution

This skill **learns your existing email organization patterns**, helps you consolidate and clean up, and then maintains your system going forward. It gets smarter and faster over time by remembering your preferences in a human-readable YAML file called the **Governance Map**.

### The Box System

Incoming email gets triaged into one of three boxes (or auto-filed):

| Box | What Goes Here | Example |
|-----|---------------|---------|
| **Box 1 — Waiting** | You're waiting for someone to reply | "Sent proposal to client, waiting for feedback" |
| **Box 2 — Action** | You need to do something | "Client asked for a revised timeline" |
| **Box 3 — Review** | You need to read it, no action required | "Team shared meeting notes" |
| **Auto-filed** | Informational, already handled | "Payment received confirmation" |

### The Confidence Graduation

The skill starts by asking you about every email. As you confirm its suggestions, it graduates rules from `always_ask` → `confirm` → `auto`. You're always in control of which rules run on autopilot.

## What It Does

- **🕰️ Historical Cleanup** — For large inboxes: bulk-archives old newsletters, social notifications, expired calendar invites, and long-dormant threads so you're not dragged through a decade of noise
- **🧹 Inbox Cleanup** — Walks you through consolidating labels/folders, archiving dead ones, and building a clean taxonomy
- **📋 Triage Sessions** — Processes your inbox using your rules, learning new patterns as it goes
- **⏰ Follow-Up Tracking** — Monitors Box 1 items and drafts nudge emails when things go stale  
- **📰 Newsletter Management** — Identifies subscriptions, lets you batch unsubscribe, and compiles a weekly digest
- **🔍 Email Search** — "What was that email from so-and-so?" with context from your governance map
- **📊 Weekly Reports** — Summarizes what got sorted, what's waiting, and what needs attention
- **🗺️ Team Reference** — Anyone on your team can ask "where does X go?" and get the answer

## Safety: What This Skill Can and Can't Do

By design, this skill operates with **constrained write access** to your Gmail. Nothing it does is irreversible.

**✅ Allowed:**
- Read messages, threads, and labels
- Create new labels (including a `To Trash` quarantine label)
- Apply, remove, and move messages between labels
- Archive (remove from inbox)
- Create Gmail drafts for your review

**❌ Not allowed:**
- Permanently deleting emails
- Moving messages to Gmail's native Trash (which auto-purges after 30 days)
- Sending emails without your explicit approval of the draft
- Emptying the `To Trash` label — only you do that, manually in Gmail

### The "To Trash" Pattern

When something would otherwise be a delete candidate, the skill moves it to a `To Trash` label and archives it out of your inbox. You review and empty `To Trash` on your own cadence in Gmail. If the skill ever mislabels something, it's a two-click fix — not lost mail.

## Before First Use: Backup (Optional but Recommended)

The skill can't delete emails (see Safety above), but onboarding can move a lot of messages between labels during the Consolidation Workshop. For peace of mind, take a backup first:

- **Quick option (built in):** During onboarding, the skill offers to save a `pre-onboarding-snapshot.yaml` with your current label structure and a sample of thread IDs per label. Opt-in, fast, and enough to spot-check a rollback — not a full backup. For a true archive, use Google Takeout.
- **Full option:** Run [Google Takeout](https://takeout.google.com) — deselect everything, select only "Mail," start the export. Google emails you an `.mbox` file when it's ready (hours for large inboxes). You don't need to wait to proceed; it runs in the background.
- **Skip:** If you trust the skill and your inbox is small, this is reasonable.

## Quick Start

### In Claude Cowork or Claude Code

1. Copy the `email-inbox-manager/` folder into your skills directory
2. Connect the Gmail MCP connector
3. Say: **"Let's set up my email system"**

### In claude.ai

1. Connect Gmail in your connectors/integrations
2. Share the `SKILL.md` content at the start of a conversation
3. Say: **"Let's set up my email system"**

## Connecting Gmail: Two Options

The skill talks to Gmail through one of two transports. Pick one.

### Why two options?

The claude.ai Gmail connector is the fastest path — one-click auth, nothing to install — but it only holds one Gmail account at a time. If you want to manage a second inbox, you either swap the connector (disrupting whatever else you were using it for) or you run the `gws` CLI locally, which can hold multiple accounts side-by-side and is called via Claude Code's Bash tool.

| | Option A: claude.ai Gmail connector | Option B: `gws` CLI via Bash |
|---|---|---|
| **Setup** | Click to connect | ~20 min (brew + GCP OAuth + `gws auth login`) |
| **Transport** | MCP tool calls | Bash tool running `gws` commands |
| **Accounts** | One at a time | Multiple, switchable per folder |
| **Runs in** | claude.ai, Cowork, Claude Code | Claude Code |
| **Delete protection** | Structural (no delete tool exists) | Skill policy (never calls delete APIs) |
| **Good for** | Single inbox, quick start | Managing personal + work, or household + business |

### Option A — claude.ai Gmail MCP connector (simpler)

This is the default path and what the skill was originally built against. One Gmail account, already wired up for most Claude surfaces.

1. In claude.ai, go to Settings → Connectors and connect Gmail (or use Cowork's built-in connector)
2. Confirm which account is connected — the skill's Phase 0 pre-check will verify this before doing anything
3. Create or `cd` to a working directory for this account (e.g., `~/projects/personal/email/pollymallen-gmail/`)
4. Say: **"Let's set up my email system"**

To manage a different account, you swap the connector in claude.ai Settings and restart the session. That's fine if you rarely switch. If you switch often, use Option B instead.

### Option B — `gws` CLI via Bash (multi-account)

[`gws`](https://github.com/googleworkspace/cli) is the official Google Workspace CLI (released March 2026). It runs locally and holds OAuth credentials for multiple Google accounts. `gws` does **not** ship an MCP server — instead, the skill calls `gws` commands through Claude Code's Bash tool, the same way it would call `git` or any other local CLI.

This path takes more upfront setup but means you can have `pollymallen@gmail.com` and `polly.allen@aicareerboost.com` both available without ever swapping a connector.

**1. Install the CLI**

```bash
brew install googleworkspace-cli
```

**2. Create a GCP OAuth Desktop client**

Each person running `gws` needs their own OAuth client. It's a one-time setup per machine, not per account. If you have `gcloud` installed, `gws auth setup` can do most of this for you (`gws auth setup --login`). Otherwise, do it manually:

- Go to https://console.cloud.google.com and create a new project (or reuse one)
- Open **OAuth consent screen** → User Type: **External** → Publishing status: **Testing**
- Under **Test users**, add your own Google account(s) — only test users can auth while the app is in Testing mode
- Go to **Credentials** → **Create credentials** → **OAuth client ID** → Application type: **Desktop app**
- Download the JSON and save it to `~/.config/gws/client_secret.json`

**3. Authenticate each account**

```bash
gws auth login --services gmail
```

Run this once per Gmail account you want to manage. `gws` will open a browser window for each one. Use `--services gmail` to limit scopes to just what this skill needs; skip `--full` (which requests pubsub + cloud-platform). See `gws auth --help` for managing credentials across accounts.

**4. No MCP registration needed**

The skill invokes `gws` through Claude Code's Bash tool — no MCP server to register, no Claude Code restart, no extra config. As soon as `gws auth status` shows you're authenticated, you're ready.

Verify:
```bash
gws auth status
gws gmail users getProfile --params '{"userId":"me"}'
```

The second command should print your email address and a message count.

**5. Folder convention: one working directory per account**

The skill persists its governance map (and all learned rules) to the current working directory. To keep accounts cleanly separated, use one folder per account:

```
~/projects/personal/email/
  pollymallen-gmail/          # personal Gmail
    email-governance-map.yaml
  pollyallen-aicareerboost/   # work Gmail
    email-governance-map.yaml
```

`cd` into the folder for whichever account you want to work on, then start Claude Code. The skill's Phase 0a pre-check calls `gws gmail users getProfile` to confirm the authenticated account matches the folder before touching anything.

To switch the active `gws` account before starting a session, use `gws auth` commands (see `gws auth --help`). Each authenticated account has its own stored credentials; the skill reads whichever is current.

## The Governance Map

The heart of the skill is a YAML file that captures everything it learns about your email system. It's designed to be:

- **Human-readable** — Open it in any text editor and understand your rules
- **Team-shareable** — Your VA, assistant, or ops lead can reference it
- **Version-controllable** — Keep it in git, track changes over time
- **Portable** — It's just a file. Move it between environments freely.

See `templates/governance-map-starter.yaml` for the blank template, or the examples in `SKILL.md` for a fully populated version.

## Commands

| Say This | What Happens |
|----------|-------------|
| "Let's set up my email system" | Full onboarding walk-through |
| "Triage my inbox" | Process inbox with your rules |
| "Check my follow-ups" | Review Box 1 waiting items |
| "Find that email from [person]" | Search with governance context |
| "Give me my email report" | Weekly summary |
| "Where does [type] go?" | Look up rules for a category |
| "Show me my email rules" | Review the governance map |

## Requirements

- Gmail account (one account at a time)
- Gmail MCP connector (built-in to Cowork, or add via Claude Code)
- A directory where the governance map YAML can be saved

## Contributing

This is an open-source skill! If you have ideas for improvements:

1. Fork the repo
2. Try the skill with your own inbox
3. Submit a PR with your improvements

Especially welcome: governance map templates for specific personas (solopreneur, VP, founder with VAs, etc.)

## License

MIT

---

*Built by [Polly Allen](https://aicareerboost.com) — Founder of AI Career Boost, former Principal PM-Technical at Amazon Alexa AI.*
