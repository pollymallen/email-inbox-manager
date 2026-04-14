# Spec: `skill-evaluator` (spin-off)

Origin: came out of evaluating `evolsb/claude-code-google-workspace` during email-inbox-manager setup (2026-04-13). Open-source skill that audits third-party Claude skills, MCP servers, and AI tools by GitHub URL before you install them.

## 1. Purpose & trigger phrases

Fires when the user is considering installing/trusting third-party AI tooling. Triggers:

- "is this skill safe to install"
- "audit this MCP server"
- "should I use this Claude skill"
- A GitHub URL + "should I use this" / "is this legit" / "review this"
- "check this repo before I install"

Non-triggers: general code review, PR review, bug triage on repos you already own.

## 2. Inputs

- **Required:** GitHub URL (repo root, subdirectory, or specific commit).
- **Optional:** Risk tolerance (`low` / `medium` / `high`), intended use case (e.g. "reads my email", "runs shell commands"), whether secrets/credentials will be passed to it.

Higher sensitivity use cases raise the bar on Security Posture and Trust dimensions.

## 3. Rubric (5 dimensions, 1–5 each)

### Active Development
- **Means:** Is someone actually maintaining this?
- **Signals:** last commit date, commit frequency over trailing 90 days, open vs closed issue ratio, PRs merged recently, release cadence.
- **1:** no commits in 12+ months, unanswered issues piling up. **3:** sporadic commits, some issue response. **5:** regular commits within 30 days, issues triaged, tagged releases.

### Adoption & Community
- **Means:** Do others trust and use this?
- **Signals:** stars, forks, contributor count, dependents, mentions in awesome-lists / blog posts, Discord or discussion activity.
- **1:** <10 stars, 1 contributor, no external mentions. **3:** ~100 stars, a handful of contributors. **5:** 1k+ stars OR from a known org, multiple active contributors, cited in curated lists.

### Security Posture
- **Means:** How risky is running this on your machine / against your accounts?
- **Signals:** permission scope (what tools/APIs it touches), dependency freshness (`npm audit` / `pip-audit` equivalents), presence of secrets in history, suspicious network calls, obfuscated code, eval/exec patterns, supply-chain red flags, CODEOWNERS, Dependabot config.
- **1:** broad scopes, stale deps with CVEs, executes shell with user input unsanitized. **3:** reasonable scopes, mostly fresh deps, no obvious red flags. **5:** least-privilege scopes, locked deps, SBOM or security policy, signed releases.

### Code Quality
- **Means:** Will this break, and is it debuggable if it does?
- **Signals:** tests present and passing in CI, linter config, type annotations, README/code coherence, error handling, modularity, CI green status.
- **1:** no tests, no CI, sprawling single file. **3:** some tests, basic CI. **5:** comprehensive tests, typed, CI green, clean structure.

### Trust & Transparency
- **Means:** Can you figure out who's behind this and what it does?
- **Signals:** license (OSI-approved?), identifiable maintainer (real name / org / history on GitHub), honest README (matches actual behavior), clear changelog, no crypto/affiliate nags, telemetry disclosed.
- **1:** no license or unclear author, README overpromises. **3:** MIT/Apache, pseudonymous but active maintainer. **5:** clear license, known maintainer with track record, honest README, changelog, telemetry disclosed.

## 4. Data-gathering method

Automatable via `gh` CLI + WebFetch:
- `gh repo view <url> --json stargazerCount,forkCount,pushedAt,licenseInfo,isArchived`
- `gh api repos/<owner>/<repo>/commits?per_page=30` — recency/frequency
- `gh api repos/<owner>/<repo>/contributors` — contributor count
- `gh api repos/<owner>/<repo>/issues?state=open` + closed — response time
- `gh api repos/<owner>/<repo>/releases`
- `gh api repos/<owner>/<repo>/contents/package.json` (or `pyproject.toml`, `requirements.txt`) — deps
- `WebFetch` README, SECURITY.md, CHANGELOG.md
- Grep the repo for `eval(`, `exec(`, `child_process`, secret-like strings

Needs human judgment: does the README match the code, is the scope appropriate for the use case, is the maintainer trustworthy, is obfuscation intentional.

## 5. Output format

```markdown
# Skill Audit: <repo-name>
**URL:** <url>  **Audited:** <date>  **Use case:** <user-supplied>

## Verdict: Use / Use with caution / Skip
<2-sentence why>

## Scores
| Dimension | Score | Notes |
|---|---|---|
| Active Development | 4/5 | Last commit 3 days ago, 12 commits in 90d |
| Adoption & Community | 2/5 | 47 stars, 1 contributor |
| Security Posture | 3/5 | Scopes OK, 2 stale deps (non-critical) |
| Code Quality | 3/5 | Tests exist, CI missing |
| Trust & Transparency | 4/5 | MIT, maintainer has 5yr GH history |

## Per-dimension reasoning
### Active Development — 4/5
<bullets>

(… repeat for each …)

## Recommended next steps
- [ ] Pin to commit `<sha>` rather than main
- [ ] Sandbox credentials / use a dedicated service account
- [ ] Re-audit in 90 days
```

## 6. Edge cases

- **Private repos:** require user to confirm `gh auth` has access; else degrade gracefully and flag the dimensions that couldn't be checked.
- **Monorepos / bundled skills:** accept a subdirectory URL; scope Code Quality and Security to that path, but Adoption/Trust stay repo-level.
- **Forks of popular skills:** audit the fork's divergence (`gh api .../compare/upstream...fork`) separately; flag if fork adds network calls or new deps.
- **No stars but from known org:** Adoption score can float up to 3 if org is recognized (Anthropic, a major vendor, etc.).
- **Archived repos:** Active Development auto-caps at 1; verdict can't exceed "Use with caution".

## 7. Reference trigger

Bootstrap this skill by invoking the existing `skill-creator` skill (at `~/.claude/skills/skill-creator/` or equivalent) with this spec as input. `skill-creator` handles SKILL.md scaffolding, trigger-phrase tuning, and install path.
