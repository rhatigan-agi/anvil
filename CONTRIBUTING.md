# Contributing to Anvil

Anvil is an open specification. Contributions that improve clarity, coverage, and adoption are welcome.

## Ways to Contribute

### 1. Feedback on the Spec

Open an issue for:

- **Ambiguities** — Places where the spec is unclear or could be interpreted multiple ways
- **Gaps** — Scenarios the spec doesn't address
- **Contradictions** — Inconsistencies between sections
- **Edge Cases** — Situations where the defined behavior seems wrong

### 2. Governance Patterns

Have domain-specific governance rules that others might use?

- Healthcare/HIPAA compliance rules
- Financial/SOX audit requirements
- Security-focused constraints
- Open source contribution policies

Share them as a Discussion or PR to `/examples/governance-patterns/`.

### 3. Schema Improvements

The JSON schemas in `/schemas/` define the artifact formats. Contributions welcome for:

- Validation improvements
- Missing fields for real-world use
- Better documentation within schemas
- JSON Schema best practices

### 4. Build an Implementation

The best contribution is a working implementation. If you're building an Anvil-compliant tool:

1. Open an issue to let us know
2. We'll add you to the Implementations table in the README
3. Share what works and what's underspecified

### 5. Documentation

- Clearer explanations
- More examples
- Tutorials for implementers
- Translations

## Pull Request Process

1. Fork the repo
2. Create a branch (`feature/your-change` or `fix/your-fix`)
3. Make your changes
4. Ensure schemas still validate (if modified)
5. Submit PR with clear description of what and why

For spec changes, explain:
- What problem does this solve?
- What's the backward compatibility impact?
- Are there alternative approaches?

## What We're Not Looking For

- Changes that break existing implementations without strong justification
- Vendor-specific extensions to core spec (propose as optional extensions instead)
- Complexity without clear benefit

## Versioning

Anvil uses semantic versioning for the spec:

- **Patch** (0.2.x): Clarifications, typo fixes, non-breaking schema additions
- **Minor** (0.x.0): New optional features, backward-compatible changes
- **Major** (x.0.0): Breaking changes (rare, with migration guide)

## Code of Conduct

Constructive, good-faith engagement. Critique ideas, not people.

## Questions?

Open a Discussion or reach out to [@rhatigan](https://github.com/rhatigan-agi).

---

*[Archon](https://github.com/rhatigan-agi/archon) is the reference implementation. For implementation-specific feedback, contribute there.*
