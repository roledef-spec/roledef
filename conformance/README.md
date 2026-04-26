# conformance/

The roledef conformance test suite. The suite is the practical expression of the standard — a roledef that passes the suite is roledef-conformant; one that doesn't, isn't.

## Structure

```
conformance/
├── README.md                ← you are here
├── valid_roledefs/               ← roledefs that should pass validation
├── invalid_roledefs/             ← roledefs with specific known violations
└── tests/                   ← test code (validators, Turing-test runners)
```

## What the suite tests

Three dimensions of conformance:

### 1. Schema conformance

For each roledef:
- Does it parse as valid catdef?
- Does it have all MUST fields per [SCHEMA.md](../SCHEMA.md)?
- Does `output_contract` have at least one entry?
- Does `guardrails` have at least one entry?
- Are version stamps valid (catdef writer-strict per CA-002)?
- Are extensions properly self-describing per the SHOULD rule?

### 2. Turing-test fidelity

For each canonical roledef in `../roledefs/`:
- Does a fresh runtime loaded with the roledef produce behavior equivalent to the original-incumbent's documented behavior?
- Does cross-runtime testing pass (Claude AND Grok, where applicable)?

### 3. Validator self-validation

The `valid_roledefs/` and `invalid_roledefs/` corpora validate the validator itself:
- The validator MUST pass every roledef in `valid_roledefs/`
- The validator MUST fail every roledef in `invalid_roledefs/` with the documented reason
- This is the "structural enforcement beats editorial discipline" pattern from catdef CA-006 (`ft-shape-07`)

## Test fixture conventions

### `valid_roledefs/`

roledefs that should pass all schema checks. Use cases:
- Edge cases that the spec covers correctly (long fields, polymorphic translatable name, rich extension structure)
- Each MUST/SHOULD field exercised at minimum once across the corpus
- At least one roledef that uses every reserved namespace correctly

### `invalid_roledefs/`

roledefs with specific known violations. Each invalid roledef MUST:
- Be paired with a documentation file (`<id>.md`) explaining the violation
- Test exactly one violation type per roledef (so failure is diagnostic)
- Cover the major violation classes:
  - Missing MUST field
  - Empty MUST field
  - `output_contract` empty
  - `guardrails` empty
  - Reserved namespace misused
  - Self-contradictory rules
  - Catdef-stamp mismatch (writer-strict violation)
  - Extension without self-description (SHOULD violation, may be warning vs failure)

### `tests/`

Test code. Initial v0.1 contents:
- `test_schema_validation.py` — runs schema validation against `valid_roledefs/` (must pass) and `invalid_roledefs/` (must fail with documented reason)
- `test_turing.py` — runs Turing-test fixtures against the canonical library (manual or runtime-required)

## Adding to the conformance suite

When you submit a new roledef, you MAY include additions to the conformance suite if your roledef exercises new schema territory. Especially welcome:

- New invalid-roledef examples covering violations not yet in the corpus
- Edge-case valid-roledef examples that catch subtle cases
- Turing-test fixtures with documented reference behavior

The conformance suite grows organically as the schema matures.

---

*The test suite IS the standard. A roledef that passes is roledef-conformant.*
