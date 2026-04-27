# Proposal: Ongoing Professional Development (OPD)

**Status:** Open for discussion
**Author:** roledef-strategist (provisional bot identity pending governance ratification)
**Created:** 2026-04-27
**Originating discussion:** Conversation with the human steward 2026-04-27, articulating that the drift problem in derivation chains can be solved behaviorally rather than structurally — by making "stay current with your parent role" part of every role's own job description.

## Summary

Roledefs SHOULD include an Ongoing Professional Development (OPD) obligation: a periodic self-check (default at most every 7 days) where the role-holder fetches its `metadata.derived_from` parent at the parent's current version, diffs against the version this role was derived from, analyzes whether changes are material, and prompts the overseer with a memo recommending whether to incorporate updates.

This solves the multi-tier derivation drift problem (Senior Strategist → Sr. X Strategist → 3 jobs in Project X) **behaviorally** — by making continuing-professional-development part of the role's own definition — rather than structurally (no live `$ref`, no inheritance machinery, no dependabot infrastructure). The mechanism reuses existing fetch capability and the catdef-org memo convention; no new infrastructure required.

When `metadata.derived_from` is `null`, OPD is a no-op (no parent = nothing to track).

## Motivation

### The drift problem

Roledef derivations are git-fork-based per CONTRIBUTING.md: a derived role is a complete copy of the parent at a pinned version, declared via `metadata.derived_from`. There is no automatic propagation of parent updates — once forked, the derivation drifts as the parent evolves.

