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

### Phase 3: Testing & Sharing
- [ ] Live test: run onboarding with actual Gmail account ("Let's set up my email system")
- [ ] Refine SKILL.md based on live testing
- [ ] Add sample "fully populated" governance map as reference
- [ ] Share at Next Level for entrepreneur feedback
- [ ] Update README with any changes from testing

## What's Next
- Test the onboarding flow with Polly's Gmail in a new session
- Iterate on SKILL.md prompts based on real-world behavior
- Consider adding a sample populated governance map for reference
- Share at Next Level and collect feedback

## Backlog (Spin-off Ideas)

- **`skill-evaluator` skill** — open-source skill that audits third-party Claude skills / MCP servers / AI tools by GitHub URL. Rubric-based 1–5 scoring across 5 dimensions: Active Development, Adoption & Community, Security Posture, Code Quality, Trust & Transparency. Output: markdown audit report with per-dimension reasoning and overall verdict (Use / Use with caution / Skip). Pull in the `skill-creator` skill to bootstrap it. Origin: came out of evaluating `evolsb/claude-code-google-workspace` during email-inbox-manager setup (2026-04-13).

## Key Files
- `SKILL.md` — skill definition (~613 lines)
- `templates/governance-map-starter.yaml` — starter template (with schedules + team fields)
- `README.md` — GitHub-facing docs
- `CONTRIBUTING.md` — contribution guidelines

## Verification
- Repo: https://github.com/pollymallen/email-inbox-manager
- Skill installed at: `~/.claude/skills/email-inbox-manager/` (symlink)
- Trigger phrases: "triage my inbox", "set up my email system", "clean up my inbox", "who was that email from...", etc.
