# Screening and Evidence

Assess employer identity, the specific opening, and candidate fit as separate questions. Cite the visible source or resume evidence for each conclusion. A platform badge or recruiter reply alone does not prove an opening is active; no reply alone does not prove it is fake.

## Evidence labels

For each fact, use `resume_fact`, `user_stated`, `inference`, or `unknown`. For each requirement, use `meets`, `partial/transferable`, `not_evidenced`, or `fails`. Missing evidence is not failure. For employer identity use `consistent`, `partially_corroborated`, `inconsistent`, or `unknown`; for opening activity use `strong_current_signal`, `plausible_activity_unconfirmed`, `uncertain_stale`, or `high_risk`.

## Employer and opening

Record the displayed company name, platform profile, employer/location consistency, job URL, observed time, posted/updated time if shown, and concrete duties, stack, pay, employment type, and hiring steps. Use independent public or official careers evidence when available. Keep the original company/role text. Do not merge subsidiaries based on fuzzy similarity; only normalize legal suffixes and use user-confirmed aliases. Similar-but-unverified names go to manual review.

Treat the opening as higher-confidence only when the listing is current and specific or there is role-specific activity. Hold or skip material identity inconsistencies, pay-to-work/training-loan demands, suspicious data requests, stale/reposted listings, bait-and-switch duties, or unexplained compensation contradictions. State the signal; do not make unsupported fraud accusations.

## Fit and transferability

Map each hard requirement to explicit resume evidence. Mark absent resume evidence `not_evidenced`; do not infer unlisted skills, ownership, or outcomes. A `partial/transferable` judgment must state (1) the genuinely shared problem or skill, (2) the concrete gap, and (3) an estimated—not promised—time/cost to close it. Do not claim the adjacent skill as already possessed.

Education:

- Explicit exclusive requirement (e.g. “must”, “only”, “not accepted”): mismatch is `fails`; quote the requirement.
- Preferred/bonus condition: mismatch is `review`, not `fails`.
- Soft, ambiguous, or missing information: `unknown`; ask the user if it materially affects the application.

Disposition: `apply_candidate` when core duties align, hard requirements are met or gaps are minor, and no high-risk signal exists; `review` for uncertain requirements/preferences; `skip` for failed hard requirements, material role mismatch, duplicate, or high-risk condition. Do not use an opaque numeric score as the sole decision rule.

## Salary normalization

Always preserve `salary_as_listed`. Parse, when possible, `salary_min`, `salary_max`, `salary_months`, `pay_type`, and `bonus_note`, plus `salary_normalized_monthly` and `salary_assumption`. A 13-salary annual package may be shown as an estimated monthly equivalent only with the assumption stated. For a daily rate, 21.75 workdays/month is an estimate, not a verified payroll basis. Base+performance+allowance remains structurally uncertain; retain the original wording. “面议”, missing, or unparsable pay is `unknown`; never guess or compare it as a known amount.

## Evidence that a job is genuine

Use confidence labels and observed signals, not a binary “real/fake” claim. Strong identity evidence and a current, concrete opening increase confidence but do not guarantee hiring. Recruiter responses are evidence only when they contain role-specific information; generic acknowledgments are not proof. Record contradictions and uncertainty for the user to decide.
