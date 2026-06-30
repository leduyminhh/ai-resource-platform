# RFC Runtime Read-Pipeline Design

**Date:** 2026-06-29
**Status:** Approved for implementation planning
**Scope:** `RFC-0201 — Discovery Engine`, `RFC-0202 — Registry Engine`, `RFC-0203 — Validation Engine`, `RFC-0204 — Dependency Resolver`

---

## 1. Context

The `300-runtime` cluster framework `RFC-0200` (Runtime Architecture) is REAL and defines eight engines plus
their handoff artifacts (`ResourceCandidate`, `CanonicalResourceSet`, `ValidationResult`, `DependencyGraph`,
`ExecutionPlan`, `ExecutionArtifact`, `PackageBundle`, `PublishResult`). `RFC-0200` forward-references all of
`RFC-0201..0209`, which are still boilerplate.

This cluster completes the **read/resolve half** of the runtime pipeline — the four engines that turn source
resources into a resolved dependency graph: Discovery → Registry → Validation → Dependency Resolver. The
build/execute half (`RFC-0205..0209`: Planning, Execution, Artifact, Caching, Observability) is a separate
later cycle. The seam is the `DependencyGraph → ExecutionPlan` handoff.

## 2. Design Goal

Rewrite the four read-pipeline engine RFCs, each owning its engine's behavior and delegating cross-cutting
contracts instead of restating them. Each engine consumes/produces the handoff artifacts named by `RFC-0200`,
using those exact names for consistency. Remove boilerplate. Do NOT redefine the runtime pipeline (`RFC-0200`),
the canonical model (`RFC-0100`), the validation model (`RFC-0106`), or the dependency graph structure (`RFC-0104`).

## 3. Approved Approach

Use the **per-RFC ownership split + validator guard + delegation** pattern (as used by foundation-rules and
resource-model-completion).

### 3.1 RFC-0201 — Discovery Engine ownership
Owns: how resources are discovered from filesystem, package and registry sources; the `ResourceCandidate`
concept; source precedence and de-duplication.
Required sections: `Discovery Sources`, `Resource Candidate`, `Source Precedence`, `Discovery Behavior`, `Examples`, `References`.
Required rules:
- Discovery reads from filesystem, package and registry sources and emits `ResourceCandidate` entries.
- Discovery MUST NOT mutate source resources (read-only phase).
- Discovery MUST produce deterministic ordering for identical inputs.
- Canonical parsing/envelope is owned by [RFC-0100]; storage/lookup by [RFC-0202]; validation by [RFC-0203].

### 3.2 RFC-0202 — Registry Engine ownership
Owns: the local **runtime** registry that stores discovered resource metadata and supports lookup by id,
kind, version, label and checksum; holds the `CanonicalResourceSet`.
Required sections: `Registry Purpose`, `Stored Metadata`, `Lookup API`, `Registry vs Distribution`, `Registry Behavior`, `Examples`, `References`.
Required rules:
- The runtime registry is an in-process store of discovered resources, distinct from the distribution
  registry/marketplace (owned by [RFC-0403]).
- Lookup MUST support id, kind, version, label and checksum.
- `metadata.id` MUST be unique within the registry.
- Metadata field semantics are owned by [RFC-0102]; naming by [RFC-0006].

### 3.3 RFC-0203 — Validation Engine ownership
Owns: validation **execution** strategy — layer ordering, short-circuiting, parallelism, caching and
incremental validation; produces `ValidationResult`.
Required sections: `Engine Responsibility`, `Execution Strategy`, `Layer Ordering`, `Caching and Incremental`, `Result Handoff`, `Examples`, `References`.
Required rules:
- The engine executes the validation model defined by [RFC-0106]; it MUST NOT redefine validation layers,
  the result envelope, or error codes.
- The engine owns execution order, short-circuiting and parallelism.
- The engine produces a `ValidationResult` per the [RFC-0106] envelope.
- Validation MUST NOT mutate source resources.

