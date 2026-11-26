# Ledger, Event, and Retention Schema

This reference is normative for persisted state. Runtime data lives outside the skill/repository, by default under `~/.codex/resume-job-apply-data/`. On startup print absolute paths, read `location.json` (`{"data_path":"..."}`), and never silently change or fall back from a missing, damaged, or unwritable path. If durable writes or the required hash algorithm cannot be performed reliably, do not start external actions.

## Files and privacy

`<data_path>/` contains `ledger.csv`, `contacted.csv`, `attempts.log` (JSONL), `config.json`, `batches/`, `messages/`, optional `purge.plan.json`, and transient `writer.lock`. The stable default entry directory contains only `location.json`. CSV is UTF-8 with BOM and one header row; JSON files are UTF-8 JSON objects; JSONL has no header. Initialization must state formats, absolute paths, and retention.

Never persist recruiter-side message text, names/contact details, resume content/attachments, passwords, verification codes, login tokens, or unnecessary sensitive data. Do not put runtime data in the repository, install package, or release. The sole text exception is the user's approved outgoing greeting, stored in `messages/<batch_id>/<candidate_id>.txt` for 90 days; it must contain no recruiter data or resume content. `resume_delivery` has `message_hash: null` and no text file.

## Initialization and local config

On first use, print both absolute paths: stable pointer directory and resolved `data_path`. If `location.json` is absent or invalid, ask whether to recreate it; never silently choose another location. Do not initialize if the selected directory cannot be written. The default pointer location is `C:\Users\LiYan\.codex\resume-job-apply-data\location.json` on this computer. The pointer contains only `data_path`; keep actual records in the resolved data directory.

`ledger.csv` and `contacted.csv` are UTF-8 with BOM plus exact headers below; `attempts.log` is an empty UTF-8 JSONL file with no header; `config.json` is a JSON object; `batches/` and `messages/` start empty. Read back and verify every file after initialization, then print paths, formats, retention, and cleanup reminder. Do not create real candidate rows during installation.

Example `config.json` (optional numeric limits stay unset until the user chooses them):

```json
{
  "ledger_retention_days": 90,
  "contacted_retention_days": 365,
  "message_retention_days": 90,
  "observation_window_hours": 72,
  "per_company_batch_limit": null,
  "daily_send_limit": null,
  "consecutive_failure_limit": null,
  "ledger_purged_through": null,
  "ledger_fingerprint": null,
  "next_event_seq": 1
}
```

`ledger_fingerprint` is `{row_count, earliest_recorded_at, sha256}`, recomputed only under the writer lock and with no unfinished transaction. A mismatch means `watermark_pending_confirmation`; do not reconcile or resume until the user confirms a new watermark.

## Lock and purge plan

`writer.lock` is acquired by exclusive create (`CreateNew`/equivalent), never by overwriting an existing lock. Its JSON body contains `pid`, random `instance_id`, `started_at`, `heartbeat_at`, and current `batch_id` (or null). Release only a lock owned by this instance. A live or stale unverified lock blocks writes; after four hours without heartbeat, ask the user before removal.

`purge.plan.json` is an atomic transaction journal, not permission to delete arbitrary paths. It contains `transaction_id`, `created_at`, `ledger_cutoff`, `contacted_cutoff`, prior/proposed watermark and fingerprint, and a fixed list of repository-external relative targets with action, expected pre-state hash/count, and expected post-state hash/count. Reject absolute paths, `..` traversal, paths outside `data_path`, unknown file classes, or plan/hash mismatches. Flush the plan before replacing any file. Replace via same-directory temporary file and atomic rename in order `ledger.csv` → `attempts.log` → `contacted.csv` → eligible batch JSON files → associated greeting files; verify each post-state, then commit watermark/fingerprint last and delete the plan. On restart, validate the plan and compare every target against pre/post state; skip completed replacements, redo only verified pre-state, and stop for user review on any third state. Never infer completion from file existence alone.

## `ledger.csv`

