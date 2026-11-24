# Resume Job Apply Skill

A reusable Codex skill for resume-based job search, careful screening, user-approved outreach on BOSS直聘 and 智联招聘, and evidence-based response analysis.

## Install

Copy the `resume-job-apply` directory into one of these locations:

- **Personal skill:** `~/.codex/skills/resume-job-apply/`
- **Project skill:** `<project-root>/.agents/skills/resume-job-apply/`

Keep `SKILL.md` at the skill directory root and preserve the `references/` folder. Restart or refresh Codex if the skill does not appear immediately. On another agent platform, adapt the frontmatter and skill-discovery directory to that platform's documented format.

Invoke explicitly with `$resume-job-apply`, or describe a resume-based job search. The skill does not contain the user's resume, credentials, or account data.

## Behavior summary

- Requires profile confirmation and a separate exact-batch confirmation before external applications.
- Prepares up to 150 qualifying openings per platform; it does not relax criteria to hit a quota.
- BOSS: one tailored greeting only, no resume attached, no automated follow-up.
- 智联招聘: submit the approved, already-uploaded resume.
- Supports parallel discovery and a separate response analyst when delegation is available, while serializing UI control per account.
- Never bypasses CAPTCHA, login, rate limits, or platform controls.

See `references/platform-workflow.md`, `references/screening-and-evidence.md`, and `references/coordination-and-reporting.md` for operational details.
