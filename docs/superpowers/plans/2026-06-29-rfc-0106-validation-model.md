# RFC-0106 Validation Model Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0106 — Validation Model` into the normative validation layers, result envelope and diagnostic/error contract for ARPS.

**Architecture:** Keep `RFC-0106` focused on validation layer contracts and result/error shape only. Extend `tools/check-rfc-links` with RFC-0106 guardrails first, then rewrite the RFC until the guard passes and close the pending delegation.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add RFC-0106-specific assertions for validation model ownership.
- Modify: `docs/200-resource-model/RFC-0106-Validation-Model.md` — replace boilerplate with validation model content.
- Modify: `TODO.md` — mark the RFC-0106 pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0106-validation-model-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: `docs/300-runtime/RFC-0203-Validation-Engine.md`, `docs/100-foundation/RFC-0007-Compatibility-Policy.md`, `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`.

---

## Task 1: Add RFC-0106 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/200-resource-model/RFC-0106-Validation-Model.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,340p' tools/check-rfc-links
```

Expected: script includes overview checks plus existing guards for RFC-0007 and RFC-0100.

- [ ] **Step 2: Add RFC-0106 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 7: RFC-0106 Validation Model ownership ---
VALIDATION="$ROOT/docs/200-resource-model/RFC-0106-Validation-Model.md"
if [ -f "$VALIDATION" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$VALIDATION" || err "missing header field ${field//\\/} in RFC-0106"
  done

  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Canonical Model' 'Runtime Flow' 'Security Considerations' 'Compatibility' 'Migration Guidance' 'Future Work' 'BUILD_ERROR'; do
    grep -qE "^#+ .*$bad|$bad" "$VALIDATION" && err "boilerplate or out-of-scope content '$bad' still in RFC-0106"
  done

  for required in 'Validation Inputs' 'Validation Layers' 'Validation Result Envelope' 'Diagnostic Object' 'Error Code Taxonomy' 'Required Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$VALIDATION" || err "missing RFC-0106 section '$required'"
  done

  for severity in 'error' 'warning' 'info'; do
    grep -q "$severity" "$VALIDATION" || err "missing RFC-0106 severity '$severity'"
  done

  for code in 'SYNTAX_ERROR' 'SCHEMA_ERROR' 'METADATA_ERROR' 'SEMANTIC_ERROR' 'DEPENDENCY_ERROR' 'COMPATIBILITY_ERROR' 'POLICY_VIOLATION' 'VALIDATOR_ERROR'; do
    grep -q "$code" "$VALIDATION" || err "missing RFC-0106 error code '$code'"
  done

  grep -q 'valid.*false.*severity.*error\|severity.*error.*valid.*false' "$VALIDATION" || err "RFC-0106 must state valid=false when any diagnostic severity is error"
  grep -q 'MUST NOT mutate source resources' "$VALIDATION" || err "RFC-0106 must state validators do not mutate source resources"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0106, including boilerplate headings such as `Motivation`, `Canonical Model`, `Runtime Flow`, `Security Considerations`, `Compatibility`, `Migration Guidance`, `Future Work`, and missing sections such as `Validation Inputs` or `Validation Result Envelope`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0106 validation model"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0106 Validation Model

**Files:**
- Modify: `docs/200-resource-model/RFC-0106-Validation-Model.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0106-validation-model-design.md`

- [ ] **Step 1: Replace RFC-0106 content**

Replace the full content of `docs/200-resource-model/RFC-0106-Validation-Model.md` with:

```markdown
# RFC-0106 — Validation Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines validation layers, validation result envelope and diagnostic/error model for ARPS resources.

