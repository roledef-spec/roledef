# Contributing to openjd

openjd is an open standard plus a community-curated reference library of AI Job Descriptions. This document describes how to contribute — both new JDs to the library and proposed changes to the spec.

## Governance

openjd is stewarded independently of any single AI vendor or implementation. The standard exists to enable interoperability between AI runtimes — Claude, Grok, GPT, Gemini, and any future runtime — not to codify any one vendor's prompt format.

**Vendors do not own the spec.** Anthropic, xAI, OpenAI, Google, Meta, and any future AI provider are equal citizens from openjd's perspective. Each is a runtime that can host JD-instantiated roles. Each can propose changes. No one gets unilateral authorship.

**The library is curated; the spec is open.** The canonical library at `github.com/openjd-spec/openjd-spec` (and its mirror at `openjd.dev`) is curated for quality, fit, and breadth. The openjd schema itself is content-neutral and freely usable — anyone can publish their own JD library at any URL using the same format. Forks are welcome.

### AI-assisted maintenance

openjd is maintained with AI-assisted review, following the pattern established by [catdef](https://github.com/catdef/catdef-spec). Routine work — drafting validation reports, reviewing JD submissions, applying patch-level fixes — is drafted by Claude sessions operating under roles defined in [CLAUDE.md](CLAUDE.md). These sessions operate under the dedicated identity `openjd-maintainer <openjd-maintainer@openjd.dev>`. Merges to `main`, schema version bumps, and governance decisions are made only by human maintainers through the normal pull-request review process; the AI-maintainer identity does not have merge authority on protected branches.

## Two kinds of contribution

Contributions to openjd fall into two categories with different processes:

1. **JD submissions** — adding a new Job Description to the library
2. **Spec proposals** — proposing changes to the openjd schema, the contribution process, or governance rules

## 1. JD submissions

### The two-stage workflow

JDs follow a two-stage process via the `proposed-jds/` and `jds/` directories:

- `proposed-jds/` — submissions in flight; publicly visible but not yet promoted to canonical
- `jds/` — the canonical library; all entries here have passed validation and review

**Submission steps:**

1. **Fork** the repo
2. **Author your JD** following the procedure in [README.md](README.md#authoring-a-jd) and the schema in [SCHEMA.md](SCHEMA.md)
3. **Add the file** to `proposed-jds/<your-jd-id>.openthing` in your fork
4. **Open a PR** against `main`
5. **Validation runs** (CI or manual openjd-validator) — produces a validation report posted on the PR
6. **Maintainer review** — checks scope, quality, library fit, attribution
7. **Strategist sign-off** — required for borderline cases (scope-edge, library-fit calls); routine clean validations are approved on maintainer alone
8. **Promotion at merge** — the maintainer updates the PR to atomically:
   - Move the file from `proposed-jds/` to `jds/`
   - Add the entry to `catalog.opencatalog`
   - Add a decision artifact to `decisions/jd-<id>.md` with the rationale
9. **Merge**

Once merged, the JD is part of the canonical library and is available via `github.com/openjd-spec/openjd-spec/blob/main/jds/<id>.openthing` and (eventually) `openjd.dev/jds/<id>`.

### Validation expectations

A submission MUST pass:

- **Schema validation** — all MUST fields present per [SCHEMA.md](SCHEMA.md), correct catdef wrapping, valid versioning, no reserved namespace misuse
- **Turing test on at least one runtime** — a fresh AI runtime loaded with the JD produces behavior equivalent to the original-incumbent's documented behavior

A submission SHOULD pass:

- **Turing test on multiple runtimes** (cross-runtime portability)
- **Self-description for all extensions** (per the SHOULD-rule in SCHEMA.md)
- **Rich `reaction_style.examples`** for downstream Turing-test validation by future implementers

### Quality bar

The library is curated. Even valid JDs may be rejected for:

- **Redundancy** — the library already has good coverage of this role; the maintainer will discuss whether the new submission improves on existing entries or whether you should fork the existing JD instead
- **Scope** — the role is too narrow (only useful for one specific company), too broad ("a generic helpful AI"), or outside openjd's intended scope
- **Quality** — the JD authoritatively describes the role but the role itself isn't well-designed (vague guardrails, contradictory rules, no clear output_contract)

If your JD is rejected for the canonical library, you can still publish it at any URL using the same format. Your JD is just as portable from your own URL as from openjd's. Curation is not censorship.

### What goes in `proposed-jds/`

- Files MUST be valid `.openthing` files conforming to [SCHEMA.md](SCHEMA.md)
- Filenames MUST match the JD's `id` field exactly: `<id>.openthing`
- IDs MUST be unique across `proposed-jds/` AND `jds/` (no shadowing)
- Each submission SHOULD include (in the PR description):
  - The role's source — extracted from a product? Self-authored from session experience? Designed from scratch?
  - Reference behavior — link to transcripts, prompts, or examples that establish the original behavior the JD is trying to capture
  - Turing test plan — which runtime(s) to test on, what scenarios

## 2. Spec proposals

Changes to the openjd schema, the contribution process, or governance rules use a separate workflow via the `proposals/` directory.

### When to propose

- The schema is missing a field your domain genuinely needs (and it doesn't fit as an `x.<domain>.<identifier>` extension)
- An existing rule has unintended consequences you've experienced empirically
- The contribution workflow has friction that's preventing legitimate contributions
- A new conformance test would catch a class of validation failures the current tests miss

### When NOT to propose

- **Domain-specific features.** If your use case is expressible as `x.<domain>.<identifier>`, use that. The schema stays minimal on purpose.
- **Wishlist items without a concrete use case.** "Wouldn't it be cool if..." without empirical motivation gets deferred.
- **Vendor-specific accommodations.** openjd is runtime-agnostic. If a feature only makes sense for one vendor's runtime, it belongs in their own tooling, not the spec.

### Proposal artifact format

Use the format described in [CLAUDE.md](CLAUDE.md#proposal-artifact-format). Each proposal is a markdown file in `proposals/`, named `<short-name>.md`. The format includes Summary, Motivation, Proposed Change, Backward Compatibility, Conformance Tests, Alternatives Considered, and Open Questions.

### Proposal workflow

1. **Open** — file the proposal as a PR adding `proposals/<name>.md`
2. **Discuss** — open period for community/maintainer/strategist input
3. **Iterate** — the proposal is refined based on feedback
4. **Decide** — the openjd-strategist files a decision in `decisions/proposal-<name>.md`: Accept | Accept with Modifications | Reject | Defer
5. **Implement** (if accepted) — the openjd-maintainer drafts the actual schema change as a follow-on PR
6. **Version bump** — accepted spec changes bump the openjd schema version per the rules in [SCHEMA.md](SCHEMA.md#versioning)

## Test suite (conformance)

The conformance suite in [`conformance/`](conformance/) is the practical expression of the standard. A change to the spec without test coverage is incomplete. Contributions to the test suite are as valuable as contributions to the spec text.

The conformance suite contains:

- **`valid_jds/`** — JDs that should pass validation; used as positive controls
- **`invalid_jds/`** — JDs with specific known violations; used to validate the validator
- **`tests/`** — the validation test code

A submitted JD's PR MAY include additions to `conformance/` if the submission exercises new schema territory.

**The test suite is the standard.** A JD that passes the suite is openjd-conformant. A JD that doesn't, isn't.

## License

openjd is MIT licensed. The schema is MIT. The reference library JDs are MIT (each carries its own `metadata.license` field; deviations from MIT must be flagged at submission time and may affect library inclusion).

If you fork openjd or publish your own JD library, you choose your own license.

## Bot identity (for AI sessions)

When AI sessions commit to openjd as part of the AI-assisted maintenance workflow, they use these identities:

- **`openjd-maintainer <openjd-maintainer@openjd.dev>`** — for maintainer drafts (proposals, validation reports, schema PRs)
- **`openjd-strategist <openjd-strategist@openjd.dev>`** — for strategist decision artifacts (provisional, pending governance ratification per the [Strategist Bot Identity](CLAUDE.md#known-work-items) Known Work Item inherited from catdef)
- **`openjd-validator <openjd-validator@openjd.dev>`** — for automated validation reports (when CI-driven)

The committer is always the human steward. AI sessions never have merge authority.

## Questions and discussion

- **Submission questions** — open a Draft PR, ask in the description
- **Spec questions** — open an issue tagged `spec-question`
- **Governance questions** — open an issue tagged `governance` or contact the maintainers directly
- **Adjacent project coordination** (catdef, openmemo, openorg, aigp.online) — issues tagged `cross-spec`

## Contact

File issues and PRs on this repository. For broader AIGP-pattern discussion, see [aigp.online](https://aigp.online).

---

*openjd v0.1.0. April 2026. An open standard for AI Job Descriptions. Licensed under MIT.*
