# RFC-0005 — Versioning

**Status:** Draft  
**Category:** 100-foundation  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines version syntax and version-change rules for ARPS specifications, resources, schemas, adapters, packages and runtime components.

This RFC owns version number semantics and range vocabulary. Compatibility classification is defined by [RFC-0007](RFC-0007-Compatibility-Policy.md); dependency version selection is defined by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md); lock snapshots are defined by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A version identifies the contract exposed by a specification, resource, schema, adapter, package or runtime component. It does not define resource identity; resource identity is defined by [RFC-0006](RFC-0006-Naming-Convention.md) and canonical resource metadata by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md).

## 2. Version Format

ARPS versions SHOULD use Semantic Versioning syntax:

```text
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

- `MAJOR`, `MINOR` and `PATCH` MUST be non-negative integers.
- `MAJOR`, `MINOR` and `PATCH` MUST NOT contain leading zeroes unless the value is `0`.
- `PRERELEASE` MAY identify unstable preview contracts.
- `BUILD` MAY identify build metadata and MUST NOT change compatibility semantics.
- Implementations SHOULD compare version precedence using Semantic Versioning rules.

## 3. Versioned Subjects

| Subject | Version meaning |
|---|---|
| Specification | Version of the ARPS specification contract. |
| Resource | Version of the resource contract identified by `metadata.id`. |
| Schema | Version of schema constraints for a resource kind or manifest. |
| Adapter | Version of adapter capabilities and output contract. |
| Package | Version of a distributable bundle. |
| Runtime Component | Version of a runtime component contract or capability set. |

A subject version MUST be interpreted in the context of its subject identity. The same version number on two different resource IDs does not imply compatibility or substitutability.

## 4. Version Ranges

Version ranges MAY be used by dependency declarations, package metadata or capability requirements.

Supported range vocabulary SHOULD include:

| Range form | Meaning |
|---|---|
| `1.2.3` | exactly version `1.2.3` |
| `>=1.2.0` | version greater than or equal to `1.2.0` |
| `<2.0.0` | version lower than `2.0.0` |
| `>=1.2.0 <2.0.0` | intersection of constraints |
| `^1.2.0` | compatible with `1.2.0` until the next major version |
| `~1.2.0` | compatible with `1.2.0` until the next minor version |

This RFC defines range vocabulary only. Version selection and conflict resolution belong to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## 5. Change Rules

- Breaking changes require a major version bump.
- Compatible additive changes SHOULD use a minor version bump.
- Patch changes MUST preserve compatibility and meaning.
- Deprecation without removal MAY be a minor or patch change depending on consumer impact.
- Removing a deprecated item is still evaluated as a removal by [RFC-0007](RFC-0007-Compatibility-Policy.md).
- Changing `metadata.id` is an identity change governed by [RFC-0006](RFC-0006-Naming-Convention.md) and compatibility rules in [RFC-0007](RFC-0007-Compatibility-Policy.md).
- When compatibility is `Unknown`, authors SHOULD NOT publish the change as a patch release.

## 6. Pre-release and Build Metadata

Pre-release versions MAY be used for drafts, previews, release candidates and experimental packages. Consumers SHOULD NOT treat pre-release versions as stable unless explicitly configured.

Build metadata MAY record build provenance, timestamp, source revision or packaging details. Build metadata MUST NOT change the meaning of the versioned contract.

## 7. Examples

### 7.1 Compatible schema addition

Adding an optional schema property can be released as `1.3.0` when [RFC-0007](RFC-0007-Compatibility-Policy.md) classifies the change as compatible.

### 7.2 Breaking resource contract change

Changing the meaning of an existing required `spec` field requires a major version bump, such as `1.4.2` to `2.0.0`.

### 7.3 Dependency range

A resource dependency may request `>=1.2.0 <2.0.0`. This RFC defines the range syntax; resolver choice is delegated to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## References

- [RFC-0006 — Naming Convention](RFC-0006-Naming-Convention.md)
- [RFC-0007 — Compatibility Policy](RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
