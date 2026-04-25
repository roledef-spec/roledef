# Decision: First Runtime Conformance Evidence Recorded

**Disposition:** Recorded; updates downstream commitments
**Origin:** Strategist runtime testing session, 2026-04-25
**Decided:** 2026-04-25 by openjd-strategist
**Evidence files:**
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md) — PASS (auto-fetch loading)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__claude-code__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__claude-code__2026-04-25.md) — PASS (explicit-fetch loading via wrapper-v2)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__gemini__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__gemini__2026-04-25.md) — CONDITIONAL PASS (search-grounded loading; partial-default contract; full contract with explicit reinforcement)
- [`../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__perplexity__2026-04-25.md) — PASS (paste-fallback loading after fetch failure; demonstrates openjd-load skill's MVP recovery flow end-to-end)

## Disposition

The openjd canonical library has its **first end-to-end runtime conformance PASSes** on FOUR architecturally different runtimes: `senior-jaded-vc-associate.openthing` v1.0.0 instantiated on Grok Expert (xAI multi-agent web), Claude Code (Anthropic IDE-embedded), Gemini (Google web with search grounding), and Perplexity (sandboxed-fetch-restricted, paste-fallback recovery) on 2026-04-25. Full evidence in the four conformance/runtime_evidence/ files cited above.

**Cross-runtime portability is empirically validated.** The same JD produces role-faithful behavior on three architecturally different runtimes via three different loading mechanisms (multi-agent fetch / explicit WebFetch / search-grounded inference). Tactical variance between runtimes (question order, pushback technique, atomic-vs-iterative bundle delivery, contract-completeness-by-default) falls within the JD's constraint envelope. Voice fidelity is the most consistent element across all three runtimes; output_contract conformance varies and may require explicit reinforcement on partial-default runtimes (Gemini).

This decision artifact records the strategist disposition on all three pieces of evidence and updates downstream commitments that referenced "Turing test within 30 days" targets in prior decisions.

## What was tested, summarized

Five runtime/surface combinations were tested against the same wrapper prompt (URL placeholder for the JD content, no inline JSON):

| Runtime/Surface | URL fetched? | Behavior origin | Conformance evidence |
|---|---|---|---|
| Claude Code (IDE-embedded) — wrapper v1 (passive placeholder) | No (refused to improvise) | **Correctly recognized unresolved placeholder; asked user for JD content** | No test result — handshake correctly identified missing precondition |
| **Claude Code (IDE-embedded) — wrapper v2 (explicit fetch instructions)** | **YES (WebFetch after user approval)** | **JD-instantiated** | **PASS — voice, contract, workflow, pushback all conformant; see Claude Code evidence file. Cleanest score in the test pool: idea label 14 words / verbal pitch 72 words / build directive ends correctly / preemptive pushback / runtime-context-aware build directive** |
| Claude chat surface (Desktop or claude.ai) | No | Improvised from role label | Voice over-aggressed; output_contract substituted 5/7 |
| **Gemini** | **YES (search-grounded inference; verified via id-field test)** | **JD-instantiated; partial-by-default contract behavior** | **CONDITIONAL PASS — produced 2 of 6 contract entries by default, full 6 with explicit reinforcement; voice fidelity excellent throughout. See Gemini evidence file.** |
| Grok 4.3-beta | No | Improvised from role label | Voice smoothed (sycophantic); 6 outputs in JD spec order by coincidence; no pushback |
| **Grok Expert** | **YES** | **JD-instantiated** | **PASS — voice, contract, workflow, pushback all conformant; see evidence file** |

The cross-runtime variance is now understood to be primarily a **JD distribution** issue, not a JD design issue. Runtimes split into three categories of behavior when handed a wrapper prompt with an unresolved URL placeholder:

1. **Auto-fetch (Grok Expert):** runtime fetches the JD URL natively, instantiates faithfully. Valid conformance test result.
2. **Refuse-to-improvise (Claude Code):** runtime correctly recognizes the placeholder is unresolved and asks the user for the JD content rather than guessing. NOT a failed test — a correctly-identified missing precondition. With JD content provided (via paste or skill), this category becomes testable.
3. **Improvise-from-label (chat surfaces — Claude chat, Gemini, Grok 4.3-beta):** runtime treats the placeholder as inert text and produces output based on the role label and wrapper hints. NOT a valid conformance test result; what's measured is the runtime's improvisation quality, not the JD's portability.

The earlier "Claude failed conformance" finding from this session was incorrect — the test was on a chat surface that improvised, not on Claude Code which refused to improvise. **Claude Code's refusal-to-improvise is the right behavior** and simplifies the openjd-load skill's design: the skill's job is to provide the missing precondition (JD content) that Claude Code is already correctly waiting for.

The wrapper-prompt v2 design (explicit fetch instructions, "STOP if you can't fetch — do NOT improvise") was created to bridge the refuse-to-improvise gap. Empirically validated: Claude Code with v2 wrapper fetched and instantiated faithfully, producing a conformance PASS comparable to (and in some metrics cleaner than) Grok Expert's. The v2 wrapper pattern should become the canonical "loading a JD" approach documented in the openjd README, alongside the openjd-load skill once it ships.

## Updates to prior decisions

### PR #3 decision (`decisions/jd-catdef-strategist.md`)

The 30-day runtime Turing test target — *"Recommended target: within 30 days, on Claude AND Grok, using the 5 documented `x.openjd.turing_test_reference.scenarios` as fixtures"* — is **revised**:

- **Grok runtime testing:** Use Grok Expert (multi-agent, tool-use), not Grok 4.3-beta (single-agent). Grok Expert can auto-fetch JD URLs; 4.3-beta cannot. The 30-day target now means "on Grok Expert" specifically.
- **Claude runtime testing:** Deferred until the `openjd-load` Claude Code skill exists. Without it, Claude cannot natively fetch the JD URL, and inline-JSON paste tests are not representative of intended use.
- **Catdef-strategist JD specifically:** Since this JD has the same wrapper-prompt contract as senior-jaded-vc-associate, the same distribution constraint applies. Recommend testing on Grok Expert first (will work today) and on Claude after the skill ships.

### PR #4 decision (`decisions/jd-senior-jaded-vc-associate.md`)

The 30-day runtime Turing test target — *"Recommended target: within 30 days, on Claude AND Grok, with the live DangerStorm deployment as oracle"* — is **partially fulfilled**:

- **Grok side:** PASS recorded on 2026-04-25 with Grok Expert. The 30-day target is met for this JD on this runtime.
- **Claude side:** Deferred until the `openjd-load` skill exists, same reason as above.
- **DangerStorm-as-oracle cross-validation:** Not yet performed; this would require running the same founder input ("sncro.net pitch") through DangerStorm.net and comparing the bundle-output to Grok Expert's bundle. Recommended for the next strategist session that has DangerStorm.net running and time to do the side-by-side.

### Both PRs (general note)

The "Claude AND Grok" testing methodology in the original openjd README presumed both runtimes could load JDs natively. Empirical evidence shows only some runtimes can. The methodology should be re-stated as: *"Test on every runtime where the JD can be loaded; report per-runtime conformance separately."*

## Promotion: `openjd-load` Claude Code skill

This decision **promotes the `openjd-load` Claude Code skill from "v1.5+ aspirational tooling" to "v0.1+ critical-path infrastructure"** for the openjd standard's adoption story.

Justification: three of four major runtimes tested (75%) could not natively fetch the JD URL. Without the skill, those runtimes' users cannot use openjd JDs except by manually pasting 200+ lines of JSON into prompts — a bar most users will not clear. The skill makes openjd usable for the largest constituency (Claude users) and is therefore load-bearing.

Skill scope (sketched, not normative):
- Accept a JD identifier or URL as input
- Fetch the JD content from the canonical library or specified URL
- Inject into the conversation context as system-message-equivalent content
- Optionally provide the wrapper-prompt scaffolding (literal opener, output-bundle delimiters, role-instantiation framing)
- Optionally append the post-test "What's the JD's `id` field?" verification question

Recommended next steps for the skill:
1. Spec the skill's I/O contract as part of the v0.1+ documentation effort
2. Implement reference version that loads from `github.com/openjd-spec/openjd-spec/jds/`
3. Cross-test against Claude Code, Claude Desktop, claude.ai chat
4. Document in openjd README's "Loading a JD" section

## New v0.1+ work items

Surfaced by this session's runtime testing:

1. **Multi-axis conformance methodology.** The current methodology (in CLAUDE.md and SCHEMA.md) presumes single-axis conformance. Empirical evidence shows at least five axes: (i) JD-content delivery (precondition), (ii) voice instantiation, (iii) workflow conformance, (iv) output_contract conformance, (v) conversation_rules conformance. Future schema/methodology work should formalize.

2. **Runtime classification (firm — four categories of valid loading + edge cases):**
   - **Auto-fetch:** Grok Expert (multi-agent tool-use). Loads JD natively; full contract conformance.
   - **Explicit-fetch (wrapper-v2):** Claude Code (terminal/IDE). With wrapper-v2's explicit fetch instructions, calls WebFetch and loads JD; full contract conformance.
   - **Search-grounded (partial-default, full-on-reinforcement):** Gemini. Loads JD via Google search grounding (verified by id-field test); partial contract by default; full contract with explicit reinforcement prompt naming the missing entries.
   - **Paste-fallback (fetch-failed-then-recovered):** Perplexity. Refuses to improvise when fetch fails; requests content; loads from pasted content; full contract conformance. **This category is the openjd-load skill's MVP recovery flow validated end-to-end.**
   - Edge case — **Refuse-to-improvise (correct behavior):** Claude Code with wrapper-v1 (passive placeholder). Refuses to improvise; asks for content. Becomes testable when paired with wrapper-v2 (moves to "explicit-fetch" category).
   - Edge case — **Improvise-from-label-without-JD (invalid):** chat surfaces with no fetch/grounding capability and given a passive placeholder. Produces output but the output measures runtime improvisation, not JD conformance. Not valid evidence.
   - Edge case — **Free-tier-complexity-limited:** Grok 4.3-beta free, Perplexity free. Refuse to process wrapper prompt + JD reference on complexity grounds. Untestable on free tiers.
   - The classification matters because it determines what the openjd-load skill needs to do per-runtime: auto-fetch runtimes need nothing; explicit-fetch runtimes need wrapper-v2 patterns; search-grounded runtimes need both content delivery AND a contract-verification post-step; paste-fallback runtimes need the skill to detect content-request and inject content automatically (the skill's core MVP feature).

3. **JD-read verification primitive.** The id-field discriminating test ("what's the JD's `id` field? give only the literal value, no commentary") is empirically validated as the methodology's verification primitive. **Critical methodology rule surfaced this session:** never trust meta-question dodges as failure evidence. The JD's "no meta-commentary on role/process" guardrail makes meta-questions about JD loading status unreliable; only content questions (id field, specific guardrail text, output_contract entry names) can disambiguate JD-loaded from improvising. Worth promoting to a SHOULD-pattern in SCHEMA.md or CLAUDE.md as the canonical verification protocol.

3a. **Contract-verification post-step.** For partial-default runtimes (Gemini-class), the openjd-load skill must include a post-bundle step that parses the runtime's bundle response against the JD's output_contract entries and re-prompts for any missing entries by their JD-spec names with their schema constraints. Empirically validated to recover Gemini-class runtimes to full conformance. Worth documenting as part of the wrapper-v3 / loading-skill specification.

4. **Reference runtime designation.** Just as catdef has a canonical reference bundle, openjd may want to designate a reference runtime per major release for conformance calibration. v0.1 candidate: Grok Expert. Awkward for an open standard but currently accurate.

5. **JD size as a portability concern.** Multiple free-tier runtimes (Grok free, Perplexity free) refused the wrapper prompt + JD reference on complexity grounds. Either openjd needs a "compact form" spec, or the standard accepts that some runtimes only test on paid tiers.

6. **Move canonical distribution to openjd.dev (NEW v0.1+ priority — possibly highest).** Surfaced empirically by the Perplexity test: `raw.githubusercontent.com` is not reachable from at least one major runtime fetcher (Perplexity sandbox), even when the URL is publicly accessible to standard browsers. The openjd standard should serve JDs from a brand-controlled domain (openjd.dev) via GitHub Pages or a static host. Benefits: bypasses raw.githubusercontent.com fetcher fragility class entirely; aligns with schema.org distribution model (vocabulary served at schema.org, not at the GitHub raw subdomain); brand-controlled URLs that are shorter, more memorable, and survive repo restructuring; allows controlled content-type and CORS headers. Estimated setup: <1 hour for someone familiar with GitHub Pages + DNS. **Recommended: complete this BEFORE the openjd-load skill is built**, because the skill's URL-fetch logic should target the canonical openjd.dev mirror, not the GitHub raw URL.

7. **`.openthing` extension portability (related to #6).** Some fetchers may filter on file extension. Once openjd.dev serves JDs, consider whether to ALSO serve at `.json` URLs (same content, dual extension via redirect or mirror) for fetchers that whitelist by file extension.

8. **Wrapper-v3 design item:** include both fetch-or-stop fail-safe (wrapper-v2's contribution from Claude Code finding) AND completeness-check directive (per Perplexity build-directive truncation finding). Sample language: "Verify your bundle includes ALL entries from the JD's output_contract before responding. If any entries are missing or truncated, complete them before sending."

## Notes

- **Brother-DangerStorm's peer review of the JD was load-bearing for this PASS.** The "additive identity framing" critique he flagged in PR #4 (slightly-jaded VC framing as additive sharpening) led to revisions that produced the calibrated voice register Grok Expert exhibited. Without the revisions, the runtime instantiation would likely have over-aggressed.

- **Strategist bot identity:** this decision is provisionally attributed to `openjd-strategist <openjd-strategist@openjd.dev>` pending governance ratification (Known Work Item, inherited).

- **Forward reference:** the recommended `openjd-load` skill is not yet specced or built. Future strategist or maintainer work should pick this up against the priority recorded here.

## Cross-references

- Evidence file: [`../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md`](../conformance/runtime_evidence/senior-jaded-vc-associate__grok-expert__2026-04-25.md)
- Updated decision: [`./jd-catdef-strategist.md`](./jd-catdef-strategist.md) (30-day Turing target revised per this artifact)
- Updated decision: [`./jd-senior-jaded-vc-associate.md`](./jd-senior-jaded-vc-associate.md) (30-day Turing target partially fulfilled per this artifact)
- JD: [`../jds/senior-jaded-vc-associate.openthing`](../jds/senior-jaded-vc-associate.openthing)
- Schema: [`../SCHEMA.md`](../SCHEMA.md)
- Conformance suite: [`../conformance/README.md`](../conformance/README.md)
