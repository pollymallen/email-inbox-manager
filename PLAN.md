# Email Inbox Manager — Plan of Record

## Context
Adaptive email management skill that learns the user's personal organization system. Designed in claude.ai conversation, now set up as an open-source GitHub repo and installed locally as a Claude Code skill.

## Architecture
- **SKILL.md** — Core skill definition (~613 lines, 6 operating modes + scheduling + team rollout)
- **templates/governance-map-starter.yaml** — Blank YAML template populated during onboarding
- **Governance map** — User-specific YAML file created per-account, NOT committed to repo (in .gitignore)

## Progress

### Phase 1: Repo Setup (COMPLETE)
- [x] Extract and organize files from claude.ai conversation
- [x] Initialize git repo
- [x] Create public GitHub repo (`pollymallen/email-inbox-manager`)
- [x] Add discovery topics (claude, ai-skill, email-management, gmail, mcp, productivity, claude-code)
- [x] Install skill locally via symlink to `~/.claude/skills/email-inbox-manager/`

### Phase 2: SKILL.md Enhancements (COMPLETE)
- [x] Team rollout workflow: migration plan document, review period gate, transition mode
- [x] Scheduling & automation: CronCreate (session) + RemoteTrigger (persistent) for triage/nudge/report
- [x] Status communication: draft email / Slack / file delivery, anomaly detection, confidence tracking
- [x] Expanded query mode: fuzzy conversational search, smart Gmail search building, governance map cross-reference

### Phase 2.5: Safety & Onboarding Hardening (COMPLETE, 2026-04-13)
- [x] Permission model: constrained write access, never deletes, `To Trash` quarantine label
- [x] Phase 0 — Backup Checkpoint (snapshot / Takeout / skip) before any action
- [x] Phase 0 Pre-check — account verification step (confirm connected Gmail matches folder)
- [x] Phase 0.5 — Historical Cleanup for large/old inboxes (bulk archive of old newsletters, social notifications, expired calendar invites, long-dormant threads) with safety rails
- [x] Stronger trigger description to prevent empty-dir design-from-scratch failure
- [x] README updated with Safety + Backup sections
- [x] Multi-account folder structure created at `~/projects/personal/email/pollymallen-gmail/`

### Phase 2.75: Pre-Live-Test Dry-Run Polish (COMPLETE, 2026-04-13 Session 3)
Added after a parallel subagent dry-run caught 16 issues before first live test.
- [x] P0: Fix duplicate "Phase 0" labels → Phase 0a (account) + Phase 0b (backup)
- [x] P0: Drop assumed Gmail whoami; use `in:sent` From-field as primary account check
- [x] P0: Document Archive/To Trash/Move tool-mapping explicitly (Archive = `unlabel_thread(INBOX)`)
- [x] P0: Move `To Trash` label creation to Phase 0a
- [x] P0: Add Phase 0.5 VIP pre-step; clarify box-rail no-op on first run
- [x] P1: Cap snapshot at labels + first-100 thread IDs (flaky-wifi safe)
- [x] P1: Mid-batch MCP failure writes `resume_token`, no silent retry
- [x] P1: Nudge counter increments on send-approval, not draft
- [x] P1: Mode 4 date math uses user's local TZ
- [x] P1: Solo-user escape hatch on team-review block
- [x] P1: Unify triage_rules condition schema under any/all; document approval_mode precedence
- [x] P1: Document when to add per-category nudge overrides
- [x] P2: Phase 3 calibration falls back to recent archived when inbox is empty
- [x] P2: Dashboard gated behind explicit request (not offered during onboarding)
- [x] P2: README snapshot description matches SKILL.md
- [x] **Phase 00 — Welcome & Orientation:** plain-language tour of all 9 phases, skip markers, honest security framing (accurate re: Anthropic data handling, no overclaim)
- [x] **"You are here" markers** at every phase header
- [x] **Progressive trust model:** three levels (always ask / session-only `session_auto: true` / persistent `auto`) with explicit downgrade path; graduation prompt has three options not yes/no
- [x] Reframe "can't delete" as structural (connector has no delete tool), not LLM preference
- [x] README "Connecting Gmail: Two Options" section (connector vs. gws) written via subagent
- [x] `SPIN-OFF-skill-evaluator-spec.md` written via subagent

