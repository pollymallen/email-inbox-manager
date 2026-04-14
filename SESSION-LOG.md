# Session Log

## Project Context
Email Inbox Manager is an open-source Claude Code skill (github.com/pollymallen/email-inbox-manager, MIT, public) that learns a user's personal email organization system and helps them clean up, consolidate, and maintain their inbox. Session 1 (Apr 6-7): extracted files from a claude.ai design conversation, set up the GitHub repo, installed locally via symlink, then iterated SKILL.md with three additions — team rollout, scheduling/automation, expanded conversational query mode. Session 2 (Apr 13): hardened safety — constrained permission model with "To Trash" quarantine (skill never deletes), Phase 0 backup checkpoint, Phase 0 account-verification pre-check, Phase 0.5 historical cleanup, strengthened trigger description. Also: multi-account folder structure at `~/projects/personal/email/`, chose `gws` CLI as MCP transport, audited and rejected `evolsb/claude-code-google-workspace` wrapper, backlogged `skill-evaluator` spin-off. Session 3 (Apr 13, plane wifi): pre-live-test dry-run caught 5 P0 + 7 P1 + 4 P2 bugs — all fixed. Added Phase 00 Welcome & Orientation with honest security framing (emphasizing that "cannot delete" is structural because the Gmail MCP connector has no delete tool, not an LLM preference). Added progressive trust model with three levels (always ask → session-only `session_auto: true` → persistent `auto`) and matching three-option graduation prompt. Added "You are here" markers to every phase. Added README "gws setup" section. Wrote `SPIN-OFF-skill-evaluator-spec.md` for the spin-off project. 6 commits ahead of origin — push pending real wifi. Current state: skill is ~870 lines, all safety + orientation + progressive trust live; gws install still blocked on OAuth setup (ground wifi).

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

## Session 3 — 2026-04-13 (plane continuation)
**Phase:** Pre-live-test hardening (completing Phase 2.5 polish before Phase 3 live test)
**Goals:** Do plane-friendly work toward the live test — spent the session on dry-run review, docs, spec, and P0/P1/P2 fixes plus a newcomer-friendly layer.

**Done (6 commits, all local, not yet pushed):**
- **Dry-run review (parallel subagent):** caught 5 P0 + 7 P1 + 4 P2 issues before first live test
- **P0 fixes (commit 1):**
  - Renamed duplicate "Phase 0" to Phase 0a (account check) and Phase 0b (backup)
  - Dropped assumption of Gmail `whoami` tool; now uses `in:sent` From-field as primary check
  - Added Archive/To Trash/Move tool-mapping block (Archive = `unlabel_thread(INBOX)`)
  - Moved `To Trash` label creation to Phase 0a so Phase 0.5 and Phase 2 can rely on it
  - Added Phase 0.5 VIP pre-step + clarified box-rail no-op on first run
- **gws README docs (commit 2, via subagent):** "Connecting Gmail: Two Options" section added to README covering claude.ai connector (Option A) and gws CLI with full OAuth walkthrough (Option B). No commands invented beyond verified context.
- **skill-evaluator spec (commit 3, via subagent):** Wrote `SPIN-OFF-skill-evaluator-spec.md` — design doc for the backlogged spin-off (5-dimension rubric, gh/WebFetch data gathering, sample audit report, edge cases).
- **P1 fixes (commit 4):** snapshot capped at labels + 100 thread IDs per label; mid-batch MCP failure writes `resume_token` to changelog (no silent retry); nudge counter increments on send-approval not draft; Mode 4 date math uses user's local TZ (PT); solo-user escape hatch on team-review block; unified `triage_rules` condition schema under `any/all` wrappers with documented precedence rules; nudge-override guidance.
- **P2 fixes + orientation layer (commit 5):** Added **Phase 00 Welcome & Orientation** — always-first step with plain-language tour of all 9 phases, skippable markers, pacing note, honest security framing (email stays in Gmail, plain-text files, no lock-in, accurate Anthropic data-handling framing without overclaiming). Added "You are here" marker to every phase header. Phase 3 falls back to recent archived when inbox is empty. Dashboard gated behind explicit request. README snapshot description corrected.
- **Progressive trust model (commit 6):** Reframed "can't delete" as a **structural guarantee** — the Gmail MCP connector has no delete tool at all, not an LLM preference. Added three-level trust (always ask / session-only via `session_auto: true` / persistent `auto`) with explicit downgrade path. Graduation prompt now asks three options ("just this session" / "from now on" / "keep asking") instead of yes/no. Listed trigger phrases for each level.

**Unfinished (still blocked on ground wifi):**
- 6 commits ahead of origin — need `git push`
- `gws` CLI install (`brew install googleworkspace-cli`)
- GCP OAuth Desktop app for `pollymallen@gmail.com`
- `gws auth login` + `gws mcp` registration in Claude Code config
- Live onboarding run
- Sample "fully populated" governance map still pending

**Blockers:** Real wifi for OAuth + brew install. Everything plane-friendly is done.

**To resume (on ground):**
1. `cd ~/projects/experiments/active/email-manager && git push` (6 commits)
2. `brew install googleworkspace-cli`
3. GCP OAuth Desktop client for `pollymallen@gmail.com` → `~/.config/gws/client_secret.json`
4. `gws auth login` per account
5. Register `gws mcp` (see `gws mcp --help` — exact command TBD)
6. Restart Claude Code; verify `mcp__gws__*` tools appear
7. `cd ~/projects/personal/email/pollymallen-gmail && claude`
8. Say "Let's set up my email system" — should fire Phase 00 welcome, then 0a account check against pollymallen@gmail.com
9. Walk through Phases 0a → 6 on a real inbox, note any skill improvements needed

**Key decisions this session:**
- Security framing must be honest, never overclaim Anthropic data handling. Point to Anthropic's current policy rather than summarizing it.
- "Can't delete" framing: lead with the structural truth (connector has no delete tool). Far stronger than "rule I follow."
- Progressive trust: always three options, never yes/no, so users don't accidentally persist session-only intent.
- Orientation runs EVERY onboarding, not just first-time — users can say "I know this, skip ahead" but the skill always offers the tour.
