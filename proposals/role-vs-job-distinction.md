# Proposal: Role-vs-Job Distinction

**Status:** Open for discussion
**Author:** roledef-strategist (provisional bot identity pending governance ratification)
**Created:** 2026-04-26
**Originating discussion:** Conversation with the human steward 2026-04-26 in the context of preparing for cross-machine portability and the broader question of where roledef artifacts should live.

## Summary

roledef currently conflates two distinct concepts under the single `roledef:Role` type: the abstract ROLE (broadly applicable, e.g., "Junior Developer", "Strategist", "Validator") and the concrete JOB (a specific instantiation of a role at a specific organization, e.g., "catdef-strategist", "Acme/payments-team Junior Developer"). This proposal separates them into two artifact types — roles and jobs — placed at two tiers of the catdef-family ecosystem with distinct ownership, locations, and privacy properties.

The role-vs-job distinction enables three things that the current conflation prevents:

1. **Same role, multiple jobs in the same org** — a company hiring three Junior Developers across three teams has one ROLE (Junior Developer) and three JOBS (each with team-specific specializations).
2. **Job-tier privacy** — a job's specifics (codebase references, internal naming, value-that-doesn't-move lists, source-project decisions) may be commercially sensitive. Roles stay public; jobs may be private.
3. **Authorization-implies-access symmetry** — jobs live in the working repo of the org that defines them, mirroring the catdef-org memo convention's per-recipient working-repo discipline.

## Motivation

### Empirical evidence from the catdef-family bootstrap (2026-04-25 → 2026-04-26)

We currently have eight roledefs in the canonical roledef library:

- `senior-open-standards-strategist` — abstract role, broadly applicable to any open-standards stewardship context
- `blackhat-tester` — abstract role, broadly applicable to any security-testing context
- `roledef-validator`, `roledef-contributor` — abstract roles for the roledef ecosystem
- `catdef-strategist v2.0.0`, `roledef-strategist v1.0.0` — JOBS that derive from senior-open-standards-strategist with catdef-family-specific specializations
- `sncro-blackhat-tester` — JOB that derives from blackhat-tester with sncro-specific specializations
- `senior-jaded-vc-associate` — JOB extracted from the DangerStorm.net production system prompt with DangerStorm-specific specializations

The job-shaped artifacts share a common pattern: they reference an abstract parent via `metadata.derived_from` and add ecosystem-specific or product-specific knowledge (substrate references, internal procedures, specific values, named precedents). They are NOT abstract; they are concrete instantiations.

Living all eight artifacts in the same canonical library is fine for the bootstrap because everything we are building is open-source. But it is the wrong shape for adopters who want to use roledef for commercial / private organizations:

- A consulting firm hiring a "Junior Developer" doesn't want their internal job description (with project codenames, team-specific procedures, compensation guidance) in a public library
- A financial services org with regulatory-sensitive role specifications can't publish those specifications publicly
- A product team's "Strategist" role with confidential roadmap context can't be a public canonical artifact

The current conflation works for catdef-family but does not generalize to the broader use case roledef is designed to enable.

### The framing the human steward articulated 2026-04-26

> "The same org may have the same role (Junior Developer) in multiple jobs within the org. The specifics of a job may contain commercially-sensitive material, which should not be in a shared repo. As for [the existing artifacts], I would argue that the roledef is for 'canonical-implementor'. catdef-canonical-implementor is the JOB with that role in catdef-std and catdef.org."

This framing crystallizes the distinction. The proposal formalizes it.

## Proposed Change

### Two-tier artifact model

| Tier | Type | Location | Visibility | Purpose |
|------|------|----------|------------|---------|
| Role | `roledef:Role` (existing type, narrowed scope) | Canonical roledef library | Public | Abstract role definition; broadly applicable; multiple jobs may instantiate one role |
| Job | `roledef:Job` (NEW type) OR `jobdef:Job` (NEW namespace) | Per-project working repo at `<project>/jobs/<id>.openthing` | Public OR private depending on org choice | Concrete instantiation of a role at a specific organization with project-specific specializations |

