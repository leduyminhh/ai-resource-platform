# Design — RFC-0106 Validation Model

- **Ngày:** 2026-06-29
- **Phạm vi:** Phase A (hoàn thiện đặc tả), sub-project — `docs/200-resource-model/RFC-0106-Validation-Model.md`
- **Approach:** B — Layer contract + result envelope
- **Trạng thái:** Draft (chờ review)

## Bối cảnh

`RFC-0106` được các tài liệu nền và RFC đã hoàn thiện xem như nguồn sở hữu validation rules và error reporting. `RFC-0100` đã chốt canonical resource envelope và forward-ref validation reporting sang `RFC-0106`. `RFC-0007` đã chốt compatibility classification và forward-ref `COMPATIBILITY_ERROR` sang `RFC-0106`.

Hiện `RFC-0106` vẫn còn boilerplate chung §2–§14, nhưng phần Abstract đã nêu đúng trách nhiệm: định nghĩa common validation layers và result format cho syntax, schema, metadata, semantic, compatibility và policy validation.

## Mục Tiêu

- Định nghĩa validation layers chung cho ARPS resources.
- Định nghĩa result envelope chuẩn cho validator output.
- Định nghĩa diagnostic object gồm code, severity, layer, message, path/location và reference.
- Chốt error code taxonomy tối thiểu để các RFC khác cross-ref.
- Chốt required behavior: deterministic diagnostics, valid=false khi có error, không mutate source resources.
- Loại bỏ boilerplate không thuộc trách nhiệm của `RFC-0106`.

## Không Làm

- Không định nghĩa execution strategy của validation engine; phần đó thuộc `RFC-0203 Validation Engine`.
- Không định nghĩa dependency resolution algorithm; phần đó thuộc `RFC-0204 Dependency Resolver`.
- Không định nghĩa compatibility classification; phần đó thuộc `RFC-0007 Compatibility Policy`.
- Không định nghĩa canonical resource envelope; phần đó thuộc `RFC-0100 Canonical Resource Model`.
- Không định nghĩa security/governance policy chi tiết; phần đó thuộc `RFC-0800` và `RFC-0802`.
- Không chốt CLI/IDE output rendering; các surface đó có thể map từ result envelope sau.

## Thiết Kế Đề Xuất

`RFC-0106` sẽ được viết lại thành RFC sở hữu validation layer contract, result envelope và diagnostic/error model, giữ header chuẩn đã chốt ở ADR-0003.

### 1. Abstract

Tóm tắt rằng RFC này định nghĩa validation model chung cho ARPS resources, bao gồm layers, result envelope và diagnostics.

### 2. Conventions

Nêu quy ước RFC 2119 và ownership boundary:
- Resource envelope: `RFC-0100`.
- Serialization/syntax: `RFC-0101`.
- Metadata semantics: `RFC-0102`.
- Dependency graph rules: `RFC-0104`.
- Compatibility classification: `RFC-0007`.
- Validation engine execution: `RFC-0203`.
- Security/governance policy: `RFC-0800`, `RFC-0802`.

### 3. Validation Inputs

Validator nhận các loại context sau, tùy layer:

| Input | Vai trò |
|---|---|
| Resource document | YAML/JSON source được validate. |
| Canonical envelope | `apiVersion`, `kind`, `metadata`, `spec`, `status` theo `RFC-0100`. |
| Schema context | Envelope schema + kind-specific schema. |
| Registry context | ID uniqueness, version set, namespace policy. |
| Dependency graph context | Edges, missing dependencies, cycle detection. |
| Compatibility context | Previous/current versions or declared compatibility policy. |
| Policy context | Organization/registry/security/governance constraints. |

### 4. Validation Layers

Layer contract nên có thứ tự khuyến nghị nhưng không ép execution engine:

| Layer | Trách nhiệm | Code nhóm |
|---|---|---|
| `syntax` | Parse YAML/JSON, document encoding, malformed source. | `SYNTAX_ERROR` |
| `schema` | Validate envelope và kind-specific schema. | `SCHEMA_ERROR` |
| `metadata` | Validate identity/version/registry metadata. | `METADATA_ERROR` |
| `semantic` | Validate kind-specific invariants không biểu diễn được bằng schema. | `SEMANTIC_ERROR` |
| `dependency` | Validate missing dependencies, graph edges, acyclic rule. | `DEPENDENCY_ERROR` |
| `compatibility` | Report classification conflicts từ `RFC-0007`. | `COMPATIBILITY_ERROR` |
| `policy` | Apply org/registry/security/governance policies. | `POLICY_VIOLATION` |

### 5. Validation Result Envelope

Result shape chuẩn:

```yaml
valid: false
diagnostics:
  - code: METADATA_ERROR
    severity: error
    layer: metadata
    message: metadata.id must be unique in the registry
    resourceId: core/clean-code
    path: /metadata/id
    location:
      file: examples/resources/skill.yaml
      line: 4
      column: 7
    reference: RFC-0102
summary:
  errorCount: 1
  warningCount: 0
  infoCount: 0
validator:
  name: arps-validator
  version: 0.1.0
```

Required fields:
- `valid`
- `diagnostics`

Optional fields:
- `summary`
- `validator`

