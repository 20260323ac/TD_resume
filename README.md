# Resume Job Apply

Skill instruction version **2.12**, updated from the reviewed v2.11 plan with the deduplication state/reason-code corrections from v2.12. It analyzes a supplied resume, screens openings, prepares an exact batch for user approval, and supports evidence-based outcome reporting on BOSS直聘 and 智联招聘.

## Install

**From a local skill-root directory:** if the directory directly contains `SKILL.md` (as `D:\Skill\resume-job-apply` does), use that directory as the skill root. Register or link that directory with the host; do not nest another `resume-job-apply` directory inside it.

**From the current flat GitHub repository:** clone directly to the target skill path, for example:

```powershell
git clone <repository-url> "$HOME\.codex\skills\resume-job-apply"
```

The clone root itself is the skill root because it contains `SKILL.md`. For an older repository revision that contains `skills/resume-job-apply/`, use that subdirectory as the skill root, not the repository root. Keep `references/` and `agents/` alongside `SKILL.md`; refresh/restart the host if the skill is not discovered.

## Use and limits

- The user manually signs in and selects the uploaded resume. The skill never handles credentials, verification codes, CAPTCHA, or platform restrictions.
- Profile confirmation and exact per-batch confirmation are separate. A batch confirmation does not carry over.
- Maximum 150 qualifying records per platform per batch; never lower screening standards to fill the target. BOSS sends one approved greeting only; 智联 submits the approved uploaded resume.
- No unattended monitoring or hidden automation. Real sending requires the user's explicit authorization for the exact batch and action.
- Runtime records are stored outside the repository, by default under `C:\Users\LiYan\.codex\resume-job-apply-data\` on this computer; approved outgoing greetings are kept for 90 days and the minimal deduplication index for 365 days. Runtime records are never included in an install package.

## Reference map

- `references/intake-contract.md`: resume facts, user statements, inferences, unknowns, and the one-time preference check.
- `references/screening-and-evidence.md`: fit, company/opening evidence, salary normalization, education, and transferability.
- `references/ledger-schema.md`: durable records, IDs, event protocol, privacy, retention, recovery, and cleanup.
- `references/coordination-and-reporting.md`: deduplication, worker boundaries, aggregation, metrics, and reporting.
- `references/platform-workflow.md`: platform-specific actions, UI evidence, pauses, and resumption.
- `references/templates.md`: exact-batch summary, outgoing greeting, and report formats.

The user's resume, credentials, and runtime ledger are not part of this package. See `CHANGELOG.md` for the human-readable version history.
