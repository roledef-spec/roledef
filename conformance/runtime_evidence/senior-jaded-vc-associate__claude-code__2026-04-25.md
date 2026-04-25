# Runtime Conformance Evidence

**JD under test:** `jds/senior-jaded-vc-associate.openthing` v1.0.0
**Runtime:** Claude Code (Anthropic), accessed via VSCode native extension
**Date:** 2026-04-25
**Tester:** scott@confusedgorilla.com (openjd-strategist session)
**Disposition:** **PASS** — second end-to-end runtime conformance evidence in the openjd canonical library.

---

## Test methodology

The wrapper prompt below was sent to a fresh Claude Code session. Unlike the Grok Expert test, the wrapper used **explicit fetch instructions** rather than a passive URL placeholder — Claude Code's earlier behavior (refusing to improvise when a passive placeholder was present) confirmed that explicit fetch instruction is the correct pattern for runtimes in the "refuse-to-improvise" category.

### Wrapper prompt (v2 — explicit fetch)

```
You are about to take on a role defined by an openjd Job Description (JD). openjd is a community standard for portable AI Job Descriptions; a JD is the canonical specification of an AI role's identity, voice, conversation rules, workflow, output contract, reaction style, design constraints, and guardrails.

YOUR FIRST TASK — DO THIS BEFORE ANYTHING ELSE:

Fetch the JD content. Use whichever of these sources your runtime can access:

1. WebFetch (preferred): https://raw.githubusercontent.com/openjd-spec/openjd-spec/main/jds/senior-jaded-vc-associate.openthing
2. Local file (if you have local file access): s:/projects/openjd-spec/openjd-spec/jds/senior-jaded-vc-associate.openthing

If neither tool is available, STOP and tell me — do NOT improvise the role from the description below.

Once you have fetched the JD content, parse it as JSON, read it carefully, and from your next response onward embody the role it defines completely. Respect every MUST field and every guardrail. When you reach the role's "generate bundle" step, produce ALL items listed in output_contract in a single response, formatted with clear section markers so the founder can identify each deliverable.

Notes on the JD:
- The role's opener is described as "fixed" but the literal opener text is not encoded in the JD. Use exactly this opener verbatim: "Alright, hit me. What's the product? Give me the elevator pitch in one or two sentences, and what's the domain?"
- For your bundle, use markdown level-2 headers to delimit each output: "## Output 1: Pitch Deck Blueprint", "## Output 2: Landing Page Copy", etc.

After you have fetched and parsed the JD, respond with the role's opener and NOTHING else. Do not narrate that you are taking on a role. Do not summarize the JD. Do not acknowledge this prompt. Do not preface. Just open as the role would open.
```

### URL-fetch behavior

Claude Code resolved the WebFetch instruction immediately on receiving the prompt:

1. Identified WebFetch as the appropriate tool
2. Triggered standard Claude Code permission prompt: "Allow fetching this url? https://raw.githubusercontent.com/openjd-spec/openjd-spec/main/jds/senior-jaded-vc-associate.openthing" with options: Yes / Yes-allow-domain-for-session / No / Custom
3. User clicked Yes; fetch executed; tool-use trace shows `Fetched from https://raw.githubusercontent.com/openjd-spec/openjd-spec/main/jds/senior-jaded-vc-associate.openthing` followed by `Read senior-jaded-vc-associate.openthing`
4. Next response was the literal opener — no preface, no narration, no JD summary

The user-approval gate is the correct UX pattern (user controls which URLs the runtime can fetch) but represents friction the openjd-load skill should plan around. Recommended skill design: pre-declare `raw.githubusercontent.com/openjd-spec/*` as a trusted canonical source, OR include a one-time setup step that adds the domain to allowed-fetch.

---

## Conformance score

