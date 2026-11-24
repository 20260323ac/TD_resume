# Coordination, Records, and Reporting

## Worker model

Use separate workers only when the host supports delegation and the user/task authorizes it:

- **BOSS discovery/application worker:** finds and screens BOSS jobs; prepares one greeting for each approved job. It does not send before batch approval.
- **智联 discovery/application worker:** finds and screens 智联 jobs; prepares the resume-submission action. It does not send before batch approval.
- **Response analyst:** observes user-authorized post-application conversations, classifies replies, and calculates outcomes. It does not answer recruiters.
- **Coordinator (primary agent):** owns profile confirmation, shared candidate ledger, cross-platform deduplication, batch approval, sending authorization, reconciliation, and final report.

Workers should return structured records to the coordinator rather than sending the user repeated progress messages. Keep user-facing updates concise: profile confirmation, exact batch approval, blockers requiring user action, and final report. “Background” means minimize interruption while the authorized task is actively running; it does not imply hidden operation, persistence after task end, or unattended account access.

Do not run concurrent workers against the same account/session. Search and preparation may run in parallel; browser actions should be serialized per account. If the UI is not stable or target identity cannot be verified, stop that queue.

## Candidate ledger fields

Use a privacy-minimized ledger with:

`platform`, `employer_name`, `role_title`, `city`, `district_or_area`, `salary_as_listed`, `job_url`, `observed_at`, `posted_or_updated_at`, `employment_type`, `experience_requirement`, `education_requirement`, `industry`, `company_size`, `financing_stage_if_shown`, `core_duties`, `hard_requirements`, `resume_evidence`, `gaps`, `employer_identity_evidence`, `opening_activity_evidence`, `risk_signals`, `fit_disposition`, `confidence_label`, `selected_resume_projects`, `planned_action`, `message_text_if_any`, `batch_approval_state`, `send_state`, `sent_at`, `reply_state`, `reply_category`, `first_reply_at`, `notes`.

Do not store credentials, verification codes, unnecessary private conversation text, or sensitive personal data. Store only information needed to avoid duplicates and produce the requested report. If a ledger is persisted, disclose its location and retain it only as long as useful for the user's job-search workflow.

## Batch rules

- Prepare up to 150 candidates for BOSS and up to 150 for 智联 per user-requested batch.
- Treat 150 as a target ceiling, not a quota. Report fewer when fewer qualify.
- Default to cross-platform deduplication on employer + role + city. A second platform touch requires explicit user opt-in for that duplicate.
- The user approves the exact list and action. Approval of profile criteria alone is insufficient.
- If job, employer, platform, resume version, message, or action changes materially after approval, return that item for renewed approval.
- Do not promise or target one submission per second. Speed must not remove identity checks, success verification, or platform rate-limit compliance.

## Response categories

Classify incoming messages as one of:

- `substantive_role_discussion`: specific duties, technical stack, team, or role scope.
- `interview_or_assessment`: concrete interview/test invitation.
- `generic_acknowledgment`: greeting or template without role-specific content.
- `role_closed_or_filled`: explicit closure or filled notice.
- `clarification_requested`: recruiter asks a question that needs the user's answer.
- `risk_or_mismatch`: asks for fees/loans, conflicts with listing, or requests suspicious/unnecessary information.
- `no_reply_in_window`: no reply by the configured observation cutoff.

The analyst may label hiring evidence as stronger or weaker, but cannot verify “genuine hiring” from a single message alone. Escalate interview scheduling, salary negotiation, personal-data requests, and any message requiring a commitment to the user.

## Metrics

Report each platform separately and state the observation window:

- qualifying jobs / jobs reviewed;
- user-approved jobs / qualifying jobs;
- verified successful sends / approved jobs attempted;
- substantive replies / verified successful sends;
- interview invitations / verified successful sends;
- median or clearly defined first-response time among replies;
- stale/uncertain/high-risk listing counts;
- duplicate and skipped counts with reasons.

Also segment cautiously by city, role family, compensation band, industry, and company size only when sample sizes support a meaningful comparison. Show denominators; avoid overinterpreting small samples. BOSS greeting and 智联 resume submission are different interventions, so raw response-rate differences do not establish platform causality.
