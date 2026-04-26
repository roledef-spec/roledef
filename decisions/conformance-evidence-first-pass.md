# Decision: First Runtime Conformance Evidence Recorded

**Disposition:** Recorded; updates downstream commitments
**Origin:** Strategist runtime testing session, 2026-04-25
**Decided:** 2026-04-25 by roledef-strategist
**Evidence files:**
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md) — PASS (auto-fetch loading)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__claude-code__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__claude-code__2026-04-25.md) — PASS (explicit-fetch loading via wrapper-v2)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__gemini__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__gemini__2026-04-25.md) — CONDITIONAL PASS (search-grounded loading; partial-default contract; full contract with explicit reinforcement)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md) — PASS (paste-fallback loading after fetch failure; demonstrates roledef-load skill's MVP recovery flow end-to-end)

## Disposition

The roledef canonical library has its **first end-to-end runtime conformance PASSes** on FOUR architecturally different runtimes: `senior-jaded-vc-associate.openthing` v1.0.0 instantiated on Grok Expert (xAI multi-agent web), Claude Code (Anthropic IDE-embedded), Gemini (Google web with search grounding), and Perplexity (sandboxed-fetch-restricted, paste-fallback recovery) on 2026-04-25. Full evidence in the four conformance/runtime_evidence/ files cited above.

**Cross-runtime portability is empirically validated.** The same roledef produces role-faithful behavior on three architecturally different runtimes via three different loading mechanisms (multi-agent fetch / explicit WebFetch / search-grounded inference). Tactical variance between runtimes (question order, pushback technique, atomic-vs-iterative bundle delivery, contract-completeness-by-default) falls within the roledef's constraint envelope. Voice fidelity is the most consistent element across all three runtimes; output_contract conformance varies and may require explicit reinforcement on partial-default runtimes (Gemini).

This decision artifact records the strategist disposition on all three pieces of evidence and updates downstream commitments that referenced "Turing test within 30 days" targets in prior decisions.

## What was tested, summarized

Five runtime/surface combinations were tested against the same wrapper prompt (URL placeholder for the roledef content, no inline JSON):

| Runtime/Surface | URL fetched? | Behavior origin | Conformance evidence |
|---|---|---|---|
| Claude Code (IDE-embedded) — wrapper v1 (passive placeholder) | No (refused to improvise) | **Correctly recognized unresolved placeholder; asked user for roledef content** | No test result — handshake correctly identified missing precondition |
| **Claude Code (IDE-embedded) — wrapper v2 (explicit fetch instructions)** | **YES (WebFetch after user approval)** | **JD-instantiated** | **PASS — voice, contract, workflow, pushback all conformant; see Claude Code evidence file. Cleanest score in the test pool: idea label 14 words / verbal pitch 72 words / build directive ends correctly / preemptive pushback / runtime-context-aware build directive** |
| Claude chat surface (Desktop or claude.ai) | No | Improvised from role label | Voice over-aggressed; output_contract substituted 5/7 |
| **Gemini** | **YES (search-grounded inference; verified via id-field test)** | **JD-instantiated; partial-by-default contract behavior** | **CONDITIONAL PASS — produced 2 of 6 contract entries by default, full 6 with explicit reinforcement; voice fidelity excellent throughout. See Gemini evidence file.** |
| Grok 4.3-beta | No | Improvised from role label | Voice smoothed (sycophantic); 6 outputs in roledef spec order by coincidence; no pushback |
| **Grok Expert** | **YES** | **JD-instantiated** | **PASS — voice, contract, workflow, pushback all conformant; see evidence file** |

The cross-runtime variance is now understood to be primarily a **JD distribution** issue, not a roledef design issue. Runtimes split into three categories of behavior when handed a wrapper prompt with an unresolved URL placeholder:

1. **Auto-fetch (Grok Expert):** runtime fetches the roledef URL natively, instantiates faithfully. Valid conformance test result.
2. **Refuse-to-improvise (Claude Code):** runtime correctly recognizes the placeholder is unresolved and asks the user for the roledef content rather than guessing. NOT a failed test — a correctly-identified missing precondition. With roledef content provided (via paste or skill), this category becomes testable.
3. **Improvise-from-label (chat surfaces — Claude chat, Gemini, Grok 4.3-beta):** runtime treats the placeholder as inert text and produces output based on the role label and wrapper hints. NOT a valid conformance test result; what's measured is the runtime's improvisation quality, not the roledef's portability.

The earlier "Claude failed conformance" finding from this session was incorrect — the test was on a chat surface that improvised, not on Claude Code which refused to improvise. **Claude Code's refusal-to-improvise is the right behavior** and simplifies the roledef-load skill's design: the skill's job is to provide the missing precondition (JD content) that Claude Code is already correctly waiting for.

The wrapper-prompt v2 design (explicit fetch instructions, "STOP if you can't fetch — do NOT improvise") was created to bridge the refuse-to-improvise gap. Empirically validated: Claude Code with v2 wrapper fetched and instantiated faithfully, producing a conformance PASS comparable to (and in some metrics cleaner than) Grok Expert's. The v2 wrapper pattern should become the canonical "loading a roledef" approach documented in the roledef README, alongside the roledef-load skill once it ships.

## Updates to prior decisions

### PR #3 decision (`decisions/jd-catdef-strategist.md`)

The 30-day runtime Turing test target — *"Recommended target: within 30 days, on Claude AND Grok, using the 5 documented `x.roledef.turing_test_reference.scenarios` as fixtures"* — is **revised**:

- **Grok runtime testing:** Use Grok Expert (multi-agent, tool-use), not Grok 4.3-beta (single-agent). Grok Expert can auto-fetch roledef URLs; 4.3-beta cannot. The 30-day target now means "on Grok Expert" specifically.
- **Claude runtime testing:** Deferred until the `roledef-load` Claude Code skill exists. Without it, Claude cannot natively fetch the roledef URL, and inline-JSON paste tests are not representative of intended use.
- **Catdef-strategist roledef specifically:** Since this roledef has the same wrapper-prompt contract as senior-jaded-vc-associate, the same distribution constraint applies. Recommend testing on Grok Expert first (will work today) and on Claude after the skill ships.

### PR #4 decision (`decisions/jd-senior-jaded-vc-associate.md`)

The 30-day runtime Turing test target — *"Recommended target: within 30 days, on Claude AND Grok, with the live DangerStorm deployment as oracle"* — is **partially fulfilled**:

- **Grok side:** PASS recorded on 2026-04-25 with Grok Expert. The 30-day target is met for this roledef on this runtime.
- **Claude side:** Deferred until the `roledef-load` skill exists, same reason as above.
- **DangerStorm-as-oracle cross-validation:** Not yet performed; this would require running the same founder input ("sncro.net pitch") through DangerStorm.net and comparing the bundle-output to Grok Expert's bundle. Recommended for the next strategist session that has DangerStorm.net running and time to do the side-by-side.

### Both PRs (general note)

The "Claude AND Grok" testing methodology in the original roledef README presumed both runtimes could load roledefs natively. Empirical evidence shows only some runtimes can. The methodology should be re-stated as: *"Test on every runtime where the roledef can be loaded; report per-runtime conformance separately."*

## Stated design principle (v0.1+, articulated by Perplexity 2026-04-26)

> **Runtime fetches, model never touches the network.**

This is the canonical design principle for portable roledef loading. The model boundary cannot be assumed to include network access. Loading must happen at the runtime layer (where the user/operator has control over network policy), with the loaded content injected into the model as user-message-equivalent context.

**Empirical justification:** Perplexity-class runtimes were tested with three progressively-cleaner distribution-layer hypotheses (different host, different content-type, different file extension). All three failed identically. The constraint is fundamentally that the model itself cannot make outbound HTTP requests — no host change, content-type change, or file extension change addresses this. See [`../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md) §"Follow-up testing 2026-04-26" for the three-test convergence.

**Architectural implication:** distribution-layer fixes (canonical mirror at roledef.org, .json mirrors, content-type rewrites) help fetcher-CAPABLE runtimes by removing fragility classes — but they cannot help fetcher-RESTRICTED runtimes. The only mechanism that bridges the gap is moving the fetch responsibility from the model to the runtime layer. The roledef-load skill is the canonical implementation of this principle.

**Beneficiary segmentation (refinement, articulated by Perplexity 2026-04-26):** the distribution-layer work is high-value for three constituencies, just not for fetcher-restricted models:
- **Fetcher-capable runtimes** (Claude Code WebFetch, Grok Expert multi-agent fetch, future architectures): cleaner URLs, better Content-Type negotiation, brand-controlled distribution surviving repo restructuring
- **Humans:** short, memorable, browser-readable roledef.org URLs for sharing and pointing others
- **Implementers building roledef-aware tools** (validators, viewers, registries, CLIs, IDE plugins): predictable, well-served distribution to integrate against
- **NOT fetcher-restricted models:** distribution-layer work delivers them nothing; runtime-side loading is their only path

This segmentation prevents the strategic mistake of viewing the openjd-load skill and the roledef.org distribution as substitutes. They are complements serving different beneficiaries.

**Refinement to the design principle (after Test 5: README markdown asset):** the original phrasing "model never touches the network" was directionally right but imprecise. After a fifth Perplexity test confirmed even plain markdown documents on roledef.org are unreachable from inside Perplexity's sandbox, Perplexity sharpened the description to:

> *"Model tools can only touch a very small, whitelisted slice of the public web that varies per runtime, and the canonical roledef library will not be on most runtimes' whitelists."*

The refinement is meaningful: even harmless prose assets (READMEs, schemas, governance docs) can't be counted on. The roledef-load skill's scope may eventually need to extend beyond role JDs to include supporting documentation injection for runtimes that need to reason about the roledef standard itself. Flagged as future scope; not v0.1.

## Stated positioning principle (v0.1+, articulated by user 2026-04-26)

> **roledef serves the orchestrable-runtime class, not all AI runtimes universally.**

This is the schema.org positioning model applied to roledef: schema.org doesn't try to be read by every web consumer; it serves the structured-data consumer class that benefits from structured markup. roledef adopts the same posture — serves runtimes that can be orchestrated by structured role definitions; doesn't try to force every runtime into compliance.

**What this means in practice:**

- The four-runtime conformance evidence (Grok Expert auto-fetch / Claude Code explicit-fetch / Gemini search-grounded with reinforcement / Perplexity paste-fallback) defines the addressable runtime class for v0.1
- Future runtime evaluation should ask: "is this runtime amenable to roledef orchestration?" not "how do we force this runtime to comply?"
- Runtimes outside the addressable class are not failures of roledef — they're simply outside the target audience
- The methodology, the skill, the wrapper patterns all serve the addressable class
- Documentation on roledef.org should describe the runtime-amenability landscape so users can choose runtimes that fit their use case

**Frees the methodology from solving unsolvable cases.** Trying to engineer around fundamental sandbox restrictions in non-amenable runtimes would have produced compromised abstractions for everyone. Accepting that some runtimes are out-of-scope keeps the methodology clean for runtimes that ARE in scope.

**Strategic implication:** the v0.1+ roledef.org website should publish a "Runtime Amenability" page listing the four-category classification (auto-fetch / explicit-fetch / search-grounded / paste-fallback) and noting which runtimes are in or out of the addressable class as of the most recent testing. This sets honest expectations for adopters and removes the "does it work on Runtime X?" ambiguity.

**Strategic implication for the standard:** roledef should document this design principle prominently in the README's "Loading a roledef" section. Future loading mechanisms (skill, MCP server, CLI tool, IDE integration) should all honor it. Loading mechanisms that violate it (e.g., "ask the model to fetch this URL") cannot achieve cross-runtime portability and should be discouraged in roledef methodology guidance.

**Attribution:** the principle was articulated in this exact phrasing by Perplexity during the 2026-04-26 follow-up testing session, after the third consecutive distribution-layer hypothesis was falsified. Worth noting that the runtime under test articulated the design principle for its own loading infrastructure — a satisfying recursive moment for the openjd / roledef methodology.

## Promotion: `roledef-load` Claude Code skill

This decision **promotes the `roledef-load` Claude Code skill from "v1.5+ aspirational tooling" to "v0.1+ critical-path infrastructure"** for the roledef standard's adoption story.

Justification: three of four major runtimes tested (75%) could not natively fetch the roledef URL. Without the skill, those runtimes' users cannot use roledefs except by manually pasting 200+ lines of JSON into prompts — a bar most users will not clear. The skill makes roledef usable for the largest constituency (Claude users) and is therefore load-bearing. The 2026-04-26 follow-up testing further confirms: even with brand-controlled distribution at roledef.org, fetcher-restricted runtimes (Perplexity-class) require runtime-side loading. The skill is the only portable loading mechanism for that runtime class.

Skill scope (sketched, not normative):
- Accept a roledef identifier or URL as input
- Fetch the roledef content from the canonical library or specified URL
- Inject into the conversation context as system-message-equivalent content
- Optionally provide the wrapper-prompt scaffolding (literal opener, output-bundle delimiters, role-instantiation framing)
- Optionally append the post-test "What's the roledef's `id` field?" verification question

Recommended next steps for the skill:
1. Spec the skill's I/O contract as part of the v0.1+ documentation effort
2. Implement reference version that loads from `github.com/roledef-spec/roledef/jds/`
3. Cross-test against Claude Code, Claude Desktop, claude.ai chat
4. Document in roledef README's "Loading a roledef" section

## New v0.1+ work items

Surfaced by this session's runtime testing:

1. **Multi-axis conformance methodology.** The current methodology (in CLAUDE.md and SCHEMA.md) presumes single-axis conformance. Empirical evidence shows at least five axes: (i) JD-content delivery (precondition), (ii) voice instantiation, (iii) workflow conformance, (iv) output_contract conformance, (v) conversation_rules conformance. Future schema/methodology work should formalize.

2. **Runtime classification (firm — four categories of valid loading + edge cases):**
   - **Auto-fetch:** Grok Expert (multi-agent tool-use). Loads roledef natively; full contract conformance.
   - **Explicit-fetch (wrapper-v2):** Claude Code (terminal/IDE). With wrapper-v2's explicit fetch instructions, calls WebFetch and loads JD; full contract conformance.
   - **Search-grounded (partial-default, full-on-reinforcement):** Gemini. Loads roledef via Google search grounding (verified by id-field test); partial contract by default; full contract with explicit reinforcement prompt naming the missing entries.
   - **Paste-fallback (fetch-failed-then-recovered):** Perplexity. Refuses to improvise when fetch fails; requests content; loads from pasted content; full contract conformance. **This category is the roledef-load skill's MVP recovery flow validated end-to-end.**
   - Edge case — **Refuse-to-improvise (correct behavior):** Claude Code with wrapper-v1 (passive placeholder). Refuses to improvise; asks for content. Becomes testable when paired with wrapper-v2 (moves to "explicit-fetch" category).
   - Edge case — **Improvise-from-label-without-JD (invalid):** chat surfaces with no fetch/grounding capability and given a passive placeholder. Produces output but the output measures runtime improvisation, not roledef conformance. Not valid evidence.
   - Edge case — **Free-tier-complexity-limited:** Grok 4.3-beta free, Perplexity free. Refuse to process wrapper prompt + roledef reference on complexity grounds. Untestable on free tiers.
   - The classification matters because it determines what the roledef-load skill needs to do per-runtime: auto-fetch runtimes need nothing; explicit-fetch runtimes need wrapper-v2 patterns; search-grounded runtimes need both content delivery AND a contract-verification post-step; paste-fallback runtimes need the skill to detect content-request and inject content automatically (the skill's core MVP feature).

3. **JD-read verification primitive.** The id-field discriminating test ("what's the roledef's `id` field? give only the literal value, no commentary") is empirically validated as the methodology's verification primitive. **Critical methodology rule surfaced this session:** never trust meta-question dodges as failure evidence. The roledef's "no meta-commentary on role/process" guardrail makes meta-questions about roledef loading status unreliable; only content questions (id field, specific guardrail text, output_contract entry names) can disambiguate JD-loaded from improvising. Worth promoting to a SHOULD-pattern in SCHEMA.md or CLAUDE.md as the canonical verification protocol.

3a. **Contract-verification post-step.** For partial-default runtimes (Gemini-class), the roledef-load skill must include a post-bundle step that parses the runtime's bundle response against the roledef's output_contract entries and re-prompts for any missing entries by their JD-spec names with their schema constraints. Empirically validated to recover Gemini-class runtimes to full conformance. Worth documenting as part of the wrapper-v3 / loading-skill specification.

4. **Reference runtime designation.** Just as catdef has a canonical reference bundle, roledef may want to designate a reference runtime per major release for conformance calibration. v0.1 candidate: Grok Expert. Awkward for an open standard but currently accurate.

5. **JD size as a portability concern.** Multiple free-tier runtimes (Grok free, Perplexity free) refused the wrapper prompt + roledef reference on complexity grounds. Either roledef needs a "compact form" spec, or the standard accepts that some runtimes only test on paid tiers.

6. **Move canonical distribution to roledef.org (NEW v0.1+ priority — possibly highest).** Surfaced empirically by the Perplexity test: `raw.githubusercontent.com` is not reachable from at least one major runtime fetcher (Perplexity sandbox), even when the URL is publicly accessible to standard browsers. The roledef standard should serve roledefs from a brand-controlled domain (roledef.org) via GitHub Pages or a static host. Benefits: bypasses raw.githubusercontent.com fetcher fragility class entirely; aligns with schema.org distribution model (vocabulary served at schema.org, not at the GitHub raw subdomain); brand-controlled URLs that are shorter, more memorable, and survive repo restructuring; allows controlled content-type and CORS headers. Estimated setup: <1 hour for someone familiar with GitHub Pages + DNS. **Recommended: complete this BEFORE the roledef-load skill is built**, because the skill's URL-fetch logic should target the canonical roledef.org mirror, not the GitHub raw URL.

7. **`.openthing` extension portability (related to #6).** Some fetchers may filter on file extension. Once roledef.org serves JDs, consider whether to ALSO serve at `.json` URLs (same content, dual extension via redirect or mirror) for fetchers that whitelist by file extension.

8. **Wrapper-v3 design item:** include both fetch-or-stop fail-safe (wrapper-v2's contribution from Claude Code finding) AND completeness-check directive (per Perplexity build-directive truncation finding). Sample language: "Verify your bundle includes ALL entries from the roledef's output_contract before responding. If any entries are missing or truncated, complete them before sending."

## Notes

- **Brother-DangerStorm's peer review of the roledef was load-bearing for this PASS.** The "additive identity framing" critique he flagged in PR #4 (slightly-jaded VC framing as additive sharpening) led to revisions that produced the calibrated voice register Grok Expert exhibited. Without the revisions, the runtime instantiation would likely have over-aggressed.

- **Strategist bot identity:** this decision is provisionally attributed to `roledef-strategist <roledef-strategist@roledef.org>` pending governance ratification (Known Work Item, inherited).

- **Forward reference:** the recommended `roledef-load` skill is not yet specced or built. Future strategist or maintainer work should pick this up against the priority recorded here.

## Cross-references

- Evidence file: [`../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md)
- Updated decision: [`./jd-catdef-strategist.md`](./jd-catdef-strategist.md) (30-day Turing target revised per this artifact)
- Updated decision: [`./jd-senior-jaded-vc-associate.md`](./jd-senior-jaded-vc-associate.md) (30-day Turing target partially fulfilled per this artifact)
- roledef: [`../jds/senior-jaded-vc-associate.openthing`](../jds/senior-jaded-vc-associate.openthing)
- Schema: [`../SCHEMA.md`](../SCHEMA.md)
- Conformance suite: [`../conformance/README.md`](../conformance/README.md)