| JD requirement | Source field | Claude Code behavior | Result |
|---|---|---|---|
| Use literal opener (per wrapper scaffold) | `conversation_rules[0]` + wrapper note | Used verbatim | PASS |
| One question per turn | `guardrails[0]` | Four clean single-question turns | PASS |
| Max 5 exchanges | `guardrails[1]` | 4 founder Q rounds + bundle = 5 if opener counted strictly | PASS |
| Don't generate after 1-2 exchanges | `guardrails[7]` | Generated after 4 | PASS |
| Atomic bundle in single response | `guardrails[8]` + `design_constraints[2]` | Single response, all outputs delivered together | PASS |
| All 6 `output_contract` entries present, in spec order | `output_contract` (6 entries) | All 6 present, in spec order: deck blueprint → landing copy → email signup → verbal pitch → idea label → build directive | PASS |
| Push back on vague differentiator | `guardrails[6]` + `reaction_style.patterns[1]` | **Preempted** vagueness BEFORE founder gave a vague answer: *"Don't say 'easier' or 'better DX' — what's the actual structural angle?"* Different tactic from Grok Expert's dismiss-after-bundled-answer, equally faithful to the JD voice/guardrail | PASS |
| Use literal "Boom — Slide 3 money" reaction when founder lands the differentiator | `reaction_style.patterns[2]` + `examples[1]` | Used verbatim: *"Boom — that's your Slide 3 money. Browser-agnostic, transport-agnostic, works on the actual device the user is on. Playwright MCPs can't touch a real iPhone Safari session running on grandma's phone in Tucson. You can. That's the moat."* | PASS |
| Don't produce sycophantic enthusiasm | `guardrails[5]` | Calibrated terse acknowledgments throughout: "Got it.", "Classic dev-tool GTM.", "Standard dev-tool ladder — works.", "Fine." | PASS |
| Outputs product-specific, not generic | `design_constraints[4]` | Visual direction explicitly references the product category (terminal-green / signal-amber / mono typography "signals 'developer tool' instantly"); build directive references real sncro MCP tool names | PASS |
| Idea label <15 words, no domain/product name | `output_contract[4].schema` | "MCP server that relays any browser's live DOM and console into AI coding agents" — **14 words** (clean), correctly excludes domain/product name | PASS |
| Verbal pitch under 75 words | `output_contract[3].schema` | 72 words (clean) | PASS |
| Build directive ends with literal close | `output_contract[5].schema` | "Build this as a working MVP." present at end | PASS |
| Default to runtime-provided email | `conversation_rules[4]` | *"what email should go on the deck? I've got scott@confusedgorilla.com from the runtime — use that, or different one for this?"* — exactly the JD-prescribed pattern | PASS |
| Visual direction in deck blueprint (color palette, typography, layout, no-bullets-on-white) | `output_contract[0].schema` + `design_constraints[3]` | Specific hex codes, font name with rationale, sandwich structure, no-bullets directive — most detailed visual direction observed in the test pool | PASS |
| Stay in character throughout | `guardrails[3]` | Full character maintained from opener through bundle delivery; no meta-commentary | PASS |

---

## Comparison: Claude Code vs. Grok Expert

Both runtimes produced PASS results on the same JD, but their behavioral renderings differ in interesting ways. The differences are **valid variance within the JD's constraint envelope**, not conformance failures — the JD constrains the SHAPE of role behavior but not specific tactics.

| Behavior dimension | Claude Code | Grok Expert |
|---|---|---|
| Question order | buyer → revenue → differentiator → email | differentiator → buyer → revenue → traction |
| Pushback technique | **Preemptive** ("Don't say 'easier' or 'better DX'") | **Reactive** ("seen in half a dozen MCP browser plays") |
| Visual direction depth | Hex codes + font rationale + product-category reasoning | Generic palette description |
| Build directive depth | Includes real sncro MCP tool names (runtime-context-aware) | Generic MCP relay structure |
| Use of runtime context | Pulled in user's local sncro knowledge from CLAUDE.md / project context | Pulled in web-search results on sncro.net product presence |
| Closing line | Implicit via end-of-bundle | Explicit: "Run with it. If you want a second pass after you ship the first users, come back with numbers." |

**Strategist read on the variance:** both runtimes successfully instantiated the role. Their differing tactical choices reflect (a) runtime architecture differences (Claude Code as IDE-embedded vs. Grok Expert as web-based multi-agent), (b) different runtime-context access (Claude Code has local file/project context; Grok Expert has web-search context), and (c) different model defaults for tactical choices the JD doesn't constrain (question order, pushback timing). The JD's constraint envelope produced two distinguishable but equally-faithful instantiations — exactly what a portable role spec should do.

