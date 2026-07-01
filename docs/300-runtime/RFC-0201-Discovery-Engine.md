# RFC-0201 — Discovery Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines how resources are discovered from filesystem, package and registry sources.

This RFC owns discovery behavior and the `ResourceCandidate` model. Canonical parsing and the envelope are owned by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md); storage and lookup of discovered resources by [RFC-0202](RFC-0202-Registry-Engine.md); validation by [RFC-0203](RFC-0203-Validation-Engine.md); overall pipeline orchestration by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Discovery Sources

| Source | Description |
|---|---|
| filesystem | Resource files (YAML/JSON) under a workspace path. |
| package | Resources bundled inside a package. |
| registry | Resources exposed by a runtime registry source. |

Discovery reads from filesystem, package and registry sources and emits candidates without interpreting business logic.

## 3. Resource Candidate

A `ResourceCandidate` is a discovered, not-yet-validated resource together with its origin.

| Field | Semantics |
|---|---|
| `source` | Origin kind (`filesystem`, `package` or `registry`). |
| `location` | Path, package reference or registry coordinate. |
| `document` | Raw serialized document for later parsing. |

- Discovery emits one `ResourceCandidate` per discovered document.
- Discovery MUST NOT mutate source resources.

## 4. Source Precedence

- When the same `metadata.id` appears in multiple sources, default precedence is `filesystem` > `package` > `registry` unless configured otherwise.
- Duplicate candidates MUST be de-duplicated deterministically by id, version and source precedence.
- Discovery MUST produce deterministic ordering for identical inputs.

## 5. Discovery Behavior

- Discovery is a read-only phase; it MUST NOT write, move or modify sources.
- Unreadable or malformed sources SHOULD be surfaced as candidates carrying an error marker for the validation engine, not silently dropped.
- Discovered candidates are handed to the runtime registry ([RFC-0202](RFC-0202-Registry-Engine.md)).

## 6. Examples

```yaml
candidate:
  source: filesystem
  location: examples/resources/skill.yaml
  document: |
    apiVersion: platform/v1
    kind: Skill
    metadata:
      id: core/clean-code
      name: clean-code
      version: 1.0.0
```

## References

- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0202 — Registry Engine](RFC-0202-Registry-Engine.md)
- [RFC-0203 — Validation Engine](RFC-0203-Validation-Engine.md)
