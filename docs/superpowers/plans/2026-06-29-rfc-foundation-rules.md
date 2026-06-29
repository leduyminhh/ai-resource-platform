# RFC Foundation Rules Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0005 — Versioning`, `RFC-0006 — Naming Convention` and `RFC-0104 — Dependency Graph` into the foundation rules cluster for ARPS.

**Architecture:** Add validator guardrails first, then rewrite one RFC per commit so each ownership boundary is reviewable. Keep versioning, naming and dependency graph as rule-level RFCs while delegating resolver algorithms, lock files, package formats and validation envelopes to their owner RFCs.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add foundation-rules assertions for RFC-0005, RFC-0006 and RFC-0104.
- Modify: `docs/100-foundation/RFC-0005-Versioning.md` — replace boilerplate with versioning rules.
- Modify: `docs/100-foundation/RFC-0006-Naming-Convention.md` — replace boilerplate with naming rules.
- Modify: `docs/200-resource-model/RFC-0104-Dependency-Graph.md` — replace boilerplate with resource-level dependency graph rules.
- Modify: `TODO.md` — mark the foundation-rules pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-foundation-rules-design.md` — approved design source.
- Verify only: `SUMMARY.md` — RFC title/path links should remain unchanged.
- No changes: `docs/300-runtime/RFC-0204-Dependency-Resolver.md`, `docs/500-build-distribution/RFC-0401-Lock-File.md`, `docs/500-build-distribution/RFC-0402-Packaging.md`.

---

## Task 1: Add Foundation Rules Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/100-foundation/RFC-0005-Versioning.md`
- Verify: `docs/100-foundation/RFC-0006-Naming-Convention.md`
- Verify: `docs/200-resource-model/RFC-0104-Dependency-Graph.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,760p' tools/check-rfc-links
```

Expected: script includes overview checks plus existing guards for completed RFCs, including RFC-0007, RFC-0100, RFC-0106, RFC-0200, RFC-0800 and RFC-0901.

- [ ] **Step 2: Add foundation-rules checks**

Append this block before the final success/exit lines:

```bash
# --- Check 11: Foundation rules ownership (RFC-0005, RFC-0006, RFC-0104) ---
VERSIONING="$ROOT/docs/100-foundation/RFC-0005-Versioning.md"
NAMING="$ROOT/docs/100-foundation/RFC-0006-Naming-Convention.md"
DEPENDENCY_GRAPH="$ROOT/docs/200-resource-model/RFC-0104-Dependency-Graph.md"

check_foundation_header() {
  local file="$1"
  local label="$2"
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$file" || err "missing header field ${field//\\/} in $label"
  done
}

check_no_foundation_boilerplate() {
  local file="$1"
  local label="$2"
  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Canonical Model' 'Required Behavior' 'Runtime Flow' 'Validation Rules' 'Error Model' 'Security Considerations' 'Migration Guidance' 'Future Work'; do
    grep -qE "^## ([0-9]+\. )?$bad[[:space:]]*$" "$file" && err "boilerplate heading '$bad' still in $label"
  done
}

if [ -f "$VERSIONING" ]; then
  check_foundation_header "$VERSIONING" "RFC-0005"
  check_no_foundation_boilerplate "$VERSIONING" "RFC-0005"
  for required in 'Version Format' 'Versioned Subjects' 'Version Ranges' 'Change Rules' 'Pre-release and Build Metadata' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$VERSIONING" || err "missing RFC-0005 section '$required'"
  done
  grep -q 'Breaking changes require a major version bump' "$VERSIONING" || err "RFC-0005 must define major bump rule"
  grep -q 'Compatible additive changes SHOULD use a minor version bump' "$VERSIONING" || err "RFC-0005 must define minor bump rule"
  grep -q 'Patch changes MUST preserve compatibility and meaning' "$VERSIONING" || err "RFC-0005 must define patch bump rule"
  grep -q 'Version selection.*RFC-0204\|RFC-0204.*version selection' "$VERSIONING" || err "RFC-0005 must delegate version selection to RFC-0204"
fi

if [ -f "$NAMING" ]; then
  check_foundation_header "$NAMING" "RFC-0006"
  check_no_foundation_boilerplate "$NAMING" "RFC-0006"
  for required in 'Naming Principles' 'Identifier Grammar' 'Resource Identity' 'Kinds and Slugs' 'Rename and Alias Rules' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$NAMING" || err "missing RFC-0006 section '$required'"
  done
  grep -q 'metadata.id' "$NAMING" || err "RFC-0006 must define metadata.id naming"
  grep -q 'namespace/name' "$NAMING" || err "RFC-0006 must define namespace/name IDs"
  grep -q 'Changing `metadata.id` is a breaking identity change' "$NAMING" || err "RFC-0006 must define metadata.id rename behavior"
  grep -q 'File paths MAY mirror resource names but MUST NOT be the source of resource identity' "$NAMING" || err "RFC-0006 must separate file paths from identity"
fi

if [ -f "$DEPENDENCY_GRAPH" ]; then
  check_foundation_header "$DEPENDENCY_GRAPH" "RFC-0104"
  check_no_foundation_boilerplate "$DEPENDENCY_GRAPH" "RFC-0104"
  for required in 'Graph Model' 'Dependency Edge' 'Dependency Constraints' 'Graph Invariants' 'Validation Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$DEPENDENCY_GRAPH" || err "missing RFC-0104 section '$required'"
  done
  grep -q 'directed and acyclic' "$DEPENDENCY_GRAPH" || err "RFC-0104 must define directed acyclic graph rule"
  grep -q 'DEPENDENCY_ERROR' "$DEPENDENCY_GRAPH" || err "RFC-0104 must reference DEPENDENCY_ERROR diagnostics"
  grep -q 'Version selection.*RFC-0204\|RFC-0204.*version selection' "$DEPENDENCY_GRAPH" || err "RFC-0104 must delegate version selection to RFC-0204"
  grep -q 'Lock snapshots.*RFC-0401\|RFC-0401.*lock snapshots' "$DEPENDENCY_GRAPH" || err "RFC-0104 must delegate lock snapshots to RFC-0401"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero. Output includes failures for current RFC-0005, RFC-0006 and RFC-0104 boilerplate headings plus missing required sections.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho cum RFC foundation rules"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0005 Versioning

**Files:**
- Modify: `docs/100-foundation/RFC-0005-Versioning.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-foundation-rules-design.md`

- [ ] **Step 1: Replace RFC-0005 content**

Replace the full content of `docs/100-foundation/RFC-0005-Versioning.md` with:

```markdown
# RFC-0005 — Versioning