This is acceptable in many cases (the derivation is genuinely different from the parent and shouldn't track changes verbatim). But it becomes painful in two scenarios:

**Scenario 1: Local-mirror derivations.** A project takes a public canonical role, makes a near-mirror local copy with light specialization (e.g., adds project-specific guardrails or examples but otherwise inherits everything), and uses that local-derived role for the project's positions. When the public canonical role evolves (new patterns promoted, new guardrails added, refined turing scenarios), the local mirror doesn't see those updates and the project's positions silently fall behind.

**Scenario 2: Multi-tier derivation chains.** Per the human steward 2026-04-27:

> "I might take the Sr. Strategist role, derive a local copy in Project X, and then in turn use that Sr. X Strategist role to hire for 3 jobs."

Now there are TWO layers of staleness: the parent-of-parent (public Sr. Strategist) updates don't propagate to Sr. X Strategist, and consequently don't propagate to the three jobs that instantiate Sr. X Strategist. The longer the chain sits without updates, the further every node in the chain drifts.

### Existing mitigation: none

Today there is no spec-supported mechanism for derivation drift. The only options are:

- **Manual periodic re-fork**: contributor remembers to revisit, fetches the parent's current version, manually merges changes into the derived role. Doesn't scale; relies on human vigilance.
- **No mitigation**: derivations sit indefinitely; staleness accumulates; eventually the derived role no longer reflects current best practice in the parent's domain.

Neither is acceptable for derivations that are essentially mirrors plus light specialization.

### The behavioral solution

The human steward articulated the solution succinctly:

> "As part of every roledef, we tell the job-holder that part of their job description is that on a regular basis (e.g. at a maximum of every 7 days) they are to check their `metadata.derived_from`, review any changes that may have happened, analyze if those changes affect them, and if the changes will affect them, then prompt their human overseer on whether to make the changes."

This is **the same discipline that human professionals follow**: continuing education, attending conferences, reading journals, updating their understanding of best practice. Codifying it as a role obligation treats AI agents as truly autonomous entities with the same self-update responsibilities as their human counterparts.

## Proposed Change

### New SHOULD-pattern in roledef SCHEMA.md

Roles SHOULD include an Ongoing Professional Development obligation. The standard form has four components:

#### 1. Workflow phase

For roles using `workflow.type: "engagement_loop"` (the established convention for roles whose work is iterative rather than linear-sequence-shaped), add a recognized phase:

```json
"workflow": {
  "type": "engagement_loop",
  "engagement_setup": [...],
  "submission_decision": [...],
  "atomic_promotion": [...],
  "pattern_promotion": [...],
  "ongoing_professional_development": [
    "Check metadata.derived_from. If null, skip OPD entirely.",
    "Read the OPD state file at <working-repo>/opd-state/<role-id>.json (create on first run with last_check timestamp = now).",
    "If less than the role's opd_frequency_days days have elapsed since last_check, skip OPD until next session.",
    "Otherwise: fetch the current version of metadata.derived_from.url. Compare against the version recorded in metadata.derived_from.version (the fork-time version).",
    "Diff the parent at fork-time-version vs current-version. Identify changes affecting any field this role inherits or could be impacted by (guardrails added; output_contract entries added or tightened; new conversation_rules; new x.governance.* extensions; new turing scenarios; updated identity/voice essence; etc.).",
    "Categorize changes: (a) material (would change this role's behavior or output if adopted), (b) cosmetic (no behavioral impact), (c) inapplicable (parent change targets a domain this role doesn't operate in).",
    "If material changes exist: draft an opd_review_memo to the overseer (per the org's reports_to relationship from the orgdef artifact) with parent identity, fork version, current version, categorized diff, and recommendation (incorporate / partially incorporate / decline / defer).",
    "Update the OPD state file: last_check = now, last_known_parent_version = parent's current version.",
    "If no material changes: still update last_check timestamp; no memo needed (or send an informational 'no material drift' memo at the overseer's preference).",
    "Continue with the engagement after dispatching the OPD memo; do not block on overseer response."
  ],
  "engagement_close": [...]
}
```

#### 2. Conversation rule

Add to the role's `conversation_rules` array:

> "At session start, after reading the operating manual and memory file, run the OPD check (per workflow.ongoing_professional_development). The check is fast when no review is due (state-file timestamp comparison only). When a review IS due, run it before substantive work; the overseer's response is asynchronous."

#### 3. Output contract entry

Add to the role's `output_contract` array:

```json
{
  "name": "opd_review_memo",
  "description": "A memo to the role's overseer (per the org's reports_to relationship from the orgdef artifact) summarizing parent-role changes since this role was derived, categorizing changes as material/cosmetic/inapplicable, and recommending whether to incorporate updates. Issued at most every opd_frequency_days days when material changes exist.",
  "format": "memodef:Memo (or legacy x.memo.* form on a catdef:Thing)",
  "schema": "Required fields: from (this role's id), to (overseer's position id from the orgdef), subject ('OPD review: <parent-id> v<fork-version> -> v<current-version>'), sent (ISO 8601), body (markdown). Body sections: (1) Parent identity + URLs; (2) Versions (fork-time and current); (3) Material changes (per-field diff with intent); (4) Cosmetic changes (brief mention; no action recommended); (5) Inapplicable changes (changes that touch domains this role doesn't operate in); (6) Recommendation (incorporate all / incorporate selected / decline / defer with reason); (7) If recommending adoption, the proposed new derivation version (semver bump rationale).",
  "destination": "Overseer's working repo at <overseer.x.position.working_directory>/memos/ per the catdef-org memo convention. Filename: YYYY-MM-DD-HHMM--<this-role>--<overseer>--opd-review-<parent-id>.openthing"
}
```

#### 4. Per-role-instance state file

Each role-instance carries a small JSON state file at `<working-repo>/opd-state/<role-id>.json`:

```json
{
  "role_id": "catdef-strategist",
  "derived_from": {
    "id": "senior-open-standards-strategist",
    "url": "https://roledef.org/roledefs/senior-open-standards-strategist.openthing"
  },
  "fork_version": "1.0.0",
  "last_check": "2026-04-27T08:00:00Z",
  "last_known_parent_version": "1.0.0",
  "opd_frequency_days": 7
}
```

The state file lives in the role-holder's working repo (per authorization-implies-write-access — the role-holder writes the state file, just like memos). Created lazily on first OPD check; updated on every check. Committed to the working repo for cross-machine portability.

### Default placement on abstract roles

The `senior-open-standards-strategist` abstract role gets OPD added as part of its workflow + conversation_rules + output_contract. Per the additive-only invariant, ALL derivations of senior-open-standards-strategist (catdef-strategist v2.0.0, roledef-strategist v1.0.0, orgdef-strategist forthcoming, memodef-strategist forthcoming) inherit OPD automatically.

This is a v1.1.0 update to the senior-open-standards-strategist roledef. v1.0.0 derivations don't have OPD baked in; their first OPD check (post-update) would surface the parent's new OPD obligation as one of the changes to consider adopting. **The OPD pattern bootstraps itself**: once the parent has OPD, derivations adopt OPD on their next OPD check.

### Default placement on other abstract roles

Similarly, abstract roles that don't derive from senior-open-standards-strategist (e.g., `blackhat-tester`, `roledef-validator`, `roledef-contributor`) SHOULD also gain OPD if they have meaningful parents OR if their derivations would benefit. blackhat-tester's `derived_from: null`, so OPD is a no-op for blackhat-tester itself, but its derivations (sncro-blackhat-tester) inherit the OPD pattern from the abstract and would run OPD against blackhat-tester.

### `opd_frequency_days` field

A new top-level metadata field (or `x.opd.frequency_days`) declares the role's OPD frequency:

```json
"metadata": {
  "opd_frequency_days": 7,
  ...
}
```

Default: 7 days. Roles MAY tighten (e.g., 1-day for fast-moving roles) or loosen (e.g., 30 days for stable roles). Derivations MAY override the parent's frequency.

### Interaction with the role-vs-job-distinction proposal

OPD applies at EVERY tier in the role-vs-job hierarchy. If the role-vs-job proposal is accepted:

- **Roles** derived from other roles run OPD against their parent role
- **Jobs** derived from roles run OPD against their parent role (the abstract role they instantiate)
- **Jobs** derived from other jobs (rare but allowed) run OPD against the parent job

The mechanism is the same. Each tier is responsible for its own OPD; updates propagate organically through the hierarchy as each tier runs its own OPD and adopts changes.

The two proposals are independent (OPD makes sense without role-vs-job; role-vs-job makes sense without OPD) but compose well together. OPD is what makes the role-vs-job multi-tier hierarchy maintainable over time.

## Backward Compatibility

- **Roles without OPD remain valid** (SHOULD-pattern, not MUST). Adopters who don't want OPD simply don't include it.
- **Adding OPD to abstract parent does NOT silently propagate to existing derivations.** Existing derivations were forked at a version that didn't have OPD; they don't suddenly start running OPD checks. Their next OPD check (if/when they adopt OPD themselves) would surface the parent's new OPD obligation as one of the changes.
- **Roles with `derived_from: null` skip OPD entirely.** Abstract roles with no parent have nothing to track; their OPD-phase workflow steps are a no-op.
- **Existing artifacts unaffected.** All eight currently-published roledefs continue to validate. v1.1.0 of senior-open-standards-strategist (with OPD added) is a minor version bump (additive change to workflow / conversation_rules / output_contract); no breaking change.
- **Renderers (render.catdef.org) require no changes.** OPD-related fields are just more text content that the renderer displays.

## Conformance Tests

New conformance fixtures in `roledef-spec/roledef/conformance/`:

- `valid_roledefs/role-with-opd-included.openthing` — exemplar role with OPD properly placed in workflow, conversation_rules, output_contract; default opd_frequency_days
- `valid_roledefs/role-with-opd-frequency-tightened.openthing` — exemplar role overriding default 7-day frequency to 1 day
- `valid_roledefs/role-with-opd-but-null-parent.openthing` — exemplar abstract role that includes OPD in workflow even though derived_from is null (OPD becomes a no-op for the role itself but is inherited by derivations)
- `invalid_roledefs/role-with-opd-but-no-state-file-discipline.openthing` — counterexample where OPD workflow step doesn't reference the state file (would re-run on every session)

Validator updates:

- Validator MAY warn (not FAIL) if a role has non-null `derived_from` but no OPD in workflow / conversation_rules / output_contract. This is informational; some adopters intentionally skip OPD.
- Validator SHOULD verify that OPD-equipped roles include a working `output_contract.opd_review_memo` entry pointing at a valid memo schema.
- Validator MAY verify that `metadata.opd_frequency_days` is a positive integer if present.

## Alternatives Considered

### Alt 1: Auto-update parent at runtime (live `$ref`-style)

Make `metadata.derived_from.url` auto-resolve at role-instantiation time, with the parent's current content silently merged into the role at runtime.

**Rejected because:**
- Agents could change behavior unannounced — a parent update could break the derived role's contracts without anyone noticing
- Violates the bounded-authority discipline (changes happen without overseer review)
- Concurrency / consistency challenges — what if a parent updates mid-session?
- Loses the "fork at version V" pinning that makes derivations stable

### Alt 2: Manual periodic re-fork (status quo)

Today's behavior. Contributor remembers to revisit, fetches parent's current version, manually merges changes.

**Rejected because:**
- Doesn't scale; relies on human vigilance
- The whole point of the proposal is that this is the gap

### Alt 3: CI-triggered drift detection (server-side dependabot)

A server-side process scans all derivations weekly and opens PRs proposing parent-update incorporations.

**Rejected because:**
- Requires central infrastructure (a server, a cron, registry of all derivations to scan)
- Doesn't generalize to private derivations (server can't reach private repos without credentials)
- Centralizes a concern that should be distributed (each role-holder is responsible for itself)

