# RFC Runtime Build-Pipeline Design

**Date:** 2026-06-29
**Status:** Approved for implementation planning
**Scope:** `RFC-0205 — Planning Engine`, `RFC-0206 — Execution Engine`, `RFC-0207 — Artifact Model`, `RFC-0208 — Caching Strategy`, `RFC-0209 — Observability`

---

## 1. Context

The read/resolve half of `300-runtime` (`RFC-0200..0204`) is REAL. This cluster completes the
**build/execute half**: Planning → Execution plus the cross-cutting concerns Artifact, Caching and
Observability. `RFC-0200` forward-references all five and defines their handoff artifacts:
Planning produces `ExecutionPlan`, Execution produces `ExecutionArtifact[]`. (The Packaging and
Publishing engines produce `PackageBundle`/`PublishResult` and are owned by the build-distribution
cluster `RFC-0402`/`RFC-0403`, not by these `020x` RFCs.) `RFC-0107` already forward-references
`RFC-0207`, and `RFC-0800` references `RFC-0206`.

Completing these five makes cluster `300-runtime` fully REAL (`RFC-0200..0209`).

## 2. Design Goal

Rewrite the five RFCs, each owning its engine/concern and delegating cross-cutting contracts
instead of restating them. Use the exact `RFC-0200` handoff artifact names. Remove boilerplate. Do
NOT redefine the runtime pipeline (`RFC-0200`), dependency resolution (`RFC-0204`), the validation
model (`RFC-0106`), the adapter contract (`RFC-0500`), or the package model (`RFC-0107`).

## 3. Approved Approach

Use the **per-RFC ownership split + validator guard + delegation** pattern.

### 3.1 RFC-0205 — Planning Engine ownership
Owns: conversion of the resolved dependency set into a deterministic `ExecutionPlan` optimized for
cache reuse and safe parallelism.
Required sections: `Planning Inputs`, `Execution Plan`, `Plan Determinism`, `Parallelism and Cache Reuse`, `Planning Behavior`, `Examples`, `References`.
Required rules:
- Planning consumes the resolved dependency graph from [RFC-0204] and produces an `ExecutionPlan`.
- The `ExecutionPlan` MUST be deterministic for identical inputs.
- Planning MUST NOT mutate source resource files.
- Cache-reuse strategy is owned by [RFC-0208]; execution by [RFC-0206].

### 3.2 RFC-0206 — Execution Engine ownership
Owns: executing an `ExecutionPlan` by invoking declared adapters and capabilities, producing
`ExecutionArtifact[]`.
Required sections: `Execution Inputs`, `Adapter Invocation`, `Execution Artifacts`, `Failure Handling`, `Execution Behavior`, `Examples`, `References`.
Required rules:
- Execution consumes an `ExecutionPlan` and MUST NOT perform dependency resolution (owned by [RFC-0204]).
- Execution MUST NOT start before required validation has passed ([RFC-0106]/[RFC-0200]).
- Execution MUST use declared adapters and capabilities; the adapter contract is owned by [RFC-0500]
  and capability/security constraints by [RFC-0800].
- Execution produces `ExecutionArtifact` outputs whose model is defined by [RFC-0207].

### 3.3 RFC-0207 — Artifact Model ownership
Owns: the immutable `ExecutionArtifact` model — identity, immutability and provenance.
Required sections: `Artifact Concept`, `Artifact Identity`, `Immutability`, `Provenance`, `Examples`, `References`.
Required rules:
- An `ExecutionArtifact` is immutable once produced.
- An artifact is identified by content checksum; signing/integrity mechanics are owned by [RFC-0801].
- Provenance links an artifact to the plan and inputs that produced it.
- Package bundling of artifacts is owned by [RFC-0107]/[RFC-0402].

