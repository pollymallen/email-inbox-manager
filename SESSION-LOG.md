# Session Log

Older session entries are in `SESSION-LOG-archive.md`.

## Project Context
Email Inbox Manager is an open-source Claude Code skill (`github.com/pollymallen/email-inbox-manager`, MIT, public) that learns a user's personal email organization system and helps clean up, consolidate, and maintain their inbox. **Sessions 1–3 (Apr 6–13):** extracted the design from a claude.ai conversation, set up the GitHub repo + local symlink install, iterated SKILL.md with team rollout / scheduling / conversational query, hardened safety (permission model, no-delete guarantee, `To Trash` quarantine, Phase 0a/0b account-check + backup, Phase 0.5 historical cleanup), added Phase 00 Welcome & Orientation, progressive trust model (always-ask / session-auto / persistent-auto) and three-option graduation prompts, honest security framing, newcomer-friendly "You are here" markers, README gws-setup section, and `SPIN-OFF-skill-evaluator-spec.md`. Dry-run subagent caught + cleared 5 P0 + 7 P1 + 4 P2 bugs before first live test. **Session 4 (Apr 14):** pushed 7 commits, installed `googleworkspace-cli`, discovered `gws` ships no MCP server — pivoted transport to **gws-via-Bash** and rewrote SKILL.md Transport Layer with an operation→transport mapping table covering both connector and gws. OAuth setup under work GCP hit `constraints/iam.allowedPolicyMemberDomains` org policy. **Session 5 (Apr 17 night → Apr 18 early AM):** unblocked the OAuth wall. Created `email-inbox-mgr-personal` under personal gcloud (outside any org), set up consent screen + Data Access scopes + test user in the new Google Auth Platform UI, re-ran `gws auth login` with explicit `gmail.modify` scope, verified `gws gmail users getProfile` — returns real inbox (54,942 messages / 51,475 threads). **Current state:** skill is ~970 lines with transport abstraction live and pushed; first live onboarding run is unblocked and queued for the next session at `~/projects/personal/email/pollymallen-gmail/`.

## Session 4 — 2026-04-14 (ground, partial)
**Phase:** Phase 3 kickoff — transport pivot + OAuth setup (blocked mid-flight)
**Goals:** Push local commits, install gws, finish OAuth, run first live test.

**Done:**
- **Pushed 7 commits** to origin/main (previously blocked on wifi)
- **Installed `googleworkspace-cli` 0.22.5** via brew
- **Discovered `gws` has no `mcp` subcommand.** Session 2/3 assumed `gws mcp` would register as an MCP server in Claude Code. It doesn't exist — `gws` is a pure CLI like `gcloud`. Architecture pivot required.
- **Chose Option 1: gws-via-Bash.** Skill invokes `gws gmail users ...` commands through Claude Code's Bash tool. No intermediate MCP wrapper (avoids rebuilding the `evolsb` problem).
- **Rewrote SKILL.md Transport Layer (~80 new lines):**
  - Detection logic (check MCP tools vs `which gws`)
  - Full operation→transport mapping table: identify account / list labels / create label / search threads / get thread / apply label / remove label / archive / To Trash / draft — with both connector calls and `gws` command strings
  - Forbidden methods under gws: `threads trash`, `threads delete`, `messages trash`, `messages delete` — NEVER call
  - Notes on Gmail label ID resolution, bulk ops, error handling
