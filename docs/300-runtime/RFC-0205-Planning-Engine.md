# RFC-0205 — Planning Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines conversion from a resolved graph into a deterministic execution plan optimized for cache reuse and safe parallelism.

This RFC owns plan construction and the `ExecutionPlan`. The resolved dependency graph is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md) and [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md); cache-reuse strategy by [RFC-0208](RFC-0208-Caching-Strategy.md); execution by [RFC-0206](RFC-0206-Execution-Engine.md); pipeline orchestration by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Planning Inputs

- Planning consumes the resolved dependency graph produced by [RFC-0204](RFC-0204-Dependency-Resolver.md).
- Planning MAY consume user intent (targets, goals) selecting which resources to build.
- Planning MUST NOT mutate source resource files.

## 3. Execution Plan

- Planning produces an `ExecutionPlan`: an ordered set of steps to execute.
- Each step names the resource, the action and its declared dependencies within the plan.
- The `ExecutionPlan` is handed to the execution engine ([RFC-0206](RFC-0206-Execution-Engine.md)).

## 4. Plan Determinism

- The `ExecutionPlan` MUST be deterministic: identical resolved inputs and intent produce an identical plan.
- Step ordering MUST be stable and derived only from the graph and declared inputs.

## 5. Parallelism and Cache Reuse

- The plan MAY mark independent steps as safe to run in parallel.
- The plan SHOULD expose cache keys so unchanged steps can be skipped; cache-key semantics are owned by [RFC-0208](RFC-0208-Caching-Strategy.md).

## 6. Planning Behavior

- Planning is read-only with respect to source resources.
- Planning MUST NOT perform dependency resolution; that is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md).

## 7. Examples

```yaml
plan:
  steps:
    - id: core/clean-code
      action: build
      dependsOn: []
      parallelSafe: true
    - id: plugins/backend
      action: build
      dependsOn: [core/clean-code]
```

## References

- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0204 — Dependency Resolver](RFC-0204-Dependency-Resolver.md)
- [RFC-0206 — Execution Engine](RFC-0206-Execution-Engine.md)
- [RFC-0208 — Caching Strategy](RFC-0208-Caching-Strategy.md)