One explicit `row_type` column distinguishes `candidate` and `attempt`; never infer row type from status. Suggested fixed header (empty cells are allowed only where a field does not apply):

```text
row_type,batch_id,batch_hash,candidate_id,platform,company_name,employer_key,role_name,role_key,city,district_or_area,job_url,salary_as_listed,salary_min,salary_max,salary_months,pay_type,bonus_note,salary_normalized_monthly,salary_assumption,observed_at,posted_or_updated_at,employment_type,experience_requirement,education_requirement,industry,company_size,financing_stage_if_shown,core_duties,hard_requirements,resume_evidence,gaps,employer_identity_evidence,opening_activity_evidence,risk_signals,fit_disposition,match_conclusion,confidence_label,selected_resume_projects,exception_flags,planned_action,action,template_id,message_hash,application_state,reply_state,reply_category,first_reply_at,attempt_id,attempt_no,attempted_at,sent_at,reason_code,stop_reason_code,approved_by,approved_at,queue_position,note_code
```

- `candidate` row: screening/approval and `reply_state`; no `attempt_id`/`attempt_no`, no `sent_*` state, and no recruiter text. Candidate states: `candidate`, `screened_in`, `screened_out`, `awaiting_approval`, `approved`, `duplicate_skipped`, `risk_hold`, `quota_blocked`, `resume_mismatch`, `closed_or_expired`, `batch_cancelled`.
- `attempt` row: one row per actual attempt, with unique `attempt_id` and increasing `attempt_no`; no `reply_state`; `application_state` is only `sent_successfully`, `sent_failed`, or `send_status_uncertain`. Never overwrite/delete an earlier attempt to record a retry.
- `reason_code` is for candidate-level screening/deduplication holds; it is separate from `stop_reason_code`, which records why an attempt/queue stopped. Use only `may_have_contacted`, `state_conflict`, and `cross_platform_duplicate` for the deduplication cases below. For same-platform confirmed contact, use `application_state=duplicate_skipped`; `reason_code` may remain empty because the state itself identifies the skip. Do not put deduplication reasons into `stop_reason_code` or `note_code`.
- Deduplication mapping: `contacted_confirmed` → `application_state=duplicate_skipped`; `contact_uncertain` → `application_state=risk_hold`, `reason_code=may_have_contacted`; `state_conflict` → `application_state=risk_hold`, `reason_code=state_conflict`; same-batch or prior cross-platform duplicate → `application_state=risk_hold`, `reason_code=cross_platform_duplicate`. Do not invent `duplicate_uncertain`, `duplicate_cross_platform`, or `state_conflict` as `application_state` values. Keep the candidate held until the user resolves the cross-platform choice or conflict; resolution does not itself authorize sending, which still requires exact batch confirmation.
- A confirmed retry may move the candidate back to `approved` only after visible evidence confirms the prior attempt was not sent. An uncertain attempt is never retried until reconciled.
- Dates are ISO 8601 with timezone. `attempted_at` is the intent/action time; `sent_at` is recorded only for confirmed sends. Report states and metrics are derived from these persisted rows plus the event log, not conversation memory.
- `reply_category` is one of `substantive_role_discussion`, `interview_or_assessment`, `generic_acknowledgment`, `role_closed_or_filled`, `clarification_requested`, `risk_or_mismatch`, or `no_reply_in_window`; `first_reply_at` is set only for an attributable reply. Store no message text. `note_code` is restricted to `prefilter`, `backfill`, or `manual_confirm`; it is not free text.

Reason-code namespaces are disjoint:

| Field | Applies to | Values |
|---|---|---|
| `reason_code` | Candidate screening/deduplication | `may_have_contacted`, `state_conflict`, `cross_platform_duplicate` |
| `stop_reason_code` | Attempt/queue stop | `page_mismatch`, `duplicate_warning`, `captcha`, `rate_limit`, `login_expired`, `timeout`, `ambiguous_behavior`, `write_failed` |

Legacy name mapping (read for migration only; write only the current name):

| Legacy value | Current value |
|---|---|
| `awaiting_batch_approval` | `awaiting_approval` |
| `send_failed` | `sent_failed` |

