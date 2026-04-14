# Session Log

## Project Context
Email Inbox Manager is an open-source Claude Code skill (github.com/pollymallen/email-inbox-manager, MIT, public) that learns a user's personal email organization system and helps them clean up, consolidate, and maintain their inbox. Session 1 (Apr 6-7): extracted files from a claude.ai design conversation, set up the GitHub repo, installed locally via symlink, then iterated SKILL.md with three major additions — team rollout workflow, scheduling/automation via CronCreate + RemoteTrigger, and expanded conversational query mode. Session 2 (Apr 13): hardened safety — added constrained permission model with "To Trash" quarantine (skill never deletes), Phase 0 backup checkpoint, Phase 0 account-verification pre-check, Phase 0.5 historical cleanup for old/large inboxes, and strengthened trigger description after a real session failed to auto-invoke. Also: created multi-account folder structure at `~/projects/personal/email/`, decided on `gws` (Google Workspace CLI) as the MCP transport instead of swapping the claude.ai Gmail connector, audited and rejected the `evolsb/claude-code-google-workspace` community wrapper, backlogged a spin-off `skill-evaluator` skill idea globally. Current state: skill is ~750 lines, all safety features live, folder ready, gws install pending.

## Session 2 — 2026-04-13
**Phase:** 2.5 Safety & Onboarding Hardening + Phase 3 kickoff planning
**Goals:** Resume project, harden the skill based on real-use concerns, start testing with `pollymallen@gmail.com`

**Done:**
- Added **Permission Model** section: constrained write access, no deletion, `To Trash` quarantine label pattern — skill can create labels, move/archive/draft, but never deletes. User empties `To Trash` manually.
- Added `to_trash` entry to `triage_boxes` in both SKILL.md and starter YAML
- Updated triage Phase 3 options: "Delete/archive" → "Archive / Send to To Trash"
- Added **Phase 0 — Backup Checkpoint**: snapshot label structure / Google Takeout / skip, before any action
- Added **Phase 0 Pre-check — Account Verification**: skill must identify connected Gmail and confirm it matches working dir before proceeding (gap caught when realized we hadn't switched connector)
- Added **Phase 0.5 — Historical Cleanup**: optional bulk pass for large/old inboxes; categorized criteria (old newsletters, social notifications, transactional, calendar, auto-notifications, long-dormant threads) with age thresholds, per-batch approval, sample preview, default Archive not Trash
- Strengthened skill description frontmatter: added exact phrase "let's set up my email system," explicit "ALWAYS use this skill — do NOT design a new system from scratch," noted empty working directory is expected starting state
- Added `historical_cleanup` section to starter YAML for audit trail
- README: added Safety and Backup sections
- Created `~/projects/personal/email/` and `~/projects/personal/email/pollymallen-gmail/` folders with top-level README explaining multi-account pattern
- Researched Google Workspace CLI (`gws`, github.com/googleworkspace/cli) released March 2026 — decided this is the right MCP transport for multi-account (no connector swap needed)
- Audited `evolsb/claude-code-google-workspace` community wrapper — 2 commits, 10 stars, 5+ weeks stale, one-person project, security gaps. Verdict: skip, go direct with official `gws`.
- Backlogged `skill-evaluator` spin-off skill to global `~/projects/BACKLOG.md` (Project Ideas section) and project PLAN.md
- 5 commits pushed to GitHub (permission model, backup checkpoint, historical cleanup, trigger strengthening, account verification, backlog note)

**Unfinished:**
- `gws` CLI not yet installed
- OAuth Desktop app not yet created in GCP for `pollymallen@gmail.com`
- `gws mcp` not yet registered in Claude Code config
- Live onboarding test not yet run
- Sample populated governance map still pending

**Blockers:**
- User was on a plane with patchy wifi at wrap time. GCP OAuth setup needs real connectivity.
- OAuth + MCP registration is ~30 min of setup before the actual skill test can run.

**To resume:**
1. Make sure you're on real wifi
2. `brew install googleworkspace-cli` (or check if already installed)
3. Create GCP OAuth Desktop client for `pollymallen@gmail.com`: https://console.cloud.google.com → create project → OAuth consent screen (External, testing, add yourself as Test User) → Credentials → Create OAuth client (Desktop app) → download JSON to `~/.config/gws/client_secret.json`
4. `gws auth setup` (if you have gcloud) or skip to `gws auth login`
5. Register `gws mcp` as an MCP server in Claude Code config (`~/.claude.json` or similar — check `gws mcp --help`)
6. Restart Claude Code, verify gws tools appear
7. `cd ~/projects/personal/email/pollymallen-gmail && claude`
8. Say "Let's set up my email system" — should trigger skill and start Phase 0 (pre-check + backup)
9. Walk through Phases 0 → 6 with a real inbox

**Key decisions made this session:**
- Permission model: no deletion, ever. `To Trash` label is the only "delete-like" action.
- Multi-account via `gws` CLI (not connector swap, not `evolsb/claude-code-google-workspace` wrapper)
- Folder convention: `~/projects/personal/email/{handle}-{provider}/` per account
- `skill-evaluator` is a separate future project, backlogged globally, not blocking email-inbox-manager
