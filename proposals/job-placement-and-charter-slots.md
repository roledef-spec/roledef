# Proposal: Job placement and charter slots

**Status:** Draft
**Author:** orgdef-strategist (provisional bot identity pending governance ratification)
**Created:** 2026-04-26
**Target version:** 0.2.0 (alongside role-vs-job-distinction acceptance)
**Origin:** Empirical dogfood at orgdef-spec, 2026-04-26. Surfaced during the bootstrap of the orgdef-spec org and the instantiation of its first job (orgdef-strategist).

## Summary

This proposal **supplements** [`role-vs-job-distinction.md`](role-vs-job-distinction.md) by adding two slots to the new Job type that proposal introduces: `placement` (structural position within the org) and `charter` (the stable slot of responsibility). Both slots emerged empirically when bootstrapping the orgdef-spec org and writing the first concrete job artifact. Both are necessary for the **completeness** criterion (a fresh AI loaded with the orgdef + role + job bundle becomes the job with zero friction).

This proposal does NOT re-litigate the role-vs-job split. It assumes role-vs-job-distinction's two-type model (or whatever final type-split that proposal lands on) and contributes the additional fields the dogfood proved load-bearing.

## Motivation

### Empirical evidence from orgdef-spec dogfood

On 2026-04-26 we bootstrapped the orgdef-spec org via two interactive facilitator roledefs (org-creation-facilitator and job-creation-facilitator, submitted as separate roledef instances). The first job we instantiated was `orgdef-strategist` — derived from `senior-open-standards-strategist`, scoped to the orgdef-spec org. The artifact lives at `orgdef-spec/orgdef-spec-org/jobs/orgdef-strategist.md` (provisional markdown form pending formal schema acceptance).

Two specific failures emerged when trying to express the job using only the fields role-vs-job-distinction defines:

1. **No way to express the job's structural placement.** The orgdef-strategist reports to the human Director, directs the orgdef-maintainer and canonical-implementor jobs, and coordinates with the catdef-strategist and roledef-strategist sibling-spec jobs. None of this fit the proposed Job schema. We worked around it with `x.orgdef.placement` as a test-namespace extension.
2. **No way to distinguish jobs that share a parent role.** A future "orgdef-maintainer" and a future "canonical-implementor" might both derive from a generic `senior-engineer` parent role. What makes them distinct as jobs is their *charter* — the stable slot of responsibility each fills. Without a charter slot, the only way to disambiguate is by id or by reading the entire identity prose.

Both slots are visible in the dogfooded artifact. Both are what the role-vs-job-distinction proposal correctly identifies as job-tier specializations — but doesn't formalize as named fields.

### Why this matters for "completeness"

The orgdef-spec org's load-bearing value is **completeness**: the orgdef + roledef + job bundle MUST be sufficient that a fresh AI runtime loaded with the bundle becomes the job, with zero context backfill. (This is a hallucination-prevention rule, not an aesthetic preference: the primary reader is a machine, and machines confabulate when given null context.)

A job artifact without placement is incomplete — the AI doesn't know who to escalate to, who to coordinate with, what to direct. A job artifact without charter is incomplete — the AI doesn't know the stable boundary of its slot, only the abstract role and the org context. Both gaps force the AI to either guess or ask the user, which violates the zero-friction bar.

## Proposed Change

Add two OPTIONAL fields to the Job type defined by role-vs-job-distinction. Both are RECOMMENDED for any operational job (any job that an AI session will actually be instantiated into); MAY be omitted for sample/template jobs in the canonical library.

### `charter` (string or polymorphic)

Single statement of the stable slot of responsibility this job fills. Distinguishes one job from another when both derive from the same parent role.

```json
"charter": "Holds delegated authority for orgdef-spec design decisions: library curation, scope narrowing, pattern promotion, MUST/SHOULD calls, and architectural deliberation. The stable slot of responsibility is 'orgdef as a credible open standard within the AIGP family.'"
```

