# RFC-0207 — Artifact Model

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines immutable outputs produced by execution and packaging.

This RFC owns the `ExecutionArtifact` model. Artifacts are produced by execution ([RFC-0206](RFC-0206-Execution-Engine.md)); bundling artifacts into packages is owned by [RFC-0107](../200-resource-model/RFC-0107-Packaging-Model.md) and [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); signing and integrity by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Artifact Concept

- An `ExecutionArtifact` is an output produced by executing a plan step.
- An artifact is data plus metadata describing what produced it.

## 3. Artifact Identity

- An artifact is identified by a content checksum plus the producing resource id and version.
- Checksum and signing mechanics are owned by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 4. Immutability

- An `ExecutionArtifact` is immutable once produced; any change MUST yield a new artifact with a new checksum.
- Consumers MUST reference artifacts by identity, not by mutable location.

## 5. Provenance

- An artifact MUST record provenance: the plan step, resource id/version and input checksums that produced it.
- Provenance enables reproducibility checks ([RFC-0208](RFC-0208-Caching-Strategy.md)).

## 6. Examples

```yaml
artifact:
  id: plugins/backend@1.0.0
  checksum: sha256:def456
  producedBy:
    step: plugins/backend
    inputs:
      - core/clean-code@1.0.0
```

## References

- [RFC-0107 — Packaging Model](../200-resource-model/RFC-0107-Packaging-Model.md)
- [RFC-0206 — Execution Engine](RFC-0206-Execution-Engine.md)
- [RFC-0208 — Caching Strategy](RFC-0208-Caching-Strategy.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0801 — Signing and Integrity](../900-security-governance/RFC-0801-Signing-and-Integrity.md)