**Status:** Draft  
**Category:** 100-foundation  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines version syntax and version-change rules for ARPS specifications, resources, schemas, adapters, packages and runtime components.

This RFC owns version number semantics and range vocabulary. Compatibility classification is defined by [RFC-0007](RFC-0007-Compatibility-Policy.md); dependency version selection is defined by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md); lock snapshots are defined by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A version identifies the contract exposed by a specification, resource, schema, adapter, package or runtime component. It does not define resource identity; resource identity is defined by [RFC-0006](RFC-0006-Naming-Convention.md) and canonical resource metadata by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md).

## 2. Version Format

ARPS versions SHOULD use Semantic Versioning syntax:

```text
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

- `MAJOR`, `MINOR` and `PATCH` MUST be non-negative integers.
- `MAJOR`, `MINOR` and `PATCH` MUST NOT contain leading zeroes unless the value is `0`.
- `PRERELEASE` MAY identify unstable preview contracts.
- `BUILD` MAY identify build metadata and MUST NOT change compatibility semantics.
- Implementations SHOULD compare version precedence using Semantic Versioning rules.

## 3. Versioned Subjects

| Subject | Version meaning |
|---|---|
| Specification | Version of the ARPS specification contract. |
| Resource | Version of the resource contract identified by `metadata.id`. |
| Schema | Version of schema constraints for a resource kind or manifest. |
| Adapter | Version of adapter capabilities and output contract. |
| Package | Version of a distributable bundle. |
| Runtime Component | Version of a runtime component contract or capability set. |

A subject version MUST be interpreted in the context of its subject identity. The same version number on two different resource IDs does not imply compatibility or substitutability.

## 4. Version Ranges

Version ranges MAY be used by dependency declarations, package metadata or capability requirements.

Supported range vocabulary SHOULD include:

| Range form | Meaning |
|---|---|
| `1.2.3` | exactly version `1.2.3` |
| `>=1.2.0` | version greater than or equal to `1.2.0` |
| `<2.0.0` | version lower than `2.0.0` |
| `>=1.2.0 <2.0.0` | intersection of constraints |
| `^1.2.0` | compatible with `1.2.0` until the next major version |
| `~1.2.0` | compatible with `1.2.0` until the next minor version |

This RFC defines range vocabulary only. Version selection and conflict resolution belong to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## 5. Change Rules

- Breaking changes require a major version bump.
- Compatible additive changes SHOULD use a minor version bump.
- Patch changes MUST preserve compatibility and meaning.
- Deprecation without removal MAY be a minor or patch change depending on consumer impact.
- Removing a deprecated item is still evaluated as a removal by [RFC-0007](RFC-0007-Compatibility-Policy.md).
- Changing `metadata.id` is an identity change governed by [RFC-0006](RFC-0006-Naming-Convention.md) and compatibility rules in [RFC-0007](RFC-0007-Compatibility-Policy.md).
- When compatibility is `Unknown`, authors SHOULD NOT publish the change as a patch release.

## 6. Pre-release and Build Metadata

Pre-release versions MAY be used for drafts, previews, release candidates and experimental packages. Consumers SHOULD NOT treat pre-release versions as stable unless explicitly configured.

Build metadata MAY record build provenance, timestamp, source revision or packaging details. Build metadata MUST NOT change the meaning of the versioned contract.

## 7. Examples

### 7.1 Compatible schema addition

Adding an optional schema property can be released as `1.3.0` when [RFC-0007](RFC-0007-Compatibility-Policy.md) classifies the change as compatible.

### 7.2 Breaking resource contract change

Changing the meaning of an existing required `spec` field requires a major version bump, such as `1.4.2` to `2.0.0`.

### 7.3 Dependency range

A resource dependency may request `>=1.2.0 <2.0.0`. This RFC defines the range syntax; resolver choice is delegated to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## References

- [RFC-0006 — Naming Convention](RFC-0006-Naming-Convention.md)
- [RFC-0007 — Compatibility Policy](RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
```

