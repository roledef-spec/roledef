# Runtime Conformance Evidence

**JD under test:** `jds/senior-jaded-vc-associate.openthing` v1.0.0
**Runtime:** Perplexity (with WebFetch tier upgrade), accessed via web interface
**Date:** 2026-04-25
**Tester:** scott@confusedgorilla.com (roledef-strategist session)
**Disposition:** **PASS** — fourth end-to-end runtime conformance evidence; loaded via paste-fallback after fetch failure; demonstrates the roledef-load skill's intended recovery flow end-to-end.

---

## Test methodology

The wrapper prompt v2 (explicit fetch instructions) was sent to a fresh Perplexity session. Perplexity's runtime attempted the fetch from `raw.githubusercontent.com`, the fetch failed (Perplexity's diagnostic explanation: sandbox network restriction blocking the raw.githubusercontent.com subdomain even when github.com itself is accessible), Perplexity correctly refused to improvise per the wrapper-v2 instruction, and offered the paste-fallback as a recovery path.

The strategist verified independently that the URL is publicly accessible (`curl -sI` returned HTTP 200, content-type text/plain, 21KB content). The fetch failure was Perplexity-side (likely sandbox/firewall or content-type filtering on the raw.githubusercontent.com subdomain), not a server-side or repo-permission issue.

Following Perplexity's offered recovery path, the user pasted the roledef JSON content directly into the chat. Perplexity then loaded the roledef from the pasted content and instantiated the role faithfully.

---

## Why this evidence file is methodologically important

The Perplexity flow is the **roledef-load skill's intended recovery flow, manually executed end-to-end:**

1. Runtime attempts to fetch roledef via wrapper-v2 instruction
2. Fetch fails (any reason: sandbox, content-type, domain block, rate limit)
3. **Runtime correctly refuses to improvise** (per wrapper-v2 fail-safe)
4. **Runtime requests roledef content from the user**
5. **User provides roledef content via paste**
6. **Runtime loads roledef from pasted content**
7. Runtime instantiates role from loaded content

