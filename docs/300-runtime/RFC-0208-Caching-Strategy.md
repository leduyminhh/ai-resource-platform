# RFC-0208 — Caching Strategy

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines cache keys, invalidation and reproducible build behavior.

This RFC owns caching semantics. Serialization determinism is owned by [RFC-0101](../200-resource-model/RFC-0101-Serialization.md); the deterministic-builds principle by [RFC-0002](../000-overview/RFC-0002-Guiding-Principles.md); plans that expose cache keys by [RFC-0205](RFC-0205-Planning-Engine.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Cache Keys

- A cache key MUST be derived from the content and context that determine a step's output (resource content, resolved inputs, adapter identity).
- Cache keys MUST be reproducible: identical inputs produce identical keys.

## 3. Invalidation

- A cache entry MUST be invalidated when any input contributing to its key changes.
- Cache invalidation MUST be conservative: when in doubt, recompute rather than serve a stale entry.

## 4. Reproducible Builds

- Reproducible builds: identical inputs MUST yield identical outputs.
- Reproducibility relies on deterministic serialization ([RFC-0101](../200-resource-model/RFC-0101-Serialization.md)) and the deterministic-builds principle ([RFC-0002](../000-overview/RFC-0002-Guiding-Principles.md)).

## 5. Caching Behavior

- Caching MUST NOT change the produced output for identical inputs; it only skips recomputation.
- A cache hit MUST be equivalent to recomputation.

## 6. Examples

```yaml
cache:
  key: sha256:aa11bb
  inputs:
    - core/clean-code@1.0.0:sha256:abc123
    - adapter: platform/build-adapter@0.1.0
  state: hit
```

## References

- [RFC-0002 — Guiding Principles](../000-overview/RFC-0002-Guiding-Principles.md)
- [RFC-0101 — Serialization](../200-resource-model/RFC-0101-Serialization.md)
- [RFC-0205 — Planning Engine](RFC-0205-Planning-Engine.md)