This RFC owns validation rules and reporting contracts. Resource envelope semantics are defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); compatibility classification is defined by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md); validation engine execution strategy is defined by [RFC-0203](../300-runtime/RFC-0203-Validation-Engine.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A validator evaluates resource documents against schemas, metadata constraints, semantic rules, dependency context, compatibility policy and governance policy. This RFC defines the layer contract and result shape. It does not define whether an engine runs layers synchronously, in parallel, incrementally or with caching.

## 2. Validation Inputs

| Input | Role |
|---|---|
| Resource document | YAML/JSON source being validated. |
| Canonical envelope | `apiVersion`, `kind`, `metadata`, `spec` and optional `status` from [RFC-0100](RFC-0100-Canonical-Resource-Model.md). |
| Schema context | Envelope schema plus kind-specific schema. |
| Registry context | IDs, versions, namespaces and registry policy required for cross-resource checks. |
| Dependency graph context | Dependency edges, referenced resources and graph constraints. |
| Compatibility context | Previous/current resources or declared compatibility policy from [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md). |
| Policy context | Organization, registry, security and governance constraints. |

Missing context MUST be reported as a diagnostic when a requested validation layer requires that context.

## 3. Validation Layers

| Layer | Responsibility | Primary code |
|---|---|---|
| `syntax` | Parse YAML/JSON, encoding and malformed document source. | `SYNTAX_ERROR` |
| `schema` | Validate envelope and kind-specific schema. | `SCHEMA_ERROR` |
| `metadata` | Validate identity, version and registry metadata constraints. | `METADATA_ERROR` |
| `semantic` | Validate kind-specific invariants not expressible by schema alone. | `SEMANTIC_ERROR` |
| `dependency` | Validate missing dependencies, graph edges and acyclic constraints. | `DEPENDENCY_ERROR` |
| `compatibility` | Report compatibility classification conflicts from [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md). | `COMPATIBILITY_ERROR` |
| `policy` | Apply organization, registry, security and governance policies. | `POLICY_VIOLATION` |

Layers are a reporting contract. Execution order, short-circuiting and parallelism are owned by [RFC-0203](../300-runtime/RFC-0203-Validation-Engine.md).

## 4. Validation Result Envelope

Validation output MUST use this envelope:

```yaml
valid: false
diagnostics:
  - code: METADATA_ERROR
    severity: error
    layer: metadata
    message: metadata.id must be unique in the registry
    resourceId: core/clean-code
    path: /metadata/id
    location:
      file: examples/resources/skill.yaml
      line: 4
      column: 7
    reference: RFC-0102
summary:
  errorCount: 1
  warningCount: 0
  infoCount: 0
validator:
  name: arps-validator
  version: 0.1.0
```

`valid` and `diagnostics` are required. `summary` and `validator` are optional.

## 5. Diagnostic Object

| Field | Required | Semantics |
|---|---|---|
| `code` | Yes | Stable machine-readable diagnostic code. |
| `severity` | Yes | One of `error`, `warning` or `info`. |
| `layer` | Yes | Validation layer that produced the diagnostic. |
| `message` | Yes | Human-readable explanation. |
| `resourceId` | No | Related resource ID when known. |
| `path` | No | JSON Pointer or YAML path to the logical value. |
| `location` | No | File, line and column when source location is available. |
| `reference` | No | RFC, schema or policy reference. |

Diagnostic messages SHOULD be clear enough for a human to act on without reading validator source code.

## 6. Error Code Taxonomy

| Code | Layer | Meaning |
|---|---|---|
| `SYNTAX_ERROR` | `syntax` | Source cannot be parsed or decoded. |
| `SCHEMA_ERROR` | `schema` | Resource fails envelope or kind-specific schema validation. |
| `METADATA_ERROR` | `metadata` | Metadata identity, version or registry constraint fails. |
| `SEMANTIC_ERROR` | `semantic` | Kind-specific invariant fails. |
| `DEPENDENCY_ERROR` | `dependency` | Dependency reference, edge or graph invariant fails. |
| `COMPATIBILITY_ERROR` | `compatibility` | Change conflicts with compatibility policy. |
| `POLICY_VIOLATION` | `policy` | Organization, registry, security or governance policy fails. |
| `VALIDATOR_ERROR` | validator infrastructure | Validator cannot load required schema, policy or context. |

Build pipeline failures are outside the core validation model and are owned by build/distribution RFCs.

## 7. Required Behavior

- `valid` MUST be `false` when any diagnostic has severity `error`.
- `valid` MAY be `true` when diagnostics contain only `warning` or `info` severities.
- Validators MUST produce deterministic diagnostic ordering for identical inputs and context.
- Validators SHOULD continue after recoverable errors to report multiple diagnostics.
- Validators MUST NOT mutate source resources during validation.
- Missing required context MUST produce a diagnostic instead of a silent pass.
- Unsupported resource kinds, schemas or policies SHOULD produce `SCHEMA_ERROR`, `POLICY_VIOLATION` or `VALIDATOR_ERROR` according to the cause.

## 8. Examples

### 8.1 Valid result

```yaml
valid: true
diagnostics: []
summary:
  errorCount: 0
  warningCount: 0
  infoCount: 0
```

### 8.2 Duplicate metadata ID

```yaml
valid: false
diagnostics:
  - code: METADATA_ERROR
    severity: error
    layer: metadata
    message: metadata.id must be unique in the registry
    resourceId: core/clean-code
    path: /metadata/id
    reference: RFC-0102
```

### 8.3 Compatibility conflict

```yaml
valid: false
diagnostics:
  - code: COMPATIBILITY_ERROR
    severity: error
    layer: compatibility
    message: removing exported resource core/clean-code is a breaking change
    resourceId: core/clean-code
    path: /spec/exports/0
    reference: RFC-0007
```

### 8.4 Policy violation

```yaml
valid: false
diagnostics:
  - code: POLICY_VIOLATION
    severity: error
    layer: policy
    message: namespace experimental is not allowed in this registry
    resourceId: experimental/demo
    path: /metadata/id
    reference: RFC-0802
```

### 8.5 Validator infrastructure failure

```yaml
valid: false
diagnostics:
  - code: VALIDATOR_ERROR
    severity: error
    layer: schema
    message: validator could not load schema for kind Adapter
    path: /kind
```

## References

- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0101 — Serialization](RFC-0101-Serialization.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0104 — Dependency Graph](RFC-0104-Dependency-Graph.md)
- [RFC-0203 — Validation Engine](../300-runtime/RFC-0203-Validation-Engine.md)
- [RFC-0800 — Security Model](../900-security-governance/RFC-0800-Security-Model.md)
- [RFC-0802 — Governance](../900-security-governance/RFC-0802-Governance.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0106**

Run:

```bash
grep -n 'RFC-0106 — Validation Model' SUMMARY.md
```

Expected: output includes `docs/200-resource-model/RFC-0106-Validation-Model.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/200-resource-model/RFC-0106-Validation-Model.md
git commit -m "docs(rfc): viet lai RFC-0106 Validation Model"
```

Expected: one commit containing only `docs/200-resource-model/RFC-0106-Validation-Model.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0106 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0106 Validation Model — nhà của Validation Rules + Error Model
```

to:

```markdown
- [x] RFC-0106 Validation Model — nhà của Validation Rules + Error Model
```

- [ ] **Step 2: Run full docs validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Verify only intended files changed**

Run:

```bash
git status --short
```

Expected: only `TODO.md` is modified after Task 1 and Task 2 commits.

- [ ] **Step 4: Commit closure note**

Run:

```bash
git add TODO.md
git commit -m "docs: dong pending RFC-0106 validation model"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0106-specific guardrails.
- `docs/200-resource-model/RFC-0106-Validation-Model.md` contains `Validation Inputs`, `Validation Layers`, `Validation Result Envelope`, `Diagnostic Object`, `Error Code Taxonomy`, `Required Behavior`, `Examples` and `References`.
- `docs/200-resource-model/RFC-0106-Validation-Model.md` no longer contains boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Runtime Flow`, `Security Considerations`, `Compatibility`, `Migration Guidance` or `Future Work`.
- RFC-0106 states severity set: `error`, `warning`, `info`.
- RFC-0106 states taxonomy codes: `SYNTAX_ERROR`, `SCHEMA_ERROR`, `METADATA_ERROR`, `SEMANTIC_ERROR`, `DEPENDENCY_ERROR`, `COMPATIBILITY_ERROR`, `POLICY_VIOLATION`, `VALIDATOR_ERROR`.
- RFC-0106 states `valid=false` when any diagnostic severity is `error`.
- RFC-0106 states validators MUST NOT mutate source resources.
- `TODO.md` marks the RFC-0106 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/200-resource-model/RFC-0106-Validation-Model.md`.
- Related RFCs such as `RFC-0203`, `RFC-0007` and `RFC-0100` remain unchanged in this plan.