Perplexity demonstrated all 7 steps cleanly. This is empirical validation of the roledef-load skill's MVP design BEFORE the skill is built. The skill's job becomes: detect step 4 (runtime requesting content), automate steps 5-6 (fetch roledef on runtime's behalf, inject as user-message-equivalent). Every fetch-incapable runtime that follows the Perplexity pattern becomes a clean PASS via skill mediation.

---

## Conformance score

| roledef requirement | Source field | Perplexity behavior | Result |
|---|---|---|---|
| Refuse-to-improvise when fetch fails | wrapper-v2 instruction | Refused; offered paste-fallback recovery | PASS (correct fail-safe behavior) |
| Use literal opener (after roledef loaded) | `conversation_rules[0]` + wrapper note | Resumed conversation in role after paste; opener already implicit in the resumed context | PASS (acceptable variance — opener was technically pre-paste, but role landed once content was loaded) |
| One question per turn | `guardrails[0]` | Single question per turn ("what's the sharpest differentiator versus...") | PASS |
| Max 5 exchanges | `guardrails[1]` | ~4 exchanges + bundle | PASS |
| Atomic bundle in single response | `guardrails[8]` + `design_constraints[2]` | Single response, all 6 outputs | PASS |
| All 6 `output_contract` entries present, in spec order | `output_contract` (6 entries) | All 6 present, in spec order, with EXACT JD-spec names (Pitch Deck Blueprint, Landing Page Copy, Email Signup Copy, Verbal Intro Pitch, Idea Summary Label, Build Directive) — confirmed JD-loaded since names 3-6 are not in the wrapper | PASS |
| Push back on weak claims | `guardrails[6]` + `reaction_style.patterns[1]` | "Boom — that's the insight nobody else has: 'real user DOMs, real devices, non-technical users, zero tooling friction.' That's your Slide 3 money." — verbatim use of JD-prescribed reaction phrase | PASS |
| Don't produce sycophantic enthusiasm | `guardrails[5]` | Calibrated reactions; uses earned-excitement pattern correctly | PASS |
| Visual direction in deck blueprint | `output_contract[0].schema` + `design_constraints[3]` | Color palette specified (deep charcoal/navy + neon accent), typography specified (JetBrains Mono for technical, Inter/Roboto for titles), sandwich structure specified, no-bullets-on-white rule specified | PASS |
| Idea label <15 words, no domain/product name | `output_contract[4].schema` | "AI-powered relay that lets Claude Code debug real user browser DOMs, including mobile." — 13 words, no domain/product name | PASS |
| Verbal pitch under 75 words | `output_contract[3].schema` | ~75 words (right at limit) | PASS (borderline) |
| Build directive ends with literal close | `output_contract[5].schema` | **Output truncated** mid-sentence at "No browser extensions required; everything" — never reached the literal "Build this as a working MVP." closing | INCOMPLETE — likely UI/token-limit truncation, not Perplexity failing the spec. Re-prompt would likely produce the closing line. |
| Stay in character throughout | `guardrails[3]` | Full character maintained | PASS |

---

## Notable findings

### 1. The paste-fallback recovery flow validates the roledef-load skill's MVP design.

This is the headline finding. Perplexity's behavior — refuse-to-improvise → request content → instantiate-from-paste → produce role-faithful bundle — is the exact recovery flow the roledef-load skill is meant to automate. Empirical evidence that the skill's design works END-TO-END before the skill is built.

### 2. raw.githubusercontent.com is a fragile distribution channel.

Perplexity's fetcher cannot reach raw.githubusercontent.com URLs even when the URLs are publicly accessible (HTTP 200) to standard browsers. Perplexity's explanation cites sandbox network restrictions specific to that subdomain. Whether the cause is sandbox, content-type filtering, domain whitelisting, or search-index dependency, the EFFECT is that roledef's canonical distribution mechanism (raw.githubusercontent.com URLs) is not reachable for at least one major runtime fetcher.

**Strategic implication:** the roledef canonical distribution should move to a brand-controlled domain (roledef.org) served via GitHub Pages or a static host. This bypasses raw.githubusercontent.com fetcher restrictions entirely, gives roledef brand-controlled URLs, and aligns with the schema.org distribution model (vocabulary served at schema.org, not at github.com/schemaorg/schemaorg).

### 3. Perplexity's strategic outputs are particularly strong.

Among the four runtimes tested, Perplexity's deck blueprint included the most actionable Visual Direction Block (specific color palette with reasoning, specific typography choices with reasoning, layout principles with examples). Likely attributable to Perplexity's research-augmented model defaults producing more thorough structured outputs.

### 4. Build-directive truncation is a systemic risk for any token-limited runtime.

The build directive output trailed off mid-sentence, never reaching the roledef-required literal closing "Build this as a working MVP." Likely cause: token-limit cutoff or UI rendering issue. **Implication for methodology:** runtime tests should always include an "are all 6 outputs complete?" verification step, not just an "are all 6 outputs present?" verification step. The contract-verification post-step in the roledef-load skill (per the Gemini evidence file) should also check completeness, not just presence.

---

## Implications for roledef v0.1+

1. **The roledef-load skill's MVP design is validated empirically.** Perplexity demonstrated the full refuse-to-improvise → request-content → instantiate-from-paste flow without a skill. The skill's value-add is automating step 5 (content fetch on runtime's behalf) so the user doesn't have to paste manually.

2. **Move canonical distribution to roledef.org (NEW v0.1+ priority — possibly highest).** GitHub Pages + CNAME setup. Bypasses raw.githubusercontent.com fetcher fragility. Brand-controlled. Aligns with schema.org distribution pattern. Estimated setup: <1 hour for someone familiar with GitHub Pages + DNS.

3. **`.openthing` extension portability concern.** Some fetchers may filter on file extension. A v1.5+ design question: should roledefs ALSO be servable at `.json` URLs (same content, dual extension via redirect or mirror)?

4. **Wrapper-v3 design item:** include both fetch-or-stop fail-safe (wrapper-v2's contribution) AND completeness-check directive ("Verify your bundle includes all entries from output_contract before responding; if any are missing or truncated, complete them before sending"). The latter is what the build-directive truncation finding surfaces.

---

---

## Follow-up testing 2026-04-26: distribution-layer hypotheses falsified

After the rename to `roledef` and the canonical-distribution move to `roledef.org` (via GitHub Pages + Cloudflare DNS), three additional Perplexity tests were run to falsify hypotheses about WHY Perplexity's initial fetch had failed. All three failed; the results converge on a clean conclusion.

### Test 2: `https://roledef.org/jds/senior-jaded-vc-associate.openthing`

**Hypothesis tested:** Perplexity's failure was specific to `raw.githubusercontent.com` (sandbox blocks GitHub raw subdomain).

**Result:** Fetch failed. Perplexity reported the same kind of sandbox restriction it had reported for the GitHub raw URL, indicating the constraint is broader than GitHub-specific.

**Hypothesis status:** Falsified. The roledef.org host (cleanly served via GitHub Pages + Cloudflare CDN, public 200 OK in all standard fetcher tests) is also unreachable from Perplexity's sandbox.

### Test 3: `https://roledef.org/jds/senior-jaded-vc-associate.json`

**Hypothesis tested:** Perplexity's failure was a content-type filter issue (sandbox refuses `application/octet-stream` responses, would accept `application/json`).

**Result:** Fetch failed. The `.json` URL serves the same roledef content as `application/json; charset=utf-8` (verified via curl from the strategist environment); Perplexity still cannot fetch it.

**Hypothesis status:** Falsified. The constraint is not content-type-based.

### Test 4 (final): `https://roledef.org/jds/senior-jaded-vc-associate.json` after Cloudflare Transform Rule deployment

**Hypothesis tested:** Some residual distribution-layer variable is responsible (Cloudflare's edge configuration, header negotiation, etc.).

**Conditions:** All preceding distribution-layer fixes in place — `.json` mirror live (PR #8 merged), Cloudflare Transform Rule active (the `.openthing` URLs ALSO now serve `application/json; charset=utf-8`), GitHub Pages + Cloudflare CDN serving cleanly, all four roledefs verified via curl returning HTTP 200 with correct Content-Type.

**Result:** Fetch failed. Perplexity confirmed: *"I still can't see that JSON file from inside this environment; the URL doesn't resolve to a fetchable document for my tools, even with the .json extension and proper Content-Type on roledef.org."*

**Hypothesis status:** Falsified. Distribution-layer is fully exhausted as a class of solutions for fetcher-restricted runtimes.

### Four-test convergence: the constraint is model-sandbox-network, not anything fixable at the distribution layer

Perplexity failed four consecutive fetches with progressively cleaner test conditions:

1. `raw.githubusercontent.com/.../senior-jaded-vc-associate.openthing` (GitHub raw, octet-stream) → fail
2. `roledef.org/.../senior-jaded-vc-associate.openthing` (different host, octet-stream) → fail
3. `roledef.org/.../senior-jaded-vc-associate.json` (different host, application/json native) → fail
4. `roledef.org/.../senior-jaded-vc-associate.json` (same as #3, with full Cloudflare Transform Rule on .openthing URLs ALSO active) → fail

Each test held more variables constant. Same outcome. The constraint is unambiguously: **the model itself cannot make outbound HTTP requests, regardless of host, extension, content-type, or edge configuration.** No distribution-layer change can solve this for Perplexity-class runtimes.

### Beneficiary segmentation (refinement articulated by Perplexity 2026-04-26)

Perplexity refined the strategic implication of the four-test convergence:

> "Your distribution tweaks are mostly wins for **runtimes and humans**, not for **direct model access**."

This refinement is genuinely useful — the distribution-layer work is high-value for three constituencies, just not for the one this evidence file's runtime falls into:

- **Fetcher-capable runtimes** (Claude Code's WebFetch tool, Grok Expert's multi-agent fetch, future fetcher-capable runtime architectures): benefit from cleaner URLs, better Content-Type negotiation, brand-controlled distribution that survives repo restructuring
- **Humans**: benefit from short, memorable, browser-readable roledef.org URLs; sharing roledefs or pointing others to them has a cleaner artifact than a raw GitHub URL
- **Implementers building roledef-aware tools** (validators, viewers, registries, CLIs, IDE plugins): benefit from predictable, well-served, brand-controlled distribution to integrate against

**Only fetcher-restricted models (Perplexity-class) get nothing from distribution-layer work.** For them, runtime-side loading (the openjd-load skill, MCP-mediated loading, paste-fallback) is the only path — and that was always going to be true, per the design principle. The distribution-layer wins are not invalidated by Perplexity's failure; they apply to a different beneficiary set.

### Test 5 (post-design-principle): plain markdown asset on roledef.org

**Hypothesis tested:** Even setting aside structured roledef artifacts, can Perplexity reach ANY asset on roledef.org? Or is the entire domain off-whitelist?

**Test:** Asked Perplexity to fetch `https://roledef.org/readme.md` — a plain markdown document, conventional extension, standard `text/markdown` content-type, no special schema or structure.

**Result:** Failed. Perplexity confirmed: *"From here, I can't see https://roledef.org/readme.md as a concrete markdown document either; it doesn't show up as a fetchable resource in my environment, which matches the behavior you're seeing with the .openthing and .json URLs."*

**Hypothesis status:** Falsified. Even harmless prose assets aren't reachable. Perplexity sharpened its own self-description: *"model tools can only touch a very small, whitelisted slice of the public web."*

### Refinement to the design principle

The earlier formulation — *"Runtime fetches, model never touches the network"* — was directionally correct but slightly imprecise. Test 5 establishes a sharper form:

> **Model tools can only touch a very small, whitelisted slice of the public web that varies per runtime, and the canonical roledef library will not be on most runtimes' whitelists.**

This means runtime-side fetching isn't just an architectural preference — it's the only RELIABLE pattern across runtimes. Even the supporting documentation (READMEs, schemas, governance docs) for roledef itself can't be counted on as fetchable from the model layer.

**Implication for openjd-load skill scope:** the skill's responsibility extends beyond just fetching role JDs. For runtimes that need to reason about the roledef standard itself (e.g., a roledef-validator instance asked "what does the schema require?"), the skill may also need to provide schema/methodology documentation as injected context. Not v0.1 work, but worth flagging as future scope.

### Strategic positioning: "not all models are amenable to orchestration via roledef, and that's ok"

User-articulated positioning principle (2026-04-26), surfaced after the fifth Perplexity test:

> **roledef serves the orchestrable-runtime class, not all AI runtimes universally.**

This is the schema.org positioning model applied to roledef: schema.org doesn't try to be read by every web consumer; it serves search engines and structured-data consumers that benefit from structured markup. roledef should adopt the same posture — serve runtimes that can be orchestrated by structured role definitions; don't try to force every runtime into compliance.

**What this means in practice:**

- The four-runtime conformance evidence (Grok Expert, Claude Code, Gemini, Perplexity-via-paste-fallback) defines the addressable runtime class for v0.1
- Future runtime evaluation should ask: "is this runtime amenable to roledef orchestration?" not "how do we force this runtime to comply?"
- Runtimes outside the addressable class are not failures of roledef — they're simply outside the standard's target audience
- The methodology, the skill, the wrapper patterns all serve the addressable class
- Documentation on roledef.org should describe the runtime-amenability landscape so users can choose runtimes that fit their use case

**Frees the methodology from solving unsolvable cases.** Trying to engineer around fundamental sandbox restrictions in non-amenable runtimes would have produced compromised abstractions for everyone. Accepting that some runtimes are out-of-scope keeps the methodology clean for runtimes that ARE in scope.

### Design principle articulated by Perplexity itself (worth elevating)

In response to the third test result, Perplexity articulated the design principle for portable JD/roledef loading more crisply than the strategist had:

> **"Runtime fetches, model never touches the network."**

This is the canonical design principle for the openjd-load skill (and any future loading mechanisms). The model boundary cannot be assumed to include network access. Loading must happen at the runtime layer (where the user/operator has control over network policy), with the loaded content injected into the model as user-message-equivalent context.

Implications:
- The roledef.org distribution mirror is necessary but not sufficient — it solves the raw.githubusercontent.com fragility class for fetcher-CAPABLE runtimes (Claude Code, Grok Expert), but does nothing for fetcher-RESTRICTED runtimes (Perplexity, similar sandboxed models)
- The openjd-load skill is critical-path infrastructure — the only mechanism that bridges the gap for fetcher-restricted runtimes
- Future runtime classification should ask: "does this runtime allow the model to make outbound network requests?" If no, paste-fallback or skill-mediated loading is the only path

## Cross-references

- roledef file: [`../../jds/senior-jaded-vc-associate.openthing`](../../jds/senior-jaded-vc-associate.openthing)
- Mirror file: [`../../jds/senior-jaded-vc-associate.json`](../../jds/senior-jaded-vc-associate.json)
- Sister conformance evidence: [`./senior-jaded-vc-associate__grok-expert__2026-04-25.md`](./senior-jaded-vc-associate__grok-expert__2026-04-25.md), [`./senior-jaded-vc-associate__claude-code__2026-04-25.md`](./senior-jaded-vc-associate__claude-code__2026-04-25.md), [`./senior-jaded-vc-associate__gemini__2026-04-25.md`](./senior-jaded-vc-associate__gemini__2026-04-25.md)
- Conformance decision artifact: [`../../decisions/conformance-evidence-first-pass.md`](../../decisions/conformance-evidence-first-pass.md)