### 6. Diagnostic Object

| Field | Required | Semantics |
|---|---|---|
| `code` | Yes | Stable machine-readable error/warning code. |
| `severity` | Yes | `error`, `warning` hoặc `info`. |
| `layer` | Yes | Validation layer phát hiện diagnostic. |
| `message` | Yes | Human-readable message. |
| `resourceId` | No | Resource liên quan nếu parse/metadata cho phép xác định. |
| `path` | No | JSON Pointer/YAML path tới vị trí logic. |
| `location` | No | File/line/column nếu có source location. |
| `reference` | No | RFC/schema/policy liên quan. |

### 7. Error Code Taxonomy

Mã tối thiểu:
- `SYNTAX_ERROR`
- `SCHEMA_ERROR`
- `METADATA_ERROR`
- `SEMANTIC_ERROR`
- `DEPENDENCY_ERROR`
- `COMPATIBILITY_ERROR`
- `POLICY_VIOLATION`
- `VALIDATOR_ERROR`

`BUILD_ERROR` không thuộc core validation model; build pipeline có thể sở hữu ở `RFC-0400` và map sang result/diagnostic riêng nếu cần.

### 8. Required Behavior

- `valid` MUST be `false` nếu có ít nhất một diagnostic severity `error`.
- `valid` MAY be `true` khi chỉ có `warning` hoặc `info`.
- Validators MUST produce deterministic diagnostic ordering for identical inputs and context.
- Validators SHOULD continue after recoverable errors to report multiple diagnostics.
- Validators MUST NOT mutate source resources during validation.
- Missing required context MUST produce diagnostic thay vì silent pass.
- Unsupported kind/schema/policy SHOULD produce `SCHEMA_ERROR`, `POLICY_VIOLATION` hoặc `VALIDATOR_ERROR` tùy nguyên nhân.

### 9. Examples

RFC nên có ví dụ ngắn:
- Valid result không diagnostic.
- Invalid metadata duplicate ID.
- Compatibility error từ breaking change theo `RFC-0007`.
- Policy violation do namespace bị cấm.
- Validator internal failure (`VALIDATOR_ERROR`) nếu validator không thể load schema.

### 10. References

Tối thiểu gồm:
- `RFC-0007 — Compatibility Policy`
- `RFC-0100 — Canonical Resource Model`
- `RFC-0101 — Serialization`
- `RFC-0102 — Metadata Model`
- `RFC-0104 — Dependency Graph`
- `RFC-0203 — Validation Engine`
- `RFC-0800 — Security Model`
- `RFC-0802 — Governance`

## Ranh Giới Với Các RFC Khác

| Chủ đề | RFC sở hữu | Cách `RFC-0106` xử lý |
|---|---|---|
| Resource envelope | `RFC-0100` | Chỉ validate envelope, không định nghĩa lại. |
| Serialization | `RFC-0101` | Syntax layer dùng làm input; không định nghĩa serializer. |
| Metadata semantics | `RFC-0102` | Metadata layer báo lỗi metadata. |
| Dependency graph | `RFC-0104` | Dependency layer báo missing/cycle, không định nghĩa resolver. |
| Compatibility | `RFC-0007` | Compatibility layer báo `COMPATIBILITY_ERROR`, không phân loại lại. |
| Engine execution | `RFC-0203` | Không chốt sync/parallel/cache/incremental strategy. |
| Security/governance | `RFC-0800`, `RFC-0802` | Policy layer báo violation. |

## Acceptance Criteria Cho Vòng Implementation Sau

- `docs/200-resource-model/RFC-0106-Validation-Model.md` không còn boilerplate chung §2–§14.
- RFC có các section riêng: `Validation Inputs`, `Validation Layers`, `Validation Result Envelope`, `Diagnostic Object`, `Error Code Taxonomy`, `Required Behavior`, `Examples`, `References`.
- RFC chốt severity set: `error`, `warning`, `info`.
- RFC chốt error code taxonomy gồm `SYNTAX_ERROR`, `SCHEMA_ERROR`, `METADATA_ERROR`, `SEMANTIC_ERROR`, `DEPENDENCY_ERROR`, `COMPATIBILITY_ERROR`, `POLICY_VIOLATION`, `VALIDATOR_ERROR`.
- RFC nêu rõ `valid=false` khi có diagnostic severity `error`.
- RFC nêu rõ validator không mutate source resources.
- RFC cross-ref đúng các RFC sở hữu envelope, metadata, dependency, compatibility, engine và governance.
- `bash tools/check-rfc-links` vẫn PASS; nếu cần rule riêng cho RFC-0106 thì thêm trong implementation plan, không tự làm trong design này.

## Rủi Ro Và Giảm Thiểu

- **Lấn sang RFC-0203:** chỉ định nghĩa contract/result, không định nghĩa engine execution strategy.
- **Lấn sang RFC-0007:** chỉ report compatibility diagnostics, không định nghĩa lại compatibility classification.
- **Diagnostic quá phức tạp:** giữ required fields tối thiểu, để `location`, `summary`, `validator` optional.
- **Policy quá rộng:** chỉ định nghĩa policy layer và `POLICY_VIOLATION`; policy cụ thể thuộc security/governance RFC.