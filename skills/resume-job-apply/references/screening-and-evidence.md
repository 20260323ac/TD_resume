# Screening and Evidence Rules

Evaluate three separate questions. Do not collapse them into a single unsupported “real/fake” judgment.

## 1. Employer and recruiter identity

Record evidence such as:

- Employer's displayed full legal/business name and platform profile.
- Whether the employer's stated business reasonably relates to the product and role.
- Whether recruiter identity/affiliation and work location are consistent with the employer's profile and job description.
- Whether independent public company information or an official careers page corroborates the identity, when available.

Use `consistent`, `partially corroborated`, `inconsistent`, or `unknown`. Platform verification is supporting evidence, not proof that the specific opening is active.

## 2. Specific opening activity and quality

Record:

- Job URL, observed date/time, posted/updated date if shown, and whether the listing remains open.
- Concrete duties, product context, platform/chip/toolchain, required experience/education, compensation structure, location, and employment type.
- Whether the recruiter can provide role-specific information, interview steps, or a realistic hiring timeline.
- Signs of stale/reposted listings, vague descriptions, bait-and-switch duties, or inconsistent salary/details.

Classify:

- `strong current signal`: recent/active listing plus concrete role details or role-specific recruiter activity.
- `plausible, activity unconfirmed`: coherent employer and concrete listing, but limited evidence of current hiring.
- `uncertain/stale`: old, incomplete, contradictory, or repeatedly reposted information.
- `high risk`: material identity mismatch, pay-to-work/training-loan demand, or other clear warning sign.

Do not label a role definitively fake solely because there is no reply. Do not label it genuinely hiring solely because a recruiter replied.

## 3. Candidate fit

Map each hard requirement to explicit resume evidence. Mark each as `meets`, `partial/transferable`, `not evidenced`, or `fails`. A missing item is not evidence that the user has it. Identify material gaps (e.g., required Linux kernel, AUTOSAR, safety certification, or a specific MCU absent from the resume) and avoid treating an adjacent keyword as equivalent experience.

Use a fit disposition:

- `apply_candidate`: core duties align, hard requirements are met or only minor gaps remain, and no high-risk signal exists.
- `review`: uncertain requirement, unclear role, or user preference needed.
- `skip`: hard requirement fails, duties materially mismatch, duplicate, or high-risk condition.

Do not use an opaque numeric score as the sole decision rule. If a score is useful for sorting, keep the evidence and hard gates visible.

## Red flags and response handling

Put a role on `risk_hold` for requests to pay a deposit, buy mandatory training, take a training loan, provide unnecessary financial credentials, or accept a role whose actual duties materially differ from the listing. Explain the specific signal; do not accuse a company of fraud without substantiation. Ask the user before sharing additional sensitive information or continuing a consequential conversation.
