# roledef Inclusion Decision: sncro-blackhat-tester

**Disposition:** Accepted into canonical library
**Origin:** PR #17
**Decided:** 2026-04-26 by roledef-strategist
**Validator:** roledef-strategist self-validation (PASS, no notes; see PR #17 description)
**Source-project peer review:** N/A (incumbent-extracted from incumbent's own engagement workspace; the extractor IS the source)
**Parent:** `blackhat-tester` v1.0.0 (PR #16, decided 2026-04-26)

## Disposition

`sncro-blackhat-tester.openthing` v1.0.0 is accepted into the canonical roledef library at `jds/sncro-blackhat-tester.openthing` (with `.json` mirror at `jds/sncro-blackhat-tester.json`). Catalog entry added (`items_count` 5 → 6); this decision artifact filed.

This is the **first derived roledef in canonical** and the **canonical worked example of the lightweight git-fork-plus-`derived_from` derivation pattern** documented in `CONTRIBUTING.md §"Derived roledefs"`. Together with its parent (PR #16), it establishes the abstract→derived showcase pair that the `CONTRIBUTING.md` example call-out promised.

## Rationale

This roledef is a **derivation** by the role's incumbent — the same incumbent who self-extracted the abstract `blackhat-tester` parent in PR #16, now writing the SNCRO-specialized derivation from the actual engagement workspace at `s:/blackhat/2/`. The lineage is declarative-only via `metadata.derived_from`; the derived file is a complete, self-contained artifact that runtimes treat as a standalone roledef. No parent fetch happens at instantiation.

The submission lands with **PASS, no notes** on schema validation. Every convention from `CONTRIBUTING.md §"Conventions for safe derivation"` was followed exactly:

- **Guardrails: 100% additive.** All 10 parent guardrails preserved verbatim; 2 SNCRO-specific guardrails added (session_secret/browser_secret value-read refusal; project_key-as-server-credential = always a finding).
- **Identity and voice: refined, not contradicted.** Parent identity opens with the parent's first 200 chars verbatim, then SNCRO surface knowledge appended (three-credential model, six documented invariants, allow_guests=True default amplifier, relay two-sided auth pattern).
- **Output contract: additive + tightenable.** All 5 parent entries preserved; 1 new entry (`invariant_check_table` — captures SNCRO's six formally documented invariants from `CLAUDE.md`/`INSTALL.md` as a structured table that drives the threat model).
- **Workflow: parent shape preserved.** `engagement_loop` type retained; all parent steps preserved across all 5 phases; SNCRO-specific operational notes appended inline to `phase_1_static_review`, `phase_nb_fix_verification`, `phase_2_dynamic_testing` with `SNCRO ADDITIVE —` prefix for derivation traceability.
- **Reaction style examples: additive.** All 5 parent examples preserved verbatim; 5 SNCRO-specific examples added — 4 grounded in real engagement findings (C-3 relay endpoint two-sided audit, NEW-5 cross-tenant XSS via guest-session git_user, H-3 project_key-as-credential with the published-in-CLAUDE.md irony, NEW-4 XFF leftmost-entry regression after rate-limits added) and 1 honestly flagged as FORMALIZATION (session_secret read refusal — generalized from the documented invariant + the role's secrets-boundary principle, not from a real engagement moment).
- **Severity thresholds: parent rubric verbatim + sncro_specific_guidance sibling.** Parent's critical/high/medium/low rubric preserved unchanged; a `sncro_specific_guidance` object added at the extension's top level (alongside `description` and `value`) with concrete SNCRO-domain examples for each severity tier. Properly self-described per SCHEMA.md SHOULD-rule.
- **Turing test scenarios: additive.** All 7 parent scenarios preserved verbatim; 4 SNCRO-specific scenarios added (relay endpoint two-sided audit; project_key-as-credential finding; session_secret read refusal; cross-tenant amplifier recognition).

## Notable design choices accepted

1. **`SNCRO ADDITIVE —` prefix convention on new behavioral items.** The new guardrails, conversation_rules, design_constraints, and workflow inline notes are explicitly prefixed to mark them as derivation specializations. Future readers can identify at the call site which items came from the parent versus from the derivation. Sets a useful precedent for derived roledefs: derivation-source visibility at the item level, complementary to the metadata-level `derived_from` declaration. **Recommend strategist consider promoting this prefix convention as a SHOULD-pattern for derived roledefs in `CONTRIBUTING.md §"Conventions for safe derivation"`.**

2. **`invariant_check_table` as a sixth output_contract entry.** Captures SNCRO's six formally documented invariants from `CLAUDE.md`/`INSTALL.md` as a structured artifact that drives the threat model. Placed before findings in the engagement workflow because invariants drive the threat model, not the other way around. Establishes a useful pattern for any target with formally enumerated invariants — the table makes invariant-vs-implementation gaps trivially auditable.

3. **`x.security.severity_thresholds.sncro_specific_guidance` sibling structure.** Rather than nest sncro examples inside `value`, brother added a sibling key at the extension's top level. The pattern preserves the parent rubric as a verbatim reference value while making domain-specific guidance discoverable as a co-equal first-class field. Self-described with its own `description` field per SCHEMA.md SHOULD-rule. **Recommend strategist consider this sibling-pattern as the standard shape for adding domain-specific guidance to a SHOULD-preserved parent extension value.**

4. **`metadata.related: ["blackhat-tester", "roledef-contributor", "roledef-validator", "catdef-strategist"]`.** First field-resolved-derivation submission — the parent reference resolves in canonical, not just to a sibling. Sets precedent for derived roledefs always listing the parent as the first `related` entry.

## Borderline-call decisions documented

The incumbent's self-rundown documents three borderline calls that were considered and rejected, each with sound reasoning:

1. **No SNCRO-specific threat tags (T-S1 etc.).** The actual engagement used a subset of the parent's T1–T10 set; adding new tags without engagement evidence would be formalization without grounding. Instead, the load-bearing subset (T1, T2, T3, T5, T7, T8, T10) is noted inside the `findings_report` schema text. Right call.

2. **No `x.sncro.invariants` extension with machine-readable invariant values.** Invariant values belong in the target's own docs, not in the roledef. The roledef can describe how to USE the invariants (which it does in conversation_rules and the new `invariant_check_table` output entry), but baking specific values would create a coupling that rots when SNCRO updates them. The roledef points at the source documents instead. Right call — preserves the "runtime fetches, model never touches the network" design principle by extension: roledefs don't bake target-specific invariant values that could drift from source.

3. **Live-MCP testing as new escalation_path trigger, not new guardrail.** The principle "Never run exploits against production systems" is already a parent guardrail. The SNCRO twist (allow_guests=True default makes "running exploits" feel like just using the product) is captured as a new escalation_path trigger so the operational guidance is at the right level. Right call — derived roledefs should resist guardrail-duplication when the parent guardrail covers the case.

## Items adjusted at promotion

None. The submission was clean as-authored; no date corrections, no content changes.

## Workflow validation (sixth pass; first derivation)

This was the **sixth clean run** of the `proposed-jds/` → `jds/` two-stage workflow and the **first derivation submission**:

1. ✓ Branch created (`submit-sncro-blackhat-tester`)
2. ✓ Parent file fetched from canonical (`s:/projects/roledef-spec/roledef/jds/blackhat-tester.openthing`)
3. ✓ Roledef authored by incumbent following the derivation procedure in CONTRIBUTING.md §"Derived roledefs"
4. ✓ Strategist self-validation — PASS, no notes
5. ✓ Pre-promotion staging in same PR (file added to both `proposed-jds/` and `jds/`, `.json` mirror added, catalog updated, decision filed)
6. ✓ Strategist sign-off (this decision)
7. ✓ Single-merge promotion

The derivation workflow is otherwise identical to the standard submission workflow — no special-casing needed. This validates the design choice in `CONTRIBUTING.md` to handle derivations with the same submission machinery as standalone roledefs (the `derived_from` declaration is the only structural difference).

## Catalog enhancement: `derived_from` field on entries

The catalog entry for this submission carries a new `derived_from: "blackhat-tester"` field — a catalog-level shortcut to the derivation lineage that complements the file-level `metadata.derived_from`. This makes derivation lineage discoverable from catalog browsing without requiring readers to fetch each file. Sets a useful precedent for future catalog entries: when a roledef has `metadata.derived_from`, the catalog entry SHOULD carry a corresponding `derived_from: "<parent-id>"` field. **Recommend strategist promote this convention to `CLAUDE.md` or `SCHEMA.md` catalog guidance.**

## Forward-reference resolution

`sncro-blackhat-tester.metadata.related: ["blackhat-tester", "roledef-contributor", "roledef-validator", "catdef-strategist"]`:

- ✓ `blackhat-tester` resolves in canonical (the parent)
- ✓ `roledef-contributor` resolves in canonical
- ✓ `roledef-validator` resolves in canonical
- ✓ `catdef-strategist` resolves in canonical

Third fully-resolved-related submission (after PR #4 and PR #16). The library is mature enough that forward-references to siblings are increasingly rare.

## Runtime test

Runtime Turing test deferred per established v0.1 pattern. The 11 documented `x.roledef.turing_test_reference.scenarios` (7 inherited + 4 SNCRO-specific) are designed for runtime instantiation comparison; recommended target within 30 days on Claude AND Grok Expert.

The role's **live runtime oracle** is the incumbent's own session running the SNCRO engagement at `s:/blackhat/2/` — a third oracle once a fresh runtime is loaded with this roledef and behavior is compared to the incumbent's documented engagement practice. SNCRO itself (the target) has live deployment at `https://relay.sncro.net` + `https://sncro.net`, providing the dynamic-testing target for future Phase 2 work.

## Strategist memory work items (logged separately to memory file)

The following pattern observations from this PR will be considered for promotion to roledef CLAUDE.md / SCHEMA.md / CONTRIBUTING.md guidance:

1. **`SNCRO ADDITIVE —` prefix convention** for derivation-source visibility at the item level (complementary to metadata-level `derived_from`)
2. **`<parent-name>_specific_guidance` sibling pattern** for adding domain-specific guidance to a SHOULD-preserved parent extension value while keeping the parent value verbatim
3. **Catalog `derived_from` field** as a catalog-level shortcut for derivation-lineage browsing
4. **First derivation establishes the abstract→derived showcase pair** referenced explicitly in CONTRIBUTING.md — the example call-out is now backed by published artifacts

## Notes

- **Strategist bot identity:** this decision is provisionally attributed to `roledef-strategist <roledef-strategist@roledef.org>` pending governance ratification (Known Work Item, inherited).

- **Library state after this PR:** 6 roledefs in canonical (2 meta-JDs + 4 non-meta, including 1 derivation). Library is mature enough to support derivation pairs; the canonical first example is now published.

- **Incumbent self-extraction continues to produce highest-fidelity outputs.** The provenance audit explicitly partitions each field into EXTRACTED / FRAMING / FORMALIZATION / OUT-OF-SCOPE; brother honestly flagged one new example (session_secret refusal) as FORMALIZATION rather than EXTRACTED, demonstrating the audit's discriminating power. Self-extraction with field-by-field provenance audit is now firmly established as the standard pattern for incumbent-authored submissions.

- **Source-project peer review unnecessary even for derivations.** The dual-review policy in `jd-senior-jaded-vc-associate.md` says self-extracted roledefs don't need source-project peer review because the extractor IS the source. This applies equally to derivations when the incumbent is also the source-project practitioner — as is the case here. Production-extracted derivations (different person extracting from a live target) would still benefit from the dual-review pattern.

## Cross-references

- PR: https://github.com/roledef-spec/roledef/pull/17
- Roledef file: [`../jds/sncro-blackhat-tester.openthing`](../jds/sncro-blackhat-tester.openthing) (mirror: [`../jds/sncro-blackhat-tester.json`](../jds/sncro-blackhat-tester.json))
- Catalog entry: [`../catalog.opencatalog`](../catalog.opencatalog) (note new `derived_from` field on entry)
- **Parent roledef:** [`../jds/blackhat-tester.openthing`](../jds/blackhat-tester.openthing)
- **Parent decision:** [`./jd-blackhat-tester.md`](./jd-blackhat-tester.md)
- Sister JDs: [`../jds/catdef-strategist.openthing`](../jds/catdef-strategist.openthing), [`../jds/senior-jaded-vc-associate.openthing`](../jds/senior-jaded-vc-associate.openthing), meta-JDs at [`../jds/roledef-contributor.openthing`](../jds/roledef-contributor.openthing) and [`../jds/roledef-validator.openthing`](../jds/roledef-validator.openthing)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Derivation guidance: [`../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles`](../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles)
- Source material: incumbent's engagement workspace at `s:/blackhat/2/` (findings, methodology notes, real owner-interaction transcripts) plus the SNCRO project's own claimed invariants at `s:/projects/sncro/CLAUDE.md` and `INSTALL.md` (six documented invariants are the first test oracles)
- Live target oracles: `https://relay.sncro.net`, `https://sncro.net` (out-of-scope for v0.1 turing-test runs but available for future Phase 2 dynamic-test reference)