### 3.4 RFC-0208 — Caching Strategy ownership
Owns: cache keys, invalidation and reproducible-build behavior.
Required sections: `Cache Keys`, `Invalidation`, `Reproducible Builds`, `Caching Behavior`, `Examples`, `References`.
Required rules:
- A cache key MUST be derived from the content and context that determine an output.
- A cache entry MUST be invalidated when any input contributing to its key changes.
- Reproducible builds: identical inputs MUST yield identical outputs; serialization determinism is
  owned by [RFC-0101] and the deterministic-builds principle by [RFC-0002].

### 3.5 RFC-0209 — Observability ownership
Owns: logs, metrics, traces, reports and diagnostic outputs emitted by runtime engines.
Required sections: `Signals`, `Logs Metrics Traces`, `Reports`, `Diagnostics vs Validation`, `Observability Behavior`, `Examples`, `References`.
Required rules:
- Engines emit logs, metrics, traces and reports as runtime telemetry.
- Observability telemetry is distinct from the validation result envelope owned by [RFC-0106]; this
  RFC MUST NOT redefine validation diagnostics.
- Telemetry MUST NOT contain plaintext secrets (secret handling owned by [RFC-0800]).

## 4. Out of Scope
- Runtime pipeline orchestration; owned by `RFC-0200`.
- Dependency resolution/version selection/lock; owned by `RFC-0204`/`RFC-0401`.
- Validation model (layers/result/codes); owned by `RFC-0106`.
- Adapter contract; owned by `RFC-0500`. Security/capabilities; owned by `RFC-0800`.
- Package model and packaging/publishing mechanics; owned by `RFC-0107`/`RFC-0402`/`RFC-0403`.

## 5. Validator Design
Extend `tools/check-rfc-links` with a runtime build-pipeline guard (Check 14) after Check 13,
reusing `check_foundation_header` and `check_no_foundation_boilerplate`. For each RFC verify header
fields, absence of boilerplate headings, the required section set, and ownership-proving key phrases:
- `RFC-0205`: `ExecutionPlan`, `MUST NOT mutate`, `RFC-0204`.
- `RFC-0206`: `ExecutionArtifact`, `RFC-0204`, `RFC-0500`.
- `RFC-0207`: `ExecutionArtifact`, `immutable`, `RFC-0107`.
- `RFC-0208`: `reproducible`, `invalidation`, `RFC-0101`.
- `RFC-0209`: `RFC-0106`, `logs`, `metrics`, `traces`.
Use exact-heading checks where possible to avoid false positives.

## 6. Implementation Plan Shape
Six commits, preserving fail-first evidence and keeping each RFC reviewable:
1. Add the build-pipeline validator guard (Check 14) and verify it fails against current boilerplate.
2. Rewrite `RFC-0205 — Planning Engine`; verify guard fails only for remaining RFCs.
3. Rewrite `RFC-0206 — Execution Engine`.
4. Rewrite `RFC-0207 — Artifact Model`.
5. Rewrite `RFC-0208 — Caching Strategy`.
6. Rewrite `RFC-0209 — Observability`; run full `tools/check-rfc-links`; close any covering `TODO.md`
   pending-delegation entry if present.

## 7. Acceptance Criteria
- `RFC-0205..0209` no longer contain boilerplate ownership sections.
- Each RFC has its required sections and references owner RFCs instead of duplicating their scope.
- `RFC-0206` does not perform/redefine resolution (`RFC-0204`) and uses declared adapters (`RFC-0500`);
  `RFC-0207` stays the artifact model (bundling → `RFC-0107`); `RFC-0209` telemetry is distinct from
  the `RFC-0106` validation result.
- Planning/Execution reference their `RFC-0200` handoff artifacts (`ExecutionPlan`, `ExecutionArtifact`) by exact name.
- `tools/check-rfc-links` passes (exit 0); `SUMMARY.md` links unchanged.
- Cluster `300-runtime` is fully REAL (`RFC-0200..0209`).

## 8. Recommendation
Proceed with this cluster as the next implementation cycle. It completes the runtime layer end-to-end
(read/resolve + build/execute), leaving the build-distribution, SDK, CLI, IDE, plugin-system and
governance clusters for later cycles.