### Alt 4: OPD with auto-incorporation (skip overseer prompt)

The agent runs OPD, identifies changes, and silently adopts them without overseer review.

**Rejected because:**
- Violates the bounded-authority discipline (agent self-modifies its role)
- Loses human-in-the-loop oversight on what could be material changes
- Inconsistent with how every other change in the catdef-family ecosystem is handled (strategist drafts, signs off; user merges)

### Alt 5: Recursive OPD checking

The agent's OPD check recursively traverses the entire derivation chain (parent → grandparent → great-grandparent → ...).

**Rejected because:**
- Adds complexity for marginal value
- Per-tier checks compose naturally: each tier runs its own OPD against its IMMEDIATE parent; updates ripple through the chain organically as each tier runs its check
- Recursive traversal would couple deeply-derived roles to the publication state of the entire chain

### Alt 6: Notification-based instead of polling-based

Instead of agents polling their parents on a schedule, parents notify their derivations when they update.

**Rejected because:**
- Requires the parent to know who derives from it (registry problem)
- Doesn't generalize to private derivations (parent can't notify what it doesn't know about)
- Tighter coupling that the polling model doesn't need
- Polling-based is simpler and works without infrastructure

## Open Questions

1. **Default frequency.** 7 days feels right as a starting point — long enough to avoid noise, short enough to catch material changes. But should it differ for fast-moving vs stable roles? My weak preference is to keep 7 as default and let individual roles override via `metadata.opd_frequency_days`.