- [ ] **Step 2: Run validator and confirm remaining failures**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero. RFC-0005-specific failures are gone; failures remain for RFC-0006 and RFC-0104 because they are still boilerplate.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0005**

Run:

```bash
grep -n 'RFC-0005 — Versioning' SUMMARY.md
```

Expected: output includes `docs/100-foundation/RFC-0005-Versioning.md`.

- [ ] **Step 4: Commit RFC-0005 rewrite**

Run:

```bash
git add docs/100-foundation/RFC-0005-Versioning.md
git commit -m "docs(rfc): viet lai RFC-0005 Versioning"
```

Expected: one commit containing only `docs/100-foundation/RFC-0005-Versioning.md`.

---

## Task 3: Rewrite RFC-0006 Naming Convention

**Files:**
- Modify: `docs/100-foundation/RFC-0006-Naming-Convention.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-foundation-rules-design.md`

- [ ] **Step 1: Replace RFC-0006 content**

Replace the full content of `docs/100-foundation/RFC-0006-Naming-Convention.md` with:

```markdown
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
```

- [ ] **Step 2: Run validator and confirm remaining failures**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero. RFC-0005 and RFC-0006-specific failures are gone; failures remain for RFC-0104 because it is still boilerplate.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0006**

Run:

```bash
grep -n 'RFC-0006 — Naming Convention' SUMMARY.md
```

Expected: output includes `docs/100-foundation/RFC-0006-Naming-Convention.md`.

- [ ] **Step 4: Commit RFC-0006 rewrite**

Run:

```bash
git add docs/100-foundation/RFC-0006-Naming-Convention.md
git commit -m "docs(rfc): viet lai RFC-0006 Naming Convention"
```

Expected: one commit containing only `docs/100-foundation/RFC-0006-Naming-Convention.md`.

---

## Task 4: Rewrite RFC-0104 Dependency Graph and Close Pending Reference

**Files:**
- Modify: `docs/200-resource-model/RFC-0104-Dependency-Graph.md`
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Replace RFC-0104 content**

Replace the full content of `docs/200-resource-model/RFC-0104-Dependency-Graph.md` with:

```markdown
# RFC-0104 — Dependency Graph

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines resource-level dependency graph semantics for ARPS resources.

This RFC owns graph nodes, dependency edges, dependency constraints and graph invariants. Version syntax is defined by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); naming and resource identity are defined by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); canonical resource envelopes are defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); validation diagnostics are defined by [RFC-0106](RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

A dependency graph describes resource relationships before runtime execution. It does not define resolver algorithms, version selection, conflict resolution, lock snapshots or package format.

## 2. Graph Model

A dependency graph is a directed graph:

- A node represents a canonical resource identified by `kind`, `metadata.id` and `metadata.version`.
- An edge points from a resource to another resource it requires.
- The graph is evaluated at resource level, not file path level.
- Dependency graphs MUST be directed and acyclic.

An implementation MAY materialize the graph in memory, in a lock file or in registry metadata, but this RFC defines only the resource-level model.

## 3. Dependency Edge

A dependency edge SHOULD identify the target resource by ID and MAY constrain kind and version range.

Conceptual shape:

```yaml
dependencies:
  - id: prompts/customer-support-summary
    kind: Prompt
    version: ">=1.2.0 <2.0.0"
    optional: false
