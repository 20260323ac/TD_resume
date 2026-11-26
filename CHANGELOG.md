# Changelog

## 2.12 - 2026-11-25

- Mapped deduplication outcomes only to declared candidate states: confirmed duplicate → `duplicate_skipped`; uncertain contact and conflict → `risk_hold` with a specific `reason_code`; cross-platform duplicate → `risk_hold` with `cross_platform_duplicate`.
- Added the separate `reason_code` field and dictionary; kept it distinct from attempt/queue `stop_reason_code` and `note_code`.

## 2.11 - 2026-11-25

Implemented the reviewed v2.11 modification plan through stages 1–4. The runtime and records remain external to this skill package.

- Added durable record schema, write-ahead attempt events, two-way recovery, single-writer locking, retention/cleanup, batch manifests, outgoing-message integrity, and candidate-level deduplication rules.
- Added intake and greeting/report templates; clarified evidence-based job screening, salary/education handling, and separate application/reply states.
- Reorganized the entry skill and install guide; retained manual login, per-batch approval, 150-per-platform ceiling, BOSS one-greeting and 智联 uploaded-resume actions.
- Real external applications remain separately authorized and were not part of this implementation.
