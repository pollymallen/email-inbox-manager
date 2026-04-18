# Session Log — Archive

Old session entries moved out of `SESSION-LOG.md` to keep it short. Most recent sessions live in `SESSION-LOG.md`.

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

**Key decisions this session:**
- Security framing must be honest, never overclaim Anthropic data handling. Point to Anthropic's current policy rather than summarizing it.
- "Can't delete" framing: lead with the structural truth (connector has no delete tool). Far stronger than "rule I follow."
- Progressive trust: always three options, never yes/no, so users don't accidentally persist session-only intent.
- Orientation runs EVERY onboarding, not just first-time — users can say "I know this, skip ahead" but the skill always offers the tour.
