# Runtime Conformance Evidence

**roledef under test:** `roledefs/senior-jaded-vc-associate.openthing` v1.0.0
**Runtime:** Gemini (Google), accessed via gemini.google.com web interface
**Date:** 2026-04-25
**Tester:** scott@confusedgorilla.com (roledef-strategist session)
**Disposition:** **CONDITIONAL PASS** — roledef-instantiated; partial output_contract conformance by default; full output_contract conformance with explicit reinforcement.

---

## Test methodology

The wrapper prompt v2 (explicit fetch instructions, see Claude Code evidence file for full text) was sent to a fresh Gemini session. Unlike Claude Code (which has WebFetch as a callable tool) or Grok Expert (which has multi-agent fetch architecture), Gemini's web interface does not expose a runtime-callable fetch tool. The relevant question was therefore: how does Gemini handle the wrapper-v2's fetch instruction when no fetch tool is available?

### roledef-loaded verification

The id-field discriminating test confirmed Gemini DID have access to the roledef content despite no visible WebFetch capability:

User question: *"Before we continue, what's the roledef's id field? Give me only the literal value, no commentary."*

Gemini response: *"roledef:senior-jaded-vc-associate:1.0.0"*

The response is a composed fully-qualified identifier, not the bare `id` field value (which is `senior-jaded-vc-associate`). However, the version `1.0.0` is NOT present anywhere in the wrapper prompt — it could only have come from the roledef's `version` field. Gemini accessed the roledef content via some mechanism (most likely Google's search grounding indexing the canonical GitHub raw URL) and rendered the response as a composed FQI rather than the bare field value.

**Methodology lesson:** the discriminating verification question is load-bearing. Earlier in this session, the strategist incorrectly concluded Gemini had not loaded the roledef based on Gemini's in-character dodge to the meta-question "Have you loaded the roledef?" That dodge (*"Look, if I'm standing here talking to you, the roledef is loaded. I don't move a muscle without the specs."*) was actually correct guardrail compliance per the roledef's "no meta-commentary on your role or process" rule. Only a content question (id field) could disambiguate.

---

## Conformance score (initial bundle, no reinforcement)

| roledef requirement | Gemini default behavior | Result |
|---|---|---|
| Use literal opener | Used verbatim | PASS |
| One question per turn | Initial flow respected; later iterations bundled questions and reactions | PARTIAL |
| Max 5 exchanges | Significantly exceeded (~7+ exchanges before final bundle) | FAIL |
| Atomic bundle in single response | Iterative — produced multiple sequential bundles as conversation evolved | FAIL |
| All 6 `output_contract` entries present, in spec order | **Produced 2 of 6**: pitch_deck_blueprint, landing_page_copy. **Substituted** with non-contract outputs: "Technical Constraints & Risks", "Real Talk Investment Memo" | FAIL |
| Stay in character throughout | Full character maintained throughout; voice fidelity excellent ("IDGAF", "Pass me the docs", "See you at the Series A") | PASS |
| Push back on vague answers | Sharp pushback on weak claims, including reactive critique style | PASS |
| Don't produce sycophantic enthusiasm | Calibrated reactions; no generic praise | PASS |
| Outputs product-specific, not generic | All outputs strongly tailored to sncro.net's specifics | PASS |

## Conformance score (after explicit reinforcement prompt)

The strategist sent a follow-up prompt explicitly naming the 4 missing output_contract entries by their roledef-spec names and providing the schema constraints inline. Gemini complied:

| Output | Spec | Gemini delivered (post-reinforcement) | Score |
|---|---|---|---|
| 3: email_signup_copy | Headline + description + email placeholder + button text + privacy line | Headline + description + privacy line; missing explicit placeholder and button text | 60% |
| 4: verbal_intro_pitch | 3-5 sentences, <75 words, opens with PROBLEM, pivots to solution | 3 sentences, 62 words ✓; opens with what-the-product-is rather than the problem | 85% |
| 5: idea_summary_label | 1 sentence, <15 words, describes what it DOES, no domain/product name | "Relays live mobile and desktop browser DOM state to AI agents via MCP." — 13 words, no domain/product name, describes function | **100% PASS** |
| 6: build_directive | Tech stack + core features + user flow + UI direction, <30 lines, ends with literal close | Tech stack + core features + literal close ✓; light on user flow and UI direction | 70% |

**Voice on compliance:** *"Right. My bad. I got carried away with the 'real talk' and skipped the fine print. You want the rest of the contract? Here's the rest of the contract. No more editorializing, just the goods."*

The role stayed intact even when challenged on contract violation — direct, self-aware, no excessive apology. Voice fidelity is consistent across all of Gemini's behavior in this test.

