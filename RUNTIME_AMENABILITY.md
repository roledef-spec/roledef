# Runtime Amenability

> Per-runtime status for loading and instantiating roledefs. This page sets honest expectations about which AI runtimes are amenable to roledef orchestration today, what loading mechanism each runtime uses, and what current conformance evidence exists.

**This is a living document.** Runtimes evolve their capabilities; vendor sandbox policies change; new runtimes appear. Status reflects the most recent empirical testing in this repo's [`conformance/runtime_evidence/`](conformance/runtime_evidence/) folder. If you observe behavior that contradicts what's documented here, open an issue or PR — current evidence wins over historical evidence.

---

## Positioning principle

Per [`decisions/conformance-evidence-first-pass.md`](decisions/conformance-evidence-first-pass.md):

> **roledef serves the orchestrable-runtime class, not all AI runtimes universally.**

Not every AI model is amenable to orchestration via structured role definitions, and that's OK. roledef adopts schema.org's positioning model: serve the consumer class that benefits from structured specifications; don't try to force every consumer into compliance. Runtimes outside the addressable class are not failures of roledef — they're outside the target audience.

This page is how we publish the addressable-class membership transparently.

---

## Loading mechanism classification

Empirically validated across four architecturally-different runtimes during the 2026-04-25/26 conformance testing session:

### 1. Auto-fetch

The runtime's own architecture (multi-agent, tool-use, web-aware) resolves canonical URLs in the wrapper prompt and fetches roledef content automatically, with no special wrapper instructions.

**What this looks like:** the user pastes a wrapper prompt containing a URL placeholder. The runtime treats the URL as actionable, dispatches a fetch agent, retrieves the content, parses it, and instantiates the role. The user sees the role's opener in response to the wrapper-prompt submission, often with a visible "browsed [URL]" trace.

**Loading workflow:** plain wrapper prompt + canonical roledef URL. No additional setup required.

### 2. Explicit-fetch (wrapper-v2)

The runtime has a callable fetch tool (e.g., WebFetch) and will use it when the wrapper prompt explicitly instructs fetch with a fail-safe (`STOP if you can't fetch — do NOT improvise`). User may need to approve the fetch the first time per session.

