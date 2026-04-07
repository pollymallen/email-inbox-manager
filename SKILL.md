---
name: email-inbox-manager
description: >
  An adaptive email inbox management skill that learns YOUR personal email organization system 
  and helps you clean up, consolidate, and maintain your inbox. Use this skill whenever the user 
  mentions email management, inbox cleanup, email triage, email organization, folder consolidation, 
  unsubscribing from newsletters, email rules, inbox zero, email overwhelm, or setting up an email 
  workflow. Also trigger when the user says things like "help me deal with my email", "my inbox is 
  a mess", "sort my email", "clean up my inbox", "manage my newsletters", "set up email rules", 
  "triage my inbox", or "I have too many folders/labels". This skill works with one Gmail account 
  at a time via the Gmail MCP connector and persists all learned rules to a YAML governance map 
  that the user and their team can reference and refine over time.
---

# Email Inbox Manager

An adaptive email management skill that learns how YOU organize email, helps you clean up and 
consolidate, and then maintains your system going forward — getting smarter and faster over time.

## Philosophy

Everyone has their own email organization style, and it's usually a mix of intentional systems 
and organic habits accumulated over years. Prescribing a rigid system rarely works. Instead, 
this skill:

1. **Learns your existing system** by examining your current labels/folders and asking questions
2. **Helps you consolidate** by identifying redundancy and suggesting merges
3. **Calibrates triage rules** by walking through real emails with you
4. **Remembers everything** in a human-readable governance map that serves as the single source 
   of truth for how your email should be managed
5. **Gets faster over time** by graduating from "ask every time" to "auto-handle with confidence"

## Prerequisites

