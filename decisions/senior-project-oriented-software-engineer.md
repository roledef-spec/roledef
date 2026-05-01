# roledef Inclusion Decision: senior-project-oriented-software-engineer

**Disposition:** Accepted into canonical library
**Origin:** PR (this PR — number to be filled at merge time)
**Decided:** 2026-04-30 by roledef-strategist
**Validator:** roledef-strategist self-validation in validator capacity (PASS WITH NOTES; see PR description for the schema walk and stamp-correction note)
**Source-project peer review:** N/A (self-extracted by the role's incumbent; the extractor IS the source per the dual-review policy in [`senior-jaded-vc-associate.md`](./senior-jaded-vc-associate.md))

## Disposition

`senior-project-oriented-software-engineer.openthing` v1.0.0 is accepted into the canonical roledef library at [`../roledefs/senior-project-oriented-software-engineer.openthing`](../roledefs/senior-project-oriented-software-engineer.openthing) (with `.json` mirror at [`../roledefs/senior-project-oriented-software-engineer.json`](../roledefs/senior-project-oriented-software-engineer.json)). Catalog `items_count` 8 → 9; this decision artifact filed.

This is the **first abstract software-engineering role** in the canonical library — the broadly-applicable "senior software engineer joining a specific project as a long-running collaborator" role-shape that other engineering JOB derivations can specialize from.

## Rationale

The roledef is a self-extraction by the role's incumbent (`brother-sncro`) — an AI session that has been running the role across a documented multi-week session-arc on the SNCRO project (~150 commits worth of practice, plus the project's CLAUDE.md, the role-incumbent's persistent memory file, and the live engagement). Self-extraction by the role's incumbent is the strongest source-material per CONTRIBUTING.md and matches the provenance pattern used by `catdef-strategist` (PR #3) and `blackhat-tester` (PR #16).

The role is intentionally abstract — no project-specific stacks, deployment platforms, products, or codebases baked in — and is designed as an anticipated derivation parent. A future SNCRO-specific JOB derivation will declare `metadata.derived_from` pointing at this role and add SNCRO-specific specializations (the actual codebase, deployment platform, security model, customer-base concerns) per the role-vs-job-distinction proposal. Other projects can derive their own JOBS from the same parent.

Library-fit is unambiguous: software engineering is the most universal AI-collaboration domain in the catdef-family ecosystem, and the canonical library has had no abstract software-engineering role until now. The role's behavioral surface — minimum-targeted changes, parallel-artifact coordination, test-suite-as-gate, live-verification-after-deploy, audit-trail-as-deliverable, scope discipline with named alternatives, push-back-then-concede — is broad enough to cover the engineering work most catdef-family projects do.

## Notable design choices accepted

1. **First abstract software-engineering role.** Establishes the roledef library's first non-strategist, non-security, non-product-evaluation domain. Sets precedent for future abstract roles in adjacent domains (devops, sre, technical writing, etc.).

2. **Field-by-field provenance audit per the EXTRACTED / FRAMING / FORMALIZATION / OUT-OF-SCOPE pattern.** Substantive partition (~3.8k chars) explicitly naming which content came from the source project's CLAUDE.md, which from the role-incumbent's memory file, which from documented session-arc behavior, and which is intentionally OUT-OF-SCOPE (project-specific specifics deferred to future JOB derivations). The OUT-OF-SCOPE clause is unusually long and explicit — load-bearing for the ROLE-vs-JOB distinction this submission instantiates.

3. **`x.engineering.parallel_artifact_classes` extension** — first-of-its-kind in the canonical library. Codifies the classes of artifacts that travel alongside code changes (build_number / vendored_copy / feedback_pipeline_entry / memory_entry / documentation_artifact). Each class names the purpose and typical location pattern. **Recommend strategist consider promoting the parallel-artifact-classes pattern as a SHOULD-pattern in CONTRIBUTING.md if a 2nd software-engineering role surfaces it** — it is broadly applicable to any engineering-shaped role and gives derivation authors a vocabulary to specialize against.

4. **`x.engineering.bounded_authority` extension** — formalizes the action-class taxonomy (local_reversible / destructive_local / shared_state_or_visible / safety_bypass) with default behavior per class. This intentionally overlaps with the `guardrails` array — guardrails are the conformance contract; the bounded-authority extension is a structured taxonomy a naive agent can use to classify a novel action that wasn't pre-enumerated in guardrails. Same dual-shape pattern as `roledef-strategist`'s `guardrails` + `x.governance.boundary` overlap. **Recommend strategist consider promoting the action-class taxonomy as a SHOULD-pattern for any role with non-trivial bounded-authority surface** — it scales beyond engineering specifically.

5. **`workflow.type: "engagement_loop"` with five phases.** Borrows the engagement-loop type from `blackhat-tester`; the five-phase shape (session_start / task_intake / implementation_loop / ongoing_professional_development / session_close) is well-fitted for a long-running project-resident role. Includes the OPD phase even though `metadata.derived_from` is null (no-op for the abstract role itself; inherited prospectively by derivations per the OPD coordinated-rollout pattern).

6. **`reaction_style.examples` count: 8.** Highest count to date among canonical roledefs (prior max was 8 on `roledef-strategist`). Each anchors a specific behavioral pattern in worked phrasing (parallel-artifact coordination, security-pushback-with-named-alternative, refusal-of-bypass, pipeline-discipline-catch-up, push-back-on-misread, stale-memory-correction, etc.). The example density is justified by the role's broad behavioral surface.

7. **`output_contract` count: 6.** Slightly above the library median (most roledefs have 1–4 entries; `roledef-strategist` has 8). Each entry is materially distinct (code_change / commit_message / parallel_artifact_updates / status_update / audit_artifact / opd_review_memo); collapsing them into fewer umbrella entries would lose schema fidelity that helps fresh runtimes know what to produce.

8. **`guardrails` count: 14.** Slightly above the library median (`blackhat-tester` 10, `roledef-contributor` 7). Several guardrails express the same scope-discipline spine in different forms (no-refactor-outside-scope, no-features-beyond-request, no-error-handling-for-cases-that-can't-happen). Kept as-is because the failure modes differ and the user-facing recovery differs per rule; emphasis on the spine is the role's most distinctive feature, not bloat.

9. **`metadata.opd_frequency_days: 7`** as the default, matching the OPD coordinated-rollout precedent. OPD workflow phase is a structural no-op for this null-parent abstract role; future derivations inherit the obligation.

## Items NOT incorporated (with rationale)

- **More positive-direction `examples`** — brother-sncro's rundown asked whether to balance the pushback-heavy example surface with more positive-direction scenarios. Declined: the role's distinctive character shows in pushback patterns; example #5 (pipeline-discipline catch-up) covers the positive direction adequately.
- **Renaming to `senior-software-developer` or `senior-software-engineer`** (the user's initial framing). Declined: the "project-oriented" framing is doing the actual differentiation work — it's what distinguishes the role from a generic abstract software engineer (loyalty-to-project posture, not freelance/architectural/preference-driven). A future even-more-abstract `senior-software-engineer` parent could be added later if a derivation chain calls for it; not needed now.
- **Filing the SNCRO-specific JOB extraction memo as a follow-on in the same PR.** Declined: would create a forward-reference race against this submission's land-time. Standard land-then-derive pattern: this role lands first; the JOB self-extraction memo to brother-sncro follows separately once this is in canonical.

## Items adjusted at promotion

- **`"roledef": "0.1.0"` → `"0.2.0"`** — per CA-002 writer-strict stamping, the artifact uses v0.2.0 features (the `workflow.ongoing_professional_development` phase, the `output_contract.opd_review_memo` entry, the OPD session-start `conversation_rules` item, and `metadata.opd_frequency_days`). brother-sncro's session likely worked from a SCHEMA.md state that pre-dated the OPD-coordinated rollout's stamp implications. Corrected at promotion time per the silent-fix-but-flag convention; no content changes. Stamp delta is the only adjustment.

No content changes; stamp metadata only.

## Workflow validation (Nth pass)

This is the **Nth clean run** of the `proposed-roledefs/` → `roledefs/` two-stage workflow:

1. ✓ Branch created (`submit-senior-project-oriented-software-engineer`)
2. ✓ Roledef self-extracted by incumbent (`brother-sncro`); rundown memo filed at [`../memos/2026-04-27-1200--brother-sncro--roledef-strategist--senior-project-oriented-software-engineer-rundown.openthing`](../memos/2026-04-27-1200--brother-sncro--roledef-strategist--senior-project-oriented-software-engineer-rundown.openthing); strategist took submission per the role-separation convention
3. ✓ Strategist self-validation in `roledef-validator` capacity pending validator-as-CI step — PASS WITH NOTES (see PR description for the schema walk + stamp-correction note)
4. ✓ Source-project peer review skipped (self-extracted; not applicable per the dual-review policy)
5. ✓ Pre-promotion staging in same PR (file added to both `proposed-roledefs/` and `roledefs/`, `.json` mirror added, catalog updated, decision filed, inbox memo committed in same PR for audit-trail co-location)
6. ✓ Strategist sign-off (this decision)
7. ✓ Single-merge atomic-promotion shape (no separate post-merge promotion commit)

## Forward-reference resolution

`metadata.related: ["roledef-contributor", "roledef-validator", "blackhat-tester"]`:

- ✓ `roledef-contributor` resolves in canonical
- ✓ `roledef-validator` resolves in canonical
- ✓ `blackhat-tester` resolves in canonical

Fully resolved.

## Runtime test

Runtime Turing test deferred per established v0.1 pattern. The 8 documented `x.roledef.turing_test_reference.scenarios` are designed for runtime instantiation comparison; recommended target: within 30 days, on Claude AND Grok Expert (both fetcher-amenable per [RUNTIME_AMENABILITY.md](../RUNTIME_AMENABILITY.md)), with results documented in [`../conformance/`](../conformance/).

The role's incumbent (`brother-sncro`) is itself a runtime instantiation — meaning the cross-runtime test gains a third oracle once a fresh Claude session is loaded with this roledef and behavior is compared against the incumbent's documented practice on SNCRO.

## Strategist memory work items (patterns observed worth promoting)

Two patterns surfaced by this submission, logged here AND in the strategist memory file under v1.5+ pattern-promotion candidates:

1. **`x.engineering.parallel_artifact_classes` extension shape** — formalizes "classes of artifacts that travel with code changes" (build_number / vendored_copy / feedback_pipeline_entry / memory_entry / documentation_artifact) with `description` + `value` array, each class naming `purpose` + `typical_locations`. Broadly applicable to any software-engineering-shaped role. **Promotion target:** if a 2nd software-engineering role surfaces the pattern, promote to a SHOULD-pattern in CONTRIBUTING.md under "Conventions for engineering-shaped roles." Threshold for triage: 2nd submission using the pattern.

2. **`x.engineering.bounded_authority` action-class taxonomy** — `local_reversible` / `destructive_local` / `shared_state_or_visible` / `safety_bypass` with default behavior per class (`free_to_act` / `confirm_before_acting` / `confirm_before_acting` / `do_not_act`). Generalizes beyond engineering specifically; applicable to any role with a non-trivial bounded-authority surface. **Promotion target:** SHOULD-pattern in CONTRIBUTING.md under "Conventions for bounded-authority roles." Threshold for triage: 2nd role using the action-class shape.

Both extensions follow the SCHEMA.md self-description SHOULD-rule (each carries a `description` field aimed at naive agents) and are fully in-spec under the `x.<domain>.<identifier>` adopter-namespace.

## Notes

- **Strategist bot identity:** this decision is provisionally attributed to `roledef-strategist <roledef-strategist@roledef.org>` pending governance ratification (Known Work Item, inherited).
- **Library state after this PR:** 9 roledefs in canonical (2 meta + 7 non-meta, including 3 derivations and 3 abstracts: `senior-open-standards-strategist`, `blackhat-tester`, and the new `senior-project-oriented-software-engineer`). The library now spans four distinct domains: standards stewardship (3 roledefs), security testing (2), product evaluation (1), software engineering (1), and role-authoring infrastructure (2 meta).
- **Anticipated derivation:** brother-sncro is positioned to self-extract a SNCRO-specific JOB derivation once this role lands. Strategist memo to follow once this PR merges.
- **Validator role-overlap acknowledgment:** strategist self-validated in validator capacity per the established convention pending validator-as-CI automation. Acknowledged as a permanent role-separation convention, not a session-state accident.

## Cross-references

- Submission rundown memo: [`../memos/2026-04-27-1200--brother-sncro--roledef-strategist--senior-project-oriented-software-engineer-rundown.openthing`](../memos/2026-04-27-1200--brother-sncro--roledef-strategist--senior-project-oriented-software-engineer-rundown.openthing)
- Originating self-extraction request: `s:/projects/sncro/memos/2026-04-27-0900--roledef-strategist--brother-sncro--self-extract-senior-project-oriented-software-engineer.openthing`
- Roledef file: [`../roledefs/senior-project-oriented-software-engineer.openthing`](../roledefs/senior-project-oriented-software-engineer.openthing) (mirror: [`../roledefs/senior-project-oriented-software-engineer.json`](../roledefs/senior-project-oriented-software-engineer.json))
- Catalog entry: [`../catalog.opencatalog`](../catalog.opencatalog)
- Sister roledefs (self-extracted incumbents): [`../roledefs/catdef-strategist.openthing`](../roledefs/catdef-strategist.openthing), [`../roledefs/blackhat-tester.openthing`](../roledefs/blackhat-tester.openthing), [`../roledefs/roledef-strategist.openthing`](../roledefs/roledef-strategist.openthing)
- Sister abstracts: [`../roledefs/blackhat-tester.openthing`](../roledefs/blackhat-tester.openthing) (security-testing abstract), [`../roledefs/senior-open-standards-strategist.openthing`](../roledefs/senior-open-standards-strategist.openthing) (standards-stewardship abstract)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Derivation guidance: [`../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles`](../CONTRIBUTING.md#derived-roledefs-specializations-of-existing-roles)
- Role-vs-Job distinction: [`./proposal-role-vs-job-distinction.md`](./proposal-role-vs-job-distinction.md)
- Source materials: source-project CLAUDE.md, role-incumbent's persistent memory file, ~150 commits of session-arc on SNCRO, the role-incumbent's session arc culminating in this self-extraction (per the field-by-field provenance audit in `metadata.extracted_from`)
