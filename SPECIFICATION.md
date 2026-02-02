# Anvil Specification

**Version 0.2.0 — Stable**

**Author:** Jeffrey Rhatigan, Rhatigan Labs

**Date:** February 2026

**License:** CC BY 4.0

---

## Abstract

Anvil is a specification for deterministic, verifiable AI-assisted software development. It defines a governed compilation pipeline—Intent, Discovery, Issue, Planning, Validation, Execution, and Governance—that treats AI as an untrusted code generator operating within structural constraints.

This document provides the formal definitions, interfaces, schemas, and conformance criteria for Anvil-compliant implementations.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Terminology](#2-terminology)
3. [Architecture Overview](#3-architecture-overview)
4. [Phase 1: Intent](#4-phase-1-intent)
5. [Phase 2: Discovery](#5-phase-2-discovery)
6. [Phase 3: Issue](#6-phase-3-issue)
7. [Phase 4: Planning](#7-phase-4-planning)
8. [Phase 5: Validation](#8-phase-5-validation)
9. [Phase 6: Execution](#9-phase-6-execution)
10. [Phase 7: Governance](#10-phase-7-governance)
11. [Governance Algebra](#11-governance-algebra)
12. [Standard Governance Rules](#12-standard-governance-rules)
13. [Domain Registry](#13-domain-registry)
14. [Provenance Protocol](#14-provenance-protocol)
15. [Human Type Casts](#15-human-type-casts)
16. [Three-Tier Memory Architecture](#16-three-tier-memory-architecture)
17. [Trust Gradients](#17-trust-gradients)
18. [Schemas](#18-schemas)
19. [Conformance](#19-conformance)
20. [Security Considerations](#20-security-considerations)
21. [References](#21-references)

---

## 1. Introduction

### 1.1 Motivation

AI-assisted code generation has reached production viability, but verification has not kept pace. Current approaches rely on behavioral constraints—prompting the AI to follow rules—which are fundamentally fragile. A prompt can be ignored; a compiler invariant cannot.

Anvil shifts the burden of safety from the model's behavior to the system's structure.

### 1.2 Design Principles

1. **Structural over Behavioral**: Constraints are enforced by architecture, not by asking the AI to comply.

2. **Deterministic Grounding**: AI cannot reference facts that were not deterministically discovered.

3. **Tiered Governance**: Different constraint types have different enforcement semantics.

4. **Complete Provenance**: Every output traces to its origins through a verifiable chain.

5. **Model Agnosticism**: The specification is independent of any specific AI model or provider.

6. **Lossless Transformation**: Intent concerns must be preserved through all pipeline phases.

### 1.3 Scope

This specification defines:
- The seven-phase compilation pipeline
- Interfaces between phases
- Standard governance rules (GOV-001 through GOV-009)
- The Governance Algebra for constraint evaluation
- Domain registry for canonical terminology
- Provenance requirements
- Conformance criteria

This specification does not define:
- Additional project-specific governance rules
- AI model selection or prompting strategies
- User interface requirements
- Deployment architectures

---

## 2. Terminology

### 2.1 Key Terms

| Term | Definition |
|------|------------|
| **Anvil** | This specification and the architectural pattern it describes |
| **Intent** | A user's expressed desire for code modification or creation |
| **Discovery Fact** | A verifiable assertion about the codebase extracted through deterministic analysis |
| **Issue** | A structured work unit derived from an intent with coverage validation |
| **Plan** | A proposed sequence of phases derived from an issue and grounded in discovery facts |
| **Phase** | An atomic unit of work within a plan with declared scope |
| **Constraint** | A rule that governs whether a plan or action is permitted |
| **Governance Stack** | An ordered collection of constraints organized by enforcement tier |
| **Verdict** | The outcome of evaluating a plan against a governance stack |
| **Fixability** | Classification of whether a violation can be repaired (AUTO, HUMAN, NEVER) |
| **Type Cast** | A human decision that resolves ambiguity and becomes authoritative fact |
| **Provenance Chain** | The complete trace linking an output to its originating inputs |
| **Domain Registry** | A symbol table enforcing canonical terminology |

### 2.2 Requirement Levels

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

---

## 3. Architecture Overview

### 3.1 Pipeline Structure

Anvil defines a seven-phase sequential pipeline with a post-execution repair loop:

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              ANVIL PIPELINE                                         │
│                                                                                     │
│  ┌────────┐   ┌───────────┐   ┌───────┐   ┌────────┐   ┌──────────┐   ┌─────────┐ │
│  │ INTENT │──▶│ DISCOVERY │──▶│ ISSUE │──▶│  PLAN  │──▶│ VALIDATE │──▶│ EXECUTE │ │
│  └────────┘   └───────────┘   └───────┘   └────────┘   └──────────┘   └─────────┘ │
│                                                                              │      │
│                                                                              ▼      │
│                                                                        ┌──────────┐ │
│                                                                        │  GOVERN  │ │
│                                                                        └──────────┘ │
│                                                                              │      │
│                                                              ┌───────────────┤      │
│                                                              ▼               │      │
│                                                        ┌──────────┐         │      │
│                                                        │  REPAIR  │◀────────┘      │
│                                                        └──────────┘                 │
│                                                              │                      │
│                                                              ▼                      │
│                                                        ┌──────────┐                 │
│                                                        │  COMMIT  │                 │
│                                                        └──────────┘                 │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Core Guarantees

These are compiler invariants, not aspirations:

| # | Guarantee | Enforcement |
|---|-----------|-------------|
| 1 | No execution without intent | Full traceability |
| 2 | No planning without discovery | Grounding before action |
| 3 | No file references without proof | Hallucination prevention |
| 4 | No violations without fixability | Every error is classified |
| 5 | No silent governance bypass | All checks logged |
| 6 | Deterministic routing | Same input = same output |
| 7 | No frontend work omission | UI concerns preserved |

### 3.3 Phase Guarantees

Each phase provides specific guarantees to subsequent phases:

| Phase | Guarantee |
|-------|-----------|
| Intent | Structured representation of user desire |
| Discovery | All facts are deterministically verifiable |
| Issue | All intent concerns are addressed or explicitly deferred |
| Planning | All claims trace to discovery facts |
| Validation | Verdict is deterministic given inputs |
| Execution | Only validated changes are applied |
| Governance | All changes evaluated against rules with fixability |

### 3.4 Information Flow

Information flows forward through phases. Later phases MUST NOT influence earlier phases within a single pipeline execution.

---

## 4. Phase 1: Intent

### 4.1 Purpose

Intent captures the user's expressed desire for code modification or creation in a structured format that serves as the root of all provenance chains.

### 4.2 Intent Structure

```typescript
interface Intent {
  id: string;                    // Unique identifier (e.g., "INTENT_007")
  description: string;           // Natural language description
  createdAt: ISO8601Timestamp;
  user: string;                  // User identifier
  tags?: string[];               // Optional categorization
  status: IntentStatus;
}

enum IntentStatus {
  CREATED = "created",
  DISCOVERY_COMPLETE = "discovery_complete",
  ISSUE_GENERATED = "issue_generated",
  PLANNED = "planned",
  VALIDATED = "validated",
  EXECUTED = "executed",
  GOVERNED = "governed",
  COMMITTED = "committed",
  FAILED = "failed"
}
```

### 4.3 Requirements

1. Every intent MUST have a unique identifier.
2. Every intent MUST have a human-readable description.
3. Intent IDs MUST be immutable once created.
4. All downstream artifacts MUST reference their originating intent.

---

## 5. Phase 2: Discovery

### 5.1 Purpose

Discovery extracts verifiable facts from the target codebase through deterministic analysis. It serves as the "linker" phase—establishing ground truth that constrains all subsequent AI operations.

### 5.2 4-Pass Discovery

Anvil defines four discovery passes:

| Pass | Name | Purpose | Method |
|------|------|---------|--------|
| **Pass 1** | Framework Detection | Identify frameworks and languages | Signal-based detection with confidence scores |
| **Pass 2** | Surface Discovery | Extract models, views, components | AST scanning (AUTHORITATIVE) |
| **Pass 3** | Ownership Binding | Determine which file executes vs renders | Interaction analysis |
| **Pass 4** | Schema Sufficiency | Check if schema supports intent | AST comparison (DETERMINISTIC, non-LLM) |

### 5.3 Pass Requirements

1. **Pass 1** MAY use LLM assistance for framework identification.
2. **Pass 2** MUST use deterministic AST scanning. Results are AUTHORITATIVE.
3. **Pass 3** MAY use LLM assistance for ownership determination.
4. **Pass 4** MUST be fully deterministic with no LLM involvement.

### 5.4 Discovery Fact Structure

```typescript
interface DiscoveryFact {
  id: string;
  factType: FactType;
  claim: string;                 // Human-readable assertion
  evidence: Evidence;
  confidence: ConfidenceLevel;
  source: FactSource;
  provenance: FactProvenance;
  extractedAt: ISO8601Timestamp;
}

enum ConfidenceLevel {
  ABSOLUTE = "absolute",         // AST-scanned, cannot be wrong
  HIGH = "high",                 // Static analysis
  SIGNAL = "signal"              // Heuristic-based
}

enum FactSource {
  DETERMINISTIC = "deterministic",
  FRAMEWORK_SCANNER = "framework_scanner",
  USER_RESOLUTION = "user_resolution"
}

interface Evidence {
  method: ExtractionMethod;
  rawData: any;
  verificationCommand?: string;
}

enum ExtractionMethod {
  AST_PARSE = "ast_parse",
  SCHEMA_INTROSPECTION = "schema_introspection",
  STATIC_ANALYSIS = "static_analysis",
  FILE_SYSTEM = "file_system",
  HUMAN_TYPE_CAST = "human_type_cast"
}
```

### 5.5 Schema Sufficiency

Pass 4 MUST determine schema sufficiency for the intent:

```typescript
interface SchemaSufficiency {
  model: string;
  migrationRequired: boolean;
  reason?: string;
}
```

If `migrationRequired: false`, the planning phase MUST NOT propose database migrations for that model.

### 5.6 User Resolution

When discovery encounters ambiguity (e.g., multiple valid frontend placements), it MUST pause for user resolution rather than guess.

```typescript
interface UserResolution {
  resolutionType: ResolutionType;
  question: string;
  options: string[];
  userChoice: string;
  resolvedAt: ISO8601Timestamp;
}

enum ResolutionType {
  FRONTEND_SCOPE = "frontend_scope",
  FRONTEND_PLACEMENT = "frontend_placement",
  OWNERSHIP_AMBIGUITY = "ownership_ambiguity"
}
```

User resolutions become authoritative facts (see §15 Human Type Casts).

### 5.7 Discovery Result

```typescript
interface DiscoveryResult {
  id: string;
  intentId: string;
  timestamp: ISO8601Timestamp;
  outcome: "success" | "failed" | "resolution_required";
  passes: PassResults;
  facts: DiscoveryFact[];
  examinedFiles: string[];
  frontendEntryPoints: string[];
  backendEntryPoints: string[];
  existingModels: ModelInfo[];
  modelFields: Record<string, FieldInfo[]>;
  schemaSufficiency: Record<string, SchemaSufficiency>;
  canonicalDomains: DomainInfo[];
  userResolution?: UserResolution;
}
```

### 5.8 The Linker Principle

**If a fact cannot be verified through discovery, it MUST NOT be available to the planning phase.**

This means:
- Files not in `examinedFiles` do not exist for planning purposes
- Schema fields not extracted cannot be referenced
- Relationships not discovered cannot be assumed

---

## 6. Phase 3: Issue

### 6.1 Purpose

Issues transform intents into structured work units with explicit coverage validation, ensuring no concerns are silently dropped.

### 6.2 Issue Structure

```typescript
interface Issue {
  id: string;
  intentId: string;
  discoveryId: string;
  title: string;
  summary: string;
  backendCriteria: AcceptanceCriterion[];
  frontendCriteria: AcceptanceCriterion[];
  technicalRequirements: string[];
  filesAffected: FileAffected[];
  coverage: CoverageValidation;
  generatedAt: ISO8601Timestamp;
}

interface AcceptanceCriterion {
  description: string;
  completed: boolean;
}

interface FileAffected {
  path: string;
  action: "modify" | "create" | "verify";
  reason: string;
}
```

### 6.3 Issue Coverage Invariant

The transformation from Intent → Issue MUST be lossless with respect to work scope.

```typescript
interface CoverageValidation {
  intentConcerns: Concern[];
  issueConcerns: Concern[];
  status: "complete" | "partial";
  missingConcerns?: Concern[];
  deferralJustification?: string;
}

enum Concern {
  BACKEND = "backend",
  FRONTEND = "frontend",
  DATABASE = "database",
  VALIDATION = "validation",
  TESTING = "testing"
}
```

**Rule:** If an intent mentions both backend AND frontend work, the issue MUST address both—or explicitly defer one with justification.

### 6.4 Frontend Preservation Invariant (v0.2.0)

If Discovery detects `is_new_frontend_interaction = true` or finds `interaction_anchors[]`, downstream phases MUST preserve this information.

Frontend work is expensive to specify. Silently dropping it forces users to re-specify from scratch. Anvil treats this as a compiler bug, not acceptable optimization.

---

## 7. Phase 4: Planning

### 7.1 Purpose

Planning generates proposed changes based on the issue, constrained by discovery facts. The AI operates in this phase but cannot escape the boundaries established by discovery.

### 7.2 Plan Structure

```typescript
interface Plan {
  id: string;
  issueId: string;
  discoveryId: string;
  intentId: string;
  timestamp: ISO8601Timestamp;
  phases: PlanPhase[];
  groundingReport: GroundingReport;
}

interface PlanPhase {
  id: string;
  sequence: number;
  phaseType: PhaseType;
  description: string;
  dependsOn: string[];           // Phase IDs this depends on
  grounding: GroundingSection;
  filesToModify: string[];
  filesThatMayBeCreated: string[];
  changes: ChangeDescription[];
  tests: TestSpecification[];
  scopeDeclaration: ScopeDeclaration;
}

enum PhaseType {
  DATABASE = "database",
  BACKEND = "backend",
  FRONTEND = "frontend",
  TESTING = "testing",
  INFRASTRUCTURE = "infrastructure"
}

interface GroundingSection {
  existingState: string;
  discoveryFactIds: string[];
  assumptions: string[];
}

interface ScopeDeclaration {
  allowedFiles: string[];
  forbiddenOperations: string[];
}
```

### 7.3 Grounding Report

Every plan MUST include a grounding report:

```typescript
interface GroundingReport {
  planId: string;
  totalClaims: number;
  groundedClaims: number;
  groundingRatio: number;        // groundedClaims / totalClaims
  verdict: GroundingVerdict;
}

enum GroundingVerdict {
  FULLY_GROUNDED = "fully_grounded",     // ratio = 1.0
  PARTIALLY_GROUNDED = "partially_grounded",
  UNGROUNDED = "ungrounded"
}
```

### 7.4 Planning Constraints

A compliant implementation MUST reject plans where:
- `groundingRatio < 1.0` (unless explicitly configured for partial grounding)
- Any phase references a file not in discovery's `examinedFiles`
- Any phase assumes a field type contradicting discovery facts
- Any phase proposes a migration when `schemaSufficiency.migrationRequired = false`
- Backend phases do not precede frontend phases (sequencing violation)

---

## 8. Phase 5: Validation

### 8.1 Purpose

Validation evaluates plans against deterministic checks before any code is generated.

### 8.2 Pre-Execution Validation

| Check | Failure Mode |
|-------|--------------|
| File references exist in discovery | Reject plan |
| Backend phases before frontend | Repair or reject |
| Schema sufficiency respected | Block migration proposals if not needed |
| No hedge language ("if exists") | Repair or reject |
| Interaction anchors included | Repair or reject |

### 8.3 Validation Result

```typescript
interface ValidationResult {
  id: string;
  planId: string;
  timestamp: ISO8601Timestamp;
  verdict: ValidationVerdict;
  checks: ValidationCheck[];
  errors: ValidationError[];
  warnings: ValidationWarning[];
}

enum ValidationVerdict {
  PASS = "pass",
  FAIL = "fail",
  REPAIRABLE = "repairable"
}

interface ValidationCheck {
  name: string;
  passed: boolean;
  message?: string;
}

interface ValidationError {
  check: string;
  message: string;
  repairable: boolean;
}
```

### 8.4 Validation Requirements

1. Validation MUST be deterministic: identical inputs produce identical verdicts.
2. Validation MUST NOT use LLM inference.
3. Same plan + same discovery = same verdict.

---

## 9. Phase 6: Execution

### 9.1 Purpose

Execution applies validated plans to the codebase, producing changes with complete provenance.

### 9.2 Execution Constraints

1. Execution MUST only proceed on plans with `ValidationVerdict.PASS`.
2. Execution MUST apply phases in sequence order.
3. Execution MUST respect `ScopeDeclaration.allowedFiles`.
4. Execution MUST be atomic: all changes succeed or all are rolled back.
5. Execution MUST produce provenance for each change.

### 9.3 Execution Result

```typescript
interface ExecutionResult {
  id: string;
  planId: string;
  validationId: string;
  timestamp: ISO8601Timestamp;
  status: ExecutionStatus;
  phaseResults: PhaseExecutionResult[];
  changes: ChangeRecord[];
  rollbackAvailable: boolean;
  duration: DurationMs;
}

enum ExecutionStatus {
  SUCCESS = "success",
  PARTIAL = "partial",
  FAILED = "failed",
  ROLLED_BACK = "rolled_back"
}

interface ChangeRecord {
  id: string;
  phaseId: string;
  file: string;
  action: "create" | "modify" | "delete";
  beforeHash?: string;
  afterHash: string;
  linesAdded?: number;
  linesRemoved?: number;
  timestamp: ISO8601Timestamp;
}
```

---

## 10. Phase 7: Governance

### 10.1 Purpose

Governance evaluates execution results against deterministic rules, producing verdicts with fixability classification.

### 10.2 Post-Execution Governance

Governance runs AFTER execution to validate that the AI's output conforms to all rules.

### 10.3 The Fixability Contract

Every violation MUST be classified:

| Classification | Behavior | Examples |
|----------------|----------|----------|
| **AUTO** | System repairs automatically | Phase sequencing, minor formatting |
| **HUMAN** | Workflow pauses for decision | New model creation, security changes |
| **NEVER** | Abort—fundamental violation | Modified migration, scope breach |

**NEVER is absolute.** The system cannot proceed. No override exists.

### 10.4 Repair Loop

If violations are fixable (AUTO or HUMAN), the system enters a repair loop:

```
execute → govern → repair → govern → ... → pass | abort
```

The loop continues until:
- All violations are resolved → PASS
- A NEVER violation is encountered → ABORT
- Human declines to fix HUMAN violation → ABORT

### 10.5 Governance Result

```typescript
interface GovernanceResult {
  id: string;
  executionId: string;
  timestamp: ISO8601Timestamp;
  verdict: GovernanceVerdict;
  checks: GovernanceCheck[];
  violations: GovernanceViolation[];
  nextAction: "commit" | "repair" | "rollback";
}

enum GovernanceVerdict {
  PASS = "pass",
  FAIL = "fail"
}

interface GovernanceCheck {
  ruleId: string;
  ruleName: string;
  status: "pass" | "fail";
  details?: string;
}

interface GovernanceViolation {
  ruleId: string;
  ruleName: string;
  tier: GovernanceTier;
  fixability: Fixability;
  file?: string;
  line?: number;
  message: string;
  remediation?: string;
}

enum Fixability {
  AUTO = "auto",
  HUMAN = "human",
  NEVER = "never"
}
```

---

## 11. Governance Algebra

### 11.1 Overview

The Governance Algebra provides a formal system for expressing, composing, and evaluating constraints.

### 11.2 Formal Definitions

**Definition 11.1 (Constraint):** A constraint C is a tuple (id, s, p, τ, f) where:
- id is a unique identifier
- s ⊆ Actions is the scope
- p: State × Action → {pass, fail} is the predicate
- τ ∈ {L0, L1, L2, L3} is the tier
- f ∈ {AUTO, HUMAN, NEVER} is the fixability

**Definition 11.2 (Governance Stack):** A governance stack S is an ordered sequence of constraints [C₁, C₂, ..., Cₙ] where constraints are ordered by tier.

### 11.3 Enforcement Tiers

| Tier | Name | Enforcement | Overridable |
|------|------|-------------|-------------|
| **L0** | Structural Invariants | Blocks execution | Never |
| **L1** | Project Policy | Blocks execution | By configuration only |
| **L2** | Behavioral Signals | Advisory (warnings) | N/A (non-blocking) |
| **L3** | Human Governance | Pauses for human input | By human decision |

### 11.4 Evaluation Semantics

```
function evaluate(S: GovernanceStack, result: ExecutionResult): GovernanceVerdict
  violations = []
  for C in S ordered by tier:
    if applies(C, result):
      if predicate(C)(result) = fail:
        violations.append(Violation(C))
        if tier(C) ∈ {L0, L1} and fixability(C) = NEVER:
          return FAIL immediately
  
  if any violation has fixability = NEVER:
    return FAIL
  else if violations is empty:
    return PASS
  else:
    return FAIL (with repair possible)
```

### 11.5 Algebraic Properties

**Property 11.1 (Determinism):** For any governance stack S and execution result R:
```
evaluate(S, R) at time t₁ = evaluate(S, R) at time t₂
```

**Property 11.2 (Monotonicity):** For any L0 constraint with fixability NEVER:
```
If predicate(C)(R) = fail, then evaluate(S, R) = FAIL
```
regardless of other constraints.

**Property 11.3 (Tier Precedence):** Constraints are evaluated in tier order. L0 before L1 before L2 before L3.

---

## 12. Standard Governance Rules

Anvil defines nine standard governance rules that implementations SHOULD support:

### 12.1 L1 Rules (HUMAN Fixability)

| ID | Name | Description |
|----|------|-------------|
| **GOV-001** | Model Creation Control | No new models without explicit intent |
| **GOV-002** | Security Change Control | No auth/security changes without security intent |
| **GOV-003** | Infrastructure Protection | No production infra mutation from non-infra intent |
| **GOV-004** | Test Deletion Control | No test deletion without justification |

### 12.2 L0 Rules (NEVER Fixability)

| ID | Name | Description |
|----|------|-------------|
| **GOV-005** | Phase Scope Enforcement | Phase writes files outside declared scope |
| **GOV-006** | Generated File Protection | No modifications to generated files |
| **GOV-007** | File Creation Scope | No file creation outside phase scope |
| **GOV-008** | Domain Uniqueness | No duplicate domain definitions |
| **GOV-009** | Canonical Domain Enforcement | Backend must use canonical domain names |

### 12.3 Generated File Patterns

GOV-006 MUST block modifications to:

```
**/migrations/**
**/package-lock.json
**/yarn.lock
**/poetry.lock
**/*_pb2.py
**/*.pb.go
**/dist/**
**/build/**
**/.next/**
```

### 12.4 Rule Structure

```typescript
interface GovernanceRule {
  id: string;                    // e.g., "GOV-005"
  name: string;
  description: string;
  tier: GovernanceTier;
  fixability: Fixability;
  scope: RuleScope;
  predicate: RulePredicate;
  message: string;
  remediation?: string;
}

interface RuleScope {
  filePatterns?: string[];
  actionTypes?: string[];
  phaseTypes?: string[];
  all?: boolean;
}
```

---

## 13. Domain Registry

### 13.1 Purpose

The Domain Registry prevents AI-induced terminology fragmentation by enforcing canonical names.

### 13.2 Registry Structure

```typescript
interface DomainRegistry {
  version: number;
  domains: Domain[];
}

interface Domain {
  canonical: string;             // e.g., "Task"
  presentationAliases: string[]; // e.g., ["TODO", "Activity"]
  backend: BackendRule;
  frontend: FrontendRule;
}

interface BackendRule {
  namingRule: "canonical_only";  // Backend MUST use canonical name
}

interface FrontendRule {
  allowedAliases: string[];      // Frontend MAY use these aliases
}
```

### 13.3 Example Registry

```yaml
domains:
  Task:
    canonical: true
    presentation_aliases:
      - TODO
      - Activity
      - ProjectTask
    backend:
      naming_rule: canonical_only
    frontend:
      allowed_aliases:
        - TODO
        - Task
```

### 13.4 Enforcement

| Location | Canonical Name | Alias |
|----------|----------------|-------|
| Backend code | ✓ Allowed | ✗ GOV-009 violation |
| Frontend code | ✓ Allowed | ✓ Allowed (if in registry) |

---

## 14. Provenance Protocol

### 14.1 Purpose

The Provenance Protocol ensures every output can be traced to its origins.

### 14.2 Provenance Chain Structure

```typescript
interface ProvenanceChain {
  id: string;
  outputId: string;
  timestamp: ISO8601Timestamp;
  nodes: ProvenanceNode[];
}

interface ProvenanceNode {
  id: string;
  type: ProvenanceNodeType;
  artifactId: string;
  artifactHash?: string;
  inputs: string[];              // Parent node IDs
  timestamp: ISO8601Timestamp;
  agent: string;                 // "system" | "discovery" | "planner" | "executor" | "governance" | "user"
}

enum ProvenanceNodeType {
  INTENT = "intent",
  DISCOVERY_FACT = "discovery_fact",
  USER_RESOLUTION = "user_resolution",
  ISSUE = "issue",
  PLAN_PHASE = "plan_phase",
  VALIDATION_RESULT = "validation_result",
  EXECUTION_CHANGE = "execution_change",
  GOVERNANCE_RESULT = "governance_result"
}
```

### 14.3 Requirements

1. Every execution output MUST have an associated provenance chain.
2. Provenance chains MUST be immutable once created.
3. **Every output can answer: "Why do you exist?"**

---

## 15. Human Type Casts

### 15.1 Purpose

Human Type Casts formalize human decisions as authoritative facts that constrain subsequent pipeline operations.

### 15.2 Type Cast Structure

```typescript
interface TypeCast {
  id: string;
  timestamp: ISO8601Timestamp;
  intentId: string;
  castType: TypeCastType;
  ambiguity: AmbiguityDescription;
  resolution: Resolution;
  scope: TypeCastScope;
}

enum TypeCastType {
  FRONTEND_PLACEMENT = "frontend_placement",
  FRONTEND_SCOPE = "frontend_scope",
  OWNERSHIP_ASSIGNMENT = "ownership_assignment",
  POLICY_OVERRIDE = "policy_override"
}

enum TypeCastScope {
  THIS_INTENT = "this_intent",
  THIS_SESSION = "this_session",
  PROJECT = "project"
}
```

### 15.3 Cardinality Reduction

User resolution is binding type narrowing:

```
Before:  frontend_owners = {FileA, FileB, FileC}
After:   frontend_owners = {FileA}
```

All other options are pruned. Once resolved, no phase may reference the discarded alternatives.

### 15.4 Type Cast as Discovery Fact

Type casts with scope PROJECT become discovery facts:

```typescript
const newFact: DiscoveryFact = {
  factType: FactType.OWNERSHIP,
  claim: resolution.selectedOption,
  evidence: {
    method: ExtractionMethod.HUMAN_TYPE_CAST,
    rawData: typeCast
  },
  confidence: "absolute",
  source: "user_resolution"
};
```

---

## 16. Three-Tier Memory Architecture

### 16.1 Purpose

The Three-Tier Memory Architecture enables implementations to learn from historical operations while maintaining deterministic guarantees.

### 16.2 Tiers

| Tier | Name | Content | Influence |
|------|------|---------|-----------|
| **Tier 1** | Stream | Raw, append-only event log | None (audit only) |
| **Tier 2** | Index | Consolidated patterns | Planning context |
| **Tier 3** | Codex | Project wisdom | Governance thresholds |

### 16.3 Memory Constraints

1. Tier 1 data MUST NOT influence discovery facts (determinism preserved)
2. Tier 2 patterns MAY influence planning context
3. Tier 3 wisdom MAY influence governance thresholds
4. All memory influence MUST be logged in provenance

---

## 17. Trust Gradients

### 17.1 Purpose

Trust Gradients track confidence in facts and derived artifacts.

### 17.2 Trust Sources

| Source | Base Score |
|--------|------------|
| AST Deterministic | 0.99 |
| Schema Introspection | 0.95 |
| Human Type Cast | 0.90 |
| Static Analysis | 0.85 |
| Pattern Match | 0.70 |
| AI Grounded | 0.55 |
| AI Ungrounded | 0.20 |

### 17.3 Trust Propagation

```
trust(derived) = min(trust(parent₁), ..., trust(parentₙ)) × decay_factor
```

Where `decay_factor` is typically 0.95.

---

## 18. Schemas

### 18.1 JSON Schema Definitions

Complete JSON Schema definitions are provided in `/schemas`:

| Schema File | Describes |
|-------------|-----------|
| `intent.schema.json` | Intent structure |
| `discovery-fact.schema.json` | Discovery fact |
| `discovery-result.schema.json` | Complete discovery result |
| `issue.schema.json` | Issue structure |
| `plan.schema.json` | Plan structure |
| `phase.schema.json` | Plan phase |
| `constraint.schema.json` | Governance constraint |
| `governance-stack.schema.json` | Complete governance stack |
| `validation-result.schema.json` | Validation result |
| `execution-result.schema.json` | Execution result |
| `governance-result.schema.json` | Governance result |
| `provenance-chain.schema.json` | Provenance chain |
| `user-resolution.schema.json` | User resolution / type cast |
| `domain-registry.schema.json` | Domain registry |
| `event.schema.json` | Timeline event |

---

## 19. Conformance

### 19.1 Conformance Levels

| Level | Requirements |
|-------|--------------|
| **Anvil Core** | Intent, Discovery (Pass 1-2), Planning, Execution, L0 governance, provenance |
| **Anvil Standard** | Core + Discovery (Pass 3-4), L1/L2/L3 governance, repair loop, domain registry, issue coverage validation |
| **Anvil Complete** | Standard + frontend preservation, memory architecture, trust gradients |

### 19.2 Core Requirements

An Anvil Core implementation MUST:

1. Implement all phases with defined contracts
2. Enforce phase sequencing (no skipping)
3. Implement Discovery Pass 1-2
4. Implement L0 governance (GOV-005, GOV-006, GOV-007)
5. Generate provenance chains for all outputs
6. Reject plans with grounding ratio < 1.0

### 19.3 Standard Requirements

An Anvil Standard implementation MUST additionally:

7. Implement Discovery Pass 3-4
8. Implement all governance rules (GOV-001 through GOV-009)
9. Implement the repair loop
10. Implement domain registry
11. Validate issue coverage vs. intent scope

### 19.4 Complete Requirements

An Anvil Complete implementation MUST additionally:

12. Implement frontend preservation invariant
13. Implement three-tier memory architecture
14. Implement trust gradients

---

## 20. Security Considerations

### 20.1 Threat Model

Anvil assumes:
- The AI is untrusted and may attempt to exceed its boundaries
- Discovery inputs (codebase) are trusted
- Human type casts are authoritative
- The governance stack is trusted configuration

### 20.2 Security Requirements

1. **Isolation**: The AI MUST NOT have direct access to execution capabilities.
2. **Validation Bypass Prevention**: No code path from planning to execution may bypass validation.
3. **Scope Enforcement**: Execution MUST verify changes match validated plan scope.
4. **Audit Trail**: All operations MUST be logged.
5. **Type Cast Authentication**: Human type casts MUST be authenticated.

### 20.3 Governance Stack Security

1. L0 constraints MUST NOT be modifiable at runtime.
2. GOV-005, GOV-006, GOV-007 have fixability NEVER and cannot be overridden.

---

## 21. References

### 21.1 Normative References

- RFC 2119: Key words for use in RFCs to Indicate Requirement Levels
- ISO 8601: Date and time format
- JSON Schema: Draft 2020-12

### 21.2 Informative References

- Rhatigan, J. (2026). Archon: Reference implementation of the Anvil specification.

---

## Appendix A: Example Governance Stack

```yaml
name: "Standard Anvil Governance"
version: "1.0.0"

rules:
  # L0: Structural Invariants (NEVER)
  - id: "GOV-005"
    name: "Phase scope enforcement"
    tier: L0
    fixability: NEVER
    message: "Phase writes files outside declared scope"

  - id: "GOV-006"
    name: "Generated file protection"
    tier: L0
    fixability: NEVER
    patterns:
      - "**/migrations/**"
      - "**/package-lock.json"
      - "**/*_pb2.py"
    message: "Cannot modify generated files"

  - id: "GOV-007"
    name: "File creation scope"
    tier: L0
    fixability: NEVER
    message: "Cannot create files outside phase scope"

  - id: "GOV-008"
    name: "Domain uniqueness"
    tier: L0
    fixability: NEVER
    message: "Duplicate domain definition"

  - id: "GOV-009"
    name: "Canonical domain enforcement"
    tier: L0
    fixability: NEVER
    message: "Backend must use canonical domain names"

  # L1: Project Policy (HUMAN)
  - id: "GOV-001"
    name: "Model creation control"
    tier: L1
    fixability: HUMAN
    message: "New model requires explicit intent"

  - id: "GOV-002"
    name: "Security change control"
    tier: L1
    fixability: HUMAN
    scope:
      filePatterns: ["**/auth/**", "**/security/**"]
    message: "Security changes require security intent"

  - id: "GOV-003"
    name: "Infrastructure protection"
    tier: L1
    fixability: HUMAN
    scope:
      filePatterns: ["**/infra/**", "**/deploy/**"]
    message: "Infra changes require infra intent"

  - id: "GOV-004"
    name: "Test deletion control"
    tier: L1
    fixability: HUMAN
    message: "Test deletion requires justification"
```

---

## Appendix B: Changelog

### v0.2.0 (February 2026)
- Added Issue phase with coverage validation
- Added Frontend Preservation Invariant
- Renumbered discovery passes to 1-4 (was 0-4)
- Added standard governance rules GOV-001 through GOV-009
- Added Domain Registry specification
- Added repair loop to governance phase
- Clarified Pass 4 is deterministic (non-LLM)
- Updated conformance levels

### v0.1.0 (January 2026)
- Initial draft specification
- Four-phase pipeline definition
- Governance Algebra formal definition
- Provenance Protocol
- Three-tier memory architecture
- Trust gradients
- Human type casts

---

*End of Specification*