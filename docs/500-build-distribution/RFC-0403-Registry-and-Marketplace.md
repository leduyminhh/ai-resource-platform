# RFC-0403 — Registry and Marketplace

**Status:** Draft  
**Category:** 500-build-distribution  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## 1. Abstract

Defines distribution layer responsibilities. Registry stores packages and metadata. Marketplace indexes, searches and presents packages.

## 2. Motivation

This RFC standardizes a core part of ARPS so multiple implementations can remain interoperable, vendor-neutral and implementation-independent.

## 3. Goals

- Define a stable contract.
- Support non-invasive migration.
- Keep the platform resource-oriented.
- Preserve deterministic behavior.
- Allow future extension without changing the core architecture.

## 4. Non-Goals

- This RFC does not mandate a specific programming language.
- This RFC does not depend on a specific AI assistant, IDE or vendor.
- This RFC does not force restructuring existing business source code.

## 5. Canonical Model

Every platform object SHOULD be represented as a canonical resource:

```yaml
apiVersion: platform/v1
kind: ResourceKind
metadata:
  id: namespace/name
  name: name
  version: 1.0.0
  labels: {}
  annotations: {}
spec: {}
status:
  lifecycle: Draft
```

## 6. Required Behavior

- Implementations MUST parse canonical resources.
- Implementations MUST validate required fields before resolution.
- Implementations SHOULD produce deterministic output.
- Implementations MUST NOT mutate source resources during read-only phases.

## 7. Runtime Flow

```text
Repository
  -> Discovery Engine
  -> Registry Engine
  -> Validation Engine
  -> Dependency Resolver
  -> Planning Engine
  -> Execution Engine
  -> Packaging Engine
  -> Publishing Engine
  -> Registry / Marketplace / Consumer
```

## 8. Validation Rules

- Required fields MUST be present.
- Resource IDs MUST be unique inside a registry.
- Versions SHOULD follow Semantic Versioning.
- Dependency graphs MUST be acyclic.
- Unknown fields MUST follow the active schema policy.

## 9. Error Model

- `SCHEMA_ERROR`
- `METADATA_ERROR`
- `DEPENDENCY_ERROR`
- `COMPATIBILITY_ERROR`
- `POLICY_VIOLATION`
- `BUILD_ERROR`

## 10. Security Considerations

- Remote resources SHOULD be verified before use.
- Packages SHOULD include checksums.
- Secrets MUST NOT be stored in plain resource manifests.
- Registries SHOULD be explicitly trusted.

## 11. Compatibility

- Breaking changes require a new major version.
- Additive fields are allowed when schema policy permits extension.
- Implementations SHOULD ignore unknown labels and annotations.

## 12. Example

```yaml
apiVersion: platform/v1
kind: Example
metadata:
  id: example/default
  name: default
  version: 1.0.0
spec: {}
```

## 13. Migration Guidance

- Discover existing assets first.
- Add metadata without moving files.
- Register resources.
- Resolve dependencies.
- Build through adapters only after validation passes.



## 14. Future Work

- Formal conformance tests.
- Reference runtime implementation.
- Registry interoperability suite.
- Extended JSON Schema and YAML Schema definitions.
