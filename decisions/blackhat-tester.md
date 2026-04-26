# roledef Inclusion Decision: blackhat-tester

**Disposition:** Accepted into canonical library
**Origin:** PR #16
**Decided:** 2026-04-26 by roledef-strategist
**Validator:** roledef-strategist self-validation (PASS WITH NOTES; see PR #16 description)
**Source-project peer review:** N/A (self-extracted; the extractor IS the source per the dual-review policy in `senior-jaded-vc-associate.md`)

## Disposition

`blackhat-tester.openthing` v1.0.0 is accepted into the canonical roledef library at `roledefs/blackhat-tester.openthing` (with `.json` mirror at `roledefs/blackhat-tester.json`). Catalog entry added (`items_count` 4 → 5); this decision artifact filed.

This is the **third non-meta roledef** in canonical and the **first roledef explicitly authored as a derivation parent** — it is the abstract methodology side of the abstract→derived showcase pair that `CONTRIBUTING.md §"Derived roledefs"` names as the canonical first example.

## Rationale

This roledef is a self-extraction by the role's incumbent — an AI session running the documented blackhat-tester role across multiple completed engagements (workspace at `s:/blackhat/`, methodology at `s:/blackhat/methodology.md`). Self-extraction is the strongest possible source-material per CONTRIBUTING.md and is the same provenance as `catdef-strategist` (PR #3).

The roledef establishes the abstract software-agnostic methodology — no specific stacks, deployment platforms, products, workspace conventions, or runtime tool names baked in. It is **designed to be derived from**: a target-specialized derivation (e.g., `sncro-blackhat-tester`) declares `metadata.derived_from` pointing at this roledef and adds domain-specific specializations as additive material per the conventions in `CONTRIBUTING.md §"Conventions for safe derivation"`.

## Notable design choices accepted

1. **First `workflow.type: "engagement_loop"`** — prior submissions used `workflow.type: "sequence"`. The `engagement_loop` shape captures the multi-phase iterative cadence of security engagements (setup → static → fix-verification iterations → dynamic → close) more faithfully than a linear sequence would. Sets a precedent for other iterative-workflow roles.

2. **First `x.security.*` extensions in canonical** — `x.security.severity_thresholds` and `x.security.escalation_path`, both following the SCHEMA.md SHOULD-rule for self-description (each carries a `description` field aimed at naive agents). Establishes the namespace pattern for security-domain roles.

3. **Highest-count `x.roledef.turing_test_reference.scenarios`** — 7 scenarios versus prior max of 5 (senior-jaded-vc-associate). Each scenario maps to a specific behavioral cluster (authorization gating, severity-by-rubric, secrets refusal, fix-verification regression detection, phase-2 gating, sub-agent re-verification, third-party scope confirmation) and lists explicit `expected_behaviors`.

4. **Field-by-field provenance audit in `metadata.extracted_from`** — follows the EXTRACTED / FRAMING / FORMALIZATION / OUT-OF-SCOPE pattern flagged in `senior-jaded-vc-associate.md` Note #3 as a candidate SHOULD-pattern. Second submission to use this format; the pattern is now de-facto standard.

5. **Explicit derivation-parent positioning in `metadata.history[0].change`** — the history note declares the abstract / derivation-parent role explicitly: *"Authored as the parent of an anticipated derivation pattern (target-specialized derivations declare `metadata.derived_from` pointing at this roledef and add domain-specific identity / examples / guardrails as additive specializations per CONTRIBUTING.md 'Conventions for safe derivation')."* Sets a precedent for abstract roles signposting derivation intent in their own history.

6. **`output_contract.delegation_prompt`** — fifth output entry captures the role's pattern of delegating large-file review to sub-agents while preserving main-thread verification discipline. Notable for being a *contract on prompts the role issues to other agents*, not just on outputs the role produces for the user. First roledef to formalize cross-agent prompt contracts; future roles that depend on sub-agent delegation can follow this pattern.

## Items adjusted at promotion

- `metadata.created` was authored as `2026-04-17` (incumbent's session lacked today's-date context); corrected to `2026-04-26` (the roledef-artifact creation date, per the convention used by all 4 prior roledefs).
- `metadata.history[0].date` correspondingly corrected to `2026-04-26`.

No content changes; date metadata only.

## Workflow validation (fifth pass)

This was the **fifth clean run** of the `proposed-roledefs/` → `roledefs/` two-stage workflow:

1. ✓ Branch created (`submit-blackhat-tester`)
2. ✓ Roledef authored by incumbent; handed to strategist for submission per the role-separation convention
3. ✓ Strategist self-validation (acting in `roledef-validator` capacity pending validator-as-CI-step) — PASS WITH NOTES (see PR #16 description)
4. ✓ Source-project peer review skipped (self-extracted; not applicable per the dual-review policy)
5. ✓ Pre-promotion staging in same PR (file added to both `proposed-roledefs/` and `roledefs/`, `.json` mirror added, catalog updated, decision filed)
6. ✓ Strategist sign-off (this decision)
7. ✓ Single-merge promotion (no separate post-merge promotion commit needed)

Step 5 — pre-staging promotion in the same PR rather than as a follow-on commit — matches the PR #4 pattern and is now de-facto standard for clean self-validations.

## Forward-reference resolution

`blackhat-tester.metadata.related: ["roledef-contributor", "roledef-validator", "catdef-strategist"]`:

- ✓ `roledef-contributor` resolves in canonical
- ✓ `roledef-validator` resolves in canonical
- ✓ `catdef-strategist` resolves in canonical

Second fully-resolved-related submission.

## Runtime test

Runtime Turing test deferred per established v0.1 pattern. The 7 documented `x.roledef.turing_test_reference.scenarios` are designed for runtime instantiation comparison; recommended target: within 30 days, on Claude AND Grok Expert (both fetcher-amenable per `RUNTIME_AMENABILITY.md`), with results documented in `conformance/`.

The role's incumbent is itself a runtime instantiation — meaning the cross-runtime test gains a third oracle once a fresh Claude session is loaded with this roledef and behavior is compared to the incumbent's documented practice across `s:/blackhat/`.

## Notable downstream item: derivation pair

This roledef is the *abstract* half of an anticipated abstract → derived pair. The derived half — `sncro-blackhat-tester` — is the next planned submission per the `CONTRIBUTING.md §"Derived roledefs"` example call-out. Authoring will follow the standard derived-roledef procedure (copy parent file, change `id`, add domain specialization, declare `metadata.derived_from`, submit normally). The pair will provide the canonical worked example of the lightweight git-fork-plus-`derived_from` derivation pattern.

## Notes

- **Strategist bot identity:** this decision is provisionally attributed to `roledef-strategist <roledef-strategist@roledef.org>` pending governance ratification (Known Work Item, inherited).

- **Library state after this PR:** 5 roledefs in canonical (2 meta-roledefs + 3 non-meta). The third non-meta roledef establishes that the schema scales across distinct domains (product evaluation, security testing, role-authoring infrastructure).

- **Incumbent-as-extractor pattern reinforced:** this is the second roledef extracted by the role's own incumbent (after `catdef-strategist`). Self-extraction continues to produce the highest-fidelity outputs — the field-by-field provenance audit confirms which fields are EXTRACTED (verbatim from source material), FRAMING (incumbent-articulated sharpenings consistent with practice), and FORMALIZATION (roledef-level structuring of procedural behavior). The pattern is mature enough to formalize as guidance in `CLAUDE.md` if not already.

## Cross-references

- PR: https://github.com/roledef-spec/roledef/pull/16
- Roledef file: [`../roledefs/blackhat-tester.openthing`](../roledefs/blackhat-tester.openthing) (mirror: [`../roledefs/blackhat-tester.json`](../roledefs/blackhat-tester.json))
- Catalog entry: [`../catalog.opencatalog`](../catalog.opencatalog)
- Sister roledefs: [`../roledefs/catdef-strategist.openthing`](../roledefs/catdef-strategist.openthing) (also self-extracted), [`../roledefs/senior-jaded-vc-associate.openthing`](../roledefs/senior-jaded-vc-associate.openthing) (production-extracted), meta-roledefs at [`../roledefs/roledef-contributor.openthing`](../roledefs/roledef-contributor.openthing) and [`../roledefs/roledef-validator.openthing`](../roledefs/roledef-validator.openthing)
- Sister decisions: [`./catdef-strategist.md`](./catdef-strategist.md), [`./senior-jaded-vc-associate.md`](./senior-jaded-vc-associate.md)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Derivation guidance: [`../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles`](../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles)
- Source material: incumbent's documented playbook at `s:/blackhat/methodology.md` and accumulated practice across `s:/blackhat/N/` engagement workspaces (workspaces are not in git per the methodology's own findings-out-of-version-control rule)
