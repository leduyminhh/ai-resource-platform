# RFC-0200 Runtime Architecture Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0200 — Runtime Architecture` into the normative runtime pipeline and engine boundary contract for ARPS.

**Architecture:** Keep `RFC-0200` focused on logical runtime flow, engine boundaries and handoff artifacts only. Extend `tools/check-rfc-links` with RFC-0200 guardrails first, then rewrite the RFC until the guard passes and close the pending delegation.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add RFC-0200-specific assertions for runtime architecture ownership.
- Modify: `docs/300-runtime/RFC-0200-Runtime-Architecture.md` — replace boilerplate with runtime architecture content.
- Modify: `TODO.md` — mark the RFC-0200 pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0200-runtime-architecture-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: engine-specific RFCs `docs/300-runtime/RFC-0201-*.md` through `docs/300-runtime/RFC-0209-*.md`.

---

## Task 1: Add RFC-0200 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/300-runtime/RFC-0200-Runtime-Architecture.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,420p' tools/check-rfc-links
```

Expected: script includes overview checks plus existing guards for RFC-0007, RFC-0100 and RFC-0106.

- [ ] **Step 2: Add RFC-0200 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 8: RFC-0200 Runtime Architecture ownership ---
RUNTIME="$ROOT/docs/300-runtime/RFC-0200-Runtime-Architecture.md"
if [ -f "$RUNTIME" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$RUNTIME" || err "missing header field ${field//\\/} in RFC-0200"
  done

  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Canonical Model' 'Validation Rules' 'Error Model' 'Security Considerations' 'Compatibility' 'Migration Guidance' 'Future Work'; do
    grep -qE "^## ([0-9]+\. )?$bad$" "$RUNTIME" && err "boilerplate heading '$bad' still in RFC-0200"
  done

  for required in 'Runtime Pipeline' 'Engine Boundary Matrix' 'Handoff Artifacts' 'Runtime Invariants' 'Failure Boundaries' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$RUNTIME" || err "missing RFC-0200 section '$required'"
  done

  for engine in 'Discovery Engine' 'Registry Engine' 'Validation Engine' 'Dependency Resolver' 'Planning Engine' 'Execution Engine' 'Packaging Engine' 'Publishing Engine'; do
    grep -q "$engine" "$RUNTIME" || err "missing RFC-0200 engine '$engine'"
  done

  for artifact in 'ResourceCandidate' 'CanonicalResourceSet' 'ValidationResult' 'DependencyGraph' 'ExecutionPlan' 'ExecutionArtifact' 'PackageBundle' 'PublishResult'; do
    grep -q "$artifact" "$RUNTIME" || err "missing RFC-0200 handoff artifact '$artifact'"
  done

  grep -q 'Execution MUST NOT start before required validation has passed' "$RUNTIME" || err "RFC-0200 must state execution does not start before validation passes"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0200, including boilerplate headings and missing sections such as `Engine Boundary Matrix` or `Handoff Artifacts`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0200 runtime architecture"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0200 Runtime Architecture

**Files:**
- Modify: `docs/300-runtime/RFC-0200-Runtime-Architecture.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0200-runtime-architecture-design.md`

- [ ] **Step 1: Replace RFC-0200 content**

Replace the full content of `docs/300-runtime/RFC-0200-Runtime-Architecture.md` with:

```markdown
# RFC-0200 — Runtime Architecture

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the ARPS runtime pipeline, engine boundaries and handoff artifacts.

This RFC owns the logical runtime flow. Engine-specific behavior is defined by [RFC-0201](RFC-0201-Discovery-Engine.md) through [RFC-0209](RFC-0209-Observability.md). Resource envelope semantics are defined by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md); validation result semantics are defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

The pipeline in this RFC defines logical handoff order. Implementations MAY run steps in one process, multiple processes or optimized execution modes when the same handoff contracts and runtime invariants are preserved.

## 2. Runtime Pipeline

The ARPS runtime pipeline is:

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

Each engine consumes a defined handoff artifact and produces the next artifact in the flow. Engine-specific RFCs MAY add details, but MUST preserve the boundaries defined here.

## 3. Engine Boundary Matrix

| Engine | Input | Output | Owns |
|---|---|---|---|
| Discovery Engine | repository, filesystem or package source | `ResourceCandidate[]` | locating candidate resource files and manifests |
| Registry Engine | `ResourceCandidate[]` | `CanonicalResourceSet` and registry index | canonical indexing, identity lookup and registry view |
| Validation Engine | `CanonicalResourceSet` plus schema and policy context | `ValidationResult` | validation execution using [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md) |
| Dependency Resolver | validated resources plus registry index | `DependencyGraph` and resolution set | dependency graph resolution and ordering |
| Planning Engine | resolution set plus user intent | `ExecutionPlan` | deterministic plan construction |
| Execution Engine | `ExecutionPlan` plus adapters and tools | `ExecutionArtifact[]` and execution result | invoking declared adapters and capabilities |
| Packaging Engine | artifacts, resources and metadata | `PackageBundle` | producing distributable package bundle |
| Publishing Engine | `PackageBundle` plus registry target | `PublishResult` | publishing, registry update and marketplace handoff |