## IDs and event log

`candidate_id` is derived from `[platform, employer_key, role_key, city]`: platform is a lowercase platform enum; employer/role keys are NFC, lowercase, trimmed, and internal whitespace collapsed; city is NFC and trimmed. Serialize the JSON array with strict RFC 8785 JCS, UTF-8 encode, SHA-256, and use `c1-` plus all 64 lowercase hex characters. Keep raw tuple fields for review. The ID is immutable once written; changed normalization affects new candidates only. Any apparent same-ID/different-tuple record is quarantined for full-tuple review and explicit audited rename, never silently merged. Maintain fixed regression samples covering Chinese, whitespace, case, and delimiter characters.

Fixed regression vectors (the strings below are the normalized tuple serialized as compact JCS JSON; SHA-256 is over the displayed UTF-8 bytes):

| Normalized tuple | Expected `candidate_id` |
|---|---|
| `["boss","甲有限公司","firmware engineer","深圳市"]` | `c1-7766e3dd5b7da63dfcfacd5619ecae3fa04dd8748b2f9ed9f7a217d048143170` |
| `["zhilian","alpha|科技","嵌入式软件工程师","郑州市"]` | `c1-bec52414589bb3fdebe280702f8f203f9b43b1be73ccf7720f24c5021bd770a5` |

The second vector intentionally includes a delimiter; a delimiter is data because the tuple is a JSON array, not joined text. Add an independent test where each tuple field changes by one normalized value and verify that the resulting full digest changes.

`attempt_id = <batch_id>#<candidate_id>#<attempt_no>`. `event_id` (ULID/UUID) is a unique reference only; never sort it. `event_seq` is a strictly increasing integer allocated by the sole writer from `config.json.next_event_seq` while holding the lock: atomically persist the increment before appending the event. Gaps are valid; duplicates are corruption and stop processing. On recovery set the next value to `max(config value, largest logged event_seq)+1`.

Every attempt follows this durable write-ahead sequence:

1. Append and flush an `intent` event before clicking. If it cannot be confirmed durable, do not click.
2. Perform the user's approved visible action once.
3. Append and flush `result` (`sent_confirmed`, `failed`, or `unverified`). If not durable, do not write the ledger.
4. Idempotently write one `attempt` row keyed by `attempt_id`; pause the send queue if the write fails.
5. Recompute the candidate's `contacted.csv` aggregate and upsert/delete accordingly.

“Written” means append, close, and explicitly flush to storage (`fsync`/equivalent), not merely process/system buffering. Event rows contain only approved IDs, batch/candidate/attempt references, platform and four-tuple, action/template, result/reason code, evidence type, correction reference, and timestamps—never page text or recruiter data. `corrects_event_id` is permitted only for `resolution_correction`.

Event shapes:

```json
{"event":"intent","event_id":"...","event_seq":1,"attempt_id":"...","candidate_id":"...","attempt_no":1,"batch_id":"...","platform":"boss","company_name":"...","employer_key":"...","role_name":"...","role_key":"...","city":"...","action":"greeting","template_id":"tpl-boss-v1","occurred_at":"..."}
{"event":"result","event_id":"...","event_seq":2,"attempt_id":"...","result":"sent_confirmed|failed|unverified","stop_reason_code":"...","occurred_at":"..."}
{"event":"resolution","event_id":"...","event_seq":3,"attempt_id":"...","candidate_id":"...","resolved_as":"sent_confirmed|failed","evidence_type":"page_bubble|delivery_record|receipt|user_attestation","reason":"short_code","occurred_at":"..."}
{"event":"resolution_correction","event_id":"...","event_seq":4,"attempt_id":"...","candidate_id":"...","resolved_as":"sent_confirmed|failed","evidence_type":"page_bubble|delivery_record|receipt|user_attestation","corrects_event_id":"...","reason":"required_short_code","occurred_at":"..."}
```

