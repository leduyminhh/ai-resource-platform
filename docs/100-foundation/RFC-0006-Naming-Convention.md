# RFC-0006 — Naming Convention

**Status:** Draft  
**Category:** 100-foundation  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines stable naming rules for ARPS namespaces, resource names, `metadata.id`, resource kinds, file slugs and package slugs.

This RFC owns naming grammar and identity rules. Canonical resource envelope fields are defined by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md); version semantics are defined by [RFC-0005](RFC-0005-Versioning.md); compatibility classification is defined by [RFC-0007](RFC-0007-Compatibility-Policy.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

Names in this RFC are portable identifiers. Human-facing display titles MAY use richer text, but portable identifiers SHOULD remain stable, lowercase and safe for filesystems, registries and URLs.

## 2. Naming Principles

- Names SHOULD be stable, readable and deterministic.
- Names and slugs SHOULD use lowercase ASCII letters, digits and hyphens unless a more specific RFC allows otherwise.
- Registry-facing identifiers SHOULD avoid whitespace, path traversal segments and shell-sensitive characters.
- A name SHOULD describe what the resource is, not where it currently lives.
- File paths MAY mirror resource names but MUST NOT be the source of resource identity.

## 3. Identifier Grammar

The default grammar is:

```text
namespace      = segment *("/" segment)
resource-name  = segment
metadata.id    = namespace "/" resource-name
segment        = lowercase letter or digit *(lowercase letter / digit / "-")
kind           = UpperCamelCase identifier
slug           = segment *("-" segment)
```

Rules:

- `metadata.id` SHOULD use `namespace/name` form.
- `namespace` SHOULD group ownership, domain or distribution scope.
- `resource-name` SHOULD be unique inside its namespace.
- `kind` SHOULD use UpperCamelCase, such as `Prompt`, `Workflow` or `Adapter`.
- `slug` SHOULD be lowercase and portable for files, packages and URLs.

## 4. Resource Identity

`metadata.id` is the stable registry identity for a resource. `metadata.name` MAY repeat the final name segment for local readability, but consumers MUST NOT treat `metadata.name` alone as globally unique.

A resource identity is evaluated with its kind and version context:

| Field | Role |
|---|---|
| `kind` | Declares the resource contract family. |
| `metadata.id` | Declares stable resource identity. |
| `metadata.name` | Provides local display or shorthand name. |
| `metadata.version` | Declares the version of that identity's contract. |

## 5. Kinds and Slugs

Resource kinds SHOULD be stable contract names. Changing `kind` changes the resource contract family and is compatibility-sensitive under [RFC-0007](RFC-0007-Compatibility-Policy.md).

File and package slugs SHOULD be derived from stable names when practical. A repository MAY choose a different directory layout, but the canonical identity remains `metadata.id`.

## 6. Rename and Alias Rules

- Changing `metadata.id` is a breaking identity change unless an alias or migration path preserves consumers.
- Renaming files without changing `metadata.id` SHOULD NOT change resource identity.
- Registry aliases MAY map an old ID to a new ID when a migration path is explicit.
- Deprecation SHOULD be preferred before removing a public resource ID.
- Alias storage and registry mechanics are owned by registry RFCs, not this naming convention.

## 7. Examples

### 7.1 Prompt resource ID

```yaml
kind: Prompt
metadata:
  id: prompts/customer-support-summary
  name: customer-support-summary
  version: 1.0.0
```

### 7.2 Workflow resource ID

```yaml
kind: Workflow
metadata:
  id: workflows/release-notes
  name: release-notes
  version: 1.2.0
```

### 7.3 File path is not identity

A file may move from `prompts/support/summary.yaml` to `resources/prompts/customer-support-summary.yaml`. If `metadata.id` remains `prompts/customer-support-summary`, consumers still refer to the same resource identity.

## References

- [RFC-0005 — Versioning](RFC-0005-Versioning.md)
- [RFC-0007 — Compatibility Policy](RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](../200-resource-model/RFC-0102-Metadata-Model.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
