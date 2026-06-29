# RFC-0901 Migration Guide Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0901 — Migration Guide` into the normative non-invasive migration playbook for ARPS.

**Architecture:** Keep `RFC-0901` focused on migration phases, gates, safety boundaries and rollback guidance only. Extend `tools/check-rfc-links` with RFC-0901 guardrails first, then rewrite the RFC until the guard passes and close the pending delegation.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add RFC-0901-specific assertions for migration guide ownership.
- Modify: `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md` — replace boilerplate with migration playbook content.
- Modify: `TODO.md` — mark the RFC-0901 pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0901-migration-guide-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: `docs/1000-enterprise-reference/RFC-0900-Enterprise-Reference-Architecture.md`, `docs/900-security-governance/RFC-0802-Governance.md`.

---

## Task 1: Add RFC-0901 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,620p' tools/check-rfc-links
```

Expected: script includes overview checks plus existing guards for RFC-0007, RFC-0100, RFC-0106, RFC-0200 and RFC-0800.

- [ ] **Step 2: Add RFC-0901 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 10: RFC-0901 Migration Guide ownership ---
MIGRATION="$ROOT/docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md"
if [ -f "$MIGRATION" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$MIGRATION" || err "missing header field ${field//\\/} in RFC-0901"
  done

  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Canonical Model' 'Required Behavior' 'Runtime Flow' 'Validation Rules' 'Error Model' 'Security Considerations' 'Compatibility' 'Future Work' 'BUILD_ERROR'; do
    grep -qE "^## ([0-9]+\. )?$bad$|$bad" "$MIGRATION" && err "boilerplate or out-of-scope content '$bad' still in RFC-0901"
  done

  for required in 'Migration Principles' 'Migration Phases' 'Phase Gate Matrix' 'Safety Boundaries' 'Rollback Guidance' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$MIGRATION" || err "missing RFC-0901 section '$required'"
  done

  for phase in 'Inventory' 'Resource Mapping' 'Metadata Overlay' 'Registry Onboarding' 'Validation Gate' 'Dependency Resolution' 'Adapter/Build Rollout' 'Publishing/Adoption'; do
    grep -q "$phase" "$MIGRATION" || err "missing RFC-0901 migration phase '$phase'"
  done

  grep -q 'metadata overlay before restructure\|Metadata overlay before restructure' "$MIGRATION" || err "RFC-0901 must state metadata overlay comes before restructure"
  grep -q 'Execution MUST NOT happen before validation and dependency gates pass' "$MIGRATION" || err "RFC-0901 must gate execution behind validation and dependency checks"
  grep -q 'Publishing MUST NOT happen before validation' "$MIGRATION" || err "RFC-0901 must gate publishing behind validation"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0901, including boilerplate headings and missing sections such as `Migration Principles`, `Phase Gate Matrix`, `Safety Boundaries` and `Rollback Guidance`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0901 migration guide"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0901 Migration Guide

**Files:**
- Modify: `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0901-migration-guide-design.md`

- [ ] **Step 1: Replace RFC-0901 content**

Replace the full content of `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md` with:

```markdown
# RFC-0901 — Migration Guide

**Status:** Draft  
**Category:** 1000-enterprise-reference  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines non-invasive migration guidance for moving existing projects into ARPS.

