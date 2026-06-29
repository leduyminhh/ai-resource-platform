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