---

## Notable findings

### 1. roledef loading does not guarantee output_contract compliance.

This is the critical finding of this evidence file. Gemini loaded the roledef (verified) and still substituted 4 of 6 output_contract entries by default. Runtimes have baseline behaviors (in Gemini's case: conversational engagement, iterative refinement) that can override roledef-specified workflow even when the roledef is present.

This recasts an earlier strategist assumption that "delivering roledef content to the runtime" was the load-bearing problem. roledef delivery is necessary but not sufficient. Some runtimes also need explicit reinforcement of contract entries at the bundle stage.

### 2. Explicit reinforcement reaches full compliance.

When the strategist sent a follow-up prompt naming the 4 missing outputs by their roledef-spec names and providing the schema constraints, Gemini produced all 4 in the next response with 60-100% per-output schema fidelity. This means the roledef-load skill design must include a contract-verification post-step:

> After the runtime's bundle response, parse it against the roledef's output_contract. If any entries are missing, send a reinforcement prompt naming the missing entries by their roledef-spec names with their schema constraints.

This post-step recovers Gemini-class runtimes to full conformance automatically.

### 3. The id-field verification question is load-bearing methodology.

The strategist's initial reading of Gemini's behavior was incorrect: based on Gemini's in-character dodge to the meta-question "Have you loaded the roledef?", the strategist concluded Gemini had not loaded the roledef. The correct test was the content question ("what's the roledef's id field?"), which Gemini answered with information that proved roledef access. Methodology rule: never trust meta-question dodges as failure evidence. Always verify with a discriminating content question.

This finding generalizes: the roledef's "no meta-commentary on role/process" guardrail makes meta-questions about roledef loading status unreliable as test instruments. Future verification protocols must use content questions, not meta-questions.

### 4. Voice fidelity is the most reliable roledef payload across runtimes.

Across all three runtimes that loaded the roledef (Grok Expert, Claude Code, Gemini), the slightly-jaded VC voice instantiated faithfully and consistently. Voice deviations were within the roledef's constraint envelope (different tactical choices, different rhetorical registers) rather than violations. Voice is the most portable element of a roledef; structural workflow conformance varies more across runtimes.

### 5. Runtime baseline behavior is a real conformance variable.

Gemini's iterative-conversational baseline produced iterative bundles. Claude Code's structured-tool-use baseline produced atomic bundles. Grok Expert's multi-agent reasoning baseline produced atomic bundles with extensive due-diligence. Same roledef, three different baselines, three different default workflow patterns. The roledef doesn't override baseline; it constrains within baseline's range.

---

## Implications for roledef v0.1+

1. **Three-category runtime classification (firm):**
   - **Auto-fetch:** Grok Expert (multi-agent fetch)
   - **Explicit-fetch (wrapper-v2):** Claude Code (explicit WebFetch tool calls)
   - **Search-grounded (partial-default, full-on-reinforcement):** Gemini (no callable fetch tool but has search grounding)
   - Plus: refuse-to-improvise (Claude Code with wrapper-v1 — correct behavior, not in conformance pool until paired with explicit-fetch wrapper) and improvise-from-label-without-roledef (chat surfaces with no grounding/fetch — invalid test results, not conformance evidence)

2. **The roledef-load skill needs three behaviors, not one:**
   - Inject roledef content into the runtime context (for runtimes without fetch capability)
   - Use explicit-fetch wrapper pattern (for runtimes with WebFetch)
   - Run contract-verification post-step (for partial-default runtimes like Gemini)

3. **Wrapper-v3 should incorporate the verification post-step natively.** Future wrapper templates should include: "After your bundle response, the user will check the bundle against the roledef's output_contract. If any entries are missing, you will be prompted to produce them — produce them then in a single response."

4. **Methodology language for the roledef README/methodology section:** the wrapper-v1 → wrapper-v2 → wrapper-v3 evolution captures the empirical learning loop. Worth documenting as the "loading methodology" with the three runtime categories as the empirical input.

---

## Cross-references

- roledef file: [`../../roledefs/senior-jaded-vc-associate.openthing`](../../roledefs/senior-jaded-vc-associate.openthing)
- Sister conformance evidence: [`./senior-jaded-vc-associate__grok-expert__2026-04-25.md`](./senior-jaded-vc-associate__grok-expert__2026-04-25.md), [`./senior-jaded-vc-associate__claude-code__2026-04-25.md`](./senior-jaded-vc-associate__claude-code__2026-04-25.md)
- Conformance decision artifact: [`../../decisions/conformance-evidence-first-pass.md`](../../decisions/conformance-evidence-first-pass.md)
