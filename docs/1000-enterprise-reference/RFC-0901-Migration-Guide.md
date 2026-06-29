# RFC-0901 — Migration Guide

**Status:** Draft  
**Category:** 1000-enterprise-reference  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## 1. Abstract

This RFC defines the non-invasive migration playbook for adopting ARPS in existing repositories. It describes migration principles, migration phases, phase gates, safety boundaries and rollback guidance.

RFC-0901 does not redefine the canonical resource model, runtime architecture, validation model, security model or compatibility policy. Those topics remain owned by their dedicated RFCs.

## 2. Migration Principles

- Non-invasive first: existing business source code SHOULD remain in place until a migration gate explicitly approves a restructure.
- Migration SHOULD perform metadata overlay before restructure: add or derive ARPS metadata before moving files or rewriting source layouts.
- Incremental adoption: teams SHOULD migrate one resource family, repository area or package boundary at a time.
- Validation-led progress: each phase SHOULD produce artifacts that can be validated before later phases consume them.
- Reversible rollout: migration actions SHOULD have a clear rollback path before they are executed.
- Human review gates: risky actions such as execution rollout, publishing or marketplace adoption SHOULD require review or policy approval.

## 3. Migration Phases

| Phase | Purpose |
| --- | --- |
| Inventory | Discover existing prompts, workflows, skills, adapters, policies, packages and documentation without changing them. |
| Mapping | Map discovered assets to ARPS resource kinds, owners, lifecycle status and registry namespaces. |
| Metadata Overlay | Add or derive `metadata.id`, `metadata.name`, `metadata.version`, labels and annotations without moving files. |
| Validation | Run schema, metadata, dependency and policy checks against candidate resources. |
| Dependency Resolution | Resolve resource dependencies and confirm the graph can be planned safely. |
| Adapter/Build Rollout | Introduce adapters, build steps or execution plans only after validation and dependency gates pass. |
| Publishing/Adoption | Package and publish stable resources for registry or marketplace consumers. |

## 4. Phase Gate Matrix

| Phase | Entry Criteria | Output | Exit Gate | Rollback |
| --- | --- | --- | --- | --- |
| Inventory | repository access | asset inventory | inventory reviewed | discard inventory |
| Mapping | reviewed inventory | resource mapping table | owner and kind accepted | revert mapping notes |
| Metadata Overlay | mapped assets | resource metadata overlay | metadata validates | remove or revert overlay metadata |
| Validation | candidate resources | validation report | schema, metadata, policy and secret checks pass | fix or remove candidate resources |
| Dependency Resolution | validated resources | dependency graph | graph is acyclic and required dependencies resolve | remove dependency metadata |
| Adapter/Build Rollout | resolved graph | adapter or build plan | validation + dependency gates pass | disable adapter or revert build plan |
| Publishing/Adoption | package bundle | published package or adoption record | validation + integrity/security publishing gates pass | publish replacement or deprecate bad version |

## 5. Safety Boundaries

- Migration MUST NOT require moving or rewriting existing business source code during inventory, mapping or metadata overlay phases.
- Execution MUST NOT begin until validation + dependency gates pass.
- Publishing MUST NOT happen until validation + integrity/security publishing gates pass.
- Secrets MUST NOT be copied into resource manifests or generated migration overlays.
- Generated migration artifacts SHOULD be clearly separated from authoritative source files unless the owner accepts them.
- Existing source-of-truth files SHOULD remain authoritative until migrated resources are reviewed and accepted.
- Registry aliases and deprecations SHOULD be used instead of silently moving resource identities.

## 6. Rollback Guidance

- Inventory artifacts can be discarded without changing source resources.
- Mapping notes can be reverted if the selected kind, namespace or owner is wrong.
- Metadata overlays can be removed or reverted if validation fails.
- Registry entries can be deregistered, marked `Draft`, or deprecated depending on registry policy.
- Published package versions SHOULD be immutable; rollback should publish a replacement or deprecate the bad version instead of mutating old artifacts.
- Adapter and build rollout should be reversible by disabling the adapter or reverting execution plans.

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
