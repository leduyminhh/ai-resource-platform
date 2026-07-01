# RFC-0202 — Registry Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the local runtime registry that stores discovered resource metadata and supports lookup by id, kind, version, label and checksum.

This RFC owns the runtime registry store and lookup. It is distinct from the distribution registry and marketplace owned by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md). Metadata field semantics are owned by [RFC-0102](../200-resource-model/RFC-0102-Metadata-Model.md); naming by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); discovery by [RFC-0201](RFC-0201-Discovery-Engine.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Registry Purpose

- The runtime registry is an in-process store of discovered resources that holds the `CanonicalResourceSet` for a run.
- It enables lookup and cross-resource checks without re-reading sources.

## 3. Stored Metadata

| Field | Use |
|---|---|
| `id` | Identity in `namespace/name` form. |
| `kind` | Resource kind. |
| `version` | Resource version. |
| `labels` | Selection and grouping. |
| `checksum` | Content integrity and de-duplication. |

`metadata.id` MUST be unique within the registry.

## 4. Lookup API

- Lookup MUST support `id`, `kind`, `version`, `label` and `checksum`.
- Lookup by id returns at most one resource per version.
- Lookup results MUST be deterministic for identical registry state.

## 5. Registry vs Distribution

- The runtime registry (this RFC) is local and ephemeral to a run.
- The distribution registry and marketplace — remote, persistent package storage and indexing — are owned by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).
- These are different concepts and MUST NOT be conflated.

## 6. Registry Behavior

- The registry stores discovered resources as a `CanonicalResourceSet`.
- Registration MUST reject a second resource with the same id and version but a differing checksum (conflict).
- The registry MUST NOT mutate stored source documents.

## 7. Examples

```yaml
lookup:
  by: id
  value: core/clean-code
  result:
    id: core/clean-code
    kind: Skill
    version: 1.0.0
    checksum: sha256:abc123
```

## References

- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0102 — Metadata Model](../200-resource-model/RFC-0102-Metadata-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0201 — Discovery Engine](RFC-0201-Discovery-Engine.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