This RFC owns the migration playbook: phases, gates, safety boundaries and rollback guidance. Enterprise target architecture is defined by [RFC-0900](RFC-0900-Enterprise-Reference-Architecture.md); resource and validation contracts are defined by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md) and [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

Migration in ARPS is non-invasive by default: existing source files remain in place while resources, metadata and registry views are introduced around them.

## 2. Migration Principles

- Discover before changing: inventory existing assets before editing files.
- Metadata overlay before restructure: add or derive metadata without moving source code by default.
- Validate before resolve/build: schema, metadata, semantic, dependency and policy checks gate later phases.
- Adapter rollout before source rewrite: use adapters to integrate existing tools and workflows before rewriting them.
- Reversible steps: early migration steps SHOULD be reversible and reviewable.
- Human review gates: risky actions such as publishing or execution SHOULD require review or policy gates.

## 3. Migration Phases

| Phase | Purpose |
|---|---|
| Inventory | Discover existing docs, prompts, scripts, workflows, configs and packages. |
| Resource Mapping | Map assets to ARPS Resource kinds such as `Skill`, `Prompt`, `Workflow`, `Plugin` or `Adapter`. |
| Metadata Overlay | Add or derive `metadata.id`, `metadata.name`, `metadata.version`, labels and annotations without moving files. |
| Registry Onboarding | Register resources into a local or remote registry view. |
| Validation Gate | Run schema, metadata, semantic, compatibility and policy validation. |
| Dependency Resolution | Build dependency graph and identify missing or ambiguous dependencies. |
| Adapter/Build Rollout | Use adapters and build pipeline only after validation and dependency gates pass. |
| Publishing/Adoption | Package and publish stable resources for registry or marketplace consumers. |

## 4. Phase Gate Matrix

| Phase | Input | Output | Gate | Rollback |
|---|---|---|---|---|
| Inventory | existing repository | asset inventory | inventory reviewed | discard inventory artifact |
| Resource Mapping | asset inventory | resource mapping table | mappings reviewed | revise mapping table |
| Metadata Overlay | mapped assets | resource metadata overlay | metadata validates | remove or revert overlay metadata |
| Registry Onboarding | resources with metadata | registry entries | lookup and index succeed | deregister or mark entries draft |
| Validation Gate | registry/resource set | `ValidationResult` | `valid=true` or accepted warnings | fix metadata, spec or policy |
| Dependency Resolution | validated resources | `DependencyGraph` or resolution set | no missing or ambiguous required dependencies | add dependency metadata or defer resource |
| Adapter/Build Rollout | resolved resources | execution/build plan | adapter capability and security gates pass | disable adapter or revert plan |
| Publishing/Adoption | package bundle | published package or adoption record | integrity, validation and review pass | publish replacement or deprecate bad version |

## 5. Safety Boundaries

- Migration MUST NOT require destructive source restructuring by default.
- Migration MUST NOT store plaintext secrets in resource manifests.
- Publishing MUST NOT happen before validation and required integrity/security checks pass.
- Execution MUST NOT happen before validation and dependency gates pass.
- Existing source-of-truth files SHOULD remain authoritative until migrated resources are reviewed and accepted.
- Registry aliases and deprecations SHOULD be used instead of silently moving resource identities.

## 6. Rollback Guidance

- Inventory artifacts can be discarded without changing source resources.
- Metadata overlays can be removed or reverted if validation fails.
- Registry entries can be deregistered, marked `Draft`, or deprecated depending on registry policy.
- Published package versions SHOULD be immutable; rollback should publish a replacement or deprecate the bad version instead of mutating old artifacts.
- Adapter/build rollout should be reversible by disabling the adapter or reverting execution plans.

## 7. Examples

### 7.1 Prompt library migration

Existing prompt files are inventoried, mapped to `Prompt` resources, given metadata overlays, validated, then registered in a local registry view. The prompt files do not need to move during the initial migration.

### 7.2 Workflow script migration

Existing scripts are mapped to `Workflow` resources. Dependency metadata is added only after inventory review. Adapter rollout starts only after validation and dependency resolution pass.

### 7.3 Stop at validation

A migration stops when validation reports a duplicate `metadata.id` or a plaintext secret in `spec`. The migration resumes after the metadata or secret handling is fixed.

## References

- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](../200-resource-model/RFC-0102-Metadata-Model.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
- [RFC-0400 — Build Pipeline](../500-build-distribution/RFC-0400-Build-Pipeline.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
- [RFC-0800 — Security Model](../900-security-governance/RFC-0800-Security-Model.md)
- [RFC-0900 — Enterprise Reference Architecture](RFC-0900-Enterprise-Reference-Architecture.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0901**

Run:

```bash
grep -n 'RFC-0901 — Migration Guide' SUMMARY.md
```

Expected: output includes `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md
git commit -m "docs(rfc): viet lai RFC-0901 Migration Guide"
```

Expected: one commit containing only `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0901 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0901 Migration Guide — nhà của Migration Guidance
```

to:

```markdown
- [x] RFC-0901 Migration Guide — nhà của Migration Guidance
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
git commit -m "docs: dong pending RFC-0901 migration guide"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0901-specific guardrails.
- `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md` contains `Migration Principles`, `Migration Phases`, `Phase Gate Matrix`, `Safety Boundaries`, `Rollback Guidance`, `Examples` and `References`.
- `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md` no longer contains boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Required Behavior`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Security Considerations`, `Compatibility` or `Future Work`.
- RFC-0901 states metadata overlay before restructure.
- RFC-0901 gates execution behind validation and dependency checks.
- RFC-0901 gates publishing behind validation and integrity/security checks.
- `TODO.md` marks the RFC-0901 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`.
- Related RFCs such as `RFC-0900` and `RFC-0802` remain unchanged in this plan.