```

Fields:

| Field | Meaning |
|---|---|
| `id` | Target resource identity using RFC-0006 naming rules. |
| `kind` | Optional target kind constraint. |
| `version` | Optional version range using RFC-0005 range vocabulary. |
| `optional` | Whether the dependency may be absent without invalidating the resource. |

## 4. Dependency Constraints

- Required dependencies MUST identify a target resource ID.
- Required dependencies MAY constrain target kind and version range.
- Optional dependencies SHOULD declare fallback behavior in the owning resource kind specification.
- Dependency declarations MUST NOT depend on repository file paths as identity.
- Version selection and conflict resolution are owned by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).

## 5. Graph Invariants

- Dependency graphs MUST be directed and acyclic.
- A required dependency MUST resolve to one acceptable target before execution or packaging decisions consume the graph.
- A dependency edge MUST NOT silently resolve to a resource with a different `metadata.id`.
- Ambiguous matches MUST NOT be accepted as deterministic resolution.
- Graph traversal SHOULD be deterministic for the same registry inputs and policy configuration.

## 6. Validation Behavior

Validation SHOULD detect missing required dependencies, ambiguous dependencies, invalid version ranges and cycles.

Missing, ambiguous or cyclic dependencies SHOULD produce `DEPENDENCY_ERROR` diagnostics through [RFC-0106](RFC-0106-Validation-Model.md).

Validation may prove that a graph is structurally acceptable. It does not choose the final concrete version when multiple acceptable versions exist; that version selection belongs to [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md). Lock snapshots belong to [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).

## 7. Examples

### 7.1 Prompt dependency

A workflow can depend on a prompt resource by ID and compatible version range:

```yaml
kind: Workflow
metadata:
  id: workflows/support-summary
  version: 1.0.0
spec:
  dependencies:
    - id: prompts/customer-support-summary
      kind: Prompt
      version: ">=1.2.0 <2.0.0"
```

### 7.2 Cycle rejected

If `workflows/a` depends on `workflows/b` and `workflows/b` depends on `workflows/a`, validation reports a `DEPENDENCY_ERROR` because the graph is cyclic.

### 7.3 Missing dependency rejected

If a required dependency points to `prompts/missing-summary` and no registry source provides that resource ID, validation reports a `DEPENDENCY_ERROR`.

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0106 — Validation Model](RFC-0106-Validation-Model.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
```

- [ ] **Step 2: Mark foundation-rules pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0005 Versioning, RFC-0006 Naming, RFC-0104 Dependency Graph — rule con được trỏ tới
```

to:

```markdown
- [x] RFC-0005 Versioning, RFC-0006 Naming, RFC-0104 Dependency Graph — rule con được trỏ tới
```

- [ ] **Step 3: Run full docs validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 4: Confirm SUMMARY still points to RFC-0104**

Run:

```bash
grep -n 'RFC-0104 — Dependency Graph' SUMMARY.md
```

Expected: output includes `docs/200-resource-model/RFC-0104-Dependency-Graph.md`.

- [ ] **Step 5: Verify only intended files changed**

Run:

```bash
git status --short
```

Expected: only `docs/200-resource-model/RFC-0104-Dependency-Graph.md` and `TODO.md` are modified after Task 1, Task 2 and Task 3 commits.

- [ ] **Step 6: Commit RFC-0104 and closure**

Run:

```bash
git add docs/200-resource-model/RFC-0104-Dependency-Graph.md TODO.md
git commit -m "docs(rfc): viet lai RFC-0104 dependency graph"
```

Expected: one commit containing only `docs/200-resource-model/RFC-0104-Dependency-Graph.md` and `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes foundation-rules guardrails for RFC-0005, RFC-0006 and RFC-0104.
- `docs/100-foundation/RFC-0005-Versioning.md` contains `Version Format`, `Versioned Subjects`, `Version Ranges`, `Change Rules`, `Pre-release and Build Metadata`, `Examples` and `References`.
- `docs/100-foundation/RFC-0006-Naming-Convention.md` contains `Naming Principles`, `Identifier Grammar`, `Resource Identity`, `Kinds and Slugs`, `Rename and Alias Rules`, `Examples` and `References`.
- `docs/200-resource-model/RFC-0104-Dependency-Graph.md` contains `Graph Model`, `Dependency Edge`, `Dependency Constraints`, `Graph Invariants`, `Validation Behavior`, `Examples` and `References`.
- The three RFCs no longer contain boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Required Behavior`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Security Considerations`, `Migration Guidance` or `Future Work`.
- RFC-0005 states breaking changes require a major version bump, compatible additive changes should use minor, and patch changes preserve compatibility and meaning.
- RFC-0006 states `metadata.id` uses `namespace/name`, file paths are not identity, and changing `metadata.id` is a breaking identity change unless alias/migration preserves consumers.
- RFC-0104 stays resource-level only, defines directed acyclic dependency graphs, and delegates version selection to RFC-0204 and lock snapshots to RFC-0401.
- `TODO.md` marks the foundation-rules pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references all three RFC paths.
- `docs/300-runtime/RFC-0204-Dependency-Resolver.md`, `docs/500-build-distribution/RFC-0401-Lock-File.md` and `docs/500-build-distribution/RFC-0402-Packaging.md` remain unchanged in this plan.
