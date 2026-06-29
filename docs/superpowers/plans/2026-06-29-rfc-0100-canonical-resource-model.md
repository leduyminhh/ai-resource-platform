# RFC-0100 Canonical Resource Model Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0100 — Canonical Resource Model` into the normative resource envelope and invariant contract for ARPS.

**Architecture:** Keep `RFC-0100` focused on top-level resource envelope semantics only. Extend `tools/check-rfc-links` with RFC-0100 guardrails first, then rewrite the RFC until the guard passes and close the pending delegation.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add RFC-0100-specific assertions for envelope ownership.
- Modify: `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md` — replace boilerplate with resource envelope content.
- Modify: `TODO.md` — mark the RFC-0100 pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0100-canonical-resource-model-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: `docs/200-resource-model/RFC-0102-Metadata-Model.md`, `docs/200-resource-model/RFC-0103-Resource-Lifecycle.md`, `docs/200-resource-model/RFC-0106-Validation-Model.md`.

---

## Task 1: Add RFC-0100 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,260p' tools/check-rfc-links
```

Expected: script includes overview checks and the existing RFC-0007 compatibility policy guard.

- [ ] **Step 2: Add RFC-0100 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 6: RFC-0100 Canonical Resource Model ownership ---
CANONICAL="$ROOT/docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md"
if [ -f "$CANONICAL" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$CANONICAL" || err "missing header field ${field//\\/} in RFC-0100"
  done

  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Runtime Flow' 'Validation Rules' 'Error Model' 'Security Considerations' 'Compatibility' 'Migration Guidance' 'Future Work'; do
    grep -qE "^#+ .*$bad" "$CANONICAL" && err "boilerplate heading '$bad' still in RFC-0100"
  done

  for required in 'Resource Envelope' 'Top-Level Fields' 'Required Behavior' 'Envelope Invariants' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$CANONICAL" || err "missing RFC-0100 section '$required'"
  done

  for field in 'apiVersion' 'kind' 'metadata' 'spec' 'status'; do
    grep -q "$field" "$CANONICAL" || err "missing RFC-0100 envelope field '$field'"
  done

  grep -q 'status.*MUST NOT.*desired configuration\|desired configuration.*MUST NOT.*status' "$CANONICAL" || err "RFC-0100 must state status does not define desired configuration"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0100, including boilerplate headings such as `Motivation`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Security Considerations`, `Compatibility`, `Migration Guidance`, or `Future Work`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0100 canonical resource model"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0100 Canonical Resource Model

**Files:**
- Modify: `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0100-canonical-resource-model-design.md`

- [ ] **Step 1: Replace RFC-0100 content**

Replace the full content of `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md` with:

```markdown
# RFC-0100 — Canonical Resource Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the canonical resource envelope shared by every ARPS Resource Kind.

This RFC owns the top-level contract for `apiVersion`, `kind`, `metadata`, `spec` and `status`. Detailed metadata fields are defined by [RFC-0102](RFC-0102-Metadata-Model.md); serialization is defined by [RFC-0101](RFC-0101-Serialization.md); validation reporting is defined by [RFC-0106](RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A Resource is a platform object represented by a common envelope plus a kind-specific `spec`. This RFC defines the envelope and invariants only. It does not define every metadata field, lifecycle state, dependency edge, validation error or package format.

## 2. Resource Envelope

Every ARPS Resource MUST use this top-level envelope:

```yaml
apiVersion: platform/v1
kind: Skill
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.0.0
spec: {}
```

`apiVersion`, `kind`, `metadata` and `spec` are required. `status` is optional and SHOULD be generated by tooling or runtime systems.

A Resource MAY include generated status:

```yaml
apiVersion: platform/v1
kind: Skill
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.0.0
spec: {}
status:
  observedGeneration: 1
  lifecycle: Published
