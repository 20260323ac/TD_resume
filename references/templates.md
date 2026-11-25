# Batch, Greeting, and Report Templates

## Exact-batch confirmation

Always bind confirmation to the immutable manifest, not a prose summary. Show the exact `batch_id`, `batch_hash`, candidate count, platform, selected resume version, and action. The summary may group ordinary items by company/role family, but every row must expand to the exact employer, role, city, pay as listed, URL, fit evidence/gaps, risk/confidence, selected resume project evidence, action, and (for greeting) the exact outgoing text. Expand all `review`, `unknown`, `risk_hold`, duplicate, and pay-exception items by default. User confirmation must explicitly identify this batch/hash and actions; no confirmation carries forward to another batch.

Any add/remove/change to a candidate, resume version, greeting, action, or template requires a new `<batch_id>-r<n>` manifest and a fresh hash/confirmation. Never overwrite a previously confirmed manifest. Verify the stored greeting hash before sending.

Suggested concise view:

```text
Batch: <batch_id> | SHA-256: <batch_hash> | candidates: <count>
Platform/action: <BOSS one greeting / 智联 uploaded resume submission>
Resume version: <exact visible version>
Groups: <company × role family, count, cities, listed pay range, exceptions>
Exceptions: <expanded item IDs and reasons, or none>
Each item expands to: employer, role, city, pay, URL, evidence/gaps, risk, project evidence, exact action/text.
Confirm this exact batch by naming its ID/hash and authorizing its stated actions.
```

## Outgoing greeting

Use only supported resume facts and the exact role's requirements. Keep Chinese text brief (normally no more than 60 characters), genuine, role-specific, and without salary/availability promises or invented experience. Natural phrasing is allowed; do not impose banned-phrase lists or claim a template improves reply rate. Save the exact approved text in the prescribed `messages/` path and record `template_id` (`tpl-boss-v1` or the user-approved revision) and `message_hash`. The coordinator must present the exact text before batch confirmation.

## Outcome report

Use the outline in `coordination-and-reporting.md`. Always show denominators and these definitions:

```text
R=<uniquely attributed responding candidates>; S=<confirmed-success candidates>;
U=<unresolved uncertain candidates>; A=<unattributed reply count/list, not included in R>
Lower endpoint = R/(S+U); upper endpoint = R/S; if denominator is zero: not calculable.
These endpoints cover denominator uncertainty only; with A>0 they are not a full bound on the true reply rate.
```

Do not compute or imply a percentage containing `A`. Distinguish confirmed facts, observed signals, inference, and unknown. State observation window and whether each candidate is full-window, partial-window, or estimated from an uncertain attempt. Include `report_generated_at` and the data/retention footer.
