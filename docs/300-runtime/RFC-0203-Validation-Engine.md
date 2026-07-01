# RFC-0203 — Validation Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines validation execution over individual resources and registries.

This RFC owns validation execution strategy. The validation model — layers, result envelope and error codes — is owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md); the runtime pipeline by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Engine Responsibility

- The engine executes the validation model defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- It MUST NOT redefine validation layers, the result envelope, or error codes.
- It produces a `ValidationResult` using the [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md) envelope.

## 3. Execution Strategy

- The engine owns execution order, short-circuiting and parallelism across layers.
- The engine MAY validate a single resource or an entire registry (`CanonicalResourceSet`).
- Validation MUST NOT mutate source resources.

## 4. Layer Ordering

- Layers run in the order syntax, schema, metadata, semantic, dependency, compatibility, policy unless a layer's required context is unavailable.
- A layer MAY be skipped only when its required context is absent, producing a diagnostic per [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 5. Caching and Incremental

- The engine MAY cache layer results keyed by resource content plus context; cache keys MUST be reproducible.
- Incremental validation MAY re-run only layers whose inputs changed.
- Caching MUST NOT change the produced `ValidationResult` for identical inputs.

## 6. Result Handoff

- The engine emits a `ValidationResult` consumed by downstream engines.
- A resource with any `error`-severity diagnostic MUST be treated as invalid downstream.

## 7. Examples

```yaml
run:
  target: core/clean-code
  layers: [syntax, schema, metadata, semantic, dependency]
  result:
    valid: true
    diagnostics: []
```

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0202 — Registry Engine](RFC-0202-Registry-Engine.md)