- **Reframed delete guarantee per transport:** structural under connector (no delete tool exposed at all); skill policy under gws (skill never calls delete APIs even though they'd technically work). Phase 00 welcome text updated to explain both honestly.
- **Updated Phase 0a account check** to cover both transports: `in:sent` for connector, `gws gmail users getProfile` for gws.
- **Updated README Option B** — dropped obsolete "register gws mcp" step, added `gws auth setup --login` path (when gcloud is present), added verification commands (`gws auth status`, `gws gmail users getProfile`).
- **OAuth setup under work GCP:**
  - Created project `email-inbox-mgr-polly` under `polly.allen@aicareerboost.com`
  - Set up OAuth consent screen (External, Testing) + added pollymallen@gmail.com as Test User
  - Created Desktop OAuth client, downloaded to `~/.config/gws/client_secret.json`
  - Enabled `gmail.googleapis.com`
  - Ran `gws auth login` — authed as pollymallen@gmail.com with readonly + labels scopes (missing `gmail.modify` — wrong scope picker selection)
- **Committed and pushed** the transport rewrite (commit `eac73d9`, 2 files changed, 128 insertions)

**Unfinished / blocked:**
- **IAM blocker (hard):** adding `user:pollymallen@gmail.com` as a principal on `email-inbox-mgr-polly` was rejected with `FAILED_PRECONDITION: User pollymallen@gmail.com is not in permitted organization` — the aicareerboost.com Workspace org has `constraints/iam.allowedPolicyMemberDomains` restricting IAM to `@aicareerboost.com` members. Cannot run Gmail API calls (403 `serviceUsageConsumer` required).
- **Missing `gmail.modify` scope** in the current `gws auth login` session — scope picker yielded readonly + labels instead.
- Live onboarding test not yet run.
- Sample populated governance map still pending.

**Decision queued (needs Polly next session):** Flip to **Option B** — redo GCP project under `pollymallen@gmail.com`'s personal gcloud identity (outside any org), which avoids the org-policy entirely. The `email-inbox-mgr-polly` project under work GCP is effectively abandoned (can be deleted for cleanup, or left as-is — no cost).

**Key decisions this session:**
- Transport: **gws-via-Bash**, not an MCP wrapper. Same pattern as `gdrive` CLI in CLAUDE.md. Simpler, no third-party wrapper to audit.
- GCP project must live outside `aicareerboost.com` org because of `allowedPolicyMemberDomains` — this is a one-time learning, write it down so future multi-account setups don't repeat the mistake.
- Scope at login: use `--scopes` flag explicitly with `gmail.modify`, not `--services gmail` which invokes the picker and led to wrong-scope selection.

## Session 5 — 2026-04-17 night → 2026-04-18 early AM
**Phase:** Phase 3 kickoff — OAuth unblock under personal gcloud
**Goals:** Flip to Option B, set up personal GCP project + OAuth, verify Gmail API end-to-end so the next session can run the live test.

**Done:**
- **Verified `gcloud` identity** — pollymallen@gmail.com was already active (Polly had run `gcloud auth login` before the session)
- **Created personal GCP project `email-inbox-mgr-personal`** (project number `272170192891`) under pollymallen@gmail.com, outside any org
- **Enabled Gmail API** (`gmail.googleapis.com`)
- **Backed up old work-org credentials** to `~/.config/gws/client_secret.json.work-org-backup`
- **Set up OAuth consent screen** via the new "Google Auth Platform" UI at `console.cloud.google.com/auth/*?project=email-inbox-mgr-personal`:
  - User type: External
  - Publishing status: Testing
  - Test users: pollymallen@gmail.com
  - **Data Access scopes registered:** `openid`, `userinfo.email`, `userinfo.profile`, `gmail.modify`
- **Created Desktop OAuth client** under the new project → downloaded `client_secret_272170192891-*.apps.googleusercontent.com.json` → moved to `~/.config/gws/client_secret.json` (confirmed `project_id: email-inbox-mgr-personal`)
- **`gws auth logout`** to clear residual work-org tokens
- **`gws auth login --scopes "https://www.googleapis.com/auth/gmail.modify"`** — authed successfully, granted 6 scopes incl. `gmail.modify`
- **Verified Gmail API:** `gws gmail users getProfile --params '{"userId":"me"}'` returns `pollymallen@gmail.com`, 54,942 messages / 51,475 threads — real inbox data

**Debug detours worth remembering:**
1. First attempt at OAuth consent screen URL showed a "You need additional access" page — caused by the browser defaulting to a different Google account than pollymallen@gmail.com. Fix: switch account via top-right avatar, or use incognito. (The CLI identity was always correct; only the browser session was wrong.)
2. First Desktop OAuth client got accidentally created in `mineral-pride-156121` ("My First Project" — GCP auto-created default) because the console's project picker top-bar overrode the `?project=` URL param after the account switch. Caught via project-number check in the downloaded filename (`921967829181` vs expected `272170192891`). Redid in the correct project.
3. First `gws auth login` attempt hit "Access blocked: Email Inbox Manager has not completed the Google verification process" with **no** Advanced link — a hard block, not the usual unverified-app warning. Root cause: **test user hadn't propagated / wasn't yet attached** in the new Auth Platform UI's Audience tab. Re-adding pollymallen@gmail.com as Test User fixed it — the block then became the soft "Advanced → Go to (unsafe)" flow.
4. First successful login granted only basic profile scopes — `gmail.modify` was missing. Root cause: the new consent flow no longer shows checkboxes. The initial login happened before Data Access scopes were registered, so Google only granted what was declared. Fix: register all four scopes in Data Access tab → `gws auth logout` → re-login. Second login correctly showed "Read, compose, and send emails from your Gmail account" on the consent page and `Continue` granted it.

**Unfinished:**
- **Live onboarding run not yet done** — queued for next session with full context (see "To resume")
- Sample "fully populated" governance map still pending (generated during/after live test)
- (Optional) Delete abandoned `email-inbox-mgr-polly` in work GCP
- (Optional) Delete the accidentally-created OAuth client in personal `mineral-pride-156121`

**Blockers:** None. Phase 3 is fully unblocked.

**To resume:**
1. `cd ~/projects/personal/email/pollymallen-gmail && claude`
2. Say "Let's set up my email system"
3. Skill should fire → Phase 00 welcome → Phase 0a account check (`gws gmail users getProfile` under the hood, should match pollymallen@gmail.com) → Phase 0b backup → Phase 0.5 historical cleanup decision → Phase 1 onwards
4. Take notes on any skill prompt / flow issues as they happen — feed back into SKILL.md refinements
5. After the first pass, generate a sample "fully populated" governance map from the real run for reference

**Key decisions this session:**
- **GCP projects for personal Gmail API access MUST live outside any Workspace org.** Workspace orgs commonly have `constraints/iam.allowedPolicyMemberDomains` which rejects personal-gmail principals. No amount of project-level config can bypass it. Always start from `gcloud auth login` as the personal identity.
- The new Google Auth Platform UI requires Data Access scopes to be registered **before** OAuth login, and test users must be attached in the Audience tab. Old "OAuth consent screen" wizard is gone — the per-tab model (Branding, Audience, Clients, Data Access) is load-bearing.
- After changing scopes or test users, always `gws auth logout` before re-running `gws auth login` — old refresh tokens won't auto-upgrade scope.
