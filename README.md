<div align="center">

# Anvil

### A Specification for Deterministic AI-Assisted Development

**The AI is not trusted. The compiler is.**

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Spec Version](https://img.shields.io/badge/spec-v0.2.0-blue.svg)](#)
[![Status](https://img.shields.io/badge/status-stable-green.svg)](#)

[The Problem](#the-problem) • [Core Insight](#the-core-insight) • [Architecture](#architecture) • [Governance](#the-governance-algebra) • [Implementations](#implementations)

</div>

---

## Why a Specification?

Every AI coding tool invents its own workflow. Cursor does chat-driven iteration. Copilot does inline completion. Aider does git-aware agents. Result: no interoperability, no shared governance, no portability.

**Anvil provides:**
- **Standard phases**: Tools can integrate at any phase boundary
- **Portable governance**: Same rules across all tools
- **Verifiable claims**: Discovery facts are tool-agnostic
- **Audit trails**: Events follow a standard schema

Think of it as LLVM IR for AI workflows. Multiple frontends (tools), one rigorous pipeline.

---

## The Problem

AI code generation has created a verification gap. The time spent verifying AI-generated code has eclipsed the time saved generating it.

The failures are concrete:

- **Hallucinated references** — File paths and functions that don't exist
- **Schema violations** — Wrong assumptions about types and relationships
- **Architectural drift** — Changes that violate ownership boundaries
- **Domain fragmentation** — Synonyms that fracture codebase consistency
- **Scope creep** — "Fixing" unrelated files without authorization

Current approaches rely on behavioral constraints—prompting models to follow rules. This is fundamentally fragile.

**Anvil takes a different approach:** treat AI as an untrusted compiler component operating within a verification pipeline. Governance becomes structural, not behavioral.

---

## The Core Insight

A traditional compiler doesn't have "vibes." It has invariants.

| Compiler Concept | Anvil Equivalent |
|------------------|------------------|
| Source code | Intent |
| Lexer/Parser | Intent → Issue → Plan transformation |
| **Linker** | **Multi-pass Discovery** |
| Type checker | Governance engine |
| Symbol table | Domain registry |
| Code generation | Execution packets |

Anvil applies compiler discipline to AI-assisted development:

> The AI cannot reference what hasn't been discovered. It cannot execute what hasn't been validated. Every output traces to its origins.

**A prompt can be ignored. A compiler invariant cannot.**

---

## Architecture

Anvil defines a sequential pipeline where each phase constrains the next.

```
┌─────────┐   ┌───────────┐   ┌───────┐   ┌──────┐   ┌──────────┐   ┌─────────┐   ┌────────┐
│ INTENT  │──▶│ DISCOVERY │──▶│ ISSUE │──▶│ PLAN │──▶│ VALIDATE │──▶│ EXECUTE │──▶│ GOVERN │
└─────────┘   └───────────┘   └───────┘   └──────┘   └──────────┘   └─────────┘   └────────┘
                                                                                       │
                                                                         ┌─────────────┘
                                                                         ▼
                                                                 ┌────────────┐
                                                                 │   REPAIR   │←─┐
                                                                 └────────────┘  │
                                                                         │       │
                                                                         └───────┘
```

### Core Guarantees

These are compiler invariants, not aspirations:

```
1. No execution without intent       → Full traceability
2. No planning without discovery     → Grounding before action
3. No file references without proof  → Hallucination prevention
4. No violations without fixability  → Every error is classified
5. No silent governance bypass       → All checks logged
6. Deterministic routing             → Same input = same output
```

---

## Phase 1: Intent

An **Intent** is a natural language description of desired change, transformed into a structured representation.

```
intent = (id, description, timestamp, user)
```

Intents are the root of all provenance chains. Every downstream artifact traces back to an intent.

---

## Phase 2: Discovery (The Linker)

Discovery extracts ground truth from the codebase through deterministic analysis. It serves as the "linker"—establishing facts that constrain all subsequent AI operations.

### 4-Pass Discovery

| Pass | Purpose | Method |
|------|---------|--------|
| **Pass 1** | Framework Detection | Signal-based detection with confidence scores |
| **Pass 2** | Surface Discovery | AST-scanned models, views, components (AUTHORITATIVE) |
| **Pass 3** | Ownership Binding | Which file *executes* vs *renders* the intent |
| **Pass 4** | Schema Sufficiency | Does the schema support this intent? **(DETERMINISTIC, non-LLM)** |

**Note:** Pass 4 is purely deterministic—AST comparison only, no LLM interpretation. If it determines `migration_required: false`, no downstream phase can propose migrations.

### The Linker Principle

**If a fact cannot be verified through discovery, it is not available to planning.**

A Discovery Fact is a tuple:

```
fact = (claim, evidence, confidence, source)
```

- **claim**: Human-readable assertion ("User.email is CharField")
- **evidence**: Method and raw data that proves it
- **confidence**: `absolute` (AST) | `high` (static analysis) | `signal` (heuristic)
- **source**: `deterministic` | `framework_scanner` | `user_resolution`

When AST scanning determines a field is a `CharField`, that's **absolute truth**. The AI cannot override it.

### User Resolution

When genuine ambiguity exists—multiple valid placements, unclear ownership—the system pauses.

```
resolution = (ambiguity, options[], user_choice, scope)
```

The user's choice is a **type cast**: it becomes the only valid option, pruning all alternatives.

---

## Phase 3: Issue

Issues transform intents into trackable work units with coverage guarantees.

```
issue = (intent_id, discovery_id, concerns[], coverage_validated)
```

### Issue Coverage Invariant

The transformation from Intent → Issue must be **lossless** with respect to work scope.

If an intent mentions both backend AND frontend work, the issue MUST address both—or explicitly defer one with justification. This prevents AI systems from silently dropping complexity.

```
Intent: "Add NPI lookup [backend] with button [frontend]"
Issue:  Only describes backend API changes
→ REJECT: Frontend concern dropped without justification
```

---

## Phase 3.5: Frontend Preservation (v0.2.0)

If Discovery detects `is_new_frontend_interaction = true` or finds `interaction_anchors[]`, downstream phases MUST preserve this information.

Frontend work is expensive to specify. Silently dropping it forces users to re-specify from scratch. Anvil treats this as a compiler bug, not acceptable optimization.

---

## Phase 4: Planning

Planning generates proposed changes constrained by discovery facts.

```
plan = (issue_id, discovery_id, phases[], grounding_report)
```

A **Phase** is an atomic unit of work:

```
phase = (sequence, description, file_targets[], dependencies[])
```

### Grounding Requirement

Every file reference, schema assumption, and architectural claim must trace to a discovery fact. 

**Grounding ratio < 1.0 → Plan rejected.**

---

## Phase 5: Validation (Pre-Execution)

Validation runs deterministic checks before any code is generated:

| Check | Failure Mode |
|-------|--------------|
| File references exist in discovery | Reject plan |
| Backend phases before frontend | Repair or reject |
| Schema sufficiency respected | Block migration proposals if not needed |
| No hedge language ("if exists") | Repair or reject |

Validation is **non-LLM**. Same plan + same discovery = same verdict.

---

## Phase 6: Execution

Execution applies the plan, generating code changes.

**Constraints:**
- Only files in declared scope may be modified
- Changes are atomic and reversible
- Every change includes provenance

### Packet Mode

For integration with external tools (Claude Code, Cursor, etc.), plans can be packaged as **execution packets**—self-contained documents for handoff.

---

## Phase 7: Governance (Post-Execution)

Governance evaluates execution results against deterministic rules.

```
verdict = evaluate(execution_result, governance_stack)
verdict ∈ {pass, fail}
```

### The Fixability Contract

Every violation is classified:

| Classification | Behavior |
|----------------|----------|
| **AUTO** | System repairs automatically |
| **HUMAN** | Workflow pauses for decision |
| **NEVER** | Abort—fundamental violation |

**NEVER is absolute.** The system cannot proceed. No override exists.

### Repair Loop

If violations are fixable (AUTO or HUMAN), the system enters a repair loop:

```
execute → govern → repair → govern → ... → pass | abort
```

The loop continues until all violations are resolved or a NEVER violation is encountered.

---

## The Governance Algebra

Anvil's governance layer is a formal system with defined properties.

### Constraint Definition

```
constraint = (id, scope, predicate, tier, fixability)
```

### Standard Rules

| ID | Description | Fixability |
|----|-------------|------------|
| **GOV-001** | No new models without explicit intent | HUMAN |
| **GOV-002** | No auth/security changes without security intent | HUMAN |
| **GOV-003** | No production infra mutation from non-infra intent | HUMAN |
| **GOV-004** | No test deletion without justification | HUMAN |
| **GOV-005** | Phase writes files outside declared scope | NEVER |
| **GOV-006** | No modifications to generated files | NEVER |
| **GOV-007** | No file creation outside phase scope | NEVER |
| **GOV-008** | No duplicate domain definitions | NEVER |
| **GOV-009** | Canonical domain names enforced | NEVER |

### Enforcement Tiers

| Tier | Name | Behavior | Override |
|------|------|----------|----------|
| **L0** | Structural Invariants | Blocks execution | Never |
| **L1** | Project Policy | Blocks execution | Configuration only |
| **L2** | Behavioral Signals | Advisory (warning) | N/A |
| **L3** | Human Governance | Pauses for input | Human decision |

### Properties

| Property | Guarantee |
|----------|-----------|
| **Determinism** | Same result + same stack = same verdict |
| **Monotonicity** | If L0 fails, nothing can override |
| **Composability** | Stacks with disjoint scopes can merge |
| **Decidability** | Evaluation terminates in bounded time |

---

## Domain Registry

AI tools fragment codebases with synonyms. Anvil specifies a **domain registry**—a symbol table for canonical terminology.

```yaml
domain:
  canonical: "Alert"
  presentation_aliases: ["Insight", "ComplianceInsight"]
  backend:
    naming_rule: canonical_only
  frontend:
    allowed_aliases: ["Insight"]
```

**Enforcement:**

| Location | "Alert" | "Insight" |
|----------|---------|-----------|
| Backend | ✓ Allowed | ✗ GOV-009 |
| Frontend | ✓ Allowed | ✓ Allowed |

---

## Generated File Protection

Certain files should never be edited:

- Lock files (`package-lock.json`, `poetry.lock`)
- Migrations (`**/migrations/**`)
- Protocol buffers (`*_pb2.py`, `*.pb.go`)
- Build artifacts (`dist/`, `build/`)

Anvil specifies that implementations must:
1. Detect generated files deterministically
2. Block modifications with `fixability: NEVER`
3. Provide remediation hints

---

## Provenance

Every output includes a provenance chain:

```
output → execution → validation → plan → issue → discovery → intent
```

Each node includes content hash, timestamp, agent, and parent references.

**Every output can answer: "Why do you exist?"**

---

## Event Log

All actions are logged to an immutable event stream:

```
event = (timestamp, intent_id, event_type, agent, status, input, output)
```

Event types: `intent_created`, `discovery_completed`, `plan_generated`, `plan_validation_failed`, `governance_passed`, `governance_failed`, `committed`

---

## Conformance

### Levels

| Level | Requirements |
|-------|--------------|
| **Anvil Core** | Intent, 4-pass Discovery, Planning, Execution, L0/L1 governance, provenance |
| **Anvil Standard** | Core + L2/L3, post-execution governance, repair loop, domain registry |
| **Anvil Complete** | Standard + memory architecture, trust gradients |

### Core Requirements

An Anvil Core implementation must:

1. Implement all phases with defined contracts
2. Enforce phase sequencing (no skipping)
3. Implement GOV-001 through GOV-009 with fixability classification
4. Generate provenance chains for all outputs
5. Reject plans with grounding ratio < 1.0
6. Block modifications to generated files

---

## Schemas

```
schemas/
├── intent.schema.json
├── discovery-fact.schema.json
├── discovery-result.schema.json
├── issue.schema.json
├── plan.schema.json
├── phase.schema.json
├── constraint.schema.json
├── governance-stack.schema.json
├── validation-result.schema.json
├── execution-result.schema.json
├── provenance-node.schema.json
├── user-resolution.schema.json
├── domain-registry.schema.json
└── event.schema.json
```

See [`/schemas`](./schemas) for complete definitions.

---

## Implementations

| Implementation | Status | Description |
|----------------|--------|-------------|
| [Archon](https://github.com/rhatigan-agi/archon) | v0.16.9 | Reference CLI implementation |

Building an Anvil-compliant tool? [Open an issue](https://github.com/rhatigan-agi/anvil/issues) to be listed.

---

## Scope and Non-Goals

Anvil covers **development-time verification**. Non-goals for v0.1:

- Runtime monitoring
- Production observability
- Model training/fine-tuning
- IDE integrations (implementation concern)

---

## Contributing

- **Feedback**: Edge cases, gaps, critiques
- **Governance patterns**: Domain-specific constraints
- **Implementations**: Build something Anvil-compliant

See [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## FAQ

**Is Anvil a tool or a specification?**
Specification. [Archon](https://github.com/rhatigan-agi/archon) is the reference implementation.

**Can I use Anvil with my existing AI coding tool?**
If the tool can accept execution packets or you can run governance on its output, yes.

**Do I need to implement all passes?**
Anvil Core requires Pass 1-2. Anvil Standard requires Pass 1-4.

**Can I skip phases for simple changes?**
No. Phase sequencing is a core invariant. However, phases can be trivial (e.g., empty discovery for greenfield).

**What about proprietary codebases?**
Discovery can be fully local (AST scanning). Governance is deterministic (no external calls required).

---

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — Use, share, adapt with attribution.

---

## Citation

```bibtex
@misc{rhatigan2026anvil,
  author = {Rhatigan, Jeffrey},
  title = {Anvil: A Specification for Deterministic AI-Assisted Development},
  year = {2026},
  publisher = {GitHub},
  url = {https://github.com/rhatigan-agi/anvil}
}
```

---

<div align="center">

**Created by [Jeff Rhatigan](https://github.com/rhatigan) at [Rhatigan Labs](https://rhatiganlabs.com)**

*A prompt can be ignored. A compiler invariant cannot.*

</div>