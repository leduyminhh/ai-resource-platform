# RFC-0206 — Execution Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines how execution plans are executed without performing dependency resolution.

This RFC owns plan execution. Dependency resolution is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md); the adapter contract by [RFC-0500](../600-sdk/RFC-0500-Adapter-SDK.md); capability and security constraints by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md); the artifact model by [RFC-0207](RFC-0207-Artifact-Model.md); validation by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Execution Inputs

- Execution consumes an `ExecutionPlan` produced by [RFC-0205](RFC-0205-Planning-Engine.md).
- Execution MUST NOT perform dependency resolution; resolution is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md).
- Execution MUST NOT start before required validation has passed ([RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md)).

## 3. Adapter Invocation

- Execution MUST use declared adapters and capabilities, not implicit vendor-specific behavior.
- The adapter contract is owned by [RFC-0500](../600-sdk/RFC-0500-Adapter-SDK.md); capability grants and security limits by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md).

## 4. Execution Artifacts

- Executing a step produces one or more `ExecutionArtifact` outputs.
- The artifact model (identity, immutability, provenance) is owned by [RFC-0207](RFC-0207-Artifact-Model.md).

## 5. Failure Handling

- A failed step MUST stop dependents that require its output.
- Execution MUST surface failures with enough context for observability ([RFC-0209](RFC-0209-Observability.md)).

## 6. Execution Behavior

- Execution MUST be deterministic given the same plan, adapters and inputs.
- Independent steps marked parallel-safe by the plan MAY run concurrently.

## 7. Examples

```yaml
execution:
  plan: build
  step: plugins/backend
  adapter: platform/build-adapter
  result:
    status: success
    artifacts:
      - id: plugins/backend@1.0.0
        checksum: sha256:def456
```

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0204 — Dependency Resolver](RFC-0204-Dependency-Resolver.md)
- [RFC-0205 — Planning Engine](RFC-0205-Planning-Engine.md)
- [RFC-0207 — Artifact Model](RFC-0207-Artifact-Model.md)
- [RFC-0500 — Adapter SDK](../600-sdk/RFC-0500-Adapter-SDK.md)
- [RFC-0800 — Security Model](../900-security-governance/RFC-0800-Security-Model.md)