- Gmail MCP connector must be connected (either via Cowork's built-in connector, Claude Code 
  with a Gmail MCP server, or claude.ai's Gmail integration)
- A working directory where the governance map YAML file can be persisted

## The Governance Map

The governance map is a YAML file (default: `email-governance-map.yaml`) that serves as the 
skill's persistent memory. It captures everything the skill learns about the user's email system.
This file is designed to be human-readable so the user, their assistant, or any team member can 
open it and understand the rules.

Before any operation, check if a governance map already exists in the working directory. If it 
does, load it and resume from the current state. If not, start the onboarding flow.

### Governance Map Structure

```yaml
# Email Governance Map
# Generated and maintained by email-inbox-manager skill
# Last updated: 2026-04-06T10:30:00-07:00
# Account: user@example.com

account:
  email: user@example.com
  onboarding_completed: true
  onboarding_date: 2026-04-06
  confidence_mode: conservative  # conservative | moderate | autonomous
  status: active  # pending_review | active
  team:
    has_team: true
    review_period_days: 5
    review_deadline: 2026-04-11  # set during onboarding if team exists
    migration_plan_shared: true

# The Box System - how incoming email gets triaged
triage_boxes:
  box_1_waiting:
    description: "Waiting for a reply from someone else"
    label: "Box 1 - Waiting"
    check_cadence: "daily"
    nudge_rules:
      default:
        nudge_after_days: [3, 7, 14]
        max_nudges: 3
        nudge_style: "polite"
      # Per-category overrides
      sales:
        nudge_after_days: [2, 5, 10]
        max_nudges: 3
        nudge_style: "professional"
      personal:
        nudge_after_days: [7]
        max_nudges: 1
        nudge_style: "casual"

  box_2_action:
    description: "Owner needs to take action on this"
    label: "Box 2 - Action Needed"
    
  box_3_review:
    description: "Owner needs to review/read this but no action required"
    label: "Box 3 - Review"

  auto_filed:
    description: "Informational updates that are complete — acknowledged and filed"
    report_cadence: "weekly"  # How often to summarize auto-filed items

# Label taxonomy — the consolidated folder structure
taxonomy:
  active_labels:
    - name: "Clients"
      purpose: "Active client communications"
      auto_rules: []
    - name: "Receipts"
      purpose: "Purchase confirmations, payment receipts"
      auto_rules:
        - condition: "subject contains 'receipt' or 'payment confirmed' or 'order confirmation'"
          action: "auto_file"
          confidence: "high"
  
  # The translation map — where old labels map to new ones
  label_migrations:
    - old_label: "Client - Acme Corp"
      new_label: "Clients"
      reason: "Consolidated per-client labels into single Clients label"
      migrated_date: 2026-04-06
    - old_label: "Purchases"  
      new_label: "Receipts"
      reason: "Renamed for clarity"
      migrated_date: 2026-04-06
  
  archived_labels:
    - name: "Old Project X"
      archived_date: 2026-04-06
      message_count: 47
      reason: "No messages received in 8+ months"

# Sender profiles — learned patterns about specific senders
sender_profiles:
  - sender: "sarah@example.com"
    name: "Sarah (Head of Ops)"
    default_triage: "box_3_review"
    exceptions:
      - condition: "subject contains 'urgent' or 'blocker'"
        triage: "box_2_action"
    notes: "Weekly ops updates auto-file; escalations go to Box 2"
    
  - sender: "notifications@stripe.com"
    name: "Stripe Notifications"
    default_triage: "auto_filed"
    file_to: "Receipts"
    notes: "Payment confirmations — auto-file and include in weekly summary"

# Newsletter management
newsletters:
  subscribed:
    - sender: "newsletter@example.com"
      name: "Industry Weekly"
      frequency: "weekly"
      action: "digest"  # digest | keep | unsubscribed
      digest_group: "Industry News"
      
  unsubscribed:
    - sender: "spam@marketer.com"
      name: "Marketing Spam Co"
      unsubscribed_date: 2026-04-06
      
  digest_settings:
    enabled: true
    delivery: "weekly"  # daily | weekly
    day: "Monday"
    format: "summary"  # summary | full | headlines

# Triage rules — the decision tree for incoming email
# Rules are evaluated in order; first match wins
triage_rules:
  - id: rule_001
    name: "Payment confirmations"
    condition: 
      any:
        - subject_contains: ["receipt", "payment confirmed", "invoice paid"]
        - sender_domain: ["stripe.com", "paypal.com", "square.com"]
    action: "auto_file"
    file_to: "Receipts"
    confidence: "high"
    approval_mode: "auto"  # auto | confirm | always_ask
    created: 2026-04-06
    source: "onboarding"  # onboarding | user_approved | learned

  - id: rule_002
    name: "Calendar notifications"
    condition:
      sender_contains: "calendar-notification"
    action: "auto_file"
    file_to: "Calendar"
    confidence: "high"
    approval_mode: "auto"
    created: 2026-04-06
    source: "onboarding"

# Communication preferences
communication:
  # When to ask for help
  uncertainty_threshold: 0.7  # Below this confidence, ask the user
  ask_method: "batch"  # batch | immediate
  batch_cadence: "end_of_session"
  
  # Reporting
  weekly_report:
    enabled: true
    day: "Monday"
    include:
      - auto_filed_summary
      - box_1_status
      - nudges_sent
      - newsletters_digested
      - rules_suggested
  
  # Dashboard preferences  
  dashboard:
    enabled: true
    format: "react"  # react | markdown | html

# Changelog — every rule change gets logged
changelog:
  - date: 2026-04-06T10:30:00-07:00
    action: "created"
    description: "Initial governance map created during onboarding"
    details: "Consolidated 45 labels to 12. Created 8 triage rules."
```

## Operating Modes

The skill operates in distinct modes depending on what the user needs. Detect the appropriate 
mode from context, or ask if ambiguous.

### Mode 1: Onboarding (First-Time Setup)

This is the interactive walk-through that builds the governance map from scratch. Run this when 
no governance map exists yet.

**Phase 1 — Discovery**

1. Use Gmail MCP to pull all existing labels/folders
2. For each label, get the message count and date of most recent message
3. Present a summary: "You have N labels. Here's the landscape:"
   - Active labels (received messages in last 3 months)
   - Dormant labels (no messages in 3-6 months)  
   - Dead labels (no messages in 6+ months)
4. Ask: "Does this match your mental model? Anything surprising?"

**Phase 2 — Consolidation Workshop**

Walk through labels in groups, starting with the dead/dormant ones:

1. For dead labels: "This label '[name]' has [N] messages, last received [date]. 
   Archive it, keep it, or merge it into another label?"
2. For dormant labels: Same question, but note they might still be relevant
3. For active labels: "Are any of these doing the same job? Could we merge them?"
4. Build the taxonomy and label_migrations sections of the governance map
5. Present the proposed new structure for approval before making any changes
6. If the user has a team, generate a **Migration Plan** document (see Team Rollout below)

Important: Do NOT move or delete any labels without explicit user approval. Present the plan 
first, get confirmation, then execute. If there's a team involved, do NOT execute until the 
team review period has passed.

**Phase 3 — Triage Calibration**

1. Pull the 20 most recent inbox messages
2. For each message, show the user: sender, subject, date, and a brief snippet
3. Ask: "What would you do with this?"
   - Box 1 (waiting for reply)
   - Box 2 (you need to act)
   - Box 3 (you need to review)
   - Auto-file (it's done, just informational)
   - Delete/archive
4. After each answer, look for patterns: "I notice you're putting all messages from 
   [sender] into Box 3. Should that be a rule?"
5. After ~15-20 calibration emails, propose a set of triage rules
6. Ask: "For each of these rules, should I: always apply it automatically, apply it 
   but tell you about it, or always ask you first?"

This is where the conservative → autonomous graduation happens. Each rule gets an 
`approval_mode`:
- `always_ask` — Always confirm before acting (conservative, good for new rules)
- `confirm` — Apply the rule but report it, user can override (moderate)
- `auto` — Apply silently, include in weekly report (autonomous, earned over time)

New rules default to `always_ask`. When the user says "yes, always do that" during 
triage, upgrade the rule to `auto`.

**Phase 4 — Newsletter Audit**

1. Search for common newsletter patterns: "unsubscribe" in body, known newsletter 
   senders, high-frequency same-sender emails
2. Present the list: "I found [N] newsletters you're subscribed to:"
3. For each: "Keep in inbox | Route to weekly digest | Unsubscribe"
4. For "unsubscribe" choices, note them in the map. Offer to help unsubscribe 
   (search for unsubscribe links in recent messages from that sender)
5. For "digest" choices, create the digest settings

**Phase 5 — Communication Preferences**

1. Ask how they want to be notified about uncertain emails
2. Ask about weekly reporting preferences
3. Ask if they want a dashboard for querying their email

**Phase 6 — Review and Commit**

1. Generate the complete governance map YAML file
2. Present a readable summary: "Here's your email system:"
   - N active labels (down from M)
   - N triage rules
   - N sender profiles
   - N newsletters managed
3. Save the governance map to the working directory
4. Log the creation in the changelog

### Mode 2: Triage Session

For ongoing inbox processing after onboarding is complete.

1. Load the governance map
2. Pull unread/inbox messages
3. For each message, evaluate against triage rules in order
4. For `auto` rules: apply silently, log for report
5. For `confirm` rules: apply but tell the user what you did
6. For `always_ask` rules or no matching rule: present to the user and ask
7. When the user makes a decision on an unmatched email, look for a generalizable 
   pattern and propose a new rule
8. Update the governance map with any new rules or profile updates
9. At the end of the session, summarize what was done

**The confidence graduation prompt:**
When a user confirms an `always_ask` action for the 3rd time with the same pattern, 
suggest: "I've asked you about emails like this 3 times now and you've always said 
[action]. Want me to just handle these automatically going forward?" If yes, upgrade 
to `auto`.

### Mode 3: Nudge Check

For checking Box 1 (waiting for reply) items.

1. Load the governance map
2. Pull all messages in Box 1
3. For each, check how long it's been waiting and the nudge policy
4. Present items that are due for a nudge: "These items have been waiting:"
5. For each nudge-ready item, draft a follow-up email for user review
6. Track nudge count in the governance map

### Mode 4: Query / Search

For fuzzy, conversational questions about email — the user shouldn't need to remember 
exact names, dates, or subjects. They'll ask things like:

- "Who was that lead I talked to last week about Blueprint? I think his name was Adam?"
- "Did Sarah ever reply about the invoice?"
- "What was that email with the contract attachment?"
- "When did I last hear from the Acme team?"
- "Find that thread where we discussed pricing"

**How to handle these queries:**

1. **Parse the intent** — extract what you can: partial names, approximate dates, 
   topics, keywords, sender/recipient hints. The user will be vague — that's fine.

2. **Build a smart search** — translate the fuzzy query into Gmail MCP search:
   - Partial names → search sender/recipient fields with what you have
   - "Last week" → convert to date range (after:YYYY/MM/DD before:YYYY/MM/DD)
   - Topics → search subject and body
   - Combine multiple signals: `from:adam subject:blueprint newer_than:7d`
   - If the first search is too broad, narrow it; if too narrow, broaden it

3. **Cross-reference the governance map** — check sender_profiles for matches. 
   If the user says "that lead named Adam," check if any sender profile matches. 
   The governance map adds context Gmail search alone can't provide: which box an 
   email is in, what triage rule applied, any notes about the sender.

4. **Present results conversationally** — don't dump a list of raw email metadata. 
   Answer the question naturally:
   - "I found 3 emails with someone named Adam about Blueprint. The most recent is 
     from Adam Torres (adam.torres@company.com) on April 2 — subject: 'Re: Blueprint 
     program details.' He's in your Box 1 (waiting for reply) — looks like you sent 
     him pricing info and haven't heard back. Want me to draft a follow-up?"
   - Include: sender full name + email, date, subject, which box/label it's in, 
     and any actionable next step

5. **Handle dead ends gracefully** — if nothing matches:
   - Try broadening the search (drop date constraints, try alternate spellings)
   - Check if the conversation might be in a thread (search for the thread, not 
     just individual messages)
   - Ask a clarifying question: "I didn't find anyone named Adam in your Blueprint 
     emails from last week. Could it be a different name, or maybe further back?"

6. **Learn from queries** — if the user asks about the same person or topic 
   repeatedly, suggest adding a sender profile or label so it's easier to find 
   next time.

### Mode 5: Weekly Report

Generate the weekly summary based on communication preferences.

1. Load the governance map
2. Compile: auto-filed items, Box 1 status, nudges sent, rule suggestions
3. If newsletter digest is enabled, compile the digest
4. Present in the user's preferred format

### Mode 6: Governance Map Review

For when the user or their team wants to understand or modify the rules.

1. Load the governance map
2. Present it in a readable format
3. Handle questions like "where does email from X go now?" by searching 
   the taxonomy, sender profiles, and triage rules
4. Allow edits: add rules, modify labels, change approval modes
5. Save updates and log to changelog

## The "Where Does This Go?" Question

One of the most valuable features: when a team member (or the user) asks 
"where should I put an email like [description]?", the skill should:

1. Search triage_rules for a matching condition
2. Search sender_profiles for the sender
3. Check label_migrations for old → new label mappings
4. If found: "That goes in [label/box]. Here's why: [rule reference]"
5. If not found: "I don't have a rule for that yet. Based on similar patterns, 
   I'd suggest [box/label]. Want me to add a rule?"

## Scheduling & Automation

After onboarding, offer to set up automated schedules so the skill runs without the user 
having to remember to invoke it. There are two scheduling tiers:

### Tier 1: Session Schedules (CronCreate)

These run while Claude is active in a session. Good for working sessions where the user 
wants background triage happening while they focus on other things.

When the user says "keep my inbox clean while I work" or "auto-triage in the background":

1. Set up a recurring cron job (e.g., every 30 minutes) that runs Mode 2 (Triage) silently
2. Only surface emails that need the user's input — everything with `auto` rules gets 
   handled and logged without interrupting
3. At the end of the session, summarize everything that was auto-handled

Note: Session schedules expire when Claude exits. Remind the user of this.

### Tier 2: Persistent Schedules (RemoteTrigger)

These run on claude.ai even when the user isn't in a session — this is the real automation 
layer. During onboarding Phase 5 (Communication Preferences), offer to set up:

| Schedule | Default Cadence | What It Does |
|----------|----------------|--------------|
| **Morning Triage** | Weekdays 9am | Run Mode 2 on inbox. Auto-handle confident rules, batch uncertain ones into a summary draft email or Slack message for the user to review |
| **Nudge Check** | Weekdays 2pm | Run Mode 3 on Box 1 items. Draft follow-ups for anything past its nudge deadline |
| **Weekly Report** | Monday 9am | Run Mode 5. Compile the full weekly summary and deliver via the user's preferred channel |
| **Newsletter Digest** | Monday 7am (if enabled) | Compile the week's newsletters into a digest |

For each schedule, ask the user:
- "Want me to set this up? What time works for you?"
- "How should I deliver the results — draft email, Slack message, or just save to a file?"
- "Should I cc or notify anyone on your team?"

Store the schedule configuration in the governance map:

```yaml
schedules:
  morning_triage:
    enabled: true
    cron: "3 9 * * 1-5"  # Weekdays ~9am
    mode: triage
    delivery: draft_email  # draft_email | slack | file
    auto_handle: true  # apply auto rules without asking
    notify_on_uncertain: true  # flag emails that need human review
  nudge_check:
    enabled: true
    cron: "7 14 * * 1-5"  # Weekdays ~2pm
    mode: nudge_check
    delivery: draft_email
  weekly_report:
    enabled: true
    cron: "57 8 * * 1"  # Monday ~9am
    mode: weekly_report
    delivery: draft_email
    recipients: []  # email addresses to cc on the report
  newsletter_digest:
    enabled: false
    cron: "13 7 * * 1"  # Monday ~7am
    mode: newsletter_digest
    delivery: draft_email
```

### Status Communication

After each automated run, the skill should:

1. **Log the run** in the governance map changelog: timestamp, what was processed, 
   what was auto-handled, what needs review
2. **Deliver a status summary** via the user's preferred channel:
   - **Draft email** (default): Create a Gmail draft with the run summary — the user 
     sees it in their drafts and can review/discard
   - **Slack message**: Post to a designated channel or DM
   - **File**: Append to a `status-log.md` in the working directory
3. **Flag anomalies**: If something unusual happens (spike in unrecognized senders, 
   rule hitting way more than expected, emails that don't match any rule), highlight 
   it prominently in the status
4. **Confidence tracking**: Include a running stat in weekly reports — "This week, X% 
   of emails were auto-handled vs Y% last week" — so the user can see the system 
   getting smarter over time

### The "What Did You Do?" Question

When the user asks "what have you been doing with my email?" or "show me your activity":

1. Pull the changelog entries since the last check-in
2. Summarize: N emails triaged, N auto-filed, N flagged for review, N nudges drafted
3. Show any rules that were proposed but not yet approved
4. Show confidence trend (% auto-handled over time)

## Error Handling and Safety

- **Never delete emails** — only archive, label, or move
- **Never send emails** without explicit user approval of the draft
- **Always confirm** before making bulk label changes (>10 messages)
- **Back up the governance map** before major updates (copy to 
  `email-governance-map.backup.yaml`)
- If the Gmail MCP connection fails, tell the user and suggest reconnecting
- If the governance map is corrupted or unreadable, offer to start fresh 
  while preserving the old file as a backup

## Quick Start Commands

Users might say any of these to invoke specific modes:

- "Let's set up my email system" → Mode 1 (Onboarding)
- "Triage my inbox" / "Sort my email" → Mode 2 (Triage)
- "Check on my follow-ups" / "What's waiting?" → Mode 3 (Nudge Check)
- "Find that email from [person]" → Mode 4 (Query)
- "Who was that person who emailed about [topic]?" → Mode 4 (Query)
- "Did [person] ever reply?" → Mode 4 (Query)
- "What was that email about [topic]?" → Mode 4 (Query)
- "Give me my email report" → Mode 5 (Weekly Report)
- "Where does [type of email] go?" → Mode 6 (Governance Map Review)
- "Show me my email rules" → Mode 6 (Governance Map Review)
- "Generate the migration plan" → Team Rollout (generates shareable migration-plan.md)
- "My team signed off" → Unlocks execution of pending label changes
- "Set up my email schedules" → Configure automated triage, nudge, and report schedules
- "Keep my inbox clean while I work" → Session-only background triage (CronCreate)
- "What have you been doing with my email?" → Activity summary from changelog

## Working with Teams

The governance map is designed to be a shared reference — not just for the inbox owner, 
but for anyone who touches their email (VAs, EAs, ops leads, team members).

### The Migration Plan Document

After the Consolidation Workshop (Phase 2), if the user has a team, generate a 
**Migration Plan** — a human-readable summary document (markdown) that includes:

1. **What's changing:** A table showing every label rename, merge, and archive with the reason
   - Format: `Old Label → New Label | Reason | # of messages affected`
2. **The new structure:** The complete label taxonomy with descriptions of what goes where
3. **The "Where does X go now?" cheat sheet:** Quick-reference lookup for the most common 
   email types the team handles — formatted as a simple table they can bookmark
4. **Timeline:** When the changes will take effect (the user sets this)
5. **How to give feedback:** Where and by when the team should raise concerns

Ask the user: "Do you have team members who manage your email? If so, I'll generate a 
migration plan you can share with them before we make any changes."

### Team Review Workflow

When a team is involved, follow this process:

1. **Generate the migration plan** after Phase 2 consolidation
2. **Ask the user how long the team needs to review** — suggest 3-5 business days as default
3. **Save the migration plan** as `migration-plan.md` in the working directory
4. **Do NOT execute label changes yet** — mark the governance map as `status: pending_review`
5. **When the user says the team has signed off**, proceed with execution
6. **After execution, keep the migration plan** as a permanent reference — the team can 
   always open it to see the old→new mapping

### Transition Period (Optional)

If the user wants a gradual transition instead of a hard cutover:

1. Keep old labels as aliases during the transition (apply both old and new labels)
2. Set a sunset date for old labels
3. During triage, if an email would have gone to an old label, apply the new label 
   and log it — this builds team muscle memory
4. After the sunset date, offer to remove the old labels entirely

### Ongoing Team Communication

- The **changelog** at the bottom of the governance map tracks every rule change with 
  a timestamp and reason — team members can check this anytime
- When rules change significantly, suggest the user notify the team: "This change 
  affects how [type] emails get handled. Want me to draft a quick heads-up?"
- The "Where does X go now?" query (Mode 6) is designed for team members — they can 
  ask in plain language and get the current routing answer

## Dashboard (Optional)

When running in claude.ai or an environment that supports React artifacts, 
the skill can generate an interactive dashboard showing:

- Current inbox status (messages per box)
- Recent auto-filed items with the ability to override
- Box 1 items with days waiting and nudge status
- Rule hit frequency (which rules fire most often)
- A search interface for finding emails

Build this as a React component using Tailwind CSS and the recharts library 
for any charts. Use the Gmail MCP connector for live data.
