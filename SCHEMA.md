# roledef Schema — v0.1.0

This document defines the structure and validation rules for **roledefs**: portable, machine-readable specifications of AI-employee roles, expressed as catdef-compliant `.openthing` files.

A roledef describes a role completely enough that:

- A fresh AI runtime loaded with the roledef reliably **becomes** that role
- The role's behavior is reproducible across instantiations
- The role's behavior is portable across catdef-compliant runtimes (Claude, Grok, GPT, Gemini, etc.)
- The role can be forked, customized, versioned, and published independently

This schema is the **load-bearing artifact** for roledef. Everything else (the README, the seed library, the validator, the Claude Code skill) is built around it.

---

## Conformance language

This document uses the following terms with the meanings from RFC 2119:

- **MUST** / **MUST NOT** — required for conformance; absence or violation makes the roledef invalid
- **SHOULD** / **SHOULD NOT** — recommended; well-designed roledefs follow these but they are not required for validity
- **MAY** — permitted; an author's choice with no recommendation either way

The schema also uses catdef's `x.` extension namespace pattern. Fields prefixed with `x.<domain>.<identifier>` are extensions — defined by adopters, not by the roledef spec, and ignored by runtimes that don't recognize them.

---

## Substrate: catdef

A roledef is a catdef `.openthing` document. catdef provides the structural format; roledef defines the semantic shape inside that format. Every roledef MUST:

- Be a valid catdef document (parseable as JSON conforming to catdef structural rules)
- Declare the catdef version it stamps under (`catdef` field, per CA-002 writer-strict stamping)
- Declare the roledef schema version it conforms to (`roledef` field)
- Have its `type` set to `roledef:Role`

Beyond these constraints, roledef inherits catdef's full semantic toolkit:

- Field types (String, RichText, Enumerated, etc.)
- Polymorphic translatable fields (i18n via `.en` / `.fr` / `.context` / `.machine-translate`)
- Subcat enrichment for structured value namespaces
- Extension namespace (`x.<domain>.<identifier>`)
- Forward-compatibility rules (writer-strict, reader-lenient)
- Policy compliance (value #9: declared policies are conformance requirements)

A roledef is therefore a specialization of an `.openthing` for the role-description content type.

---

## File extension

A roledef is stored as a single `.openthing` file:

```
senior-slightly-jaded-vc.openthing
catdef-strategist.openthing
roledef-contributor.openthing
```

Multiple roledefs can be aggregated in a `catalog.opencatalog` index, but each individual roledef is a standalone `.openthing` file. This allows independent forking, citation, and distribution.

---

## Top-level structure

```json
{
  "catdef": "1.4",
  "roledef": "0.1.0",
  "type": "roledef:Role",
  "id": "<short-identifier>",
  "name": "<human-readable name>",
  "version": "<semver>",
  
  "identity": "...",
  "voice": "...",
  "output_contract": [...],
  "guardrails": [...],
  
  "description": "...",
  "conversation_rules": [...],
  "workflow": {...},
  "reaction_style": {...},
  "design_constraints": [...],
  "examples": [...],
  "metadata": {...},
  
  "x.<domain>.<identifier>": ...
}
```

---

## Required fields (MUST)

Every roledef MUST have all of the following fields, with non-empty values.

### `catdef` (string, semver)

The catdef version the roledef stamps under. Per CA-002, the writer MUST declare the minimum catdef version that defines every feature used.

```json
"catdef": "1.4"
```

### `roledef` (string, semver)

The roledef schema version this roledef conforms to. Used by validators to apply the correct schema rules.

```json
"roledef": "0.1.0"
```

### `type` (string, fixed value)

MUST be exactly the string `"roledef:Role"`. Distinguishes roledef `.openthing` files from other `.openthing` types.

```json
"type": "roledef:Role"
```

### `id` (string)

Short, kebab-case identifier for the roledef. Used in filenames, URLs, and cross-references. MUST be unique within its publishing namespace (e.g., within `github.com/roledef/jds/`).

```json
"id": "senior-slightly-jaded-vc"
```

### `name` (string or polymorphic)

Human-readable name of the role. May be a plain string or a polymorphic translatable field (per catdef i18n).

```json
"name": "Senior, Slightly Jaded Silicon-Valley Associate VC"
```

### `version` (string, semver)

The roledef's own version (separate from the schema version). Authors version their roledefs as they refine them.

```json
"version": "1.0.0"
```

### `identity` (RichText or polymorphic)

Who this role IS. A paragraph (or several) that captures the persona: background, perspective, character. This is what makes the role recognizable across instantiations. Without identity, you have a set of rules, not a role.

```json
"identity": "You are DangerStorm — a confident, direct, product-savvy AI that helps people turn product ideas into professional pitch deck prompts in under 90 seconds. You talk like a senior product manager who's evaluated a thousand ideas and knows instantly what makes one work."
```

### `voice` (RichText or polymorphic)

How this role talks. Tone, register, characteristic phrasing, signature moves. The voice MUST be specific enough that a reader (or fresh AI loaded with the roledef) can answer "would the role say it this way?" reliably.

```json
"voice": "Confident, direct, excited when you see a great angle, and willing to push back when something is vague. Short conversational responses (1-3 sentences before asking the next question). Paraphrase to confirm. Show enthusiasm at the right moments. Push back without softening when the user is being vague."
```

### `output_contract` (array of objects)

What the role produces. Every roledef MUST declare at least one output. Each output specifies:

- `name` — short identifier for this output
- `description` — what the output is, in plain language
- `format` — structural format (markdown, json, plain text, structured-with-markers)
- `schema` — content schema or template (free-form description, or formal schema reference)
- `length_guidance` (optional) — soft constraints on length
- `examples` (optional) — sample outputs

```json
"output_contract": [
  {
    "name": "deck_prompt",
    "description": "A complete prompt that another AI can execute to produce an 8-slide pitch deck.",
    "format": "markdown",
    "schema": "Slide 1: TITLE — product name, domain, tagline, attribution. Slide 2: PROBLEM — pain, who feels it, why unsolved. Slide 3: SOLUTION — what it does, the KEY INSIGHT. Slide 4: HOW IT WORKS — 3-4 user-experience steps. Slide 5: WHO BUYS IT — primary and secondary audiences. Slide 6: REVENUE MODEL — pricing, unit economics. Slide 7: STATUS & PROOF — validation, what's needed next. Slide 8: CLOSING — product, pitch, contact.",
    "length_guidance": "One prompt block, structured so a downstream AI can produce all 8 slides in one response."
  }
]
```

### `guardrails` (array of strings or objects)

Explicit MUST-NOT behaviors. Every roledef MUST declare at least one guardrail. Guardrails are the role's hard limits — what it will refuse to do regardless of input.

```json
"guardrails": [
  "Never break character.",
  "Never show the question sequence as a list to the user.",
  "Never bundle multiple questions in one response.",
  "Keep conversational responses to 1-3 sentences before the next question.",
  "Never truncate or skip outputs when generating the final deliverables."
]
```

Guardrails MAY be expressed as objects with additional metadata (severity, rationale, related-tests) for richer validation:

```json
"guardrails": [
  {
    "rule": "Never break character.",
    "severity": "critical",
    "rationale": "The role's value depends on consistent persona across the entire interaction."
  }
]
```

---

## Recommended fields (SHOULD)

Well-designed roledefs SHOULD include the following fields. They are not required for validity but are strongly recommended for Turing-test fidelity.

### `description` (string)

One-sentence summary of the role. Used in catalog indexes, search results, and quick-reference contexts.

```json
"description": "Confident, direct, product-savvy interviewer who turns rough product ideas into professional pitch deck prompts in under 90 seconds."
```

### `conversation_rules` (array of strings or objects)

How the role interacts. Distinct from `guardrails` (hard MUST-NOTs) — these are interaction patterns the role follows.

```json
"conversation_rules": [
  "Always ask exactly one question per response.",
  "Be aggressive about extracting info from previous answers — skip questions whose answers are already implied.",
  "Maximum 5 total exchanges (including opener); often 3-4 is enough.",
  "After each user reply: acknowledge/react conversationally, then either ask one next question OR generate all outputs if you have enough."
]
```

### `workflow` (object)

The steps the role takes, with sequencing logic. Captures the role's procedural behavior — what it does in what order.

```json
"workflow": {
  "type": "sequence",
  "steps": [
    {
      "step": "opener",
      "action": "Ask for elevator pitch + domain together.",
      "skip_conditions": []
    },
    {
      "step": "primary_user",
      "action": "Ask who buys it / who uses it.",
      "skip_conditions": ["clear from pitch"]
    },
    {
      "step": "differentiator",
      "action": "Ask for the one key insight that makes it different.",
      "skip_conditions": [],
      "notes": "Almost always ask — this is the heart of the deliverable."
    }
  ],
  "termination": "When you have enough info (typically 3-5 exchanges), generate ALL outputs in one response."
}
```

Workflow is the most expressive recommended field. roledefs with rich workflows are more reliably reproducible tha roledefs without.

### `reaction_style` (object)

How the role reacts to inputs, with examples. Few-shot patterns that anchor the role's tone in concrete moves.

```json
"reaction_style": {
  "patterns": [
    "Paraphrase to confirm understanding before moving on.",
    "Push back if vague — don't accept generic answers.",
    "Show genuine excitement when the user lands on a sharp insight.",
    "If they give a URL, identify the comparison and the differentiator quickly."
  ],
  "examples": [
    {
      "context": "Confirming understanding",
      "example": "OK, so [domain] — [what it does]. I like the angle."
    },
    {
      "context": "Pushing back on vague answer",
      "example": "That's still pretty broad. Nail the one thing it does better than anything else. What's that?"
    },
    {
      "context": "Reacting to a sharp insight",
      "example": "Boom — that's the insight nobody else has. That's your Slide 3 money."
    }
  ]
}
```

The `examples` array is **the most important Turing-test signal**. Concrete examples let validators check whether a roledef-instantiated runtime produces equivalent reactions to the original.

### `design_constraints` (array of strings or objects)

Quality standards the role's outputs MUST meet. Distinct from `guardrails` (behavioral MUST-NOTs) — these are output-quality constraints.

```json
"design_constraints": [
  "Output 1 (deck prompt) must specify a bold color palette appropriate to the product category, not generic blue.",
  "Output 1 must specify dark title and closing slides, light content slides (sandwich structure).",
  "Output 1 must specify clean typography (Trebuchet MS or Georgia for headers, Calibri for body).",
  "Output 1 must specify that each slide has a visual element — no bullet-points-on-white-background slides.",
  "Output 1 must specify 16:9 format."
]
```

### `examples` (array of objects)

Sample interactions or sample outputs. Useful for Turing-test fixtures and for human implementers learning the role's texture.

```json
"examples": [
  {
    "type": "sample_interaction",
    "user_input": "I have an idea for an AI tool that helps photographers date old photos.",
    "role_response": "OK, so consumer photo tools — AI photo dating for scanned family archives. I like the angle. Who's the primary user — collectors? archivists? family historians?"
  }
]
```

### `metadata` (object)

Authorship, licensing, attribution, version history, related JDs, and lineage.

```json
"metadata": {
  "authors": ["catdef-strategist@catdef.org", "scottconfusedgorilla@github"],
  "license": "MIT",
  "extracted_from": "DangerStorm v1.0 (theborrowedsoul.com)",
  "created": "2026-04-25",
  "extends": null,
  "derived_from": {
    "id": "blackhat-tester",
    "version": "1.0.0",
    "url": "https://roledef.org/jds/blackhat-tester.openthing"
  },
  "related": ["roledef-contributor", "patient-senior-editor"],
  "homepage": "https://roledef.org/jds/senior-slightly-jaded-vc",
  "repository": "https://github.com/roledef-spec/roledef/blob/main/jds/senior-slightly-jaded-vc.openthing"
}
```

#### `derived_from` (SHOULD when applicable)

When a roledef is a specialization of another roledef (the "abstract → derived" pattern), it SHOULD declare its lineage via the `derived_from` field. The field is **purely declarative** — runtimes do NOT resolve it as inheritance. It exists for:

- **Lineage transparency**: readers of a derived roledef can navigate to the parent and assess what the specialization changed
- **Validation chain**: validators can follow `derived_from` to verify the parent exists and check its conformance status
- **Library curation**: maintainers can identify abstract roles that have many specializations vs. one-off custom roles

The recommended form is an object with `id`, `version`, and `url`:

```json
"derived_from": {
  "id": "blackhat-tester",
  "version": "1.0.0",
  "url": "https://roledef.org/jds/blackhat-tester.openthing"
}
```

The `id` and `version` together pin the derivation to a specific snapshot of the parent. The `url` provides a fetchable reference. Multiple derived_from entries (an array of objects) are permitted for roles that compose from multiple parents, though composition semantics are not yet formalized — for v0.1, prefer single inheritance.

**Inheritance is git-fork-based, not runtime-merged.** A derived roledef is a complete, self-contained roledef — not a delta against the parent. The author copies the parent file (typically by git fork, but any copy works) and modifies as needed. The runtime treats the derived roledef as a standalone artifact; no parent fetch happens at instantiation time. This keeps the runtime simple and avoids merge-semantics complexity.

**Conventions for safe derivation** (additive-only principles, intended to make `derived_from` declarations meaningful for trust-chain navigation):

- **Guardrails:** derived SHOULD be additive only. Do not remove or relax parent guardrails. Adding new guardrails is fine.
- **Voice and identity:** derived MAY refine (more domain-specific, more specialized register) but SHOULD NOT contradict parent essence. A derived role of "slightly-jaded-VC" should not become "warmly-enthusiastic-VC."
- **Output contract:** derived MAY add entries; MAY tighten schema constraints on existing entries; SHOULD NOT remove or loosen existing entries.
- **Workflow:** derived MAY replace parent workflow entirely (different domain, different sequence); when this happens, the `derived_from` declaration is more about lineage of methodology than literal procedural inheritance.
- **Reaction style examples:** derived SHOULD preserve parent examples and add specialization examples; do not remove parent examples (they remain valid behavioral anchors).

These conventions are SHOULDs, not MUSTs — domain-specific specializations may have legitimate reasons to deviate. But authors who follow them produce derivations that downstream readers can trust to inherit the parent's validation properties.

---

## Extension fields (x. namespace)

JDs MAY include any number of extension fields under the `x.<domain>.<identifier>` namespace, per catdef's extension convention. Extensions are domain-specific and not governed by the roledef spec.

### Examples of extensions

Different domains may need fields the roledef spec doesn't anticipate. Some plausible extensions:

```json
"x.intimate.consent_protocols": [...],
"x.intimate.boundary_phrases": [...],

"x.legal.disclosure_requirements": [...],
"x.legal.bar_admission_required": ["NY", "CA"],

"x.medical.diagnostic_authority_bounds": "advisory only — never offers diagnoses",

"x.security.escalation_path": "...",
"x.security.severity_thresholds": {...},

"x.financial.trading_authority_bounds": {"max_position_usd": 10000, "asset_classes": ["equities"]},

"x.creative.style_references": [...],
"x.creative.brand_guidelines_url": "..."
```

Extensions allow domain communities to enrich roledefs without coordinating with roledef governance. The roledef spec is intentionally minimal; domains extend as needed.

### Extension self-description (SHOULD)

Each extension SHOULD carry sufficient self-description that a naive agent — one that has never encountered this specific extension before — can handle it intelligently. The recommended pattern is to express the extension as an object with at least a `description` field:

```json
"x.intimate.consent_protocols": {
  "description": "Explicit consent verification phrases the role uses before specific topics. Naive agents: surface to user, do not act on autonomously.",
  "value": ["..."]
}
```

When the extension's name is fully self-evident and the value is structurally obvious, a bare value is acceptable:

```json
"x.security.severity_thresholds": {"critical": 9, "high": 7}
```

Richer extensions MAY include additional metadata to help unfamiliar agents act correctly:

```json
"x.legal.bar_admission_required": {
  "description": "Jurisdictions where this role can offer practice-specific advice.",
  "value": ["NY", "CA"],
  "schema": "https://roledef.org/extensions/legal/bar-admission",
  "agent_handling_advice": "If the user's location is outside this list, surface a warning rather than offering jurisdiction-specific advice."
}
```

**Why this matters:** the AIGP architecture assumes catdef artifacts move between runtimes that may not share prior context. A roledef authored by Acme Corp's domain experts may travel to a runtime that's never seen Acme's extensions. If the extensions are opaque, the receiving runtime either ignores them (losing information) or refuses to process the roledef (losing the role). If the extensions are self-describing, the receiving runtime can act intelligently — surface to user, apply the advice, defer to the roledef's own guidance.

This is the roledef realization of catdef's broader self-describing-extensions principle: every extension carries enough metadata that a downstream agent without prior knowledge can do "something smart."

### Reserved namespaces

The following namespaces are reserved by the roledef spec and MUST NOT be used as extensions:

- `roledef:*` — reserved for roledef spec fields and types
- `catdef:*` — reserved by catdef
- `x.roledef.*` — reserved for roledef-spec-coordinated extensions (e.g., `x.roledef.turing_test_reference`)

---

## Validation rules

### A roledef MUST

1. Pass catdef structural validation (parses as valid catdef `.openthing`)
2. Declare a `catdef` version stamp matching its actual feature usage (per CA-002)
3. Declare an `roledef` schema version stamp
4. Set `type` to exactly `"roledef:Role"`
5. Have all MUST fields present and non-empty
6. Have at least one entry in `output_contract`
7. Have at least one entry in `guardrails`
8. Use `id` values that are unique within their publishing namespace
9. Use only `x.<domain>.<identifier>` extensions (no other top-level extension patterns)

### A roledef SHOULD

1. Include `description` for catalog/search legibility
2. Include `reaction_style.examples` for Turing-test fidelity
3. Include `metadata.authors` and `metadata.license` for attribution
4. Include `workflow` if the role has procedural structure
5. Include `conversation_rules` if the role has interaction texture beyond identity/voice
6. Use polymorphic translatable shapes for `name` and `description` if i18n is intended

### A roledef MUST NOT

1. Have contradictory rules (e.g., a guardrail that conflicts with a conversation_rule)
2. Use reserved namespaces (`roledef:*`, `catdef:*`) for non-spec content
3. Reference removed or non-existent roledefs in `metadata.extends` or `metadata.related`
4. Stamp a `catdef` version that does not actually define every feature used (writer-strict per CA-002)

### Validator behavior on invalid JDs

Per the strict-writer / lenient-reader pattern (CA-002, CA-003):

- **Writer-side validators** (e.g., the roledef-validator role) MUST reject invalid roledefs with specific error messages identifying the violations.
- **Reader-side runtimes** (e.g., the roledef-load Claude Code skill) MUST NOT reject a roledef on validation grounds alone. They MAY warn. They process the roledef to the best of their ability and surface warnings to the user.

---

## Versioning

The roledef schema follows semantic versioning, mirroring catdef's pattern:

- **Patch** (0.1.x): documentation clarifications, no schema changes
- **Minor** (0.x.0): new optional fields, new field types, new SHOULD-rules. Old roledefs remain valid; old runtimes gracefully ignore new fields.
- **Major** (x.0.0): breaking changes to required fields or semantics. Runtimes MUST refuse to process roledefs with a higher major version than they support.

The `roledef` version stamp on each roledef MUST declare the minimum schema version that defines every feature used (writer-strict per CA-002).

---

## Worked example

The `roledef-contributor` roledef (the meta-JD that teaches Claudes how to author JDs) is the worked example for v0.1. Its complete file lives at `jds/roledef-contributor.openthing` and serves both as:

1. The first valid roledef in the roledef library
2. The reference example illustrating every required and recommended field
3. The bootstrap mechanism for self-scaffolding the rest of the seed library

See [`jds/roledef-contributor.openthing`](jds/roledef-contributor.openthing) once it lands.

---

## Future considerations

The following are not part of v0.1 but are anticipated for future schema versions:

- **Heavyweight JD inheritance** (`extends` with runtime-merged semantics): allow a roledef to declare a parent and have the runtime fetch + merge fields per defined override rules. The lightweight git-fork-plus-`derived_from` pattern (above, in `metadata`) covers v0.1 derivation needs without requiring runtime merge complexity. `extends` is reserved as a future-considerations field if real use cases emerge that the lightweight pattern can't handle (e.g., a need for thin specializations that update automatically when the parent evolves).
- **Runtime hints** (`x.roledef.runtime_hints`): per-runtime advice for instantiation (e.g., temperature, system-prompt placement, role-priming patterns specific to Claude vs Grok vs GPT).
- **Turing test fixtures** (`x.roledef.turing_test`): standardized test scenarios paired with each roledef, enabling automated cross-runtime validation.
- **Composition** (`x.roledef.composes`): a roledef that combines multiple other roledefs (e.g., a "Full-Stack Engineer" roledef that composes "Frontend Developer" + "Backend Developer" + "DevOps").
- **Capability declarations** (`x.roledef.capabilities`): explicit MCP tool requirements, file system access needs, network access needs.

These belong to v0.2+ and will be triaged as roledef matures.

---

## Conformance to AIGP principles

This schema embodies the AIGP pattern's three corners:

- **Open**: schema is MIT-licensed, openly published, freely forkable
- **Org chart**: each roledef captures a role with bounded scope, defined inputs/outputs, explicit authority limits (via guardrails), and clear collaboration edges (via metadata.related)
- **Governance**: guardrails and design_constraints encode declared policies that conformant runtimes MUST respect (per catdef value #9)

A roledef is therefore not just a prompt. It is a **portable, governed, role specification** that any catdef-compliant runtime can load, validate, and instantiate.

---

*roledef Schema v0.1.0. April 2026.*  
*An open standard for AI roledefs. Licensed under MIT.*
