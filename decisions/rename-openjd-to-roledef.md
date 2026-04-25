# Decision: Rename openjd → roledef

**Disposition:** Project renamed from `openjd` to `roledef`, effective 2026-04-25
**Origin:** Strategist runtime testing session, 2026-04-25 (same session that produced the four runtime conformance evidence files)
**Decided:** 2026-04-25 by roledef-strategist (formerly openjd-strategist), with explicit human steward sign-off
**Pre-rename project name:** `openjd` (in use 2026-04-23 through 2026-04-25, ~48 hours)

## Disposition

The project formerly known as `openjd` (and informally as "openjd Job Descriptions" / "JDs") is renamed to `roledef`. Specific renames applied across the repository:

| Old | New |
|---|---|
| `openjd` (project name, lowercase) | `roledef` |
| `OpenJD` (project name, TitleCase) | `RoleDef` |
| `openjd` schema version field | `roledef` schema version field |
| `openjd:JobDescription` (type) | `roledef:Role` (type) |
| `openjd:JDLibraryCatalog` (catalog type) | `roledef:Library` (catalog type) |
| `openjd:*` reserved namespace | `roledef:*` reserved namespace |
| `x.openjd.*` extension namespace | `x.roledef.*` extension namespace |
| `openjd-spec` (GitHub org) | `roledef-spec` (GitHub org) |
| `openjd-spec/openjd-spec` (repo) | `roledef-spec/roledef` (repo) |
| `openjd.dev` (canonical domain) | `roledef.org` (canonical domain) |
| `openjd-strategist@openjd.dev` (bot identity) | `roledef-strategist@roledef.org` (bot identity) |
| `openjd-validator`, `openjd-contributor` (meta-roledef IDs) | `roledef-validator`, `roledef-contributor` |
| "Job Description" / "JD" (artifact noun) | `roledef` (artifact noun — no abbreviation) |

The `.openthing` file extension is **unchanged** — that extension belongs to the catdef substrate, not to the consumer spec. All catdef-substrate documents are `.openthing` regardless of which consumer spec they conform to.

## Rationale

Three independently-discovered naming collisions surfaced during the conformance testing session of 2026-04-25:

### 1. AWS Thinkbox OpenJD (established, render-farm spec)

`OpenJD` is an established specification for batch render-farm job submission, descended from Deadline and primarily used in VFX/animation rendering pipelines. Repository at github.com/OpenJobDescription/openjd-cli. Multi-year history, substantial industry adoption in the rendering/HPC space.

The "OpenJD" acronym and the "Open Job Description" expansion are both in use. Continuing to use "openjd" for an AI-role-definition standard would have created direct namespace confusion with an established standard in an adjacent technical domain.

### 2. chrisbarry/openagent (recent, solo project)

Individual developer's spec at github.com/chrisbarry/openagent, created Dec 2025. Same problem domain as ours ("AI agents in a technology-agnostic, portable format"). 8 stars, low traction, but exists. Uses "agent" terminology that the strategist had already explicitly rejected (per the "employee not agent" framing established earlier in the catdef-strategist work).

### 3. Oracle Open Agent Specification (recent, enterprise weight)

Oracle's "Open Agent Spec" at github.com/oracle/agent-spec. Created Oct 2025, 339 stars, 45 forks, 29 open issues, very active development. Defines two primary objects (Agents and Flows), provides Python SDK (PyAgentSpec), reference runtime (WayFlow), adapters for common agentic frameworks. Enterprise-vendor backing makes this the most concerning of the three collisions.

The strategist evaluated whether to adopt Oracle's spec instead of building a separate standard. Decision: NO, because the design centers are genuinely different. Oracle's Agent Spec is "configure an agent runtime" (LLM provider + system_prompt + tool bindings + adapter pattern); roledef is "define a role's behavior" (voice + guardrails + reaction_style + output_contract + cross-runtime Turing testing). The two are complementary, not competitive — a roledef defines the role; an Agent Spec configuration defines the runtime to execute it.

### Combined finding: "Open[X]" prefix is becoming generic in this design space

Three independent parties (AWS Thinkbox, chrisbarry, Oracle) have converged on "Open[X]" naming for AI-role/agent specifications in the last six months. The "Open[X]" namespace in this design space is becoming generic. Picking ANY "Open[X]" name now risks future collision with the next entrant. A naming pattern that escapes the "Open[X]" family entirely is more durable.

## Why "roledef"

The strategist evaluated several candidate replacements (`OpenRole`, `OpenPersona`, `OpenEmployee`, `roledef`, others). `roledef` selected for these reasons:

1. **Matches the catdef family naming pattern exactly:** `catdef` (catalog definitions) → `roledef` (role definitions). Same naming morphology. Reinforces the catdef ecosystem brand instead of fragmenting it.

2. **Outside the contested "Open[X]" namespace entirely.** Nobody else is using "[X]def" naming for this design space, and the pattern is associated with the catdef family.

3. **Short, descriptive, distinctive.** "roledef" is what the spec is — definitions of roles.

4. **Clean coexistence story with Oracle Agent Spec.** "roledef defines AI roles (what behavior to instantiate); Oracle's Agent Spec defines AI agent runtime configurations (what infrastructure to instantiate them with)." Clear lane separation.

