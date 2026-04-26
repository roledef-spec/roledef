# proposed-roledefs/

Submission staging area for new roledefs. roledefs in this directory are **in flight** — submitted but not yet promoted to the canonical library at [`../roledefs/`](../roledefs/).

## Why this directory exists

The two-stage workflow (per [CONTRIBUTING.md](../CONTRIBUTING.md#the-two-stage-workflow)) gives in-flight submissions a publicly visible home. Anyone browsing the repo can see what's pending without needing to query GitHub PRs.

## What's in this directory

- `.openthing` files for roledefs that have been submitted via PR but not yet merged
- Submissions awaiting validation, review, or strategist sign-off

## What's NOT in this directory

- Accepted roledefs (those live in `../roledefs/`)
- Rejected roledefs (those are closed without merge; the rejection rationale is in the closed PR)
- Spec proposals (those live in `../proposals/`)

## Lifecycle

A roledef lands here when a contributor opens a PR. It leaves here in one of two ways:

1. **Approved** — at merge time, the maintainer atomically moves the file from `proposed-roledefs/` to `../roledefs/`, updates `../catalog.opencatalog`, and records a decision in `../decisions/<id>.md`. Single merge action.
2. **Rejected** — the PR is closed without merge. Rationale is in the PR's closing comment.

## Submitting

See [CONTRIBUTING.md §1 roledef Submissions](../CONTRIBUTING.md#1-roledef-submissions) for the full process.

In brief:
1. Fork
2. Add your roledef as `proposed-roledefs/<your-roledef-id>.openthing`
3. Open a PR
4. Validation runs, maintainer reviews, strategist signs off if needed
5. Approved → maintainer promotes; rejected → closed with reason

---

*roledefs in this directory are in active review. They are NOT part of the canonical roledef library until promoted to `../roledefs/`.*
