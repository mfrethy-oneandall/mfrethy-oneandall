# CP&A (Confidentiality Policy & Agreement) Workflow

## What It Is
A Rock RMS automated workflow that manages volunteer onboarding. Every new volunteer must sign a Confidentiality Policy & Agreement before being granted access/visibility to sensitive church data.

## How It Works
1. A staff member (Requester) initiates a Rock Workflow for a person (volunteer)
2. Rock sends Mike an email: "Confidentiality Policy & Agreement: [Name]"
3. Mike provisions IT access → workflow moves to "Waiting for Agreement"
4. Volunteer must sign the agreement → "Waiting for Review"
5. Mike reviews and approves

## Current Queue States
- **Waiting for Review** — IT action needed (Mike's lane)
- **Waiting for Agreement** — Volunteer needs to sign (Audrey C follows up)

## Triage
Snapshot task (86b8r7uxt) closed 2026-03-09 — superseded by Claude (Cowork) daily 10am sync.
Codex `daily-clickup-triage` automation retired same date.

## Key People
- **Audrey C** (Audrey Ceballos) — handles "Waiting for Agreement" follow-ups
- Requesters are typically campus staff (e.g., Joshua Adams, Yesenia Gomez, Daniel Leon)

## Open Rock Workflows (as of 2026-03-09, confirmed from Rock)
| Person | Rock Workflow # | State | Requester | ClickUp Task |
|--------|----------------|-------|-----------|-------------|
| Amy Meza | 2031001 | Waiting for Review | Yesenia Gomez | [86b8rmm89](https://app.clickup.com/t/86b8rmm89) |
| Lisa Ulloa | 2027416 | Waiting for Review | Jolene Pascasio | [86b8r7wpw](https://app.clickup.com/t/86b8r7wpw) |
| Raquel Leon | 2026168 | Waiting for Review | Daniel Leon | [86b8rmm8d](https://app.clickup.com/t/86b8rmm8d) |
| Chaz Kumlander | 2024794 | Waiting for Review | Joshua Adams | [86b8ttvq0](https://app.clickup.com/t/86b8ttvq0) |
| John Davila | 2020876 | Waiting for Agreement | Joshua Adams | (Comms task: 86b8mff1z) |
| Anthony Macchia | 1901135 | Waiting for Agreement | — | ⚠️ 97 days pending |
| Ray Fernandez | 1967972 | Waiting for Agreement | — | 47 days pending |
| Richard Koon | 1916757 | Waiting for Agreement | — | ⚠️ Possible duplicate |
| Richard Koon | 1901133 | Waiting for Agreement | — | ⚠️ Possible duplicate |

## Access Decision Method (Least Privilege)
_Absorbed from Codex cpa-comparator-standard, 2026-03-09_

- Scope all edits/comments to IT Team > IT Service Desk > IT Requests only.
- Do not baseline access from requester permissions.
- Pull comparable **completed** Rock workflows (WorkflowTypeId = 144).
- Rank by: matching UsageType (Rock / Email Account / Rock+Email Account) + role-intent keywords in Reason (kids, outreach, women, baptism, lead/admin).
- Prefer explicit completion outcomes in workflow Results; fallback to IT task closeout comments.
- Propose least-privilege access from overlap across top matches.
- Require requester/owner confirmation before elevated grants.
- Record in ClickUp task: requested usage, comparable workflow IDs used, final granted roles/groups + any exclusions with reason.

## Rock → ClickUp Pipeline Notes
- Individual ClickUp tasks now exist for all Waiting for Review items (4/4 covered).
- Codex pipeline is fully retired; Claude daily sync owns this going forward.
- ZZ-prefixed test tasks in ClickUp (from Feb 12 Codex tests) are safe to close.
