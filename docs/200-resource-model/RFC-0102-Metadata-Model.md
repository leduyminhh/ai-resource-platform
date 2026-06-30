# RFC-0102 — Metadata Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines metadata fields used for identity, ownership, discovery, governance, search and compatibility.

This RFC owns metadata field semantics. The canonical envelope that contains metadata is defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); identifier grammar by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); version format by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); compatibility classification by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md); lifecycle state by [RFC-0103](RFC-0103-Resource-Lifecycle.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Metadata Fields

| Field | Required | Semantics |
|---|---|---|
| `id` | Yes | Stable resource identity in `namespace/name` form. |
| `name` | Yes | Short human-readable name. |
| `version` | Yes | Resource version. |
| `owner` | No | Owning team or person. |
| `labels` | No | Key/value pairs for selection and grouping. |
| `annotations` | No | Non-identifying auxiliary metadata. |

## 3. Identity

- `metadata.id` MUST be present and stable within a registry namespace.
- Identifier grammar (namespace/name form, case and characters) is defined by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md).
- `metadata.version` format is defined by [RFC-0005](../100-foundation/RFC-0005-Versioning.md).
- `metadata.name` is a display name and MUST NOT be used as resource identity.

## 4. Discovery and Governance

- `labels` SHOULD be used for discovery, selection and grouping.
- `owner` SHOULD identify the party responsible for governance.
- `annotations` MAY carry tooling, audit or provenance information that does not affect identity.

## 5. Compatibility Fields

- Compatibility classification of metadata changes is defined by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md).
- Changing `metadata.id` is an identity change and is compatibility-sensitive.
- Adding optional labels or annotations is a compatible change.

## 6. Extensibility

- `labels` and `annotations` are open string maps.
- Unknown metadata fields MUST follow the active schema policy.
- Unknown labels and annotations SHOULD be ignored by tools that do not understand them.

## 7. Examples

```yaml
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.2.0
  owner: platform-team
  labels:
    tier: core
  annotations:
    source: legacy-import
```

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0103 — Resource Lifecycle](RFC-0103-Resource-Lifecycle.md)