```

## 3. Top-Level Fields

| Field | Required | Semantics |
|---|---|---|
| `apiVersion` | Yes | API/spec family used to interpret the resource envelope. It is distinct from `metadata.version`. |
| `kind` | Yes | PascalCase Resource Kind selecting the kind-specific contract. Naming rules are owned by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md). |
| `metadata` | Yes | Identity, version and registry/search metadata. Detailed fields are owned by [RFC-0102](RFC-0102-Metadata-Model.md). |
| `spec` | Yes | Desired configuration, capability declaration or authored intent. The schema is owned by the RFC for the specific kind. |
| `status` | No | Observed/generated state produced by tooling or runtime systems. Lifecycle details are owned by [RFC-0103](RFC-0103-Resource-Lifecycle.md). |

## 4. Required Behavior

- Producers MUST emit `apiVersion`, `kind`, `metadata` and `spec`.
- Producers MAY omit `status`.
- Consumers MUST NOT require `status` for initial parse, validation or dependency resolution.
- Read-only phases MUST NOT mutate source resource files.
- Tools that generate `status` MUST keep desired configuration in `spec`, not `status`.
- Parsers SHOULD preserve known fields.
- Parsers SHOULD NOT drop unknown extension fields unless the active schema policy requires rejection.
- Unknown top-level fields MUST be handled by schema and validation policy, not by ad-hoc implementation behavior.

## 5. Envelope Invariants

- `apiVersion` and `kind` together select the schema family used to interpret `spec`.
- `metadata.id` identifies the resource inside a registry context.
- `metadata.version` identifies the resource instance version and is distinct from `apiVersion`.
- `spec` represents authored desired state or declared capability.
- `status` represents observed/generated state and MUST NOT define desired configuration.
- `metadata` MUST NOT be used as a substitute for kind-specific behavioral configuration.
- A resource without `status` can still be valid, resolvable and packageable.

## 6. Examples

### 6.1 Minimal valid resource

```yaml
apiVersion: platform/v1
kind: Prompt
metadata:
  id: core/summarize
  name: summarize
  version: 1.0.0
spec: {}
```

### 6.2 Resource with metadata extensions

```yaml
apiVersion: platform/v1
kind: Skill
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.0.0
  labels:
    domain: engineering
  annotations:
    owner: platform-team
spec: {}
```

Labels and annotations are metadata extensions. Their detailed policy is owned by [RFC-0102](RFC-0102-Metadata-Model.md) and compatibility behavior is owned by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md).

### 6.3 Resource with generated status

```yaml
apiVersion: platform/v1
kind: Workflow
metadata:
  id: core/build
  name: build
  version: 1.0.0
spec: {}
status:
  observedGeneration: 3
  lifecycle: Published
```

The `status` block is generated state. Removing it MUST NOT change the desired configuration represented by `spec`.

### 6.4 Invalid status usage

```yaml
apiVersion: platform/v1
kind: Adapter
metadata:
  id: adapters/example
  name: example
  version: 1.0.0
spec: {}
status:
  desiredRuntime: production
```

This pattern is invalid because desired configuration is stored in `status`. Desired runtime configuration belongs in `spec` under the adapter-specific contract.

## References

- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
- [RFC-0101 — Serialization](RFC-0101-Serialization.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0103 — Resource Lifecycle](RFC-0103-Resource-Lifecycle.md)
- [RFC-0105 — Manifest](RFC-0105-Manifest.md)
- [RFC-0106 — Validation Model](RFC-0106-Validation-Model.md)
- [RFC-0107 — Packaging Model](RFC-0107-Packaging-Model.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0100**

Run:

```bash
grep -n 'RFC-0100 — Canonical Resource Model' SUMMARY.md
```

Expected: output includes `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md
git commit -m "docs(rfc): viet lai RFC-0100 Canonical Resource Model"
```

Expected: one commit containing only `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0100 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0100 Canonical Resource Model — nhà của Canonical Model, Required Behavior
```

to:

```markdown
- [x] RFC-0100 Canonical Resource Model — nhà của Canonical Model, Required Behavior
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
git commit -m "docs: dong pending RFC-0100 canonical resource model"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0100-specific guardrails.
- `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md` contains `Resource Envelope`, `Top-Level Fields`, `Required Behavior`, `Envelope Invariants`, `Examples` and `References`.
- `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md` no longer contains boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Security Considerations`, `Compatibility`, `Migration Guidance` or `Future Work`.
- RFC-0100 states required fields: `apiVersion`, `kind`, `metadata`, `spec`.
- RFC-0100 states `status` is optional/generated and MUST NOT define desired configuration.
- `TODO.md` marks the RFC-0100 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/200-resource-model/RFC-0100-Canonical-Resource-Model.md`.
- Related RFCs such as `RFC-0102`, `RFC-0103` and `RFC-0106` remain unchanged in this plan.