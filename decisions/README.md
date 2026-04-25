# decisions/

Strategist decision artifacts. Each file records a decision made by roledef-strategist on a non-routine matter.

This directory is the institutional memory of roledef's evolution. Future strategist sessions read this to understand the rationale for past decisions.

## What goes here

Two main file types:

### `jd-<id>.md` — roledef inclusion decisions

Filed at promotion time when a roledef moves from `proposed-jds/` to `jds/`. Records:

- Which roledef was promoted
- Validation summary (schema pass, Turing test results)
- Rationale for inclusion (scope fit, quality, breadth contribution)
- Any modifications made during review
- Cross-references to the originating PR

Created automatically by roledef-strategist (or maintainer with strategist sign-off) at merge time.

### `proposal-<name>.md` — Spec proposal decisions

Filed when a spec proposal is decided. Records:

- The proposal name and link
- Disposition (Accept, Accept with Modifications, Reject, Defer)
- Rationale
- Build directive (for the maintainer, if Accepted)
- Cross-references to the proposal artifact and PR

Mirrors the decision-artifact pattern from catdef-spec.

## Format

Each decision is a markdown file with structured frontmatter:

```markdown
# roledef Inclusion Decision: <jd-id> (or Proposal Decision: <name>)

**Disposition:** Accepted (or other)
**Origin:** PR #N (or proposal-<name>.md)
**Decided:** YYYY-MM-DD by <strategist identity>

## Disposition
[The decision in one line]

## Rationale
[Why this decision was made]

## Notes
[Modifications, conditions, follow-on work, cross-references]
```

## Why this directory matters

Decisions made and lost are decisions remade. Without artifacts, every strategist session re-litigates everything. With artifacts, prior reasoning is accessible to anyone reading the repo.

This is the roledef realization of the AIGP "open governance" principle: every meaningful decision leaves a public, auditable trail.

---

*Strategist decisions are the institutional memory. Read them before re-deciding anything.*
