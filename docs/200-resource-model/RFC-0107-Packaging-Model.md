# RFC-0107 — Packaging Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the logical model of immutable packages produced by the platform.

This RFC owns the logical package model: what a package is, what it contains and how it is identified. Packaging mechanics (compression, signing, verification) are defined by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); publishing by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md); the artifact relationship by [RFC-0207](../300-runtime/RFC-0207-Artifact-Model.md); signing and integrity by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Package Concept

A package is an immutable, distributable unit that bundles one or more canonical resources together with a manifest describing its contents.

- A package is produced by the build pipeline and is not edited in place.
- A package is described by a package manifest ([RFC-0105](RFC-0105-Manifest.md)).

## 3. Package Contents

- A package MUST contain a package manifest listing the resources it bundles by `metadata.id` and resolved version.
- A package MAY contain resolved dependency snapshots; lock semantics are owned by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).
- A package MUST NOT contain plaintext secrets.

## 4. Package Identity

- A package is identified by name, version and content checksum.
- The checksum and signing mechanics are defined by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).
- Two packages with the same name and version MUST have identical content (same checksum).

## 5. Immutability

- Packages are immutable once produced; any change MUST produce a new version.
- Republishing the same name and version with different content MUST be rejected.
- Compression, signing and verification mechanics are owned by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md).

## 6. Examples

```yaml
package:
  name: plugins/backend
  version: 1.0.0
  checksum: sha256:0d1f...c3
  contents:
    - id: plugins/backend
      version: 1.0.0
    - id: core/clean-code
      version: 1.0.0
```

## References

- [RFC-0105 — Manifest](RFC-0105-Manifest.md)
- [RFC-0207 — Artifact Model](../300-runtime/RFC-0207-Artifact-Model.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
- [RFC-0801 — Signing and Integrity](../900-security-governance/RFC-0801-Signing-and-Integrity.md)
