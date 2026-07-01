# RFC-0204 — Dependency Resolver

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines dependency resolution, version selection, conflict handling and lock generation.

This RFC owns the resolution algorithm. The dependency graph structure and invariants are owned by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md); version range grammar by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); lock file format by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md); diagnostics by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Resolution Inputs

- The resolver consumes the dependency graph defined by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md) plus the runtime registry ([RFC-0202](RFC-0202-Registry-Engine.md)).
- It MUST NOT redefine graph structure or the acyclic invariant.

## 3. Version Selection

- The resolver selects one concrete version per dependency from the versions satisfying the declared range.
- Version range grammar is owned by [RFC-0005](../100-foundation/RFC-0005-Versioning.md).
- Selection MUST be deterministic for identical inputs and registry state.

## 4. Conflict Handling

- When no version satisfies all constraints, the resolver MUST surface a `DEPENDENCY_ERROR` diagnostic (via [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md)).
- Cyclic or missing required dependencies MUST also surface as `DEPENDENCY_ERROR`.

## 5. Lock Generation

- The resolver produces a resolved `DependencyGraph` and a lock describing the exact resolved versions.
- The lock file format is owned by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md); this RFC owns only the act of producing the resolved set.

## 6. Resolver Behavior

- Resolution is read-only with respect to source resources.
- Given the same registry state and inputs, the resolved `DependencyGraph` and lock MUST be identical.

## 7. Examples

```yaml
resolve:
  root: plugins/backend
  selected:
    - id: plugins/backend
      version: 1.0.0
    - id: core/clean-code
      version: 1.0.0
  lock: generated
```

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