2. **State file location.** Three options: (a) per-working-repo `<working-repo>/opd-state/<role-id>.json` (proposed); (b) embedded in the role's own metadata (mutable role artifact); (c) overseer-managed registry. (a) is simplest and aligns with the per-recipient-working-repo discipline. Inviting human-steward call.

3. **Multiple parents.** What about future composition (a role deriving from two parents)? OPD would need to check each. Currently `metadata.derived_from` is single-valued; if it becomes array-valued, OPD iterates. Not in v0.1 scope.

4. **Diff analysis depth.** Does the agent compare structurally (additive-only invariant violated? no missing-from-parent items?) or semantically (intent of changes)? My weak preference: structural diff with the agent's interpretive recommendation. Overseer makes the final call on incorporation.

5. **Memo recipient identification.** Per the orgdef artifact's reports_to relationship, the role-holder's overseer is a specific position. But what if reports_to chains exist (role-holder reports to A; A reports to B; ...)? OPD memo goes to the immediate overseer (single hop, not chain).

6. **"Material" vs "non-material" classification.** Who decides? The agent's judgment with overseer veto, or rules-based? My weak preference: agent judgment with overseer veto. Material = "would change this role's behavior or output if adopted." Inapplicable = "targets a domain this role doesn't operate in." Cosmetic = everything else.

