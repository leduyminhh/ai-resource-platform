# RFC-0007 Compatibility Policy Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0007 — Compatibility Policy` into the normative compatibility policy and matrix for ARPS.

**Architecture:** Keep `RFC-0007` focused on classification and compatibility matrix only. Use `tools/check-rfc-links` as the deterministic guard: first extend it to catch RFC-0007 boilerplate/missing policy sections, then rewrite the RFC until the guard passes.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — extend the existing docs validator with RFC-0007-specific assertions.
- Modify: `docs/100-foundation/RFC-0007-Compatibility-Policy.md` — replace boilerplate with the policy/matrix content.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0007-compatibility-policy-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: `docs/100-foundation/RFC-0005-Versioning.md`, `docs/100-foundation/RFC-0006-Naming-Convention.md`.

---

## Task 1: Add RFC-0007 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/100-foundation/RFC-0007-Compatibility-Policy.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,220p' tools/check-rfc-links
```

Expected: script currently checks link integrity, overview header conformance, overview boilerplate removal, and RFC-0001 terminology coverage.

- [ ] **Step 2: Add RFC-0007 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 5: RFC-0007 Compatibility Policy ownership ---
COMPAT="$ROOT/docs/100-foundation/RFC-0007-Compatibility-Policy.md"
if [ -f "$COMPAT" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$COMPAT" || err "missing header field ${field//\\/} in RFC-0007"
  done

  for bad in 'Canonical Model' 'Required Behavior' 'Runtime Flow' 'Error Model' 'Security Considerations' 'Migration Guidance' 'Future Work'; do
    grep -qE "^#+ .*$bad" "$COMPAT" && err "boilerplate heading '$bad' still in RFC-0007"
  done

  for required in 'Compatibility Classes' 'General Rules' 'Compatibility Matrix' 'Deprecation Policy' 'Decision Rules' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$COMPAT" || err "missing RFC-0007 section '$required'"
  done

  for class in 'Compatible' 'Conditionally Compatible' 'Breaking' 'Unknown'; do
    grep -q "$class" "$COMPAT" || err "missing RFC-0007 compatibility class '$class'"
  done

  for subject in 'Resource' 'Schema' 'Adapter' 'Package' 'Registry'; do
    grep -q "$subject" "$COMPAT" || err "missing RFC-0007 matrix subject '$subject'"
  done
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0007, including boilerplate headings such as `Canonical Model`, `Runtime Flow`, `Error Model`, `Security Considerations`, `Migration Guidance`, or `Future Work`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0007 compatibility policy"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0007 Compatibility Policy

**Files:**
- Modify: `docs/100-foundation/RFC-0007-Compatibility-Policy.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0007-compatibility-policy-design.md`

- [ ] **Step 1: Replace RFC-0007 content**

Replace the full content of `docs/100-foundation/RFC-0007-Compatibility-Policy.md` with:

```markdown
# RFC-0007 — Compatibility Policy

**Status:** Draft  
**Category:** 100-foundation  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines how compatibility is classified for ARPS resources, schemas, adapters, packages and registries.

This RFC owns compatibility policy. Version number semantics are defined by [RFC-0005](RFC-0005-Versioning.md); canonical resource fields are defined by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md); validation reporting is defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

Compatibility classification answers one question: can an existing valid consumer continue to read, validate, resolve or execute a platform object without changing its own code or configuration?

`RFC-0007` defines the classification policy. It does not define SemVer syntax, canonical resource schema, validator output shape, runtime flow or package publishing mechanics.

## 2. Compatibility Classes

| Class | Meaning |
|---|---|
| `Compatible` | Existing valid consumers can continue without code or configuration changes. |
| `Conditionally Compatible` | Consumers can continue only when they support a declared capability, schema policy, alias or migration path. |
| `Breaking` | A previously valid consumer can fail, misinterpret data or produce artifacts with different meaning. |
| `Unknown` | The implementation lacks enough information to classify the change safely. |

Implementations MUST NOT treat `Unknown` as `Compatible` by default.

## 3. General Rules

- Adding an optional field is `Compatible` when the active schema policy allows extensions.
- Adding a required field is `Breaking` for existing producers or manifests that do not provide it.
- Removing or renaming a field is `Breaking` unless a declared alias or migration path preserves the old meaning.
- Changing a field type is `Breaking` unless the old representation remains accepted.
- Changing the meaning of an existing field is `Breaking` even when the serialized type is unchanged.
- Tightening validation constraints is `Breaking` for existing valid values that become invalid.
- Loosening validation constraints is `Compatible` when consumers can safely ignore the additional accepted values.
- Adding enum values is `Conditionally Compatible` unless all known consumers are required to ignore unknown enum values.
- Deprecating an item is not itself `Breaking`; removing the item is evaluated as a removal.
- When multiple rules apply, implementations MUST choose the stricter classification.

Strictness order is: `Breaking` > `Unknown` > `Conditionally Compatible` > `Compatible`.

## 4. Compatibility Matrix

This matrix defines default classifications. RFCs that own a subject MAY define stricter rules, but they SHOULD reference this policy instead of repeating it.

