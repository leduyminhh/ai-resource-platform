# Design — RFC-0901 Migration Guide

- **Ngày:** 2026-06-29
- **Phạm vi:** Phase A (hoàn thiện đặc tả), sub-project — `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md`
- **Approach:** B — Phase-based playbook + gates
- **Trạng thái:** Draft (chờ review)

## Bối cảnh

`RFC-0901` được `TODO.md` xem là nguồn sở hữu Migration Guidance. Hiện RFC này vẫn còn boilerplate chung §2–§14, trong đó Migration Guidance chỉ gồm chuỗi ngắn: discover existing assets, add metadata, register resources, resolve dependencies, build through adapters.

Sau khi đã hoàn thiện các nền tảng `RFC-0100` (resource envelope), `RFC-0106` (validation model), `RFC-0200` (runtime flow), `RFC-0800` (security model) và `RFC-0007` (compatibility policy), `RFC-0901` có đủ nền để trở thành playbook migration không xâm lấn cho existing projects.

## Mục Tiêu

- Định nghĩa migration playbook không xâm lấn từ existing projects vào ARPS.
- Chốt các phase migration, input/output và gate để đi tiếp.
- Chốt safety boundaries: không restructure phá hủy, không capture secrets, không publish/execute trước validation.
- Chốt rollback guidance cho metadata overlay, registry onboarding và package publishing.
- Loại bỏ boilerplate không thuộc trách nhiệm của `RFC-0901`.

## Không Làm

- Không định nghĩa enterprise target architecture; phần đó thuộc `RFC-0900`.
- Không định nghĩa governance approval workflow; phần đó thuộc `RFC-0802`.
- Không định nghĩa runtime/build internals; phần đó thuộc `RFC-0200` và `RFC-0400`.
- Không định nghĩa schema/resource model chi tiết; phần đó thuộc `RFC-0100..0107`.
- Không định nghĩa registry protocol chi tiết; phần đó thuộc `RFC-0403`.

## Thiết Kế Đề Xuất

`RFC-0901` sẽ được viết lại thành migration guide/playbook sở hữu migration phases, gates, safety boundaries và rollback guidance, giữ header chuẩn đã chốt ở ADR-0003.

### 1. Abstract

Tóm tắt rằng RFC này định nghĩa non-invasive migration guide từ existing projects vào ARPS.

### 2. Conventions

Nêu quy ước RFC 2119 và ownership boundary:
- Resource envelope/metadata: `RFC-0100`, `RFC-0102`.
- Dependency graph: `RFC-0104`.
- Validation gates: `RFC-0106`.
- Runtime flow: `RFC-0200`.
- Build/package/registry: `RFC-0400`, `RFC-0402`, `RFC-0403`.
- Security boundaries: `RFC-0800`.
- Enterprise reference architecture: `RFC-0900`.

### 3. Migration Principles

- **Discover before changing:** inventory existing assets before editing files.
- **Metadata overlay before restructure:** add/derive metadata without moving source code by default.
- **Validate before resolve/build:** schema, metadata, semantic, dependency and policy checks gate later phases.
- **Adapter rollout before source rewrite:** use adapters to integrate existing tools/workflows before rewriting them.
- **Reversible steps:** early migration steps should be reversible and reviewable.
- **Human review gates:** risky actions such as publishing or execution require review/policy gates.

### 4. Migration Phases

| Phase | Purpose |
|---|---|
| Inventory | Discover existing docs, prompts, scripts, workflows, configs and packages. |
| Resource Mapping | Map assets to ARPS Resource kinds such as `Skill`, `Prompt`, `Workflow`, `Plugin`, `Adapter`. |
| Metadata Overlay | Add or derive `metadata.id`, `metadata.name`, `metadata.version`, labels and annotations without moving files. |
| Registry Onboarding | Register resources into a local or remote registry view. |
| Validation Gate | Run schema, metadata, semantic, compatibility and policy validation. |
| Dependency Resolution | Build dependency graph and identify missing/ambiguous dependencies. |
| Adapter/Build Rollout | Use adapters/build pipeline only after validation and dependency gates pass. |
| Publishing/Adoption | Package and publish stable resources; adopt through registry/marketplace consumers. |

### 5. Phase Gate Matrix

Mỗi phase nên có:
- input
- output
- proceed gate
- rollback action

Ví dụ:

