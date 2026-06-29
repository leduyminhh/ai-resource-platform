# RFC-0104 — Dependency Graph

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines resource-level dependency graph semantics for ARPS resources.

This RFC owns graph nodes, dependency edges, dependency constraints and graph invariants. Version syntax is defined by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); naming and resource identity are defined by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); canonical resource envelopes are defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); validation diagnostics are defined by [RFC-0106](RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A dependency graph describes resource relationships before runtime execution. It does not define resolver algorithms, version selection, conflict resolution, lock snapshots or package format.

## 2. Graph Model

A dependency graph is a directed graph:

- A node represents a canonical resource identified by `kind`, `metadata.id` and `metadata.version`.
- An edge points from a resource to another resource it requires.
- The graph is evaluated at resource level, not file path level.
- Dependency graphs MUST be directed and acyclic.

An implementation MAY materialize the graph in memory, in a lock file or in registry metadata, but this RFC defines only the resource-level model.

## 3. Dependency Edge

A dependency edge SHOULD identify the target resource by ID and MAY constrain kind and version range.

Conceptual shape:

```yaml
dependencies:
  - id: prompts/customer-support-summary
    kind: Prompt
    version: ">=1.2.0 <2.0.0"
    optional: false
```

Fields:

| Field | Meaning |
|---|---|
| `id` | Target resource identity using RFC-0006 naming rules. |
| `kind` | Optional target kind constraint. |
| `version` | Optional version range using RFC-0005 range vocabulary. |
| `optional` | Whether the dependency may be absent without invalidating the resource. |

## 4. Dependency Constraints

- Required dependencies MUST identify a target resource ID.
- Required dependencies MAY constrain target kind and version range.
- Optional dependencies SHOULD declare fallback behavior in the owning resource kind specification.
- Dependency declarations MUST NOT depend on repository file paths as identity.
- Version selection and conflict resolution are owned by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## 5. Graph Invariants

- Dependency graphs MUST be directed and acyclic.
- A required dependency MUST resolve to one acceptable target before execution or packaging decisions consume the graph.
- A dependency edge MUST NOT silently resolve to a resource with a different `metadata.id`.
- Ambiguous matches MUST NOT be accepted as deterministic resolution.
- Graph traversal SHOULD be deterministic for the same registry inputs and policy configuration.

## 6. Validation Behavior

Validation SHOULD detect missing required dependencies, ambiguous dependencies, invalid version ranges and cycles.

Missing, ambiguous or cyclic dependencies SHOULD produce `DEPENDENCY_ERROR` diagnostics through [RFC-0106](RFC-0106-Validation-Model.md).

Validation may prove that a graph is structurally acceptable. It does not choose the final concrete version when multiple acceptable versions exist; that version selection belongs to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md). Lock snapshots belong to [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).

## 7. Examples

### 7.1 Prompt dependency

A workflow can depend on a prompt resource by ID and compatible version range:

```yaml
kind: Workflow
metadata:
  id: workflows/support-summary
  version: 1.0.0
spec:
  dependencies:
    - id: prompts/customer-support-summary
      kind: Prompt
      version: ">=1.2.0 <2.0.0"
```

### 7.2 Cycle rejected

If `workflows/a` depends on `workflows/b` and `workflows/b` depends on `workflows/a`, validation reports a `DEPENDENCY_ERROR` because the graph is cyclic.

### 7.3 Missing dependency rejected

If a required dependency points to `prompts/missing-summary` and no registry source provides that resource ID, validation reports a `DEPENDENCY_ERROR`.

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0106 — Validation Model](RFC-0106-Validation-Model.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
