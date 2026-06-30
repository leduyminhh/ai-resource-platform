# RFC-0101 — Serialization

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines YAML and JSON as serialization mappings of the canonical resource model.

This RFC owns serialization format policy, encoding and round-trip rules. The resource envelope and top-level fields are defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); metadata field semantics by [RFC-0102](RFC-0102-Metadata-Model.md); validation of serialized documents by [RFC-0106](RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Format Policy

YAML and JSON are supported serialization mappings of the same canonical object model. The canonical object model is the source of truth; YAML and JSON are interchangeable encodings of it.

- A resource MAY be authored in YAML or JSON.
- Both encodings MUST represent the same canonical object for the same resource.
- Runtime engines MUST operate on parsed canonical objects, not raw YAML or JSON text.

## 3. Encoding

- Serialized documents MUST be UTF-8 encoded.
- Map keys MUST be strings; map key ordering is not semantically significant.
- Documents SHOULD NOT rely on encoding-specific escapes that do not survive a round-trip.

## 4. Serialization Rules

- Serialization SHOULD be deterministic: identical canonical objects SHOULD produce byte-identical output under the same serializer settings.
- Tools SHOULD emit map keys in a stable order to support reproducible diffs and checksums.
- Empty optional fields SHOULD be omitted rather than serialized as null, unless null is semantically meaningful.
- Scalar types (string, number, boolean, null) MUST be preserved across YAML and JSON.

## 5. Round-trip

- Parsing a serialized document and re-serializing it MUST preserve the canonical object (round-trip fidelity).
- Converting YAML to JSON and back MUST preserve the canonical object.
- A round-trip MUST NOT silently drop unknown fields permitted by the active schema policy.

## 6. Examples

### 6.1 YAML

```yaml
apiVersion: platform/v1
kind: Skill
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.0.0
spec:
  dependencies: []
status:
  lifecycle: Published
```

### 6.2 Equivalent JSON

```json
{
  "apiVersion": "platform/v1",
  "kind": "Skill",
  "metadata": { "id": "core/clean-code", "name": "clean-code", "version": "1.0.0" },
  "spec": { "dependencies": [] },
  "status": { "lifecycle": "Published" }
}
```

## References

- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0106 — Validation Model](RFC-0106-Validation-Model.md)