| Phase | Input | Output | Gate | Rollback |
|---|---|---|---|---|
| Inventory | existing repo | asset inventory | inventory reviewed | discard inventory artifact |
| Metadata Overlay | mapped assets | metadata overlay | metadata validates | remove overlay metadata |
| Registry Onboarding | resources with metadata | registry entries | lookup/index succeeds | deregister or mark draft |
| Validation Gate | registry/resource set | `ValidationResult` | `valid=true` or accepted warnings | fix metadata/spec/policy |
| Publishing | package bundle | published package | integrity + review pass | publish replacement/deprecate old version |

### 6. Safety Boundaries

- Migration MUST NOT require destructive source restructuring by default.
- Migration MUST NOT store plaintext secrets in resource manifests.
- Publishing MUST NOT happen before validation and required integrity/security checks pass.
- Execution MUST NOT happen before validation and dependency gates pass.
- Existing source-of-truth files SHOULD remain authoritative until migrated resources are reviewed and accepted.
- Registry aliases/deprecations SHOULD be used instead of silently moving resource identities.

### 7. Rollback Guidance

- Inventory artifacts can be discarded without changing source resources.
- Metadata overlays can be removed or reverted if validation fails.
- Registry entries can be deregistered, marked `Draft`, or deprecated depending registry policy.
- Published package versions SHOULD be immutable; rollback should publish a replacement or deprecate the bad version instead of mutating old artifacts.
- Adapter/build rollout should be reversible by disabling the adapter or reverting execution plans.

### 8. Examples

RFC nên có ví dụ ngắn:
- Prompt library migration: prompt files → `Prompt` resources → metadata overlay → validation → registry onboarding.
- Workflow script migration: scripts → `Workflow` resources → dependency graph → adapter rollout.
- Stop at validation: metadata duplicate or plaintext secret blocks migration.

### 9. References

Tối thiểu gồm:
- `RFC-0100 — Canonical Resource Model`
- `RFC-0102 — Metadata Model`
- `RFC-0104 — Dependency Graph`
- `RFC-0106 — Validation Model`
- `RFC-0200 — Runtime Architecture`
- `RFC-0400 — Build Pipeline`
- `RFC-0402 — Packaging`
- `RFC-0403 — Registry and Marketplace`
- `RFC-0800 — Security Model`
- `RFC-0900 — Enterprise Reference Architecture`

## Ranh Giới Với Các RFC Khác

| Chủ đề | RFC sở hữu | Cách `RFC-0901` xử lý |
|---|---|---|
| Enterprise target architecture | `RFC-0900` | Chỉ hướng dẫn migration vào kiến trúc, không định nghĩa kiến trúc đích. |
| Resource envelope | `RFC-0100` | Dùng resource model làm target mapping. |
| Metadata detail | `RFC-0102` | Chỉ hướng dẫn overlay metadata. |
| Dependency graph | `RFC-0104` | Chỉ dùng làm migration gate. |
| Validation result | `RFC-0106` | Chỉ dùng `ValidationResult` làm gate. |
| Runtime/build | `RFC-0200`, `RFC-0400` | Chỉ dùng trong rollout phase. |
| Security | `RFC-0800` | Chỉ áp safety/security gate. |
| Governance | `RFC-0802` | Không định nghĩa approval workflow. |

## Acceptance Criteria Cho Vòng Implementation Sau

- `docs/1000-enterprise-reference/RFC-0901-Migration-Guide.md` không còn boilerplate chung §2–§14.
- RFC có các section riêng: `Migration Principles`, `Migration Phases`, `Phase Gate Matrix`, `Safety Boundaries`, `Rollback Guidance`, `Examples`, `References`.
- RFC chốt migration không xâm lấn: metadata overlay trước restructure.
- RFC chốt validation/dependency/security gates trước build/execution/publish.
- RFC chốt rollback guidance cho metadata, registry và package publish.
- RFC cross-ref đúng các RFC sở hữu resource model, metadata, dependency graph, validation, runtime, build/package/registry, security và enterprise architecture.
- `bash tools/check-rfc-links` vẫn PASS; nếu cần rule riêng cho RFC-0901 thì thêm trong implementation plan, không tự làm trong design này.

## Rủi Ro Và Giảm Thiểu

- **Lấn sang RFC-0900:** chỉ hướng dẫn migration playbook, không thiết kế enterprise architecture.
- **Lấn sang governance:** human review/policy gates chỉ nêu ở mức boundary, process chi tiết thuộc `RFC-0802`.
- **Quá implementation-specific:** giữ ở phase/gate/output level, không định nghĩa CLI hoặc runtime command.
- **Migration phá hủy:** nhấn mạnh non-invasive default, rollback và review gates.