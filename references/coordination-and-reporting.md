# Coordination and Reporting

## Worker boundaries

Use delegation only when the host supports it and the current user/task authorizes it. Discovery can be parallel; a worker returns evidence-backed candidate records and never submits before coordinator approval. A response analyst may classify responses during the authorized active task but does not answer recruiters. The primary coordinator owns profile confirmation, deduplication, manifests, batch confirmation, the sole writer lock, reconciliation, and final reporting. Never let concurrent workers control the same account/session or write the same data path. Keep updates concise, but surface any needed user decision immediately.

“Background” means minimize interruptions during the active authorized task only; it does not mean hidden operation, unattended persistence, or post-task account monitoring.

## Deduplication and state

Follow the three distinct keys and priority in `ledger-schema.md`, using only declared `application_state` values: a confirmed same-platform historical four-tuple becomes `duplicate_skipped`; an uncertain contact becomes `risk_hold` + `reason_code=may_have_contacted`; a `state_conflict` becomes `risk_hold` + `reason_code=state_conflict`; same-batch or prior cross-platform duplicates become `risk_hold` + `reason_code=cross_platform_duplicate` until the user chooses or opts in. Same company but different role is not a duplicate. Record the state and reason on the candidate ledger row; do not invent `duplicate_uncertain`, `duplicate_cross_platform`, or use `state_conflict` as an application state. Keep `reason_code` separate from attempt/queue `stop_reason_code` and from `note_code`.

`contacted.csv` is only a privacy-minimized deduplication index; never compute response or employer statistics from it. All statistics come from persisted `ledger.csv` rows, grouped by `candidate_id` so retries do not inflate denominators. Apply the highest-priority `state_conflict` rule before any ordinary success/uncertain/failure classification.

## Reply attribution and metrics

Count a reply only when its timestamp is after the relevant confirmed send, the visible conversation/application record identifies the specific role, and it is not another role at the same employer. If any check fails, leave send state unchanged and add the item to attribution-pending `A`; do not store recruiter text. One candidate counts at most once in `R`; multiple messages for that candidate do not multiply it. A later reply must not overwrite the application state: update `reply_state` independently.

At report time, calculate per platform and per candidate:

- `R`: uniquely attributed responding candidates;
- `S`: candidates whose aggregate application state is `sent_successfully`;
- `U`: candidates whose aggregate state remains `send_status_uncertain`;
- `A`: count and privacy-minimized list of replies with unresolved attribution, excluded from `R`.

Show the two denominator endpoints using the same `R`: lower endpoint `R/(S+U)`, upper endpoint `R/S`; these are not a complete bound on the true rate if `A > 0`. Report bases and formula; do not show any ratio containing `A`. If denominator is zero, say “not calculable,” not zero percent. Clearly state that BOSS greetings and 智联 resume submissions are different treatments; response-rate comparison does not prove platform causality.

Include reviewed/qualifying/approved counts, verified sends, failed/uncertain/held/skipped reasons, replies, substantive replies, interview invitations, first-response latency among attributed replies, company/opening confidence, and sample-size caveats. Report by platform separately. Segment by city, role family, pay band, industry, company size, or template only when the sample supports a useful comparison.

Observation window is 72 hours as a configurable working default, not a platform fact. It starts per candidate at confirmed `sent_at`; for uncertain candidates use `attempted_at` marked estimated. Batch-level full-window time is the last confirmed successful send plus 72 hours, but mark individual candidates still within their own windows. The skill ends after the batch and does not poll in the background. Only when the user runs it again, generate a report with `report_generated_at` and full/partial-window flags.

## Stop reason codes

Use only concise codes, never page or conversation excerpts: `page_mismatch`, `duplicate_warning`, `captcha`, `rate_limit`, `login_expired`, `timeout`, `ambiguous_behavior`, `write_failed`. Visible signals trigger pause; do not invent numeric platform quota assumptions. Company/batch limits, daily limits, and consecutive-failure limits remain unset until the user chooses values. The 150-per-platform-per-batch maximum is a ceiling and does not authorize maximizing throughput.

## Retention and report footer

Use `ledger-schema.md` for the complete retention, cleanup transaction, and privacy rules. Default retention: ledger/closed attempt evidence and batch/greeting content 90 days; contacted dedup index 365 days. Prompt before deletion and list affected counts/date ranges. Keep all runtime records outside Git and installation packages. End reports with the absolute data location, active retention periods, and next cleanup reminder; do not include recruiter-side message text or personal contact details.

## Report outline

1. Batch ID/hash, approved count, selected resume version, action, and report timestamp.
2. Per-platform reviewed/qualified/approved/sent counts and exceptions.
3. `R/S/U/A` bases, `A` count/list, both endpoints and formula, denominator/window caveats.
4. Candidate-level outcome detail grouped by employer, with evidence and concise reason codes.
5. Risks, conflicts, uncertain states, and items requiring user action.
6. Observed patterns with sample-size limitations; next-batch proposals (not applied automatically).
7. Data path, retention, and next cleanup reminder.