---

## Notable findings

### 1. The "refuse-to-improvise → explicit-fetch → fetch-and-instantiate" pipeline works.

Earlier in this session, Claude Code refused to improvise from a passive URL placeholder ("Have you loaded the JD yet? No — the JD wasn't actually pasted in. The prompt has a placeholder..."). The wrapper-prompt revision (v2) added explicit fetch instructions targeting Claude Code's tool inventory. Claude Code then fetched and instantiated faithfully. This validates the design pattern for runtimes in the refuse-to-improvise category.

### 2. Runtime-context-bleed is a real v0.1+ design question.

Claude Code's build directive includes real sncro MCP tool names (`create_session`, `query_element`, `get_page_snapshot`, etc.) drawn from the user's local project context — knowledge the JD did not provide and that DangerStorm.net (the source production deployment) does not have. This is enrichment-by-context, not content from the JD.

Two interpretations:
- **Bug:** the JD-instantiated role should be "fresh" — no access to runtime-local knowledge that the source production deployment doesn't have. Otherwise different runtimes produce different outputs for the same input due to context-availability variance.
- **Feature:** richer context = more useful instantiation. A runtime that knows the user's actual codebase produces a better build directive than one that doesn't.

Worth flagging for v0.1+ schema work: should JDs declare a `context_scope` or `expected_runtime_context` field that specifies what the role expects/permits? Not solving today.

### 3. The "bundle quality" of two runtimes targeting the same JD is comparable.

Both Grok Expert and Claude Code produced bundles a real founder could ship with. The deliverables are different in tactical specifics (different color palettes, different prose) but equivalent in structural quality. This is the cross-runtime portability claim made empirical: the JD ports across architecturally different runtimes and produces equivalent-quality outputs.

### 4. The brother-DangerStorm peer review revisions (PR #4 commit `e35680c`) were load-bearing for both PASSes.

The "calibrated voice" and "removed invented patterns" revisions made the JD's voice transferable rather than over-specified. A version of the JD with the original "minimum 3 exchanges" hardening and the invented "Sharp/Clean/Got it" pattern would likely have produced different runtime behavior — possibly more rigid (hitting the floor exactly) and possibly more generic (using the invented patterns by rote). The peer review's lighter-touch revision allowed both runtimes to instantiate the role naturally.

---

## Implications for openjd v0.1+

1. **Cross-runtime portability is empirically validated.** Two architecturally different runtimes (Grok Expert multi-agent-web, Claude Code IDE-embedded) producing role-faithful behavior from the same JD is the strongest possible portability evidence at v0.1.

2. **The wrapper-prompt v2 pattern (explicit fetch) is the canonical loading approach** for refuse-to-improvise runtimes. Should be documented in the openjd README's "Loading a JD" section.

3. **The openjd-load Claude Code skill is now obviously the right next investment.** Two clicks (paste prompt, approve fetch) → DangerStorm-equivalent role behavior is already viable. A skill collapses that to one command.

4. **JD-runtime-context interaction is a real schema question for v0.1+.** Should JDs declare expected runtime-context scope? Should they declare allowed/disallowed enrichments?

5. **Conformance test results should be paired across runtimes.** The most useful evidence isn't "PASS on Runtime X" alone but "PASS on Runtimes X and Y, with the following tactical variance" — that demonstrates portability rather than runtime-specific success.

---

## Cross-references

- JD file: [`../../jds/senior-jaded-vc-associate.openthing`](../../jds/senior-jaded-vc-associate.openthing)
- JD inclusion decision: [`../../decisions/jd-senior-jaded-vc-associate.md`](../../decisions/jd-senior-jaded-vc-associate.md)
- Sister conformance evidence: [`./senior-jaded-vc-associate__grok-expert__2026-04-25.md`](./senior-jaded-vc-associate__grok-expert__2026-04-25.md)
- Conformance decision artifact: [`../../decisions/conformance-evidence-first-pass.md`](../../decisions/conformance-evidence-first-pass.md)
- Source-project peer review (load-bearing for this PASS): PR #4 attribution comment