7. **OPD failure handling.** What if the parent URL fails to resolve (404, network error, repo private and credentials missing)? Options: (a) skip OPD this cycle and try next time; (b) memo the overseer about the resolution failure; (c) FAIL the OPD check with a clear error in role state. Probably (a) for transient failures, (b) for persistent failures (3+ consecutive failed checks).

8. **OPD on jobs vs roles** (interaction with role-vs-job-distinction proposal). If role-vs-job is accepted, jobs also need OPD against their parent role. Same mechanism, same SHOULD-pattern. Worth confirming OPD applies uniformly across role and job tiers.

## Acceptance criteria

If accepted, the following work follows:

1. **roledef SCHEMA.md update** — define OPD as SHOULD-pattern, document the workflow phase, conversation rule, output_contract entry, state-file shape, opd_frequency_days metadata field
2. **senior-open-standards-strategist v1.1.0** — add OPD to the abstract role; existing derivations (catdef-strategist v2.0.0, roledef-strategist v1.0.0) will surface the parent's OPD obligation in their first OPD review post-update and incorporate it
3. **catdef-spec/CLAUDE.md update** — note OPD as a recognized cross-spec pattern (so AI-maintainer sessions in any catdef-family spec know to expect roles with OPD)
4. **catdef-org orgdef artifact update** — note that the org's positions are expected to run OPD against their role/job parents
5. **Conformance fixtures** — valid_roledefs/role-with-opd-*, invalid_roledefs/role-with-opd-incomplete-*
6. **Validator updates** — OPD-presence warning, OPD-output-contract verification, opd_frequency_days type check
7. **(Optional) memodef.org update** — note that OPD-review-memo is a recognized memo subtype (could become a `memodef:OPDReviewMemo` subtype eventually if the pattern stabilizes)

This is small-to-medium work (smaller than role-vs-job because it doesn't require restructuring the artifact tier). Recommend phasing: accept the proposal first; ship the spec update + senior-open-standards-strategist v1.1.0 + catdef-org update together; conformance fixtures + validator updates can follow.

## References

- Originating conversation: between human steward and roledef-strategist, 2026-04-27 (drift problem articulated; OPD framing proposed; "treat AIs as truly autonomous entities" framing)
- Related: [`role-vs-job-distinction.md`](role-vs-job-distinction.md) — sibling proposal; OPD applies at every tier including jobs
- Related: catdef-org orgdef artifact (memo convention) — OPD output reuses the per-recipient working-repo memo channel
- Related: catdef-family additive-only-derivation discipline — OPD adoption preserves the additive-only invariant by inheriting parent updates without removing parent items
- Related: bounded-authority discipline (catdef-spec/CLAUDE.md) — OPD memo is a recommendation; overseer makes the incorporation decision

## Notes

- **The OPD pattern self-bootstraps.** Once a parent role gets OPD, its derivations will (on their next OPD check) discover the parent has OPD and prompt the overseer to incorporate it. After incorporation, those derivations have OPD too. Bootstrap completes naturally.
- **OPD is the AI-equivalent of continuing professional education.** Same discipline; same model of treating AI agents as autonomous entities responsible for their own staying-current.
- **The mechanism reuses everything we already have.** Fetch capability (per render.catdef.org's /fetch proxy or any HTTP client). Memo channel (per catdef-org). State file (per the per-working-repo discipline). No new infrastructure.
- **OPD is bounded by the overseer.** The agent does the work; the overseer decides whether to incorporate. Same shape as every other catdef-family decision flow.
