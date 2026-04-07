# Email Inbox Manager — Plan of Record

## Context
Adaptive email management skill that learns the user's personal organization system. Designed in claude.ai conversation, now set up as an open-source GitHub repo and installed locally as a Claude Code skill.

## Architecture
- **SKILL.md** — Core skill definition (425 lines, 6 operating modes)
- **templates/governance-map-starter.yaml** — Blank YAML template populated during onboarding
- **Governance map** — User-specific YAML file created per-account, NOT committed to repo (in .gitignore)

## Progress

- [x] Extract and organize files from claude.ai conversation
- [x] Initialize git repo
- [x] Create public GitHub repo (`pollymallen/email-inbox-manager`)
- [x] Add discovery topics (claude, ai-skill, email-management, gmail, mcp, productivity, claude-code)
- [x] Install skill locally via symlink to `~/.claude/skills/email-inbox-manager/`
- [ ] Live test: run onboarding with actual Gmail account
- [ ] Refine SKILL.md based on live testing
- [ ] Share at Next Level for feedback (week of 2026-04-06)
- [ ] Add sample "fully populated" governance map as reference

## What's Next
- Test the onboarding flow with Polly's Gmail (say "Let's set up my email system" in a new session)
- Iterate on SKILL.md prompts based on real-world behavior
- Consider adding a sample populated governance map for reference

## Key Files
- `SKILL.md` — skill definition
- `templates/governance-map-starter.yaml` — starter template
- `README.md` — GitHub-facing docs
- `CONTRIBUTING.md` — contribution guidelines

## Verification
- Repo: https://github.com/pollymallen/email-inbox-manager
- Skill installed at: `~/.claude/skills/email-inbox-manager/` (symlink)
- Trigger phrases: "triage my inbox", "set up my email system", "clean up my inbox", etc.