### 3.4 RFC-0204 — Dependency Resolver ownership
Owns: dependency resolution algorithm, version selection, conflict handling and lock generation; produces a
resolved `DependencyGraph`.
Required sections: `Resolution Inputs`, `Version Selection`, `Conflict Handling`, `Lock Generation`, `Resolver Behavior`, `Examples`, `References`.
Required rules:
- The resolver consumes the dependency graph structure and invariants defined by [RFC-0104] and selects
  concrete versions; it MUST NOT redefine graph structure or acyclicity rules.
- Version range grammar is owned by [RFC-0005]; lock file format by [RFC-0401].
- Unresolvable, conflicting or cyclic dependencies MUST surface as `DEPENDENCY_ERROR` diagnostics (via [RFC-0106]).
- Resolution MUST be deterministic for identical inputs and registry state.

## 4. Out of Scope
- Runtime pipeline orchestration and engine boundaries; owned by `RFC-0200`.
- Canonical envelope; owned by `RFC-0100`.
- Validation layers/result/error codes (the model); owned by `RFC-0106`.
- Dependency graph structure/invariants; owned by `RFC-0104`.
- Planning, execution, artifacts, caching, observability; owned by `RFC-0205..0209` (next cycle).
- Distribution registry/marketplace and lock-file format mechanics; owned by `RFC-0403`/`RFC-0401`.
- Packaging/publishing; owned by `RFC-0402`/`RFC-0403`.

## 5. Validator Design
Extend `tools/check-rfc-links` with a runtime read-pipeline guard (Check 13) after Check 12, reusing the
`check_foundation_header` and `check_no_foundation_boilerplate` helpers. For each RFC verify header fields,
absence of boilerplate headings, the required section set above, and ownership-proving key phrases:
- `RFC-0201`: `ResourceCandidate` and `MUST NOT mutate`.
- `RFC-0202`: `CanonicalResourceSet`, `RFC-0403`, and lookup keys `id`/`kind`/`version`/`label`/`checksum`.
- `RFC-0203`: `ValidationResult` and `RFC-0106`.
- `RFC-0204`: `DependencyGraph`, `RFC-0104`, `RFC-0401`, and `DEPENDENCY_ERROR`.
Use exact-heading checks where possible to avoid false positives in prose/references.

## 6. Implementation Plan Shape
Five commits, preserving fail-first evidence and keeping each RFC reviewable:
1. Add the read-pipeline validator guard (Check 13) and verify it fails against current boilerplate.
2. Rewrite `RFC-0201 — Discovery Engine`; verify guard fails only for remaining RFCs.
3. Rewrite `RFC-0202 — Registry Engine`.
4. Rewrite `RFC-0203 — Validation Engine`.
5. Rewrite `RFC-0204 — Dependency Resolver`; run full `tools/check-rfc-links`; close any covering `TODO.md`
   pending-delegation entry if present.

## 7. Acceptance Criteria
- `RFC-0201`, `RFC-0202`, `RFC-0203`, `RFC-0204` no longer contain boilerplate ownership sections.
- Each RFC has its required sections and references owner RFCs instead of duplicating their scope.
- `RFC-0202` distinguishes the runtime registry from the distribution registry (`RFC-0403`); `RFC-0203`
  does not redefine the `RFC-0106` model; `RFC-0204` does not redefine the `RFC-0104` graph.
- Each engine references its `RFC-0200` handoff artifact by the exact name.
- `tools/check-rfc-links` passes (exit 0); `SUMMARY.md` links unchanged.
- The runtime read/resolve pipeline (`RFC-0201..0204`) is REAL.

## 8. Recommendation
Proceed with this cluster as the next implementation cycle. It completes the read/resolve half of the runtime
pipeline anchored by the already-REAL `RFC-0200`, and sets up the build/execute half (`RFC-0205..0209`) for a
following cycle at the natural `DependencyGraph → ExecutionPlan` seam.
