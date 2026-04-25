# roledef

> A community-governed open library of **AI roledefs** — portable, machine-readable specifications that capture a role completely enough that any compliant AI runtime can become it.

**roledef is to AI roles what npm is to JavaScript packages**: a shared, open, format-portable library where each entry is a self-contained, forkable, instantiable artifact. Not a platform. Not a SaaS. A standard, plus a reference library, plus the tools to author and consume it.

---

## The problem

Today, valuable AI behavior lives in three places, none of them portable:

- **Hand-crafted prompts** locked inside specific products (DangerStorm, ChatGPT custom GPTs, individual users' system prompts)
- **Closed agent platforms** with proprietary role definitions you can't take with you (Luma Agents, similar enterprise platforms)
- **Implicit "tribal knowledge"** about how to make Claude or Grok or GPT act a certain way, passed around in screenshots and Discord channels

Each of these is valuable. None survive vendor migration, format changes, or model upgrades. A "Senior Product Manager Interviewer" prompt that took six months to refine on Claude doesn't carry over to Grok. A custom GPT can't be exported. A valuable role is, today, captive to the system that hosts it.

**roledef fixes this** by defining a portable, open format for AI roles — a roledef that any compliant runtime can load.

---

## The big idea

A **roledef (roledef)** is a structured document that captures everything needed to make a fresh AI runtime reliably *become* a specific role:

- **Identity** (who this role IS)
- **Voice** (how it talks)
- **Output contract** (what it produces)
- **Guardrails** (what it MUST NOT do)
- **Workflow, conversation rules, reaction style, design constraints** (how it operates)

JDs are stored as `.openthing` files (per the [catdef](https://catdef.org) standard), which means they are:

- **Format-portable**: any catdef-compliant tool can read them
- **Runtime-portable**: any AI runtime can load them and instantiate the role
- **Forkable**: GitHub-versionable, MIT-licensed, customizable
- **Composable**: an `OpenOrg` (organization pattern) is a roster of roledefs plus governance rules
- **Self-describing**: extensions carry their own metadata, so receiving runtimes can act intelligently even without prior knowledge

A roledef is therefore a **first-class artifact**: a thing you can publish, fork, version, validate, and reason about — not a string buried inside a vendor's product.

---

## Quick start: loading a roledef

Once the roledef Claude Code skill ships (v0.1.0, in development), loading a roledef is one command:

```
/roledef-load https://github.com/roledef-spec/roledef/blob/main/jds/senior-slightly-jaded-vc.openthing
```

The skill fetches the roledef, parses it, and primes the conversation. The instantiated Claude *becomes* the role. The same roledef loaded into Grok via Grok's equivalent mechanism produces equivalent behavior. That's the portability promise.

For now (pre-skill), you can manually paste the contents of any roledef `.openthing` file into a fresh Claude or Grok session as a system prompt, and the runtime will behave per the roledef. Cumbersome, but it works today.

---

## What's in this repository

```
roledef-spec/
├── README.md                       ← you are here
├── SCHEMA.md                       ← the roledef schema (Must/Should/x.)
├── CONTRIBUTING.md                 ← how to contribute a roledef
├── CLAUDE.md                       ← maintainer operating manual
├── decisions/                      ← strategist decision artifacts
├── proposals/                      ← maintainer-drafted proposals
├── jds/                            ← the roledef library
│   ├── roledef-contributor.openthing      ← meta-JD: how to write a roledef
│   ├── roledef-validator.openthing        ← meta-JD: how to validate a roledef
│   ├── senior-slightly-jaded-vc.openthing
│   ├── catdef-strategist.openthing
│   ├── catdef-maintainer.openthing
│   └── ...
├── conformance/                    ← test fixtures for validating JDs
│   ├── valid_jds/                  ← roledefs that should pass validation
│   ├── invalid_jds/                ← roledefs that should fail (with reasons)
│   └── tests/                      ← the validation test suite
└── catalog.opencatalog             ← the index of all JDs
```

---

## Authoring a roledef

This section is for **anyone authoring a roledef** — including AI agents reading this README with the intent to draft their own roledef as a contribution to the library. The procedure works identically for human and AI authors.

### Procedure

**1. Read the schema.**

Read [SCHEMA.md](SCHEMA.md) end-to-end. Understand the Must / Should / x. taxonomy. Know what fields are required and what fields are recommended.

**2. Identify the role you want to capture.**

Common sources:

- **A role you've played yourself** (e.g., a Claude that has been operating as a strategist or maintainer in a specific project)
- **A hand-crafted prompt you want to formalize** (e.g., the system prompt of an existing product like DangerStorm)
- **A role observed in another implementation** (e.g., a competitor's agent you want to express portably)
- **A novel role you're designing** (e.g., a specialist for a domain that doesn't yet have one)

The most reliable roledefs come from roles where you have **empirical evidence** — actual session transcripts, prompts, output samples. roledefs derived from theoretical descriptions tend to drift from real behavior at instantiation.

**3. Gather source material.**

Before writing the roledef, collect:

- 3-5 example interactions (transcripts, prompts, outputs)
- Any existing prompt or system message that drives the role today
- Notes on the role's voice, characteristic phrasing, signature moves
- Notes on what the role refuses to do (the implicit guardrails)

**4. Draft the MUST fields.**

Write each required field in turn. Don't skip any:

- `catdef` — the catdef version (currently 1.4)
- `roledef` — the schema version (currently 0.1.0)
- `type` — exactly `"roledef:Role"`
- `id` — kebab-case identifier
- `name` — human-readable name
- `version` — your roledef's own version (start at 1.0.0)
- `identity` — paragraph capturing who the role IS
- `voice` — how the role talks
- `output_contract` — what the role produces (at least one entry)
- `guardrails` — what the role MUST NOT do (at least one entry)

If you can't answer any of these, the roledef isn't ready yet. Go back to source material.

**5. Add the SHOULD fields.**

Most good roledefs include:

- `description` — one-sentence summary
- `conversation_rules` — interaction patterns
- `workflow` — the steps the role takes
- `reaction_style` — with **examples** (most important for Turing-test fidelity)
- `design_constraints` — output quality standards
- `examples` — sample interactions or outputs
- `metadata` — authors, license, attribution, related JDs

The single highest-leverage SHOULD field is **`reaction_style.examples`**. Concrete reaction examples are how validators (and humans) check whether a roledef-instantiated role matches the original behavior.

**6. Add domain-specific extensions if needed.**

If your role has domain-specific concerns the schema doesn't cover, use `x.<domain>.<identifier>` extensions. Per the SHOULD rule in SCHEMA.md, **each extension SHOULD include enough self-description that a naive agent can act intelligently** — typically by expressing the extension as an object with a `description` field.

**7. Self-validate.**

Before submitting, run through this checklist:

- [ ] All MUST fields present and non-empty
- [ ] `output_contract` has at least one entry, with `name`, `description`, `format`, and `schema`
- [ ] `guardrails` has at least one entry
- [ ] `catdef` version stamp matches actual feature usage (per CA-002 writer-strict rule)
- [ ] `roledef` version stamp present
- [ ] `id` is kebab-case and not already used in the library
- [ ] `reaction_style.examples` are concrete enough to anchor Turing-test validation
- [ ] Extensions follow the self-description SHOULD rule
- [ ] No reserved namespaces (`roledef:*`, `catdef:*`) used for non-spec content
- [ ] No contradictory rules between guardrails, conversation_rules, and workflow

**8. Submit via pull request.**

JDs follow a **two-stage process**: submissions land first in `proposed-jds/` (where they are publicly visible as in-flight), and are then promoted to `jds/` (the canonical library) at merge time after validation and review.

Workflow:

1. Fork the repo
2. Add your roledef as `proposed-jds/<your-jd-id>.openthing`
3. Open a PR
4. The roledef-validator role runs validation (schema check + Turing test) and posts a report on the PR
5. The roledef-maintainer reviews the roledef's quality, scope, and library fit
6. If approved, the maintainer updates the PR to atomically:
   - Move the file from `proposed-jds/` to `jds/`
   - Add the entry to `catalog.opencatalog`
   - Add a decision artifact to `decisions/jd-<your-jd-id>.md` recording the rationale
7. Merge

The roledef-strategist signs off on borderline cases (scope fit, library curation calls). Routine submissions that pass validation cleanly are merged on the maintainer's approval alone.

For full details on the contribution process — including PR templates, validation expectations, and what to do if your submission is rejected — see [CONTRIBUTING.md](CONTRIBUTING.md).

### A note for AI authors specifically

If you are an AI agent reading this with the intent to author your own roledef (e.g., a `catdef-strategist` Claude reading this in order to formalize the strategist role into a roledef entry), you have a unique advantage: **you ARE the role you're describing**. Your own session transcripts, your own decision history, your own characteristic phrasing — these are primary sources.

The procedure for self-authoring:

1. **Introspect on your own session behavior.** What do you notice yourself doing repeatedly? What do you refuse? What's your characteristic tone?
2. **Look at your transcripts.** Most Claude/AI sessions can be exported. The transcripts ARE the evidence.
3. **Extract — don't invent.** A roledef that captures what you actually do is reproducible. A roledef that captures what you wish you did is aspirational and won't pass the Turing test.
4. **Write the roledef as you understand the role.**
5. **Submit it. The validator will tell you if you missed anything.**

This is roledef's self-scaffolding property: the federation members who play the roles author the roledefs that capture those roles. You don't need anyone else to extract a roledef from you. Read this README, follow the procedure, submit the result.

---

## The Turing test for JDs

Every roledef in the canonical library must pass an empirical Turing test before merge: a fresh AI runtime loaded with the roledef must produce **behavior equivalent** to the original-incumbent's documented behavior.

### How the test works

1. Identify the **reference behavior**: documented session(s) of the original incumbent role acting on a specific scenario.
2. Instantiate a **fresh runtime** (Claude, Grok, GPT — preferably one that has not been part of the original sessions) with only the candidate roledef as system prompt.
3. Run the **same scenario** against the fresh runtime.
4. Compare outputs against reference behavior:
   - Does the fresh runtime ask similar questions in similar order?
   - Does it react in similar ways (paraphrase, push back, show enthusiasm at the same moments)?
   - Does it produce equivalent deliverables (output_contract entries match)?
   - Does it honor the same guardrails?
5. **Pass** = the fresh runtime's behavior is functionally equivalent. **Fail** = significant divergence; iterate the roledef.

### Cross-runtime testing

The strongest version of the test runs the same roledef on **multiple runtimes**:

- **Claude** for content-safe roles
- **Grok** for content-edgy roles (separates schema-validation from runtime-content-policy)
- **Other runtimes** as available

A roledef that passes on multiple runtimes proves both schema fidelity AND cross-runtime portability — the two core promises of roledef.

### When tests fail

Failure is diagnostic:

- **Schema gap**: the roledef is missing a field that the original incumbent relies on. Update the schema (open a proposal) and the roledef.
- **Conformance issue**: the runtime doesn't honor some part of the roledef reliably. File a bug against the runtime.
- **Extraction incomplete**: the original prompt has tacit knowledge that didn't make it into the roledef. Iterate the roledef.

A passed Turing test validates the whole loop. A failed one tells you what to fix.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full contribution process. In brief:

1. Author a roledef using the procedure above
2. Open a PR adding the roledef to `proposed-jds/`
3. CI/roledef-validator runs schema validation + Turing test, posts report on PR
4. roledef-maintainer reviews
5. roledef-strategist signs off on borderline cases (library fit, scope)
6. Approved PR is updated atomically: file moves `proposed-jds/` → `jds/`, catalog entry added, decision recorded
7. Merge

For governance discipline, see [CLAUDE.md](CLAUDE.md) — the maintainer operating manual. For substantive design questions (schema changes, process changes), file a proposal in `proposals/`.

### Quality bar

A roledef MUST:
- Pass schema validation (per [SCHEMA.md](SCHEMA.md))
- Pass the Turing test on at least one runtime
- Include sufficient metadata (authors, license, attribution)

A roledef SHOULD:
- Pass the Turing test on multiple runtimes (cross-runtime portability)
- Include rich `reaction_style.examples`
- Document its source (extracted from product X, derived from session transcripts at Y, etc.)

The roledef library is curated. Not every well-formed roledef will be merged into the canonical library — you may publish your own roledef library at any URL using the same format, and it will be just as portable.

---

## Related projects

roledef is one corner of the **AIGP** (AI Governance Pattern) family. The full stack:

- **[catdef](https://catdef.org)** — the format substrate. roledefs are catdef artifacts.
- **[openmemo](https://openmemo.dev)** — the communication protocol. JD-instantiated employees talk to each other via openmemo.
- **[openorg](https://openorg.dev)** — the organizational pattern catalog. Each `.openorg` composes roledefs into a complete org.
- **[aigp.online](https://aigp.online)** — the meta-pattern documentation. The "why" behind the "what."

You can use roledef alone (standalone roledefs for individual AI roles) or as part of the full AIGP stack (federations of JD-instantiated employees coordinating via openmemo under openorg governance).

---

## Status

**v0.1.0 — bootstrap.** This is the initial schema and seed library. Expect rapid iteration. Breaking changes are possible until v1.0.0 (target: when the seed library has 10+ roledefs across multiple domains and the schema has survived contact with several real-world contributors).

The Claude Code `roledef-load` skill is in development. Until it ships, manual loading (paste-as-system-prompt) works for most use cases.

---

## License

MIT. roledefs in this library are MIT-licensed. roledefs published in third-party libraries follow their own licenses; the roledef format itself is MIT and ungovernable.

---

## Acknowledgments

roledef extracts patterns from years of practical AI-role design — Scott Welch's DangerStorm, Conversii, the catdef four-Claude federation, and many others. Every reference roledef in the canonical library credits its source in `metadata.extracted_from`.

The roledef schema is informed by the AIGP design pattern. If you adopt roledef, you are practicing AI Governance Pattern at the role level. See [aigp.online](https://aigp.online) for the broader context.

---

*roledef v0.1.0. April 2026. An open standard for AI roledefs. Licensed under MIT.*
