# sample-jobs/

**Canonical sample `roledef:Job` artifacts** for demonstration purposes. These are not operational jobs — no live role-holder occupies any of these positions. They document patterns and serve as adopter starting-points.

## What goes here

Job artifacts that have **broad pedagogical or pattern-establishing value** for the roledef ecosystem. The criteria for sample-jobs/ inclusion (strategist call) include:

- The Job exemplifies a generally-useful pattern (e.g., the abstract→derived showcase, the sample-of-record for a self-extracted role)
- The Job is referenced as a worked example in the spec, CONTRIBUTING.md, README, or canonical decision artifacts
- The Job's owning project is willing to maintain a canonical sample-of-record copy alongside its operational copy

Most Jobs do NOT belong here. Operational Jobs live in their owning project's working repo (per [CONTRIBUTING.md §"Job artifacts"](../CONTRIBUTING.md#2-job-artifacts-roledefjob)). The canonical library curates Roles (broadly applicable) and sample Jobs (canonical demonstrations); routine operational Jobs are not curated.

## How sample Jobs differ from operational Jobs

A sample Job MAY:

- Omit `metadata.placement` (no concrete org-chart placement; the sample exists outside any specific operational org)
- Use an illustrative-only `metadata.org_definition` URL (e.g., a sample orgdef in the same repo)
- Omit `charter` if the Job exists purely as a structural template

A sample Job MUST still:

- Validate against the schema as a `roledef:Job` (all required fields present, additive-only invariant against parent Role honored)
- Stamp `roledef: "0.2.0"` or higher
- Declare `metadata.role_definition` pointing at a real Role artifact

## Submission

Sample Jobs follow the same workflow as Role submissions (see [CONTRIBUTING.md §"1. Role submissions"](../CONTRIBUTING.md#1-role-submissions-roledefrole)) — PR to `proposed-roledefs/<id>.openthing`, validator runs, maintainer reviews, strategist signs off on library fit, promotion at merge moves the file to `sample-jobs/<id>.openthing` (rather than `roledefs/<id>.openthing`).

The strategist sign-off is more rigorous for sample Jobs than for routine Roles — every sample Job is a deliberate library-curation call, not a routine canonical addition.

## Relationship to operational copies

Several sample Jobs in this directory have **operational copies** living in their owning project repos:

| Sample (here) | Operational copy |
|---|---|
| `catdef-strategist.openthing` | `catdef-spec/jobs/strategist.openthing` |
| `roledef-strategist.openthing` | `roledef-spec/roledef/jobs/strategist.openthing` |
| `sncro-blackhat-tester.openthing` | `sncro/jobs/blackhat-tester.openthing` |
| `senior-jaded-vc-associate.openthing` | `dangerstorm/jobs/vc-associate.openthing` |

The sample is pinned at the published version (immutable except for explicit re-bumps). The operational copy evolves with its owning project — adding org-specific placement, additional specializations, the `charter` for that specific org context, etc.

When the operational copy adds material specializations that have broader pedagogical value, those changes MAY be reflected back into the sample via a follow-on PR — but this is intentional curation, not automatic sync.

---

*sample-jobs/ directory introduced in roledef v0.2.0 alongside the role-vs-job-distinction acceptance.*