## 4. Handoff Artifacts

- `ResourceCandidate`: discovered source path plus minimal parse or discovery metadata.
- `CanonicalResourceSet`: normalized set of resources keyed by registry identity.
- `ValidationResult`: validation output defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- `DependencyGraph`: resource dependency nodes and edges defined by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md).
- `ExecutionPlan`: deterministic ordered plan produced before execution.
- `ExecutionArtifact`: result generated by execution adapters or tools.
- `PackageBundle`: package output defined by packaging and build RFCs.
- `PublishResult`: result of publishing to a registry or marketplace target.

## 5. Runtime Invariants

- Execution MUST NOT start before required validation has passed.
- Dependency resolution MUST use validated resources.
- Planning MUST NOT mutate source resource files.
- Execution MUST use declared adapters and capabilities, not implicit vendor-specific behavior.
- Packaging MUST use validated resource metadata and produced artifacts.
- Publishing MUST operate on a package bundle, not raw unchecked resources.
- Given identical inputs and context, handoff artifacts SHOULD be deterministic.

## 6. Failure Boundaries

- Discovery failures are owned by [RFC-0201](RFC-0201-Discovery-Engine.md).
- Validation failures use `ValidationResult` from [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- Compatibility conflicts use classification from [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md) and diagnostics from [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- Dependency failures are owned by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md) and [RFC-0204](RFC-0204-Dependency-Resolver.md).
- Planning, execution, packaging and publishing failures are owned by their engine RFCs and build/distribution RFCs.

## 7. Examples

### 7.1 Happy path

```text
Repository
  -> ResourceCandidate[]
  -> CanonicalResourceSet
  -> ValidationResult(valid=true)
  -> DependencyGraph
  -> ExecutionPlan
  -> ExecutionArtifact[]
  -> PackageBundle
  -> PublishResult
```

### 7.2 Stop at validation

```text
CanonicalResourceSet
  -> ValidationResult(valid=false)
  -> stop before Dependency Resolver
```

When validation fails, dependency resolution, planning, execution, packaging and publishing MUST NOT continue for the invalid resource set.

### 7.3 Stop at dependency resolution

```text
ValidationResult(valid=true)
  -> Dependency Resolver reports missing dependency
  -> stop before Planning Engine
```

When dependency resolution fails, planning and execution MUST NOT run for the unresolved graph.

## References

- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0107 — Packaging Model](../200-resource-model/RFC-0107-Packaging-Model.md)
- [RFC-0201 — Discovery Engine](RFC-0201-Discovery-Engine.md)
- [RFC-0202 — Registry Engine](RFC-0202-Registry-Engine.md)
- [RFC-0203 — Validation Engine](RFC-0203-Validation-Engine.md)
- [RFC-0204 — Dependency Resolver](RFC-0204-Dependency-Resolver.md)
- [RFC-0205 — Planning Engine](RFC-0205-Planning-Engine.md)
- [RFC-0206 — Execution Engine](RFC-0206-Execution-Engine.md)
- [RFC-0207 — Artifact Model](RFC-0207-Artifact-Model.md)
- [RFC-0208 — Caching Strategy](RFC-0208-Caching-Strategy.md)
- [RFC-0209 — Observability](RFC-0209-Observability.md)
- [RFC-0400 — Build Pipeline](../500-build-distribution/RFC-0400-Build-Pipeline.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0200**

Run:

```bash
grep -n 'RFC-0200 — Runtime Architecture' SUMMARY.md
```

Expected: output includes `docs/300-runtime/RFC-0200-Runtime-Architecture.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/300-runtime/RFC-0200-Runtime-Architecture.md
git commit -m "docs(rfc): viet lai RFC-0200 Runtime Architecture"
```

Expected: one commit containing only `docs/300-runtime/RFC-0200-Runtime-Architecture.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0200 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0200 Runtime Architecture — nhà của Runtime Flow
```

to:

```markdown
- [x] RFC-0200 Runtime Architecture — nhà của Runtime Flow
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
git commit -m "docs: dong pending RFC-0200 runtime architecture"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0200-specific guardrails.
- `docs/300-runtime/RFC-0200-Runtime-Architecture.md` contains `Runtime Pipeline`, `Engine Boundary Matrix`, `Handoff Artifacts`, `Runtime Invariants`, `Failure Boundaries`, `Examples` and `References`.
- `docs/300-runtime/RFC-0200-Runtime-Architecture.md` no longer contains boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Validation Rules`, `Error Model`, `Security Considerations`, `Compatibility`, `Migration Guidance` or `Future Work`.
- RFC-0200 states all engine names and all handoff artifact names.
- RFC-0200 states execution does not start before required validation has passed.
- `TODO.md` marks the RFC-0200 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/300-runtime/RFC-0200-Runtime-Architecture.md`.
- Engine-specific RFCs `RFC-0201..RFC-0209` remain unchanged in this plan.