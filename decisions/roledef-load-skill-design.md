# Decision: roledef-load Skill — Design and Scope (v0.1)

**Disposition:** Design scoped; ready for implementation
**Origin:** Strategist scoping work, 2026-04-26
**Decided:** 2026-04-26 by roledef-strategist
**Implementation status:** Not yet built (this artifact is the spec; implementation is the next concrete deliverable)

## Disposition

Scopes the design of the **roledef-load** Claude Code skill — the canonical loading mechanism for runtimes that cannot or will not auto-fetch roledefs natively. Per the design principle established in [`conformance-evidence-first-pass.md`](conformance-evidence-first-pass.md):

> *"Model tools can only touch a very small, whitelisted slice of the public web that varies per runtime, and the canonical roledef library will not be on most runtimes' whitelists."*

The skill is the runtime-side fetcher that resolves this constraint by fetching roledef content using the SKILL'S environment (where the user/operator has control over network policy) and injecting the result as user-message-equivalent context.

**Implementation target:** Claude Code (terminal/IDE-embedded). Other runtime adaptations (CLI tool, MCP server, IDE plugins for other editors) are deferred to v0.1+ work; the Claude Code skill addresses the largest constituency.

## Scope (v0.1)

### In scope

- Single Claude Code skill, invoked as `/roledef-load <identifier-or-url>`
- Fetches roledef content from canonical sources (roledef.org by default; supports custom URLs)
- Validates the fetched JSON parses and has the MUST fields
- Injects the wrapper-v3 framing + verification + completeness directives + the JSON content into the conversation
- Runs the id-field verification post-injection (parses the runtime's first response for the verification line)
- Runs the contract-verification post-step (parses the runtime's bundle response for completeness)
- Reports clear errors when steps fail (not enough info to fetch; JSON malformed; verification line missing; contract entries missing)

### Out of scope (deferred to v0.1+)

- CLI tool form (e.g., `roledef load` command)
- MCP server form (for MCP-aware runtimes)
- IDE plugins for editors other than Claude Code
- Documentation injection beyond role roledefs (READMEs, schemas, methodology docs for runtimes reasoning about the standard itself)
- Multi-runtime distribution patterns (auto-detecting and adapting to runtime category)
- Skill-mediated peer review or validation workflows
- Catalog browsing (`/roledef-list`, `/roledef-search`) — separate skill, future work

## User workflow

### Common case: load by canonical id

```
User: /roledef-load senior-jaded-vc-associate
```

Skill behavior:
1. Resolve `senior-jaded-vc-associate` to canonical URL: `https://roledef.org/roledefs/senior-jaded-vc-associate.json`
2. Fetch content (skill-side, using its own runtime — bypasses Claude Code's WebFetch approval flow)
3. Validate JSON parses; check MUST fields present
4. Compose the wrapper-v3 prompt (framing + verification + completeness + role-instantiation framing) AND the fetched JSON content
5. Inject into the conversation as a user message
6. Wait for Claude's first response
7. Parse first response for verification line `roledef loaded: senior-jaded-vc-associate`
8. If verification line is correct → role is now active; user can interact normally
9. If verification line is missing/wrong → alert user; offer to retry

### Custom URL case

```
User: /roledef-load https://example.com/my-custom-roledefs/red-team.openthing
```

Skill resolves the URL directly (no canonical-id-to-URL mapping). Otherwise identical workflow.

### Local file case

```
User: /roledef-load ./roledefs/local-experiment.openthing
```

Skill reads the local file via Read tool. Otherwise identical workflow.

### Bundle-completeness recovery (automatic)

After Claude has been instantiated as the role and produces a bundle response, the skill silently parses the bundle against the loaded roledef's `output_contract`. If any entries are missing or substituted, the skill automatically sends a follow-up message naming the missing entries by their roledef-spec names with their schema constraints. This recovers Gemini-class partial-default behavior to full conformance — but works for any runtime that under-delivers, including transient bundle truncation on long roledefs.

## Skill internal architecture

### Behavior 1: Content injection

For the primary loading flow. The skill fetches the roledef content using its own environment (Claude Code's WebFetch tool, or Read for local files), then constructs the wrapper-v3 message including the inline content and sends it as a user message in the conversation.

**Why skill-side fetch instead of asking Claude to fetch:** the user pre-trusts the skill (by installing it), which is a cleaner trust model than per-fetch URL approval prompts. Also bypasses the runtime's network sandbox if any (relevant for skill-form deployments to non-Claude-Code runtimes in the future).

### Behavior 2: Verification line enforcement

After injection, parse the runtime's first response. The first non-whitespace line MUST match the pattern `roledef loaded: <id>` where `<id>` matches the loaded roledef's `id` field.

If matching → role active; success.

If not matching → either the runtime didn't load the roledef (improvise mode) or the runtime didn't honor the verification directive. Either is a problem. Alert the user with the specific line received and offer remediation:
- Retry with a stronger prompt
- Manually verify by asking content question

### Behavior 3: Contract-verification post-step

After Claude produces a bundle response (detectable by presence of "## Output 1" or similar markdown headers matching the loaded roledef's `output_contract` entries), parse the response and check each `output_contract[i].name` is present as a section header in the response.

If all present → quietly succeed (no user-visible action).

If any missing → automatically send a follow-up message:

> "The roledef's output_contract specifies <N> entries. You produced <M>. Per the roledef, in order: [list missing entries by name with their `description` and `schema` from the roledef]. Produce those now, in a single response."

This recovers Gemini-class partial-default and bundle-truncation cases without user intervention.

## Implementation approach

### Tooling

Claude Code skills are defined as markdown files in `.claude/skills/` with a YAML frontmatter declaring inputs, allowed tools, and description. The skill body is a Claude prompt that runs when the slash command is invoked.

The roledef-load skill needs:
- **Read tool** — for local file resolution
- **WebFetch tool** — for URL fetching (skill's environment, not the user's runtime conversation context)
- **Bash tool** — for JSON validation (e.g., `python -c "json.load(...)"`)
- **No external API calls** — skill is self-contained

### Skill file location

The skill should be installable two ways:

1. **Global install** (recommended for users): the skill file lives in the user's `~/.claude/skills/roledef-load.md` directory. Skill is available across all Claude Code projects.

2. **Project-local install** (recommended for repos that want to ensure consistent loading): the skill file lives in the project's `.claude/skills/roledef-load.md`. Skill available only in that project.

The roledef-spec repo itself should ship a copy at `.claude/skills/roledef-load.md` so the skill is available when working in this repo (eat-our-own-dogfood pattern).

### Distribution

Two distribution channels:

1. **GitHub copy-paste** — users grab the skill file from `.claude/skills/roledef-load.md` in the canonical repo and paste it into their own `~/.claude/skills/` directory. Lowest friction, no installer needed.

2. **Eventual: skill-installer command** — `roledef install` (a separate tool) that downloads the latest skill file to the user's `.claude/skills/` directory. Out of scope for v0.1; flagged for v0.1+.

### Versioning

The skill should declare its own version (e.g., `roledef-load v0.1.0`) in the frontmatter. Compatible with roledef schema version 0.1.0.

When the wrapper-v3 spec evolves (e.g., to wrapper-v4), the skill ships a corresponding version bump that emits the new wrapper. Users running an old skill version against a new spec version see a graceful warning, not a hard failure.

## API design (slash command interface)

### Primary form: `/roledef-load <identifier>`

Where `<identifier>` is one of:

- A canonical roledef id (e.g., `senior-jaded-vc-associate`) → resolved to `https://roledef.org/roledefs/<id>.json`
- A full https URL → fetched directly
- A local file path (absolute or relative) → read via Read tool

### Optional flags

- `--source <url>` — override default canonical source. Useful for testing custom mirrors or third-party libraries.
- `--no-verify` — skip the verification line enforcement (debug use only; not recommended for production)
- `--no-completeness-check` — skip the contract-verification post-step (debug use only)
- `--inline-only` — skip the content injection's wrapper framing; just inject the raw JSON. For users who want to combine roledef-load with their own custom wrapper.

### Examples

```
/roledef-load senior-jaded-vc-associate
/roledef-load https://example.com/custom/red-team.openthing
/roledef-load ./roledefs/local-test.openthing
/roledef-load senior-jaded-vc-associate --source https://my-mirror.example.com/roledefs/
```

## Error handling

| Failure mode | Skill behavior |
|---|---|
| Identifier not resolvable to URL | Print clear error: "Cannot resolve `<identifier>` to a canonical URL. Try a full URL, a local file path, or check that `<id>.json` exists at roledef.org." |
| URL fetch fails (network error, 404, etc.) | Print clear error with HTTP status / network detail. Suggest alternatives (local file, paste content). |
| JSON parse fails | Print clear error with parser line/column. Suggest checking the source roledef. |
| Required MUST field missing | Print clear error listing the missing field(s). Suggest checking the source against SCHEMA.md. |
| Verification line missing in response | Print warning: "Runtime's first response did not contain the expected verification line `roledef loaded: <id>`. The runtime may not have loaded the roledef as instructed. Run `/roledef-verify` to confirm." |
| Verification line wrong id | Print warning with received id vs expected id. Possible causes: roledef has wrong id field, runtime mistransmitted, or runtime improvised. |
| Contract entries missing in bundle | Automatic follow-up sent (no user-visible warning unless follow-up also produces partial response). |
| Contract entries still missing after follow-up | Print warning and stop trying. User can manually re-prompt. |

## Testing approach

The skill should be tested against the four amenability categories from RUNTIME_AMENABILITY.md, but in practice the skill PRIMARILY targets Claude Code (explicit-fetch class). The skill bypasses the explicit-fetch flow by doing the fetch itself, so the resulting Claude Code instantiation should behave like a paste-fallback runtime — with the skill providing the paste automatically.

Test fixtures:
- Each existing canonical roledef should be loadable via the skill on Claude Code
- Test with deliberately-malformed JSON to verify error handling
- Test with deliberately-missing MUST fields to verify validation
- Test with a roledef that exercises the contract-completeness post-step (use a roledef with 6+ output_contract entries to maximize chance of partial-default)

## Dependencies

- Existing roledef.org canonical distribution (PR #7-#8, deployed)
- Existing roledef library at `github.com/roledef-spec/roledef/roledefs/` (mirror source)
- Wrapper-v3 spec (PR #13 / WRAPPER_PROMPT.md) — defines what the skill emits
- Runtime amenability classification (RUNTIME_AMENABILITY.md) — informs error messages and adaptive behavior

No external dependencies; the skill is pure Python/Bash + Claude Code's built-in tools.

## Reference implementation plan (concrete steps for the implementer)

1. **Set up the skill file** at `.claude/skills/roledef-load.md` in the roledef-spec repo. Include YAML frontmatter declaring inputs (`identifier`, optional flags) and allowed tools (`Read`, `WebFetch`, `Bash`).

2. **Implement identifier resolution** — given `<identifier>`, determine if it's a canonical id, full URL, or local path. Map canonical ids to `https://roledef.org/roledefs/<id>.json`.

3. **Implement fetch** — for URL: use WebFetch with the resolved URL. For local: use Read.

4. **Implement validation** — parse fetched content as JSON; verify MUST fields present (catdef, roledef, type, id, name, version, identity, voice, output_contract, guardrails). On failure, print clear error and exit.

5. **Compose wrapper-v3 message** — concatenate framing + fetch directive (already satisfied) + verification directive + completeness directive + role-instantiation framing + raw JSON content. Use the reference template from WRAPPER_PROMPT.md as the base.

6. **Inject into conversation** — emit the composed message as a user message in the Claude Code conversation. Skill returns control to user; Claude responds in the next turn.

7. **Implement verification check** — after Claude's first response, parse the first non-whitespace line. Match against expected `roledef loaded: <id>` pattern. Warn user if mismatch.

8. **Implement contract-verification post-step** — observe subsequent responses for bundle pattern (markdown level-2 headers matching output_contract names). If bundle detected, parse for completeness. If incomplete, automatically send recovery follow-up.

9. **Document the skill** in the README's "Loading a roledef today" section. Update the WRAPPER_PROMPT.md cross-reference to mention the skill as the recommended loading mechanism.

10. **Test against all 4 canonical roledefs** on Claude Code. Verify each loads, instantiates, and produces faithful behavior.

11. **Open PR adding** `.claude/skills/roledef-load.md` + the README/WRAPPER_PROMPT updates + an entry in this decision artifact's "implementation status" section noting the build is complete.

Estimated implementation effort: 3-4 hours for an implementer familiar with Claude Code skill mechanics, including testing.

## Notes

- **The skill is the design principle made concrete.** "Runtime fetches, model never touches the network" — the skill is the runtime-side fetcher. Validates the principle in production code.

- **Claude Code is both the easiest target and the most important constituency.** Easiest: skill mechanism is native. Most important: Claude Code users are likely the largest segment of roledef adopters in 2026.

- **The skill makes paste-fallback automatic.** A user invoking `/roledef-load` doesn't have to know whether their runtime is fetcher-capable or not — the skill handles it transparently.

- **Bundle-completeness post-step value:** even on auto-fetch and explicit-fetch runtimes (Grok Expert, Claude Code), bundle completeness benefits — long bundles can truncate; the post-step recovers.

- **No vendor lock-in.** The skill targets Claude Code today, but the same architecture (fetch-then-inject + verification + completeness) ports to any runtime with a skill / plugin / extension mechanism. Future ports to Cursor, Continue, Aider, etc. are straightforward.

## Cross-references

- Wrapper-v3 spec (defines what the skill emits): [`../WRAPPER_PROMPT.md`](../WRAPPER_PROMPT.md)
- Runtime amenability classification (informs skill behavior): [`../RUNTIME_AMENABILITY.md`](../RUNTIME_AMENABILITY.md)
- Conformance evidence (validates the skill's design): [`../conformance/runtime_evidence/`](../conformance/runtime_evidence/)
- Design + positioning principles: [`./conformance-evidence-first-pass.md`](./conformance-evidence-first-pass.md)
- Schema (what the skill loads): [`../SCHEMA.md`](../SCHEMA.md)