Format guidance: prose, 1–4 sentences. Polymorphic translatable (i18n) form per catdef. Should articulate *what stable purpose this position serves*, not what tasks the role performs (those belong in the parent roledef's `output_contract` and `workflow`).

### `placement` (object)

Structural placement within the org. Sub-fields:

```json
"placement": {
  "org": "<orgdef-id>",
  "reports_to": "<job-id or position-id>",
  "directs": ["<job-id>", "<job-id>"],
  "coordinates_with": ["<job-id>", "<job-id>"],
  "peer_to": ["<job-id>"]
}
```

| Sub-field | Cardinality | Purpose |
|---|---|---|
| `org` | exactly one | The orgdef this job belongs to (id or full coordinate) |
| `reports_to` | zero-or-one | The job/position this job escalates to |
| `directs` | zero-or-many | Jobs/positions this job has authority over |
| `coordinates_with` | zero-or-many | Peer jobs (often in sibling orgs) this job collaborates with |
| `peer_to` | zero-or-many | Jobs at the same authority level within the same org (e.g., co-strategists) |

All sub-fields are optional individually; `placement` itself is optional. A job at the top of an org chart has no `reports_to`. A leaf job has no `directs`. A standalone solo job may have only `org`.

References use the same shape as `metadata.role_definition` (id + version + url) where pinning matters; bare ids acceptable for intra-org references.

### Where the slots live

The placement and charter slots live ON the Job artifact (not the orgdef Position, not the parent Role). Rationale:

- A Job is portable — it travels with its placement (e.g., a private job moves between orgs and carries its context)
- Placement and charter are properties of the **specialization**, not the abstract role
- Bootstrapping AI loads the Job and immediately knows its context; no second lookup required

## Backward Compatibility

This proposal is purely additive on top of role-vs-job-distinction:

- Both slots are OPTIONAL — existing Job artifacts (once role-vs-job-distinction lands) remain valid without them
- Generic Roles in the canonical roledef library are unaffected (they don't have these slots; nothing changes for them)
- The schema version bump that role-vs-job-distinction triggers covers this proposal too — no separate version bump needed if both proposals land together

Migration story: as adopters create operational jobs, they add placement and charter. Sample jobs in the canonical library MAY include them as exemplars but aren't required to.

## Conformance Tests

New conformance fixtures in `roledef-spec/roledef/conformance/`:

- `valid_jobs/job-with-full-placement.openthing` — exemplar with all placement sub-fields populated
- `valid_jobs/job-with-minimal-placement.openthing` — exemplar with only `placement.org`
- `valid_jobs/job-with-charter-only.openthing` — exemplar with charter but no placement (e.g., a solo job)
- `invalid_jobs/job-placement-references-nonexistent.openthing` — counterexample where `placement.reports_to` doesn't resolve in the referenced orgdef
- `invalid_jobs/job-placement-circular.openthing` — counterexample where placement creates a reports_to cycle

Validator updates:

- Validator MUST verify `placement.org` resolves to a valid orgdef artifact (when public)
- Validator SHOULD verify `placement.reports_to`, `directs`, `coordinates_with`, `peer_to` references resolve within the referenced orgdef
- Validator SHOULD detect and warn on placement cycles (e.g., A reports_to B, B reports_to A)
- For private orgs, validator MAY skip resolution checks (with WARN, not FAIL)

## Alternatives Considered

### Alt 1: Express placement only via `orgdef.relationships`

orgdef already has `orgdef:Relationship` (typed edges between positions: `{type: "reports_to", from: "X", to: "Y"}`). Don't add placement to jobs at all; rely on orgdef relationships.

**Rejected because:** the AI bootstrapping into the job needs a *job-centric* view, not an org-graph view to traverse. Forcing the AI to reconstruct its placement by graph-walking the orgdef adds friction on every cold start. The authoritative source can still be the orgdef relationships; the placement on the job can be a denormalized convenience view validated for consistency.

### Alt 2: Inline placement into `orgdef:Position` instead of the Job

Add the placement sub-fields to orgdef:Position rather than to the Job. The position carries the structural info; the Job carries only the role-specialization.

**Rejected because:** breaks portability. Jobs that travel between orgs (e.g., a contractor's job moved to a new client) lose their context. Also: a position references a job, but a job may be referenced by multiple positions; placement on the Job means the same placement applies across all referencing positions.

### Alt 3: Compute placement at projection time from `orgdef.relationships`

The facilitator (or a render-time tool) projects placement into the Job at the moment of AI instantiation. The Job artifact never persists placement.

**Rejected because:** requires runtime resolution machinery that doesn't yet exist; complicates the "load the bundle, BE the job" flow. Snapshot semantics (placement on the Job at authoring time) is consistent with roledef's existing git-fork derivation model.

### Alt 4: Make placement and charter part of the Job's `metadata`

Place these in `metadata.placement` and `metadata.charter` rather than as top-level fields, mirroring `metadata.derived_from` and `metadata.role_definition`.

**Considered:** would keep relational/contextual info clustered in metadata. Open question: does this serve the AI cold-start use case as well? If a fresh AI reads top-level fields first, top-level placement may be more discoverable. Author's weak preference: top-level for discoverability, but bikeshed-with-real-content welcome.

## Open Questions

1. **Single source of truth: `job.placement` vs `orgdef.relationships`.** Both can express the same structural facts. If both exist, which is authoritative? Validator should check consistency, but the spec needs to declare a winner. Author's weak proposal: `orgdef.relationships` is authoritative for the org-chart view; `job.placement` is a mandatory denormalized cache that the validator keeps in sync.

2. **Naming: `placement` vs alternatives.** "Placement" is descriptive but slightly bureaucratic. Alternatives: `context`, `org_position`, `position_context`. Avoid `relationships` to disambiguate from `orgdef.relationships`. Inviting bikeshed.

3. **Top-level vs `metadata`.** See Alt 4. Whichever roledef-strategist prefers; both work structurally.

4. **`peer_to` in the placement object.** Real shape for some orgs (co-strategists, co-leads), but adds another sub-field. Could defer to `coordinates_with` for v0.1 and add `peer_to` only if real demand surfaces.

5. **Charter as string vs structured object.** Currently proposed as prose (string or polymorphic). Could be structured (e.g., `{purpose, scope, accountability, success_criteria}`). Author's preference: keep prose for v0.1; promote to structure if patterns emerge across canonical jobs.

6. **Cross-spec coordination with orgdef.** This proposal touches orgdef substrate — orgdef may want a parallel SHOULD that says "if a Job references your Org, your relationships SHOULD reflect the Job's placement (or vice versa)." Requires orgdef-strategist coordination (which is the same person — me — but the proposal needs to land in orgdef-spec separately).

## Acceptance Criteria

If accepted (and assuming role-vs-job-distinction is also accepted):

1. **Roledef SCHEMA.md update** — define `placement` and `charter` as OPTIONAL/SHOULD fields on the Job type
2. **Conformance fixtures** — `valid_jobs/` and `invalid_jobs/` additions per Conformance Tests above
3. **Validator updates** — placement reference resolution and cycle detection
4. **Cross-spec coordination** — companion orgdef proposal flagging the `orgdef.relationships` ↔ `job.placement` consistency rule
5. **Migration of dogfood artifact** — `orgdef-spec/orgdef-spec-org/jobs/orgdef-strategist.md` migrated from markdown + `x.orgdef.*` extensions to formal `.openthing` JSON with the new fields

## References

- [`role-vs-job-distinction.md`](role-vs-job-distinction.md) — parent proposal that establishes the Job type. This proposal supplements it.
- `orgdef-spec/orgdef-spec-org/jobs/orgdef-strategist.md` — empirical exemplar from dogfood
- `orgdef-spec/orgdef-spec-org/orgdef.md` — the org charter the job sits within
- Originating conversation: Director (Scott) + orgdef-strategist, 2026-04-26 — bootstrap of orgdef-spec org through interactive facilitator roledefs
- Related: orgdef SCHEMA.md `orgdef:Relationship` — the org-chart-side expression of placement
- Related: roledef SCHEMA.md `metadata.derived_from` — the same lineage-pinning pattern this proposal mirrors for placement references
