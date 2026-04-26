# roledef Inclusion Decision: senior-jaded-vc-associate

**Disposition:** Accepted into canonical library
**Origin:** PR #4
**Decided:** 2026-04-25 by roledef-strategist
**Validator:** roledef-validator (PASS on revised state — see PR #4 validation report comment)
**Source-project peer review:** brother-DangerStorm (PASS-WITH-NOTES on initial state; all warranted notes addressed in revision commit `e35680c`)

## Disposition

`senior-jaded-vc-associate.openthing` v1.0.0 is accepted into the canonical roledef library at `roledefs/senior-jaded-vc-associate.openthing`. Catalog entry added (4th entry, `items_count` updated to 4); this decision artifact filed.

This is the **second non-meta roledef** in canonical and the **first roledef extracted from a real-world production AI product** — establishing the **production-extraction pattern** alongside the self-extraction pattern set by PR #3.

## Rationale

This roledef is a behavioral extraction from the DangerStorm.net production system prompt (the role implemented in `s:/projects/dangerstorm/server.py SYSTEM_PROMPT`, currently shipping live at https://dangerstorm.net). The extraction was authorized by the source product's owner under the site's "X-Ray Mode" open-posture publication of the prompt.

The submission landed with PASS on schema validation (post-revision) and PASS-WITH-NOTES on source-project peer review (pre-revision; all warranted notes addressed in revision commit `e35680c`).

## The dual-review pattern (new precedent set by this PR)

This is the first roledef submission to undergo TWO complementary reviews before promotion:

1. **Source-project peer review** by brother-DangerStorm (the resident Claude session in the source repo). Caught three meaningful fidelity drifts that strategist self-validation missed:
   - "Minimum 3 exchanges" guardrail hardened beyond production's descriptive "usually 3-5"
   - Invented "Sharp./Clean./Got it." reaction pattern (voice-consistent but not in source)
   - Identity reframing additions ("slightly jaded", "you're not the partner") not literally extracted

2. **roledef-validator review** confirmed schema conformance and internal consistency on the revised state.

**Strategist call: dual review becomes the standard for production-extracted roledefs.** Self-extracted roledefs (e.g., catdef-strategist) do not need source-project review because the extractor IS the source. Production-extracted roledefs benefit substantially from source-project review because the extractor is one degree removed from the source-of-truth and may not see additive framing as additive.

## Notable design choices accepted

1. **6-entry `output_contract`** — richest in the library so far (was 1-4 in prior submissions). Validates schema scales to bundle-producing roles. No conformance issue; sets useful precedent for future bundle-producing roles.

2. **Live runtime behavioral oracle in `x.roledef.turing_test_reference`** — first roledef to cite an external production deployment as a runtime cross-validation target. Unlocks a stronger turing-test methodology than internal-scenario-comparison alone. Recommend strategist consider promoting this pattern in future schema/CLAUDE.md guidance for production-extracted roledefs.

3. **Field-by-field provenance audit in `metadata.extracted_from`** — first submission to explicitly partition each field into EXTRACTED / FRAMING / FORMALIZATION / OUT-OF-SCOPE categories. Dramatically stronger than prior coarse provenance statements. Recommend strategist consider promoting this audit format as a SHOULD-pattern for future submissions in CLAUDE.md or SCHEMA.md.

4. **First fully-resolved `metadata.related`** — all three referenced roledefs resolve in canonical. No forward references. Possible because the bootstrap is complete; subsequent submissions inherit a fuller library to reference.

5. **Workflow `evaluation` step retained as formalization-with-honesty-annotation** — the production prompt enforces an implicit completeness criterion through the question sequence. The roledef makes it explicit so a fresh runtime can instantiate the role faithfully. Acceptable per the production-extraction pattern when annotated as formalization rather than passed off as extraction.

## Items NOT incorporated into roledef revision (with rationale)

- **6-output normalization** retained — production prompt has internal inconsistency at server.py line 149 ("ALL SIX") vs line 201 ("ALL FIVE"). roledef correctly normalizes to 6 (matches output structure). Source-side fix is brother-DangerStorm's responsibility in the DangerStorm repo, not a roledef revision. **Strategist note:** this is a positive externality of cross-project review — extraction work surfaces source-material bugs.

## Workflow validation (fourth pass)

This was the **fourth clean run** of the `proposed-roledefs/` → `roledefs/` two-stage workflow:

1. ✓ Branch created (`submit-senior-jaded-vc-associate`)
2. ✓ roledef authored and added to `proposed-roledefs/`
3. ✓ PR opened (#4) with self-validation pre-completed
4. ✓ **NEW: source-project peer review** (brother-DangerStorm, PASS-WITH-NOTES)
5. ✓ Revision commit incorporates warranted notes (`e35680c`)
6. ✓ Validation report posted by roledef-validator role (PASS on revised state)
7. ✓ Maintainer review (no requested changes)
8. ✓ Strategist sign-off (this decision)
9. ✓ Promotion commit moves file to `roledefs/`, updates `catalog.opencatalog`, files this decision

Step 4 is a new addition to the workflow for production-extracted roledefs. The workflow is otherwise unchanged.

## Forward-reference resolution

`senior-jaded-vc-associate.metadata.related: ["roledef-contributor", "roledef-validator", "catdef-strategist"]`:

- ✓ `roledef-contributor` resolves in canonical
- ✓ `roledef-validator` resolves in canonical
- ✓ `catdef-strategist` resolves in canonical

First fully-resolved-related submission.

## Runtime test

Runtime behavioral oracle is **live** at https://dangerstorm.net. A fresh-runtime instantiation of this roledef can be cross-validated against the production deployment using the 5 documented `x.roledef.turing_test_reference.scenarios` as fixtures. Recommended target: within 30 days, on Claude AND Grok, with results documented in `conformance/`. Cross-runtime test harness deferred per established v0.1 pattern, but the live oracle makes runtime testing easier and stronger than for prior submissions (which had no oracle, only documented examples).

## Strategist memory work items (logged separately to memory file)

The following pattern observations from this PR will be added to the catdef-strategist memory file:

1. **Dual-review pattern for production-extracted roledefs** (source-project peer review + roledef-validator)
2. **Live runtime oracle pattern** (citing production deployment in `x.roledef.turing_test_reference`)
3. **Field-by-field provenance audit pattern** in `metadata.extracted_from`

These should be considered for promotion to roledef CLAUDE.md or SCHEMA.md guidance in a follow-on PR.

## Notes

- **Strategist bot identity:** this decision is provisionally attributed to `roledef-strategist <roledef-strategist@roledef.org>` pending governance ratification (Known Work Item, inherited).

- **Library state after this PR:** 4 roledefs in canonical (2 meta-roledefs + 2 non-meta). Library is fully operational and growing.

- **Recommended next submissions** (per user-confirmed sequencing): rest of seed library — catdef-maintainer self-extraction, brother-blackhat-tester self-extraction, etc.

- **Surfaced source-product bug:** server.py line 149 says "ALL SIX outputs" while line 201 says "ALL FIVE". Logged here for brother-DangerStorm to fix in the DangerStorm repo as a separate issue.

## Cross-references

- PR: https://github.com/roledef-spec/roledef/pull/4
- roledef file: [`../roledefs/senior-jaded-vc-associate.openthing`](../roledefs/senior-jaded-vc-associate.openthing)
- Catalog entry: [`../catalog.opencatalog`](../catalog.opencatalog)
- Sister roledefs: [`../roledefs/roledef-contributor.openthing`](../roledefs/roledef-contributor.openthing), [`../roledefs/roledef-validator.openthing`](../roledefs/roledef-validator.openthing), [`../roledefs/catdef-strategist.openthing`](../roledefs/catdef-strategist.openthing)
- Sister decisions: [`./roledef-contributor.md`](./roledef-contributor.md), [`./roledef-validator.md`](./roledef-validator.md), [`./catdef-strategist.md`](./catdef-strategist.md)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Source material: `s:/projects/dangerstorm/server.py SYSTEM_PROMPT` (lines 115-206); live oracle at https://dangerstorm.net
- Source-project peer review: PR #4 attribution comment (full report at `c:/Users/edsby/roledef-pr4-dangerstorm-review.md`)
- Validation report: PR #4 comment by roledef-validator role