5. **No abbreviations, "roledef" is the noun for the artifact too.** "Shoot me the roledef" / "I'll write a roledef for that role" / "the roledef-contributor roledef" (the meta-roledef that defines how to write roledefs). Like calling a `.json` a "json" — the format name IS the noun for the artifact. This eliminates the JD / Job Description abbreviation that's been awkward for AI roles (HR-tinged).

## Strategic positioning post-rename

> **roledef is a catdef-family consumer specification for AI role definitions. It is complementary to (not competitive with) Oracle's Agent Spec, which addresses a different layer of the AI-role-instantiation stack.**

- **catdef** — vendor-neutral catalog substrate (the wire format)
- **roledef** — AI role definition consumer spec (built on catdef substrate; defines voice, guardrails, output_contract, conversation_rules)
- **Oracle Agent Spec** — agent runtime configuration spec (defines LLM provider, model, tool bindings, adapter)
- **AWS Thinkbox OpenJD** — render-farm job submission spec (different technical domain entirely; no longer a naming collision)

A complete AI worker specification could compose these layers: catdef-substrate `.openthing` containing roledef role definition fields, optionally referencing an Oracle Agent Spec runtime config block for runtimes that want one. Each layer addresses what it's good at.

## Migration

Executed as part of this PR (rename-openjd-to-roledef branch):

1. Content sweep: `openjd` → `roledef`, `OpenJD` → `RoleDef`, `Job Description` → `roledef`, `JD` → `roledef`, plus all derived strings (URLs, emails, namespaces, IDs)
2. File renames via `git mv`:
   - `decisions/jd-openjd-contributor.md` → `decisions/jd-roledef-contributor.md`
   - `decisions/jd-openjd-validator.md` → `decisions/jd-roledef-validator.md`
   - `jds/openjd-contributor.openthing` → `jds/roledef-contributor.openthing`
   - `jds/openjd-validator.openthing` → `jds/roledef-validator.openthing`
3. Cleanup pass: removed duplication artifacts from chained replacements (e.g., "roledef roledef" → "roledef")
4. JSON validity verification: all 4 roledefs + catalog parse correctly post-sweep
5. Decision artifact (this file) recording the rename, rationale, and migration steps

Post-merge migration steps (not in this PR):

6. Transfer repo from `openjd-spec/openjd-spec` to `roledef-spec/roledef` via `gh repo transfer` (preserves history; GitHub auto-creates redirects from old URLs)
7. Set up `roledef.org` canonical distribution: GitHub Pages on the new repo + CNAME for roledef.org (DNS managed by Cloudflare per user's transfer-in-progress as of 2026-04-25)
8. Strategist memory file update: rename history, new repo/domain, Oracle/AWS/chrisbarry collision findings logged

## Conformance evidence preserved

The four runtime conformance evidence files merged in PR #5 (Grok Expert / Claude Code / Gemini / Perplexity) have been content-renamed in this sweep but their methodology findings, conformance scores, and dispositions are unchanged. The historical conformance evidence remains valid; only the naming has been updated.

The same four runtimes will produce equivalent conformance results on the renamed roledef artifacts because the JD content, the schema structure, and the role behaviors are unchanged — only the names and namespaces are different.

## Notes

- **Pre-rename openjd is preserved in git history.** Anyone who needs to reference the original openjd naming can navigate to commits dated before this rename PR's merge timestamp. Git log preserves the full history.

- **GitHub auto-redirects from old URLs.** Once the repo is transferred to `roledef-spec/roledef`, all existing references to `openjd-spec/openjd-spec` URLs will auto-redirect via GitHub's repo-rename redirect mechanism. Existing bookmarks, links, and external references remain functional.

- **Canonical distribution at roledef.org bypasses raw.githubusercontent.com fragility.** Per the conformance evidence in [`conformance-evidence-first-pass.md`](./conformance-evidence-first-pass.md), `raw.githubusercontent.com` is unreachable from at least one major runtime fetcher (Perplexity sandbox). Serving roledefs at roledef.org via GitHub Pages + CNAME bypasses this fragility class entirely AND gives the standard brand-controlled URLs aligned with the schema.org distribution model.

- **The `openjd` name remains historically valid for ~48 hours of work** (project bootstrap through PR #5 merge, 2026-04-23 through 2026-04-25). All four runtime conformance PASSes were achieved under the openjd name; the rename does not invalidate that work, it migrates the project's naming to a more durable footing before broader adoption.

## Cross-references

- Pre-rename PR landing four conformance evidence files: PR #5 (merged 2026-04-25 as commit dc32140)
- Conformance evidence files (renamed in this PR): `conformance/runtime_evidence/`
- Conformance methodology decision artifact: [`./conformance-evidence-first-pass.md`](./conformance-evidence-first-pass.md)
- Schema reference: [`../SCHEMA.md`](../SCHEMA.md)
- Project README: [`../README.md`](../README.md)
- catdef family reference: catdef-spec at github.com/catdef/catdef-spec
- Adjacent specifications:
  - AWS Thinkbox OpenJD: github.com/OpenJobDescription/openjd-cli (render farms; different technical domain)
  - Oracle Agent Spec: github.com/oracle/agent-spec (agent runtime configuration; complementary layer)
  - chrisbarry/openagent: github.com/chrisbarry/openagent (solo experimental spec; same domain)