For an `unverified` result, allow one `resolution` root per attempt. Follow `corrects_event_id` references to the unique chain tip; `event_seq` is for audit, not deciding the tip. Corrections must point to the current tip. Multiple roots, forks, cycles, dangling/cross-attempt references, or disagreement with page evidence are E-class conflicts; never choose a “last” event automatically. A valid tip resolves the attempt as `sent_confirmed` or `failed`, may rebuild its attempt row, and releases U restrictions. Only confirmed `failed` permits retry.

## Recovery and queue safety

At every startup and before search/filtering: acquire the single-writer lock; read log and ledger fully; index by `attempt_id`; reconcile both directions; then recompute `contacted.csv`; print a reconciliation list. Classify: A=intent without result (verify page, otherwise append `unverified`); B=known result but missing attempt row (idempotently rebuild; sending remains paused until write succeeds); U=`unverified` without valid resolution (no resend/cleanup); C=attempt row missing intent (log mismatch); D=duplicate attempt ID; E=conflicting state/chain. Candidate rows never require log events. A/U/C (for that candidate)/D/E block affected sending; resolve before resuming. Do not rely on in-memory state after interruption.

If a confirmed result exists but the ledger row is missing, the log can rebuild only attempt facts. It cannot reconstruct `reply_state`, reply time, approver, or fit rationale; mark those `unknown` and ask for review. Never claim the full ledger can be recovered from the event log.

## `contacted.csv` and aggregation

Exact 11 columns:

```text
platform,employer_key,role_key,city,contacted_at,source_batch_id,template_id,send_state,note_code,last_event_seq,source_attempt_id
```

It is only a deduplication index, not a reporting source. Match exact platform + normalized employer + normalized role + city. Aggregate all attempts per candidate as a set, with the conflict rule first:

1. Let `S` be the earliest `sent_confirmed` conclusion by `event_seq`. If any later conclusive `sent_confirmed` or `failed` event has a larger `event_seq`, record `state_conflict` and freeze that candidate, even if it succeeded before. Point the source fields to the later conclusion that creates the conflict. A `failed` followed by success is a valid retry, not conflict.
2. Otherwise any confirmed success gives `contacted_confirmed`; `contacted_at` and source fields point to the earliest successful send.
3. With no success but any unresolved A/U/`unverified`, use `contact_uncertain` and conservatively block repeat contact.
4. If all attempts are confirmed `failed`, do not keep a row; delete an existing row so a later batch may retry.
5. Missing legacy source anchors become `unknown` pending review. `send_state` values are only `contacted_confirmed`, `contact_uncertain`, `state_conflict`, `unknown`.

Replaying the same event set must produce the same row. `last_event_seq` is an audit anchor, not a last-write-wins rule. A conflict freezes only that candidate, not unrelated candidates or the whole batch. Log any deletion in the event/audit record, not as a failed-contact row.

Deduplication priority: (1) same-platform historical four-tuple: skip; (2) same-batch cross-platform duplicate: ask the user to choose one; (3) prior cross-platform contact: require explicit user opt-in. Same company/different role is not a duplicate; flag for review only. Record each decision in the ledger.

## Batch manifest and hashes

`batches/<batch_id>.json` is immutable. Any removal/addition or change to resume version, action, template, or message creates `<old>-r<n>`; never overwrite. Candidate entries are sorted by `candidate_id`. Closed candidate fields are `candidate_id`, `platform`, `company_name`, `employer_key`, `role_name`, `role_key`, `city`, `job_url`, `salary_as_listed`, `resume_version`, `template_id`, `message_hash`, `action`, matching conclusion, and exception flags. No resume, greeting, recruiter identity, or contact text. The manifest identity is outside the entry fields; `batch_hash` is SHA-256 lowercase hex of strict JCS serialization of the complete manifest payload, excluding only the self-referential `batch_hash`. Do not normalize strings at hash time: JCS is strict RFC 8785 with no NFC/case/whitespace preprocessing. Keep manifest field definition and hash coverage in sync. No timestamps, random values, local paths, or self-hash in hashed payload.

