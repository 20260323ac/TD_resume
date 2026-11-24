# Platform Workflow

## Human login and entry points

The user manually signs into the job-seeker account and handles CAPTCHA, account recovery, and verification prompts. Do not use stored credentials or automate login.

Use official entry points and select the confirmed city in the platform UI. Avoid hard-coded city paths (for example, a Shenzhen-specific BOSS URL) when the user's confirmed target city differs. A registration/advertising landing page is not assumed to be a job-seeker page; verify the visible site and account role before proceeding. Do not create accounts, change account identity, or upload a resume unless explicitly authorized.

Confirm that the resume currently selected on 智联 is the version the user approved. If the resume is unavailable or ambiguous, pause for the user.

## Platform actions

### BOSS直聘

For each approved role:

1. Confirm the visible employer, job title, and city match the ledger entry.
2. Confirm the control performs one initial communication action.
3. Send one short, role-specific greeting using only supported resume facts.
4. Do not attach/submit a resume through the greeting, send a second message, or continue the conversation automatically.
5. Verify the platform visibly records the greeting as sent; otherwise mark uncertain and inspect before retrying.

### 智联招聘

For each approved role:

1. Confirm the visible employer, job title, city, and application form match the ledger entry.
2. Confirm the selected, already-uploaded resume is the version shown in the approved batch.
3. Submit the application with that resume. Do not add unapproved attachments, answers, salary claims, or personal details.
4. Verify the platform visibly records successful submission; otherwise mark uncertain and inspect before retrying.

## Detailed end-to-end diagram

```mermaid
flowchart TD
    A[Resume or explicit job-application request arrives] --> B{Is this a job-search/application request?}
    B -- No --> X[Answer the request without starting applications]
    B -- Yes --> C[Extract resume facts, user preferences, inferences, and unknowns]
    C --> D[Propose role families, cities, filters, and project evidence]
    D --> E{User confirms or edits profile}
    E -- Edit --> D
    E -- Confirm --> F[User manually signs in to both job-seeker accounts]
    F --> G{Correct account, city controls, and resume available?}
    G -- No --> H[Pause and hand control to user]
    H --> G
    G -- Yes --> I[Coordinator dispatches discovery tasks]
    I --> J1[BOSS discovery worker]
    I --> J2[智联 discovery worker]
    J1 --> K1[Read full JD and collect evidence]
    J2 --> K2[Read full JD and collect evidence]
    K1 --> L[Coordinator checks employer, opening activity, fit, risk, and duplicates]
    K2 --> L
    L --> M{Pass hard gates and no high-risk signal?}
    M -- No --> N[Skip or hold with reason]
    M -- Yes --> O[Select supported project evidence and prepare platform action]
    O --> P[Prepare up to 150 BOSS and up to 150 智联 candidates]
    P --> Q[Present exact roles, resume version, actions, and messages]
    Q --> R{User approves this exact batch?}
    R -- No --> S[Edit, defer, or cancel batch]
    R -- Yes --> T[Run one serial queue per account; separate accounts may proceed independently]
    T --> U1[BOSS: send one greeting only]
    T --> U2[智联: submit approved uploaded resume]
    U1 --> V[Verify visible outcome and update ledger]
    U2 --> V
    V --> W{Result ambiguous, page mismatch, CAPTCHA, or rate limit?}
    W -- Yes --> Y[Stop that queue and request user action if needed]
    W -- No --> Z[Continue with next approved record]
    Z --> AA[Response analyst observes only within authorized active scope]
    AA --> AB[Classify replies; do not answer for the user]
    AB --> AC[Report separate platform metrics, evidence, caveats, and next-batch proposals]
```

## User communication

Keep messages concise. Normally ask for only two confirmations: (1) the search profile, and (2) the exact batch and platform actions. Send brief progress only when a user decision or intervention is needed, then provide one final report. Do not hide a material issue in the name of minimizing chatter.