### Phase 2.9: Transport Pivot (COMPLETE, 2026-04-14 Session 4)
Added after discovering `gws` CLI does not ship an MCP server — Session 2/3 assumption was wrong. Skill now supports claude.ai connector OR gws-via-Bash (no MCP wrapper).
- [x] Push 7 local commits to origin
- [x] `brew install googleworkspace-cli` (0.22.5)
- [x] Discover `gws` has no `mcp` subcommand; architecture pivot required
- [x] Choose Option 1: gws-via-Bash transport (skill invokes `gws` via Bash tool, same pattern as `gdrive` CLI)
- [x] Write new SKILL.md "Transport Layer" section with full operation→transport mapping table
- [x] Update Prerequisites, Phase 0a account check, delete-guarantee language (structural vs policy per transport), tool mapping, error handling
- [x] Update README Option B — drop obsolete `gws mcp` registration step, add `gws auth setup --login` path, add verification commands
- [x] Commit + push transport rewrite (eac73d9)

### Phase 3: Live Testing & Sharing
- [x] Push local commits to origin
- [x] Install `gws` CLI
- [x] OAuth setup under work GCP (`email-inbox-mgr-polly`) — consent screen, Desktop client, client_secret.json, Gmail API enabled, `gws auth login` as pollymallen@gmail.com
- [ ] **BLOCKED: Redo GCP project under personal gcloud** — work GCP org policy `constraints/iam.allowedPolicyMemberDomains` prevents granting pollymallen@gmail.com IAM roles on aicareerboost.com-org projects. Option A is dead. Flip to Option B: `gcloud auth login` as pollymallen@gmail.com, create project outside any org, redo OAuth consent + Desktop client, replace `~/.config/gws/client_secret.json`
- [ ] Re-run `gws auth login --scopes "https://www.googleapis.com/auth/gmail.modify"` (explicit scope; avoid picker which previously selected readonly)
- [ ] Verify `gws gmail users getProfile` succeeds without 403
- [ ] Live test: run onboarding against `pollymallen@gmail.com` via gws — walk all phases (00 → 6)
- [ ] Refine SKILL.md based on live testing
- [ ] Add sample "fully populated" governance map as reference
- [ ] Share at Next Level for entrepreneur feedback
- [ ] Update README with any changes from testing
- [ ] (Optional cleanup) Delete abandoned `email-inbox-mgr-polly` project from work GCP

## What's Next
- Redo GCP project under `pollymallen@gmail.com` personal gcloud (outside any org — avoids `allowedPolicyMemberDomains`). See SESSION-LOG.md Session 4 "To resume" for full 10-step checklist.
- After that: first live test at `~/projects/personal/email/pollymallen-gmail/`, exercising Phase 00 welcome, progressive trust, gws transport end-to-end.
- Iterate on SKILL.md prompts based on real behavior.
- Eventually: add `polly.allen@aicareerboost.com` as a second gws-authenticated account (same OAuth client works for multiple test users).

## Backlog (Spin-off Ideas)

- **`skill-evaluator` skill** — open-source skill that audits third-party Claude skills / MCP servers / AI tools by GitHub URL. Rubric-based 1–5 scoring across 5 dimensions: Active Development, Adoption & Community, Security Posture, Code Quality, Trust & Transparency. Output: markdown audit report with per-dimension reasoning and overall verdict (Use / Use with caution / Skip). Pull in the `skill-creator` skill to bootstrap it. Origin: came out of evaluating `evolsb/claude-code-google-workspace` during email-inbox-manager setup (2026-04-13).

## Key Files
- `SKILL.md` — skill definition (~970 lines; new Transport Layer section added Session 4)
- `templates/governance-map-starter.yaml` — starter template
- `README.md` — GitHub-facing docs (now includes gws setup section)
- `CONTRIBUTING.md` — contribution guidelines
- `SPIN-OFF-skill-evaluator-spec.md` — design doc for the backlogged skill-evaluator spin-off

## Verification
- Repo: https://github.com/pollymallen/email-inbox-manager
- Skill installed at: `~/.claude/skills/email-inbox-manager/` (symlink)
- Trigger phrases: "triage my inbox", "set up my email system", "clean up my inbox", "who was that email from...", etc.