| Subject | Change | Default class | Notes |
|---|---|---|---|
| Resource | Add optional `metadata.labels` or `metadata.annotations` key | `Compatible` | Consumers SHOULD ignore unknown labels and annotations. |
| Resource | Change `metadata.id` | `Breaking` | Registry identity changes unless an alias maps old ID to new ID. |
| Resource | Change `kind` | `Breaking` | The resource is no longer the same contract. |
| Resource | Add optional `spec` field | `Compatible` | Requires schema policy that permits additive fields. |
| Resource | Add required `spec` field | `Breaking` | Existing manifests become invalid. |
| Resource | Add or change `status` field | `Compatible` | Applies when `status` is observational and not required for resolution. |
| Schema | Add optional property | `Compatible` | Compatible when unknown-field handling is defined. |
| Schema | Add required property | `Breaking` | Existing valid documents can fail validation. |
| Schema | Remove property | `Breaking` | Unless an alias or migration preserves the old property. |
| Schema | Add enum value | `Conditionally Compatible` | Compatible only for consumers that handle unknown enum values. |
| Schema | Narrow numeric/string constraints | `Breaking` | Existing valid values can become invalid. |
| Schema | Widen numeric/string constraints | `Compatible` | Compatible when downstream consumers do not assume the narrower range. |
| Adapter | Add declared capability | `Compatible` | Existing consumers can ignore unused capabilities. |
| Adapter | Remove declared capability | `Breaking` | Plans requiring that capability can no longer execute. |
| Adapter | Change output shape | `Breaking` | Unless the previous output shape remains available. |
| Adapter | Add supported resource kind | `Compatible` | Existing kinds keep behavior. |
| Adapter | Change vendor mapping semantics | `Breaking` | Same input can produce a different artifact meaning. |
| Package | Add resource without changing existing exports | `Compatible` | Existing consumers can keep resolving previous exports. |
| Package | Remove exported resource | `Breaking` | Existing dependencies can no longer resolve. |
| Package | Narrow dependency range | `Breaking` | Existing lock or resolver choices can become invalid. |
| Package | Widen dependency range | `Conditionally Compatible` | Compatible when resolver policy remains deterministic. |
| Package | Add integrity metadata | `Compatible` | Existing content remains the same; stronger verification may be optional. |
| Registry | Add new resource ID | `Compatible` | Existing IDs remain stable. |
| Registry | Move namespace without alias | `Breaking` | Existing references cannot resolve. |
| Registry | Move namespace with alias | `Conditionally Compatible` | Requires consumers to support alias resolution. |
| Registry | Mark resource deprecated | `Compatible` | Existing references still resolve. |
| Registry | Delete resource version | `Breaking` | Existing locks or dependency ranges can fail. |

## 5. Deprecation Policy

Deprecated items MUST continue to document their current meaning until they are removed.

A deprecation notice SHOULD include:

- the replacement item, when one exists;
- the reason for deprecation;
- the compatibility window or version boundary;
- the migration path for consumers.

Deprecation without removal is `Compatible`. Removal after deprecation is still classified by the removal rules in this RFC and by versioning rules in [RFC-0005](RFC-0005-Versioning.md).

## 6. Decision Rules

Implementations evaluating compatibility MUST apply these rules in order:

1. Identify the subject: Resource, Schema, Adapter, Package or Registry.
2. Apply the most specific matrix row for that subject.
3. Apply the general rules when no matrix row matches.
4. Return `Unknown` when required metadata, schema policy or capability declarations are missing.
5. Choose the stricter classification when two rules apply.

Compatibility reporting and error codes, including `COMPATIBILITY_ERROR`, are owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 7. Examples

### 7.1 Add optional field

Adding `spec.timeoutSeconds` as an optional field is `Compatible` when older consumers can ignore it and the active schema policy allows unknown or additive fields.

### 7.2 Add required field

Adding required `spec.runtime` is `Breaking` because existing manifests that omit it become invalid.

### 7.3 Add enum value

Adding lifecycle value `Archived` is `Conditionally Compatible`: consumers that ignore unknown lifecycle values can continue; consumers that exhaustively match known values need an update.

### 7.4 Deprecate adapter capability

Deprecating capability `generate-plan-v1` in favor of `generate-plan-v2` is `Compatible` while both capabilities remain declared. Removing `generate-plan-v1` is `Breaking` for plans that require it.

### 7.5 Move registry namespace

Moving `core/clean-code` to `foundation/clean-code` is `Breaking` without an alias. It is `Conditionally Compatible` when the registry declares an alias from the old ID to the new ID and consumers support alias resolution.

## References

- [RFC-0005 — Versioning](RFC-0005-Versioning.md)
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0107 — Packaging Model](../200-resource-model/RFC-0107-Packaging-Model.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0007**

Run:

```bash
grep -n 'RFC-0007 — Compatibility Policy' SUMMARY.md
```

Expected: output includes `docs/100-foundation/RFC-0007-Compatibility-Policy.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/100-foundation/RFC-0007-Compatibility-Policy.md
git commit -m "docs(rfc): viet lai RFC-0007 Compatibility Policy"
```

Expected: one commit containing only `docs/100-foundation/RFC-0007-Compatibility-Policy.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0007 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0007 Compatibility Policy — nhà của Compatibility
```

to:

```markdown
- [x] RFC-0007 Compatibility Policy — nhà của Compatibility
```

- [ ] **Step 2: Run full docs validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Verify only intended files changed**

Run:

```bash
git status --short
```

Expected: only `TODO.md` is modified after Task 1 and Task 2 commits.

- [ ] **Step 4: Commit closure note**

Run:

```bash
git add TODO.md
git commit -m "docs: dong pending RFC-0007 compatibility policy"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0007-specific guardrails.
- `docs/100-foundation/RFC-0007-Compatibility-Policy.md` contains compatibility classes, general rules, matrix, deprecation policy, decision rules, examples and references.
- `docs/100-foundation/RFC-0007-Compatibility-Policy.md` no longer contains boilerplate headings `Canonical Model`, `Required Behavior`, `Runtime Flow`, `Error Model`, `Security Considerations`, `Migration Guidance` or `Future Work`.
- `TODO.md` marks the RFC-0007 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/100-foundation/RFC-0007-Compatibility-Policy.md`.
- `RFC-0005` and `RFC-0006` remain unchanged in this plan.