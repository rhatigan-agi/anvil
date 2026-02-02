# Anvil Specification Schemas

**Version:** 0.2.0

This directory contains JSON Schema definitions for all data structures in the Anvil specification.

## Schema Files

### Core Pipeline Schemas

| Schema | Description |
|--------|-------------|
| [`intent.schema.json`](./intent.schema.json) | User intent structure |
| [`discovery-fact.schema.json`](./discovery-fact.schema.json) | Individual discovery fact |
| [`discovery-result.schema.json`](./discovery-result.schema.json) | Complete discovery phase result |
| [`issue.schema.json`](./issue.schema.json) | Issue with coverage validation (v0.2.0) |
| [`plan.schema.json`](./plan.schema.json) | Complete plan structure |
| [`phase.schema.json`](./phase.schema.json) | Individual plan phase |
| [`validation-result.schema.json`](./validation-result.schema.json) | Pre-execution validation result |
| [`execution-result.schema.json`](./execution-result.schema.json) | Execution phase result |
| [`governance-result.schema.json`](./governance-result.schema.json) | Post-execution governance result (v0.2.0) |

### Governance Schemas

| Schema | Description |
|--------|-------------|
| [`constraint.schema.json`](./constraint.schema.json) | Governance constraint definition |
| [`governance-stack.schema.json`](./governance-stack.schema.json) | Complete governance stack |
| [`domain-registry.schema.json`](./domain-registry.schema.json) | Canonical terminology registry (v0.2.0) |

### Provenance & Resolution Schemas

| Schema | Description |
|--------|-------------|
| [`provenance-chain.schema.json`](./provenance-chain.schema.json) | Complete provenance chain |
| [`user-resolution.schema.json`](./user-resolution.schema.json) | User resolution / type cast |
| [`event.schema.json`](./event.schema.json) | Timeline event |

## Usage

### Validation

Validate a JSON document against a schema:

```bash
# Using ajv-cli
npx ajv validate -s schemas/plan.schema.json -d my-plan.json

# Using jsonschema (Python)
jsonschema -i my-plan.json schemas/plan.schema.json
```

### Code Generation

Generate TypeScript types from schemas:

```bash
# Using json-schema-to-typescript
npx json-schema-to-typescript schemas/*.schema.json -o types/
```

Generate Python dataclasses:

```bash
# Using datamodel-codegen
datamodel-codegen --input schemas/ --output models.py
```

## Schema Versioning

All schemas include a `$id` field with the version:

```json
{
  "$id": "https://anvil-spec.dev/schemas/v0.2.0/plan.schema.json"
}
```

When validating, ensure you're using schemas that match your Anvil implementation version.

## Schema Relationships

```
intent.schema.json
       │
       ▼
discovery-result.schema.json ──► discovery-fact.schema.json
       │                                    │
       │◄─────── user-resolution.schema.json◄┘
       ▼
issue.schema.json
       │
       ▼
plan.schema.json ──► phase.schema.json
       │
       │◄── governance-stack.schema.json ──► constraint.schema.json
       ▼
validation-result.schema.json
       │
       ▼
execution-result.schema.json ──► provenance-chain.schema.json
       │
       ▼
governance-result.schema.json
       │
       ▼
event.schema.json
```

## v0.2.0 Changes

- **Added:** `issue.schema.json` — Coverage validation for lossless Intent→Issue transformation
- **Added:** `governance-result.schema.json` — Post-execution governance with fixability
- **Added:** `domain-registry.schema.json` — Canonical terminology enforcement
- **Renamed:** `plan-step.schema.json` → `phase.schema.json`
- **Renamed:** `type-cast.schema.json` → `user-resolution.schema.json`
- **Renamed:** `tier1-event.schema.json` → `event.schema.json`
- **Removed:** `tier2-pattern.schema.json`, `tier3-wisdom.schema.json`, `trust-assignment.schema.json` (moved to Anvil Complete, optional)

## License

These schemas are part of the Anvil Specification, licensed under CC BY 4.0.