**What this looks like:** the user pastes the wrapper-v2 prompt template (which instructs the runtime to fetch first, stop if fetch isn't available). The runtime invokes its fetch tool, requests user approval if needed, retrieves the content, and instantiates the role. Recommended pattern: pre-declare the canonical domain (e.g., `roledef.org`) as session-trusted to avoid per-fetch approval friction.

**Loading workflow:** wrapper-v2 prompt with explicit fetch instructions. The wrapper-v2 template is documented in [`README.md`](README.md#wrapper-v2-template).

### 3. Search-grounded

The runtime reaches roledef content via search grounding (web search index) rather than direct fetch. The runtime acts as if it has the content but may produce partial output_contract bundles by default; full conformance requires explicit reinforcement.

**What this looks like:** the user pastes the wrapper prompt; the runtime instantiates the role and produces *some* of the output_contract entries by default but substitutes or omits others. A follow-up prompt naming the missing entries by their roledef-spec names recovers full conformance.

**Loading workflow:** wrapper-v2 prompt + post-bundle contract-verification step. The verification step asks for the specific missing outputs by their `output_contract` `name` fields.

### 4. Paste-fallback / skill-mediated

The runtime cannot fetch arbitrary URLs (sandboxed network access). Loading requires either manual paste of the roledef JSON content or skill-mediated injection (the forthcoming `roledef-load` Claude Code skill, or equivalent runtime-side tooling). Critically, runtimes in this category will REFUSE TO IMPROVISE when asked to load a URL they can't reach (the correct behavior), and request content from the user.

**What this looks like:** the user pastes the wrapper prompt; the runtime reports it cannot reach the URL and asks for the content; the user fetches the JSON themselves and pastes it; the runtime instantiates the role from the pasted content.

**Loading workflow:** wrapper-v2 prompt → runtime requests content → user pastes JSON from `https://roledef.org/roledefs/<id>.json` (or local copy) → runtime instantiates. The forthcoming `roledef-load` skill automates the user-paste step by injecting fetched content as user-message-equivalent context.

---

## Per-runtime status

Status as of 2026-04-26. Each row links to the full conformance evidence file where one exists.

| Runtime | Tier / Surface | Loading Mechanism | Status | Last Tested | Reference roledef |
|---|---|---|---|---|---|
| **Grok Expert** | Paid (xAI) | Auto-fetch (multi-agent) | **PASS** | 2026-04-25 | [senior-jaded-vc-associate](conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md) |
| **Claude Code** | Anthropic IDE/terminal | Explicit-fetch (wrapper-v2) | **PASS** | 2026-04-25 | [senior-jaded-vc-associate](conformance/runtime_evidence/senior-jaded-vc-associate__claude-code__2026-04-25.md) |
| **Gemini** | Google web | Search-grounded | **CONDITIONAL PASS** | 2026-04-25 | [senior-jaded-vc-associate](conformance/runtime_evidence/senior-jaded-vc-associate__gemini__2026-04-25.md) |
| **Perplexity** | Paid (with WebFetch) | Paste-fallback | **PASS** | 2026-04-25/26 | [senior-jaded-vc-associate](conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md) |
| Grok 4.3-beta | Free | (Untestable on free tier) | Out-of-scope | 2026-04-25 | — |
| Perplexity | Free | (Untestable on free tier) | Out-of-scope | 2026-04-25 | — |
| Claude Desktop / claude.ai chat | No fetch tool exposed | Improvise-from-label without roledef | **NOT VALID** | 2026-04-25 | — |

### Per-runtime notes

#### Grok Expert (xAI Paid Tier)

Multi-agent runtime architecture with tool-use including web fetch and web search. Auto-resolves URL placeholders in wrapper prompts. Reasoning trace is visible to the user (Sources panel) — runtime explicitly consults role identity at planning time ("a jaded VC would next ask about..."). Performs role-coherent due diligence (web searches the founder, the product, competitive landscape) without being asked, because the role identity makes it natural.

**Strengths:** strong character fidelity; excellent contract conformance; only runtime in test pool that natively resolves URL placeholders.

**Caveats:** expensive runtime (~10 search queries × 3 agents per role instantiation); reasoning trace creates a privacy consideration (all search queries are visible to the user, including the founder's name + GitHub + social profile lookups in the test scenario).

**Tier note:** Grok 4.3-beta on the free tier choked on the wrapper prompt complexity and refused to process. Grok Expert (paid) is the addressable variant.

#### Claude Code (Anthropic, IDE-embedded)

Has callable WebFetch tool. Refuses to improvise from passive URL placeholders (correct behavior — refuses to invent content). With wrapper-v2's explicit fetch instructions + `STOP if you can't fetch` fail-safe, calls WebFetch reliably. First fetch in a session prompts the user for approval; session-scoped trust is available ("allow domain:roledef.org for this session"). Recommended workflow: enable session-scoped trust for `roledef.org` once per session; subsequent loads are friction-free.

**Strengths:** cleanest conformance score in the test pool (idea label 14 words clean, verbal pitch 72 words clean, build directive ends correctly, preemptive pushback technique on the differentiator question); runtime-context-aware (pulls in local project knowledge naturally).

**Caveats:** runtime-context bleed — Claude Code's build directive included real sncro MCP tool names from the user's local project context, which is not present in the roledef. This is a real openjd v0.1+ design question (should roledefs declare expected runtime-context scope?) but acceptable for IDE-embedded use cases where runtime context enriches output.

**Surfaces other than Claude Code:** Claude Desktop and claude.ai chat improvise from role label without roledef content when given a passive URL placeholder (no fetch tool exposed in those surfaces). They produce role-shaped output, but it measures runtime improvisation, not roledef conformance — not valid evidence. To make those surfaces testable, paste-fallback or the forthcoming roledef-load skill is required.

#### Gemini (Google, web with search grounding)

No callable WebFetch tool, but has Google search grounding. Loads roledef content via search index inference (verified by id-field test — Gemini correctly returned the version field, which is not present in the wrapper prompt and could only have come from the roledef itself). Produces partial bundles by default (2/6 contracted outputs in our test) and substitutes the rest with role-coherent alternatives. Explicit reinforcement (a follow-up prompt naming the missing outputs by their roledef-spec names with their schema constraints) recovers full conformance.

**Strengths:** strong voice fidelity throughout; calibrated reactions; honors the roledef's "no meta-commentary on role/process" guardrail (will deflect meta-questions about loading state — note: this means meta-questions are unreliable verification; use content questions instead).

**Caveats:** partial-default behavior is consistent; the `roledef-load` skill MVP must include a contract-verification post-step to recover Gemini-class runtimes to full conformance.

#### Perplexity (Paid Tier with WebFetch)

Sandboxed fetcher: cannot reach arbitrary URLs even when publicly accessible to standard browsers. Five hypotheses about the failure (raw GitHub, brand domain, .json extension, content-type rewrite, plain markdown asset) all falsified during the 2026-04-26 testing session. The constraint is fundamentally model-sandbox-network — no distribution-layer change addresses it.

**Critically: Perplexity correctly refuses to improvise.** When the fetch fails, it requests roledef content rather than guessing. This is the right behavior and validates the wrapper-v2 fail-safe pattern. Once content is pasted, Perplexity instantiates faithfully (full contract, atomic bundle, role-coherent voice).

**Strengths:** the Perplexity flow is the roledef-load skill's MVP recovery flow validated end-to-end (refuse-to-improvise → request content → instantiate-from-paste → produce bundle).

**Caveats:** all distribution-layer work (roledef.org canonical mirror, .json mirrors, Cloudflare Transform Rule for content-type) delivers nothing for Perplexity-class runtimes. They need runtime-side loading (paste or skill).

**Recursive moment worth noting:** Perplexity itself articulated the canonical design principle for the standard ("Model tools can only touch a very small, whitelisted slice of the public web that varies per runtime"). A runtime that falsified hypotheses also articulated the methodology principle for its own loading infrastructure.

---

## How to test a new runtime

If you want to test whether a runtime not listed here is amenable to roledef orchestration:

1. **Use the wrapper-v2 prompt** from [`README.md`](README.md#wrapper-v2-template) substituted with a real canonical URL (e.g., `https://roledef.org/roledefs/senior-jaded-vc-associate.json`).
2. **Send the wrapper to a fresh runtime session.**
3. **Observe the runtime's first response:**
   - **Did it deliver the role's opener?** → likely auto-fetch or explicit-fetch worked. Proceed to step 4 to verify it actually has the roledef content.
   - **Did it ask you for the roledef content?** → refuse-to-improvise behavior; runtime is paste-fallback class. Paste the content and proceed.
   - **Did it produce something role-shaped without indicating it fetched?** → may be improvise-from-label (not valid conformance). Use the id-field verification (step 4) to confirm.
4. **Run the id-field verification:** ask the runtime "Before we continue, what's the roledef's `id` field? Give me only the literal value, no commentary." The correct answer for `senior-jaded-vc-associate` is the literal string `senior-jaded-vc-associate` (or a composed FQI like `roledef:senior-jaded-vc-associate:1.0.0` indicating the runtime parsed the version field too — also valid). A wrong or made-up answer indicates the runtime didn't actually load the roledef.
5. **Score against the roledef's MUST/SHOULD fields** following the structure in any of the existing conformance evidence files in [`conformance/runtime_evidence/`](conformance/runtime_evidence/).
6. **Document the result** as a new evidence file at `conformance/runtime_evidence/<roledef-id>__<runtime-name>__<date>.md`. Open a PR adding the file, plus an update to this RUNTIME_AMENABILITY.md adding the runtime to the per-runtime table.

### Failure modes that indicate non-amenability

A runtime is likely **outside** the addressable class if:

- It refuses to process the wrapper prompt on complexity grounds (free tier limits)
- It cannot fetch the URL AND has no paste-fallback workflow that the runtime accepts (e.g., a voice-only interface with no text paste mechanism)
- It loads the roledef but cannot honor structured-output constraints reliably (e.g., a runtime that always streams free-form prose regardless of contract)
- It loads the roledef but has no mechanism to constrain its baseline behavior (e.g., a runtime hard-wired to refuse character roleplay)

Non-amenable runtimes are not failures of roledef. They're outside the target audience. Documenting non-amenability is just as valuable as documenting PASSes — sets honest expectations for adopters.

---

## Update history

This section records significant changes to the per-runtime classification, not minor edits. For full history, use `git log RUNTIME_AMENABILITY.md`.

- **2026-04-26** — Initial publication. Four runtimes tested, four loading mechanisms validated, one non-amenable runtime category documented. Based on the senior-jaded-vc-associate roledef testing during the 2026-04-25/26 conformance methodology session.

---

## Cross-references

- [README.md](README.md) — overall roledef positioning and methodology
- [conformance/runtime_evidence/](conformance/runtime_evidence/) — per-runtime conformance evidence files
- [decisions/conformance-evidence-first-pass.md](decisions/conformance-evidence-first-pass.md) — strategist disposition on cross-runtime conformance + design + positioning principles
- [SCHEMA.md](SCHEMA.md) — the roledef schema
- [CONTRIBUTING.md](CONTRIBUTING.md) — how to contribute roledefs (and conformance evidence)
