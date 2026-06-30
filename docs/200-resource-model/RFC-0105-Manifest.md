# RFC-0105 — Manifest

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines manifest structure for resources, packages and registries.

This RFC owns manifest structure: how resources are declared and collected. The canonical resource envelope is defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); the logical package model by [RFC-0107](RFC-0107-Packaging-Model.md) and packaging mechanics by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); registry mechanics by [RFC-0202](../300-runtime/RFC-0202-Registry-Engine.md) and [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Manifest Purpose

A manifest declares or collects resources by `metadata.id` and version so tooling can discover, group and distribute them.

- A manifest MUST reference resources by `metadata.id`.
- A manifest MUST NOT redefine the canonical resource envelope (owned by [RFC-0100](RFC-0100-Canonical-Resource-Model.md)).
- A manifest is itself a canonical resource and follows the envelope rules.

## 3. Resource Manifest

A resource manifest lists resources that belong together.

```yaml
apiVersion: platform/v1
kind: Manifest
metadata:
  id: core/manifest
  name: core
  version: 1.0.0
spec:
  resources:
    - id: core/clean-code
      version: ^1.0.0
```

## 4. Package Manifest

A package manifest declares the resources contained in a package. The package format and mechanics are owned by [RFC-0107](RFC-0107-Packaging-Model.md) and [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md).

- A package manifest MUST list contained resources by `metadata.id` and resolved version.

## 5. Registry Manifest

A registry manifest declares which packages or resources a registry exposes. Registry mechanics are owned by [RFC-0202](../300-runtime/RFC-0202-Registry-Engine.md) and [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).

## 6. Manifest Rules

- Referenced resource IDs MUST be unique within a manifest.
- A manifest SHOULD declare version ranges for resource references; range resolution is owned by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).
- A manifest MUST NOT embed full resource bodies in place of references unless the format explicitly allows inlining.

## 7. Examples

```yaml
apiVersion: platform/v1
kind: Manifest
metadata:
  id: plugins/backend-manifest
  name: backend-manifest
  version: 1.0.0
spec:
  resources:
    - id: plugins/backend
      version: ^1.0.0
    - id: core/clean-code
      version: ^1.0.0
```

## References

- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0107 — Packaging Model](RFC-0107-Packaging-Model.md)
- [RFC-0202 — Registry Engine](../300-runtime/RFC-0202-Registry-Engine.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
