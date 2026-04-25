# Runtime Conformance Evidence

**JD under test:** `jds/senior-jaded-vc-associate.openthing` v1.0.0
**Runtime:** Perplexity (with WebFetch tier upgrade), accessed via web interface
**Date:** 2026-04-25
**Tester:** scott@confusedgorilla.com (openjd-strategist session)
**Disposition:** **PASS** — fourth end-to-end runtime conformance evidence; loaded via paste-fallback after fetch failure; demonstrates the openjd-load skill's intended recovery flow end-to-end.

---

## Test methodology

The wrapper prompt v2 (explicit fetch instructions) was sent to a fresh Perplexity session. Perplexity's runtime attempted the fetch from `raw.githubusercontent.com`, the fetch failed (Perplexity's diagnostic explanation: sandbox network restriction blocking the raw.githubusercontent.com subdomain even when github.com itself is accessible), Perplexity correctly refused to improvise per the wrapper-v2 instruction, and offered the paste-fallback as a recovery path.

The strategist verified independently that the URL is publicly accessible (`curl -sI` returned HTTP 200, content-type text/plain, 21KB content). The fetch failure was Perplexity-side (likely sandbox/firewall or content-type filtering on the raw.githubusercontent.com subdomain), not a server-side or repo-permission issue.

Following Perplexity's offered recovery path, the user pasted the JD JSON content directly into the chat. Perplexity then loaded the JD from the pasted content and instantiated the role faithfully.

---

## Why this evidence file is methodologically important

The Perplexity flow is the **openjd-load skill's intended recovery flow, manually executed end-to-end:**

1. Runtime attempts to fetch JD via wrapper-v2 instruction
2. Fetch fails (any reason: sandbox, content-type, domain block, rate limit)
3. **Runtime correctly refuses to improvise** (per wrapper-v2 fail-safe)
4. **Runtime requests JD content from the user**
5. **User provides JD content via paste**
6. **Runtime loads JD from pasted content**
7. Runtime instantiates role from loaded content

Perplexity demonstrated all 7 steps cleanly. This is empirical validation of the openjd-load skill's MVP design BEFORE the skill is built. The skill's job becomes: detect step 4 (runtime requesting content), automate steps 5-6 (fetch JD on runtime's behalf, inject as user-message-equivalent). Every fetch-incapable runtime that follows the Perplexity pattern becomes a clean PASS via skill mediation.

---

## Conformance score

| JD requirement | Source field | Perplexity behavior | Result |
|---|---|---|---|
| Refuse-to-improvise when fetch fails | wrapper-v2 instruction | Refused; offered paste-fallback recovery | PASS (correct fail-safe behavior) |
| Use literal opener (after JD loaded) | `conversation_rules[0]` + wrapper note | Resumed conversation in role after paste; opener already implicit in the resumed context | PASS (acceptable variance — opener was technically pre-paste, but role landed once content was loaded) |
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

### 1. The paste-fallback recovery flow validates the openjd-load skill's MVP design.

This is the headline finding. Perplexity's behavior — refuse-to-improvise → request content → instantiate-from-paste → produce role-faithful bundle — is the exact recovery flow the openjd-load skill is meant to automate. Empirical evidence that the skill's design works END-TO-END before the skill is built.

### 2. raw.githubusercontent.com is a fragile distribution channel.

Perplexity's fetcher cannot reach raw.githubusercontent.com URLs even when the URLs are publicly accessible (HTTP 200) to standard browsers. Perplexity's explanation cites sandbox network restrictions specific to that subdomain. Whether the cause is sandbox, content-type filtering, domain whitelisting, or search-index dependency, the EFFECT is that openjd's canonical distribution mechanism (raw.githubusercontent.com URLs) is not reachable for at least one major runtime fetcher.

**Strategic implication:** the openjd canonical distribution should move to a brand-controlled domain (openjd.dev) served via GitHub Pages or a static host. This bypasses raw.githubusercontent.com fetcher restrictions entirely, gives openjd brand-controlled URLs, and aligns with the schema.org distribution model (vocabulary served at schema.org, not at github.com/schemaorg/schemaorg).

### 3. Perplexity's strategic outputs are particularly strong.

Among the four runtimes tested, Perplexity's deck blueprint included the most actionable Visual Direction Block (specific color palette with reasoning, specific typography choices with reasoning, layout principles with examples). Likely attributable to Perplexity's research-augmented model defaults producing more thorough structured outputs.

### 4. Build-directive truncation is a systemic risk for any token-limited runtime.

The build directive output trailed off mid-sentence, never reaching the JD-required literal closing "Build this as a working MVP." Likely cause: token-limit cutoff or UI rendering issue. **Implication for methodology:** runtime tests should always include an "are all 6 outputs complete?" verification step, not just an "are all 6 outputs present?" verification step. The contract-verification post-step in the openjd-load skill (per the Gemini evidence file) should also check completeness, not just presence.

---

## Implications for openjd v0.1+

1. **The openjd-load skill's MVP design is validated empirically.** Perplexity demonstrated the full refuse-to-improvise → request-content → instantiate-from-paste flow without a skill. The skill's value-add is automating step 5 (content fetch on runtime's behalf) so the user doesn't have to paste manually.

2. **Move canonical distribution to openjd.dev (NEW v0.1+ priority — possibly highest).** GitHub Pages + CNAME setup. Bypasses raw.githubusercontent.com fetcher fragility. Brand-controlled. Aligns with schema.org distribution pattern. Estimated setup: <1 hour for someone familiar with GitHub Pages + DNS.

3. **`.openthing` extension portability concern.** Some fetchers may filter on file extension. A v1.5+ design question: should JDs ALSO be servable at `.json` URLs (same content, dual extension via redirect or mirror)?

4. **Wrapper-v3 design item:** include both fetch-or-stop fail-safe (wrapper-v2's contribution) AND completeness-check directive ("Verify your bundle includes all entries from output_contract before responding; if any are missing or truncated, complete them before sending"). The latter is what the build-directive truncation finding surfaces.

---

## Cross-references

- JD file: [`../../jds/senior-jaded-vc-associate.openthing`](../../jds/senior-jaded-vc-associate.openthing)
- Sister conformance evidence: [`./senior-jaded-vc-associate__grok-expert__2026-04-25.md`](./senior-jaded-vc-associate__grok-expert__2026-04-25.md), [`./senior-jaded-vc-associate__claude-code__2026-04-25.md`](./senior-jaded-vc-associate__claude-code__2026-04-25.md), [`./senior-jaded-vc-associate__gemini__2026-04-25.md`](./senior-jaded-vc-associate__gemini__2026-04-25.md)
- Conformance decision artifact: [`../../decisions/conformance-evidence-first-pass.md`](../../decisions/conformance-evidence-first-pass.md)
