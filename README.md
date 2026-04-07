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

- **🧹 Inbox Cleanup** — Walks you through consolidating labels/folders, archiving dead ones, and building a clean taxonomy
- **📋 Triage Sessions** — Processes your inbox using your rules, learning new patterns as it goes
- **⏰ Follow-Up Tracking** — Monitors Box 1 items and drafts nudge emails when things go stale  
- **📰 Newsletter Management** — Identifies subscriptions, lets you batch unsubscribe, and compiles a weekly digest
- **🔍 Email Search** — "What was that email from so-and-so?" with context from your governance map
- **📊 Weekly Reports** — Summarizes what got sorted, what's waiting, and what needs attention
- **🗺️ Team Reference** — Anyone on your team can ask "where does X go?" and get the answer

## Quick Start

### In Claude Cowork or Claude Code

1. Copy the `email-inbox-manager/` folder into your skills directory
2. Connect the Gmail MCP connector
3. Say: **"Let's set up my email system"**

### In claude.ai

1. Connect Gmail in your connectors/integrations
2. Share the `SKILL.md` content at the start of a conversation
3. Say: **"Let's set up my email system"**

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