The choice between `roledef:Job` (under the roledef namespace) versus `jobdef:Job` (a new catdef-family consumer-spec sibling) is an open question; see Alternatives Considered below.

### Three-tier artifact graph (position → job → role)

orgdef positions currently have `position.role_definition` pointing directly at a roledef. Under this proposal, the canonical chain becomes:

```
orgdef:Position
  └─ position.job_definition → roledef:Job (or jobdef:Job)
                                 └─ job.role_definition → roledef:Role
```

A position references a JOB (the concrete instantiation, where the org-specific specializations live). The job in turn references a ROLE (the abstract definition). The position-to-role lookup goes through two hops; the job is the intermediate that carries the specializations.

For backward compatibility, `position.role_definition` (pointing directly at a role) MAY remain valid — interpreted as "this position uses the abstract role with no org-specific specializations." Most positions in real use will have `position.job_definition` instead.

### Sample/canonical jobs in the public library

Per the human steward's clarification: the public roledef library MAY host sample / canonical jobdefs for demonstration purposes, mirroring the memodef-spec/memodef `templates/` pattern (the spec repo curates templates; operational instances live in adopters' working repos).

For roledef-spec/roledef this means a new directory:

```
roledef-spec/roledef/
├── roledefs/         ← abstract ROLES (canonical library, primary content)
│   ├── senior-open-standards-strategist.openthing
│   ├── blackhat-tester.openthing
│   ├── canonical-implementor.openthing      (forthcoming, currently informal)
│   ├── maintainer.openthing                 (forthcoming)
│   └── ...
├── sample-jobs/      ← canonical sample JOBS (demonstration, not operational)
│   ├── catdef-strategist.openthing                  (sample job for catdef-spec context)
│   ├── sncro-blackhat-tester.openthing              (sample job for security-engagement context)
│   └── senior-jaded-vc-associate.openthing          (sample job for DangerStorm-style production extraction)
└── ...
```

Sample jobs document the pattern; adopters copy and adapt for their own working repos.

### Migration of existing artifacts

The eight existing canonical roledefs split into two tiers:

| Current artifact | Type today | Tier under proposal | Future location |
|---|---|---|---|
| `senior-open-standards-strategist` | role | role | stays in `roledefs/` |
| `blackhat-tester` | role | role | stays in `roledefs/` |
| `roledef-validator` | role (meta-JD) | role | stays in `roledefs/` |
| `roledef-contributor` | role (meta-JD) | role | stays in `roledefs/` |
| `catdef-strategist v2.0.0` | role (claimed) | **JOB** | move to sample-jobs/ AND publish operational copy at catdef-spec/jobs/strategist.openthing |
| `roledef-strategist v1.0.0` | role (claimed) | **JOB** | move to sample-jobs/ AND publish operational copy at roledef-spec/roledef/jobs/strategist.openthing |
| `sncro-blackhat-tester` | role (claimed) | **JOB** | move to sample-jobs/ AND publish operational copy at sncro/jobs/blackhat-tester.openthing |
| `senior-jaded-vc-associate` | role (claimed) | **JOB** | move to sample-jobs/ AND publish operational copy at dangerstorm/jobs/vc-associate.openthing |

The existing artifacts can remain at their current canonical URLs (with backward-compat redirects or aliases) to avoid breaking anything that references them. Migration is incremental.

### Privacy story

- **Public roles**: any role in `roledef-spec/roledef/roledefs/` is MIT-licensed and freely usable
- **Public sample jobs**: any sample job in `roledef-spec/roledef/sample-jobs/` is MIT-licensed and serves as adopter starting-point
- **Operational jobs** in adopter working repos: licensed at the adopter's discretion. Public projects (catdef-family, DangerStorm-style open-extraction projects) publish their operational jobs in public repos. Commercial / private orgs keep their jobs in private repos.
- **orgdef position references** to private jobs gracefully fail to resolve for outsiders (404); the org's own sessions resolve them via authenticated git access. This mirrors how private package-manager dependencies work.

### Required field changes

`roledef:Job` (or `jobdef:Job`) artifacts have these required fields beyond the catdef envelope:

- `job` (string) — short kebab-case job id within the org
- `name` (string or polymorphic) — human-readable job name
- `version` (semver)
- `metadata.role_definition` (object: id + version + url) — the abstract role this job instantiates
- `metadata.org_definition` (object: id + version + url) — the org this job exists in (references an `orgdef:Organization`)

Plus the role-tier specializations (identity / voice / output_contract / guardrails / conversation_rules / workflow / reaction_style / examples / metadata), which under the additive-only convention extend the abstract role's specifications.

`orgdef:Position` gains an OPTIONAL `job_definition` field (id + version + url) pointing at a job. The existing `role_definition` field remains valid for positions that don't need job-tier specialization.

## Backward Compatibility

- Existing `metadata.derived_from` references on currently-public artifacts continue to resolve.
- Existing `position.role_definition` references continue to work (interpreted as "no job-tier specialization").
- Existing artifacts can be left at their current canonical URLs; new `sample-jobs/` directory is additive.
- The new `roledef:Job` type is a new addition under the `roledef:` namespace (or under a new `jobdef:` namespace if we go that way); existing `roledef:Role` artifacts are unaffected.
- Renderers (render.catdef.org) already handle catdef-family namespaced types; rendering `roledef:Job` artifacts requires no renderer changes.

Migration is incremental: as adopters move their operational jobs out of the public library into their own working repos, existing canonical-library entries can be aliased or redirected. No big-bang restructuring required.

## Conformance Tests

New conformance fixtures needed in `roledef-spec/roledef/conformance/`:

- `valid_jobs/canonical-implementor-job.openthing` — exemplar job with role_definition + org_definition references and additive-only specializations
- `valid_jobs/job-with-private-org.openthing` — exemplar job whose `org_definition.url` is a private (non-resolvable for outsiders) URL
- `invalid_jobs/job-without-role.openthing` — counterexample missing required `metadata.role_definition`
- `invalid_jobs/job-with-fork-not-derivation.openthing` — counterexample where the job removes parent guardrails (violates additive-only)

Validator updates:

- Validator MUST verify that `metadata.role_definition.url` resolves for jobs published in public repos
- Validator MUST verify additive-only invariant against the parent role
- Validator MAY skip role-resolution check for jobs whose URL pattern indicates a private repo (with WARN, not FAIL)

## Alternatives Considered

### Alt 1: Single `roledef:Role` type, no separation (status quo)

What we have today. Works for the catdef-family bootstrap because everything is open. Doesn't generalize to commercial / private use cases. Doesn't capture the "same role, multiple jobs" concept.

**Rejected because:** doesn't generalize.

### Alt 2: `roledef:Job` under the roledef namespace

What this proposal currently specifies. Roles and jobs are sibling types within roledef-spec.

**Considered:** the simplest evolution; doesn't require bootstrapping a new consumer-spec; preserves the role-vs-job tight coupling at the spec level.

### Alt 3: `jobdef:Job` as a new catdef-family consumer-spec sibling

Bootstrap a separate `jobdef-spec/jobdef` repo with its own SCHEMA / CONTRIBUTING / CLAUDE / etc. Jobs become a first-class catdef-family member alongside roledef.

**Considered:** cleaner separation of concerns; allows jobdef to evolve independently from roledef; better matches the catdef-family pattern of one-spec-per-domain.

**Open question:** which is right? My weak preference is **Alt 2** (`roledef:Job` under roledef namespace) for v0.1 of this proposal — minimum new infrastructure, ships sooner. We can promote to **Alt 3** (separate `jobdef-spec`) later if patterns surface that warrant the separation. But the human steward's call.

### Alt 4: Two-tier without a new type — just convention

Use convention to distinguish roles from jobs. Roles live in `roledefs/`; jobs live in `jobs/` directories in working repos. Both are `roledef:Role` typed artifacts. The only difference is location.

**Rejected because:** loses machine-readability of the role-vs-job distinction. Validators can't enforce different rules for roles vs jobs without a type discriminator. Renderers can't render them differently.

