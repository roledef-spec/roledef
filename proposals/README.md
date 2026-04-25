# proposals/

Spec-level proposals — changes to the roledef schema, the contribution workflow, or governance rules.

This directory is **not** for new roledef submissions. Those go in [`../proposed-jds/`](../proposed-jds/) and follow a different workflow.

## What goes here

- Schema changes (new fields, modifications to MUST/SHOULD/x. taxonomy)
- Workflow proposals (changes to the contribution process, governance, validation requirements)
- Conformance test additions (new test categories, new fixture conventions)
- Anything else that affects the spec or the project structure

## What does NOT go here

- New roledefs for the library (use `../proposed-jds/`)
- Editorial typos and patches (just open a small PR; no proposal artifact needed)
- Implementation-specific feature requests for one runtime (file with that runtime's project)

## Format

Each proposal is a markdown file using the format in [CLAUDE.md §Proposal artifact format](../CLAUDE.md#proposal-artifact-format). Filename is `<short-name>.md`.

## Workflow

1. **Open** — file the proposal as a PR adding `proposals/<name>.md`
2. **Discuss** — open period for community/maintainer/strategist input
3. **Iterate** — proposal is refined based on feedback
4. **Decide** — roledef-strategist files a decision in `../decisions/proposal-<name>.md`: Accept | Accept with Modifications | Reject | Defer
5. **Implement** (if accepted) — roledef-maintainer drafts the actual schema/workflow change as a follow-on PR
6. **Version bump** — accepted spec changes bump the roledef schema version per [SCHEMA.md §Versioning](../SCHEMA.md#versioning)

## Status of proposals

A proposal's status is in its file's header:

- **Draft** — under active iteration
- **Under review** — awaiting strategist decision
- **Accepted** — merged, ready for implementation
- **Accept with Modifications** — accepted with changes specified in the decision
- **Rejected** — closed; rationale in the decision
- **Superseded** — replaced by a newer proposal; cross-reference in the file
- **Deferred** — moved to a future cycle; cross-reference in the decision

---

*Proposals are how roledef evolves. Submissions are how the library grows. Don't confuse them.*