For `action=greeting`, save the exact approved outgoing text as UTF-8 without BOM under `messages/<batch_id>/<candidate_id>.txt`. `message_hash` preprocessing is separate from JCS: Unicode NFC, normalize line endings to `\n`, trim leading/trailing whitespace, UTF-8 encode, SHA-256 lowercase hex. Recompute before sending; mismatch or missing text blocks the item and requires a new confirmed batch. For `resume_delivery`, retain the `message_hash` key with JSON `null` and create no text file.

Fixed synthetic `batch_hash` vector (test-only; not a real job record). This is the complete payload for one mock `resume_delivery` candidate; `batch_hash` itself is excluded because self-reference is forbidden. UTF-8 bytes of the exact canonical serialization below must hash to the expected lowercase hex:

```json
[{"action":"resume_delivery","candidate_id":"c1-test","city":"郑州市","company_name":"测试公司","employer_key":"测试公司","exception_flags":[],"job_url":"https://example.invalid/job/1","match_conclusion":"meets","message_hash":null,"platform":"zhilian","resume_version":"mock-v1","role_key":"embedded software engineer","role_name":"嵌入式软件工程师","salary_as_listed":"面议","template_id":null}]
```

Expected SHA-256: `e021ca2aa0cadeccaf7bb6eb8b8ba5224c8913f74b8a34d41509346e0ab73c51`. Recompute twice and compare bytes and digest. Change each field, including `action` or `message_hash: null`, and confirm the digest changes.

## `config.json`, locks, and cleanup

`config.json` holds only local settings: retention/window values, user-confirmed optional limits, `ledger_purged_through`, `ledger_fingerprint` (row count + earliest time + content hash), and `next_event_seq`. Do not invent limit defaults. `writer.lock` is exclusive per `data_path`; page viewing may be parallel, writes are serialized. A second instance is read-only or refuses execution. Never auto-steal a stale lock; after four hours without update ask the user. Cleanup holds the lock exclusively and blocks sends.

Retention defaults: ledger and closed attempt events 90 days; `contacted.csv` 365 days; manifests and greeting text 90 days. User gets a deletion-range summary and confirms before deletion. Before cleanup, reconcile; refuse while A, unresolved U, D, or E remain. Prepare `purge.plan.json`, atomically replace each file via temp + rename in order `ledger → attempts log → contacted → batch manifests → messages`, verify each, commit watermarks/fingerprint last, then remove plan and release lock. On restart, finish an existing plan idempotently before other work. Delete log events only when closed, their ledger rows are deleted, and they are beyond retention. Keep batches until all their ledger rows are purged; delete associated greetings with the batch. A manual ledger edit/deletion that changes fingerprint pauses reconciliation/resume until user confirms the new watermark. Manual deletion of logs, contacts, config, or message text must be disclosed with the corresponding recovery/deduplication limitation.

Repository isolation checks must inspect tracked file paths, not directory names. Never add runtime data, locks, temp files, `batches/**/*.json`, or `messages/**/*.txt` to Git or packages. If no Git repository is available, state that tracked-file isolation could not be verified; do not claim it passed.

On Windows/PowerShell, the repository-side check is the following and passes only when it prints no paths:

```powershell
git ls-files | Where-Object {
  $_ -match '^(location\.json|config\.json|ledger\.csv|contacted\.csv|purge\.plan\.json|attempts\.log.*|.*\.tmp|.*\.lock|batches/.+\.json|batch-.+\.json|messages/.+\.txt)$'
}
```

This output must be empty; it does not prove runtime data exists. Separately, with `$d` set to the absolute external data path, verify data-side completeness:

```powershell
$d = 'C:\Users\LiYan\.codex\resume-job-apply-data'
(Get-ChildItem "$d\batches" -File -Filter *.json).Count
(Get-ChildItem "$d\messages" -Recurse -File -Filter *.txt).Count
```

The batch count must match retained manifests. The message count must equal the number of retained `action=greeting` candidates across those manifests; verify each batch separately. `resume_delivery` requires no text file. If this directory is not a Git worktree, report the repository-side check as unavailable rather than empty/passed.
