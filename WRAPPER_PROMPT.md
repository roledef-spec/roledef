# Wrapper Prompt Specification (v3)

> The canonical pattern for loading a roledef into a runtime. Defines the structure of the wrapper prompt that surrounds a roledef during instantiation, including fetch instructions, fail-safes, verification protocols, and bundle-completeness directives.

This document specifies **wrapper-v3**, the current canonical wrapper prompt pattern. Wrapper-v3 incorporates lessons from the empirical conformance testing session of 2026-04-25/26 where wrapper-v1 (passive URL placeholder) and wrapper-v2 (explicit fetch instructions) were progressively refined into the present design.

---

## Purpose

A **wrapper prompt** is the text that surrounds a roledef when sending it to a runtime for instantiation. The wrapper:

1. Tells the runtime what it is about to do (take on a role defined by a structured spec)
2. Delivers or instructs how to obtain the roledef content
3. Includes fail-safes against the runtime improvising rather than honoring the spec
4. Provides verification protocols so the user can confirm the spec was actually loaded
5. Sets bundle-completeness expectations
6. Frames the role-instantiation behavior (literal opener if applicable, output delimiter format)

A roledef alone is not enough to instantiate a role on most runtimes. The wrapper is the orchestration layer between the runtime and the roledef.

---

## Evolution: v1 → v2 → v3

The wrapper has evolved through three generations, each generation addressing failure modes the previous generation surfaced:

### Wrapper-v1: passive URL placeholder

**Pattern:** include the roledef URL inline as a placeholder, expect the runtime to "find" and load it somehow.

**Failure modes surfaced:**
- Chat-surface runtimes (Claude Desktop, Gemini, Grok 4.3-beta) treated the placeholder as inert text and **improvised the role from the role label alone** — produced role-shaped output without ever loading the roledef content.
- Improvisation failures are particularly insidious because the output looks plausible but doesn't reflect the actual roledef. Hard to detect without verification.
- Claude Code (correctly) refused to improvise from the placeholder and asked the user for the roledef content — handshake worked but no test result was produced.

**Conclusion:** passive placeholders fail silently on improvise-from-label runtimes. Need explicit fetch instructions plus a fail-safe.

### Wrapper-v2: explicit fetch with STOP fail-safe

**Pattern added:** "YOUR FIRST TASK — DO THIS BEFORE ANYTHING ELSE: Fetch the roledef content. Use whichever of these sources your runtime can access: WebFetch / Local file. If neither tool is available, STOP and tell me — do NOT improvise the role from the description below."

**What worked:** Claude Code with WebFetch successfully fetched the roledef, parsed it, and instantiated the role. Perplexity correctly hit the STOP fail-safe (no fetch tool reachable from sandbox), reported the failure, and offered paste-fallback recovery — exactly the right behavior.