### Alt 5: Inline job specializations in orgdef positions (no separate job artifact)

Keep `position.role_definition` pointing at the abstract role; let positions inline all the specializations (in `position.x.org.specializations` or similar). No separate job artifact.

**Rejected because:** inlining specializations in positions denies job artifacts the ability to be referenced by multiple positions (the "same role multiple jobs" case). Also denies the privacy / per-project-repo placement that's the main motivation.

## Open Questions

1. **Type namespacing: `roledef:Job` or `jobdef:Job`?** (See Alternatives Considered Alt 2 vs Alt 3.) Inviting human-steward call.

2. **Migration ordering**: do we migrate the existing four job-shaped artifacts (catdef-strategist, roledef-strategist, sncro-blackhat-tester, senior-jaded-vc-associate) all at once, or incrementally as their owning projects pick up the work? Incrementally is less risky but creates a temporary inconsistency.

3. **Sample jobs in public library**: which existing artifacts become sample jobs vs which get fully migrated to their owning projects? The senior-jaded-vc-associate JOB extracted from DangerStorm could either: (a) move to dangerstorm/jobs/ (operational), (b) stay at roledef-spec/roledef/sample-jobs/ (canonical demonstration), or (c) BOTH (operational at dangerstorm/, sample-of-record at roledef-spec/). My weak preference is (c) for the bootstrap-era artifacts, since they have value as worked examples.

4. **Position field naming**: `position.job_definition` is symmetric with `position.role_definition`. Alternative: `position.instance_of` or `position.fills` or other names. Inviting bikeshed-with-real-content.

5. **Privacy-flag at the job tier**: should jobs carry an explicit `x.job.commercial_sensitivity` field? Or is this purely conveyed by location (private repo vs public repo)?

6. **Renderer updates**: render.catdef.org currently renders `roledef:Role` cleanly. Does it need anything special to render `roledef:Job` (or `jobdef:Job`)? Probably not (catdef-family namespaced-type support handles arbitrary new types per catdef.org PR #2), but worth confirming.

7. **catdef-org orgdef artifact update**: positions currently declare `role_definition` references; would need updating to declare `job_definition` once jobs exist for those positions. Follow-on PR to orgdef.

8. **Authorization-implies-access for jobs**: per the catdef-org memo convention, a memo to position X lives in X's working repo at `./memos/`. By the same discipline, would a JOB for position X also live in X's working repo? (Yes — that's the whole point of the per-project-repo placement.) Worth documenting explicitly.

## Acceptance criteria

If this proposal is accepted, the following work follows:

1. **roledef SCHEMA.md update** — define `roledef:Job` (or `jobdef:Job`) type, required fields, position.job_definition addition, sample-jobs/ directory convention
2. **roledef CONTRIBUTING.md update** — submission workflow for jobs (different from roles in important ways: jobs may live in adopter repos, not all jobs come through canonical library)
3. **Conformance fixtures** — valid_jobs/, invalid_jobs/
4. **Migration of existing 4 job-shaped artifacts** — to sample-jobs/ + operational copies in owning project repos
5. **catdef-org orgdef artifact update** — positions declare job_definition where applicable
6. **Validator updates** — additive-only-vs-parent-role check, role_definition URL-resolution check
7. **(Optional) jobdef-spec/jobdef bootstrap** — if Alt 3 chosen, the new consumer-spec scaffold

This is meaty work. Recommend phasing: accept the proposal first, then sequence the work post-acceptance with the human steward's pacing.

## References

- Originating conversation: between human steward and roledef-strategist, 2026-04-26 (cross-machine portability discussion that surfaced the role-vs-job distinction)
- Related: catdef-org orgdef artifact (orgdef PR #1) — current `position.role_definition` pattern that this proposal extends
- Related: memodef-spec/memodef bootstrap (2026-04-26) — precedent for the spec-curates-templates pattern that this proposal mirrors with sample-jobs/
- Related: catdef-family precedent for additive-only derivations (sncro-blackhat-tester, catdef-strategist v2.0.0, roledef-strategist v1.0.0) — the same discipline applies between jobs and parent roles