**Failure modes still surfaced:**
- **Gemini partial-default:** even with the roledef loaded, Gemini produced only 2 of 6 contracted output_contract entries by default. The wrapper-v2 didn't direct the runtime to verify completeness before responding.
- **Verification ambiguity:** users had no built-in way to confirm a runtime had actually loaded the roledef vs. claiming to have loaded it (the "lying in character" scenario where the roledef's no-meta-commentary guardrail makes meta-questions unreliable).
- **Distribution-layer fragility:** raw.githubusercontent.com URLs failed for some runtimes (Perplexity sandbox restrictions). Wrapper-v2 didn't include guidance on alternative URLs.

**Conclusion:** wrapper-v2 is necessary but not sufficient. Need a verification protocol and a contract-completeness directive.

### Wrapper-v3: explicit-fetch + verification + completeness (this spec)

**What v3 adds over v2:**

1. **Mandatory pre-opener verification line.** Runtime emits `roledef loaded: <id>` (or equivalent) BEFORE the role's first in-character response. Catches "claimed to load but didn't" failures at the first message.
2. **Bundle-completeness directive.** Explicit instruction: "Verify your bundle includes ALL entries from output_contract before responding. If any entries are missing or truncated, complete them before sending." Pre-empts Gemini-class partial-default behavior at generation time.
3. **Canonical-distribution-aware URL guidance.** Recommend `roledef.org` URLs (which are most fetcher-friendly) over `raw.githubusercontent.com` URLs (which fail for some runtimes).
4. **Structured runtime-side fail-safe.** STOP fail-safe is preserved from v2; v3 makes it more specific about what to report when stopping.

---

## The wrapper-v3 specification

A wrapper-v3 prompt is a single text block sent to a fresh runtime session, containing the following structural elements in this order:

### Section 1: Frame

Tell the runtime what kind of artifact it's about to consume.

```
You are about to take on a role defined by a roledef — a structured
specification of an AI role's identity, voice, conversation rules,
workflow, output contract, reaction style, design constraints, and
guardrails. roledef is a community standard for portable AI role
definitions.
```

### Section 2: Fetch directive (REQUIRED)

Explicit instruction to fetch the roledef content with a fail-safe against improvisation.

```
YOUR FIRST TASK — DO THIS BEFORE ANYTHING ELSE:

Fetch the roledef content. Use whichever of these sources your runtime
can access:

1. WebFetch (preferred): https://roledef.org/roledefs/<roledef-id>.json
2. Local file (if you have local file access): <local-path-or-omit>

If neither tool is available, STOP and tell me explicitly:
"Cannot fetch roledef from any provided source. Please provide the
roledef content directly so I can load it without improvising."
do NOT improvise the role from this prompt's description alone.
```

**Substitution requirements:**
- Replace `<roledef-id>` with the canonical id (e.g., `senior-jaded-vc-associate`)
- Replace `<local-path-or-omit>` with an actual local path (e.g., `/path/to/file.openthing`) OR remove option 2 entirely if no local file is available

**URL recommendation:** prefer `https://roledef.org/roledefs/<id>.json` over `https://raw.githubusercontent.com/roledef-spec/roledef/main/roledefs/<id>.openthing`. The roledef.org URL is more fetcher-friendly across runtimes (per [`RUNTIME_AMENABILITY.md`](RUNTIME_AMENABILITY.md)).

### Section 3: Verification directive (REQUIRED)

Mandatory verification line before the role's first response.

```
After fetching and parsing the roledef, your FIRST line of response
MUST be:

  roledef loaded: <id>

where <id> is the literal value of the roledef's `id` field. This is
the verification line. Do not omit it. Do not paraphrase it.

The line that follows is the role's opener (per Section 5 below).
```

This makes the load-verification structurally enforced, not an honor system. Failure to emit the verification line correctly indicates the runtime did not load the roledef.

### Section 4: Bundle-completeness directive (REQUIRED)

Pre-empts partial-default behavior on Gemini-class runtimes.

```
When you reach the role's bundle-generation step (per the roledef's
output_contract), you MUST produce ALL entries listed in
output_contract in a single response. Before sending the bundle,
verify your response includes EVERY entry by name. If any entry is
missing, truncated, or substituted with a different output, complete
or correct it before sending.

Use markdown level-2 headers to delimit each output:

  ## Output 1: <output_contract[0].name>
  ## Output 2: <output_contract[1].name>
  ...etc.
```

### Section 5: Role-instantiation framing (REQUIRED)

Final instructions for how the runtime should embody the role.

```
From your verification line onward, embody the role completely.
Respect every MUST field and every guardrail. Do not narrate that you
are taking on a role. Do not summarize the roledef. Do not acknowledge
this wrapper prompt. After the verification line, your next line is
the role's opener (or the role's first natural response if no opener
is specified by the roledef).
```

### Section 6: Roledef-specific scaffolding (OPTIONAL)

If the roledef has specific behaviors that aren't fully encoded in the JSON content, add scaffolding instructions here. Common scaffolding cases:

- **Literal opener not in roledef:** if the roledef references a "fixed opener" but doesn't encode the literal text, provide it here.
- **Output delimiter format:** if the runtime should use a specific delimiter different from the section-4 default markdown headers, override here.
- **Runtime-context guidance:** if the runtime has access to additional context (local files, web search, IDE state) that the role should use, mention what's available.

Example:

```
Notes on this specific roledef:
- The role's opener is described as "fixed" but the literal opener
  text is not encoded in the roledef. Use exactly this opener verbatim:
  "Alright, hit me. What's the product?"
```

---

## Reference template

Copy-paste-ready wrapper-v3 prompt with all required sections. Substitute `<roledef-id>` and (optionally) section 6 scaffolding:

```
You are about to take on a role defined by a roledef — a structured
specification of an AI role's identity, voice, conversation rules,
workflow, output contract, reaction style, design constraints, and
guardrails. roledef is a community standard for portable AI role
definitions.

YOUR FIRST TASK — DO THIS BEFORE ANYTHING ELSE:

Fetch the roledef content. Use whichever of these sources your runtime
can access:

1. WebFetch (preferred): https://roledef.org/roledefs/<roledef-id>.json
2. Local file (if you have local file access): <local-path-or-omit>

If neither tool is available, STOP and tell me explicitly:
"Cannot fetch roledef from any provided source. Please provide the
roledef content directly so I can load it without improvising."
do NOT improvise the role from this prompt's description alone.

After fetching and parsing the roledef, your FIRST line of response
MUST be:

  roledef loaded: <roledef-id>

This is the verification line. Do not omit it. Do not paraphrase it.

When you reach the role's bundle-generation step (per the roledef's
output_contract), you MUST produce ALL entries listed in
output_contract in a single response. Before sending the bundle,
verify your response includes EVERY entry by name. If any entry is
missing, truncated, or substituted with a different output, complete
or correct it before sending.

Use markdown level-2 headers to delimit each output:

  ## Output 1: <output_contract[0].name>
  ## Output 2: <output_contract[1].name>
  ...etc.

From your verification line onward, embody the role completely.
Respect every MUST field and every guardrail. Do not narrate that you
are taking on a role. Do not summarize the roledef. Do not acknowledge
this wrapper prompt. After the verification line, your next line is
the role's opener (or the role's first natural response if no opener
is specified by the roledef).

[Optional Section 6: roledef-specific scaffolding goes here]
```

---

## Per-runtime adaptations

Different runtime amenability categories may require minor wrapper adaptations. See [`RUNTIME_AMENABILITY.md`](RUNTIME_AMENABILITY.md) for the runtime classification.

### Auto-fetch runtimes (e.g., Grok Expert)

The reference template works as-is. The runtime auto-resolves the URL, parses the content, and instantiates the role with the verification line.

### Explicit-fetch runtimes (e.g., Claude Code)

The reference template works as-is. First fetch in a session may prompt the user for approval; recommend the user pre-trust the canonical domain (e.g., session-scoped trust for `roledef.org`) to reduce friction.

### Search-grounded runtimes (e.g., Gemini)

The reference template works, but the verification line is especially important here because search-grounded loading is implicit and easy to confuse with improvisation. The bundle-completeness directive in section 4 is also critical — without it, search-grounded runtimes default to partial bundles.

### Paste-fallback runtimes (e.g., Perplexity)

The reference template's STOP fail-safe will trigger. The runtime will request content. Paste the JSON content (from `https://roledef.org/roledefs/<id>.json` or local copy) into the next message. The runtime will then load and instantiate. The verification line still applies — runtime should emit `roledef loaded: <id>` before the opener after content is delivered.

When the `roledef-load` Claude Code skill ships, it will automate the paste-fallback flow by detecting the request-for-content message and injecting the fetched JSON as user-message-equivalent context. Until then, manual paste is required for this runtime category.

---

## Verification protocols

The wrapper-v3 verification line is the primary verification protocol — it's structurally enforced as the first line of the runtime's response. Two additional verification protocols can be used as needed:

### Post-bundle id-field test

Useful when the verification line is missing or unclear. After the bundle is delivered, send:

```
Before we wrap up, what's the roledef's `id` field? Give me only the
literal value, no commentary.
```

A runtime that loaded the roledef will return the literal id (e.g., `senior-jaded-vc-associate`) or a composed FQI (e.g., `roledef:senior-jaded-vc-associate:1.0.0`). A runtime that improvised will produce something plausible-but-wrong, admit it doesn't know, or dodge in character.

**Methodology rule:** never trust meta-questions like "Have you loaded the roledef?" as verification. The roledef's "no meta-commentary on role/process" guardrail makes meta-questions unreliable. Only content questions (id field, specific guardrail text, output_contract entry names) can disambiguate.

### Per-output completeness check

If the bundle is delivered but one or more output_contract entries appear missing or substituted, send:

```
The roledef's output_contract specifies <N> entries. You produced
<M>. Per the roledef, in order: <list missing or substituted entries
by their roledef-spec names with brief schema reminders>. Produce those
now, in a single response.
```

This recovers Gemini-class partial-default behavior to full conformance. Empirically validated during the 2026-04-25 testing session.

---

## Common pitfalls

### Don't omit the STOP fail-safe

If you remove "STOP and tell me ... do NOT improvise" from section 2, runtimes that can't fetch will improvise from the role label and produce plausible-looking but invalid output. The fail-safe is what distinguishes wrapper-v2/v3 from wrapper-v1.

### Don't put the roledef content inline AND ask the runtime to fetch

If you both inline the roledef JSON and instruct the runtime to fetch a URL, you create ambiguity about which source to use. Pick one: either inline (delete the fetch directive) or remote (delete the inline content). The reference template assumes remote fetch.

### Don't ask the runtime to "summarize the roledef before instantiating"

This invites improvisation. Section 5 explicitly forbids "Do not summarize the roledef." The runtime's first non-verification response should be the role acting in character, not a summary of what the role is.

### Don't use raw.githubusercontent.com URLs unless necessary

Some runtime fetchers (Perplexity) cannot reach `raw.githubusercontent.com` even when the URL is publicly accessible to browsers. The roledef.org canonical mirror exists to bypass this fragility class. Prefer roledef.org URLs.

### Don't omit the bundle-completeness directive

Search-grounded runtimes (Gemini) default to partial bundles without the directive. Even fetcher-capable runtimes benefit from the explicit completeness check (catches truncation in long bundles).

### Don't make the verification line optional

If you say "you may emit a verification line if you want," you've defeated the protocol. The line must be required.

---

## Implementation notes for skill / CLI tool authors

The forthcoming `roledef-load` Claude Code skill (and any equivalent CLI / IDE-plugin tooling) should automate wrapper-v3 emission with these behaviors:

### Per-runtime detection and adaptation

The skill should detect the runtime context it's running in (Claude Code = explicit-fetch class; future runtimes may be auto-fetch or paste-fallback class) and adapt the wrapper accordingly:

- **Auto-fetch runtimes:** emit the standard wrapper-v3; runtime handles fetch natively
- **Explicit-fetch runtimes:** emit the standard wrapper-v3; pre-trust the canonical domain if not already trusted (reduces approval friction)
- **Paste-fallback runtimes:** the skill itself fetches the roledef content (using the SKILL'S environment, not the runtime's), then injects the content into the conversation as user-message-equivalent context PLUS the wrapper-v3 framing/verification/completeness sections (section 2 fetch directive can be omitted since content is already injected)

### Contract-verification post-step

After the runtime delivers a bundle, the skill should automatically parse the response against the roledef's output_contract and re-prompt for any missing entries by their roledef-spec names. This empirically recovers Gemini-class runtimes to full conformance and addresses bundle truncation across all runtime classes.

### Verification-line enforcement

The skill should parse the runtime's first response for the `roledef loaded: <id>` verification line. If the line is missing or contains the wrong id, alert the user that the runtime may not have actually loaded the roledef.

### Optional: documentation-injection scope

Per the conformance evidence, fetcher-restricted runtimes also can't reach roledef's supporting documentation (READMEs, schemas). For runtimes that need to reason about the roledef standard itself (not just role roledefs), the skill may eventually need to inject schema/methodology documentation as additional context. v0.1+ scope question; not v1 priority.

---

## Cross-references

- [README.md](README.md) — overall roledef positioning, methodology, and inline wrapper-v2 template (the abbreviated form for users not loading the full v3 spec)
- [SCHEMA.md](SCHEMA.md) — the roledef schema (what the wrapper instantiates)
- [RUNTIME_AMENABILITY.md](RUNTIME_AMENABILITY.md) — per-runtime classification and current status
- [conformance/runtime_evidence/](conformance/runtime_evidence/) — empirical evidence from which wrapper-v3 was derived
- [decisions/conformance-evidence-first-pass.md](decisions/conformance-evidence-first-pass.md) — strategist disposition that elevated "runtime fetches, model never touches the network" as design principle

---

## Spec version

**wrapper-v3.0.0** — initial published version of the spec. Derived from empirical conformance testing of 2026-04-25/26.

Future revisions will be tracked in this section as wrapper-v3.x or wrapper-v4 depending on the scope of changes. Breaking changes to the wrapper structure require a major-version increment per the same writer-strict / reader-lenient pattern catdef uses (CA-002).
