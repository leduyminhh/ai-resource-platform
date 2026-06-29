# Design — RFC-0800 Security Model

- **Ngày:** 2026-06-29
- **Phạm vi:** Phase A (hoàn thiện đặc tả), sub-project — `docs/900-security-governance/RFC-0800-Security-Model.md`
- **Approach:** B — Trust boundary model
- **Trạng thái:** Draft (chờ review)

## Bối cảnh

`RFC-0800` được `TODO.md` xem là nguồn sở hữu Security Considerations. Hiện RFC này vẫn còn boilerplate chung §2–§14, trong đó Security Considerations chỉ gồm vài rule ngắn: remote resources should be verified, packages should include checksums, secrets must not be stored in manifests, registries should be trusted.

Sau khi đã hoàn thiện `RFC-0100` (resource envelope), `RFC-0106` (validation diagnostics) và `RFC-0200` (runtime boundaries), `RFC-0800` cần chốt security principles và trust boundaries để các RFC khác cross-ref thay vì lặp nội dung security chung.

## Mục Tiêu

- Định nghĩa security principles nền tảng cho ARPS.
- Chốt trust boundaries giữa local workspace, remote registry, package/artifact, adapter/execution và secrets.
- Chốt required controls tối thiểu cho remote resources, packages, secrets, registries và execution.
- Chốt validation hooks để security/policy failures map sang `RFC-0106` diagnostics.
- Loại bỏ boilerplate không thuộc trách nhiệm của `RFC-0800`.

## Không Làm

- Không định nghĩa signing/checksum/integrity algorithm chi tiết; phần đó thuộc `RFC-0801 Signing and Integrity`.
- Không định nghĩa governance approval workflow hoặc policy lifecycle; phần đó thuộc `RFC-0802 Governance`.
- Không định nghĩa execution engine internals; phần đó thuộc `RFC-0206 Execution Engine`.
- Không định nghĩa diagnostic result shape; phần đó thuộc `RFC-0106 Validation Model`.
- Không định nghĩa package format chi tiết; phần đó thuộc `RFC-0402 Packaging`.
- Không định nghĩa registry protocol chi tiết; phần đó thuộc `RFC-0403 Registry and Marketplace`.

## Thiết Kế Đề Xuất

`RFC-0800` sẽ được viết lại thành RFC sở hữu security contract và trust boundaries, giữ header chuẩn đã chốt ở ADR-0003.

### 1. Abstract

Tóm tắt rằng RFC này định nghĩa security model cho trust, secrets, remote resources, package verification và execution boundaries.

### 2. Conventions

Nêu quy ước RFC 2119 và ownership boundary:
- Validation diagnostics: `RFC-0106`.
- Runtime boundaries: `RFC-0200`.
- Execution engine: `RFC-0206`.
- Packaging/registry: `RFC-0402`, `RFC-0403`.
- Signing/integrity: `RFC-0801`.
- Governance/policy workflow: `RFC-0802`.

### 3. Security Principles

- **Explicit trust:** remote registries, packages and execution targets are not trusted by default.
- **Least privilege:** adapters/tools receive only capabilities and context required for the operation.
- **No plaintext secrets:** resource manifests must not contain plaintext credentials, tokens or key material.
- **Verify before use:** remote resources and packages should be verified before validation/execution/publishing decisions.
- **Deterministic and auditable:** security-relevant decisions should be reproducible and traceable.
- **Fail closed for unknown trust:** unknown or missing trust context should not silently pass for high-risk actions.

### 4. Trust Boundaries

| Boundary | Meaning | Main risk |
|---|---|---|
| Local Workspace | Repository files and local resource manifests. | unreviewed local changes, accidental secret inclusion |
| Registry / Marketplace | Remote metadata/resource/package source. | dependency confusion, untrusted source, stale or malicious metadata |
| Package / Artifact | Distributable bundles and generated artifacts. | tampering, missing integrity metadata, artifact substitution |
| Adapter / Execution | Tool/model/vendor execution boundary. | undeclared capability use, unsafe side effects, data exfiltration |
| Secret Boundary | External secret provider or runtime secret injection. | plaintext secret leakage, logging secrets, manifest persistence |

### 5. Required Controls

- Remote registries and marketplace sources SHOULD be explicitly trusted before use.
- Remote resources SHOULD be verified before use; high-risk actions MAY require policy to make verification mandatory.
- Packages SHOULD include integrity metadata; signing/checksum mechanism details are owned by `RFC-0801`.
- Resource manifests MUST NOT contain plaintext secrets.
- Execution MUST use declared adapters and capabilities.
- Tools MUST NOT log secrets or write them into generated manifests/artifacts.
- Security policy failures SHOULD surface as `POLICY_VIOLATION` diagnostics using `RFC-0106`.

### 6. Validation Hooks

RFC-0800 should define hooks, not engine internals:

| Hook | Layer | Example diagnostic |
|---|---|---|
| Secret scan | policy/semantic | plaintext token-like value found in manifest |
| Registry trust check | policy | registry source is not in trusted set |
| Package integrity check | policy | package is missing required integrity metadata |
| Adapter capability check | policy/semantic | execution requires undeclared adapter capability |
| Remote source check | policy | remote resource source cannot be verified |

### 7. Failure Behavior

- Unknown trust context SHOULD produce warning or error based on policy risk level; high-risk execution/publishing SHOULD fail closed.
- Plaintext secret detection MUST be an error.
- Missing required package integrity metadata MUST produce `POLICY_VIOLATION` when policy requires it.
- Untrusted registry/package/execution target MUST NOT be silently allowed.
- Security diagnostics should include enough reference/path/location information for remediation when available.

### 8. Examples

RFC nên có ví dụ ngắn:
- Trusted registry accepted because it is explicitly configured.
- Package rejected or warned because integrity metadata is missing.
- Manifest rejected because `spec.token` contains a plaintext token.
- Execution blocked because adapter capability is not declared.

### 9. References

Tối thiểu gồm:
- `RFC-0106 — Validation Model`
- `RFC-0200 — Runtime Architecture`
- `RFC-0206 — Execution Engine`
- `RFC-0402 — Packaging`
- `RFC-0403 — Registry and Marketplace`
- `RFC-0801 — Signing and Integrity`
- `RFC-0802 — Governance`

## Ranh Giới Với Các RFC Khác

| Chủ đề | RFC sở hữu | Cách `RFC-0800` xử lý |
|---|---|---|
| Validation result | `RFC-0106` | Chỉ yêu cầu security failures map sang diagnostics. |
| Runtime boundary | `RFC-0200` | Dùng runtime handoff để đặt security boundary. |
| Execution details | `RFC-0206` | Chỉ yêu cầu declared adapters/capabilities. |
| Package format | `RFC-0402` | Chỉ yêu cầu integrity metadata ở mức policy. |
| Registry protocol | `RFC-0403` | Chỉ yêu cầu explicit trust/verification. |
| Signing/integrity | `RFC-0801` | Không định nghĩa thuật toán checksum/signature. |
| Governance workflow | `RFC-0802` | Không định nghĩa approval/policy lifecycle. |

## Acceptance Criteria Cho Vòng Implementation Sau

- `docs/900-security-governance/RFC-0800-Security-Model.md` không còn boilerplate chung §2–§14.
- RFC có các section riêng: `Security Principles`, `Trust Boundaries`, `Required Controls`, `Validation Hooks`, `Failure Behavior`, `Examples`, `References`.
- RFC chốt nguyên tắc no plaintext secrets.
- RFC chốt remote registries/resources/packages không được trusted ngầm.
- RFC chốt execution phải dùng declared adapters/capabilities.
- RFC chốt policy/security failures map sang `POLICY_VIOLATION` / diagnostics của `RFC-0106`.
- RFC cross-ref đúng các RFC sở hữu validation, runtime, execution, packaging, registry, signing, governance.
- `bash tools/check-rfc-links` vẫn PASS; nếu cần rule riêng cho RFC-0800 thì thêm trong implementation plan, không tự làm trong design này.

## Rủi Ro Và Giảm Thiểu

- **Lấn sang RFC-0801:** chỉ yêu cầu integrity metadata, không định nghĩa thuật toán signing/checksum.
- **Lấn sang RFC-0802:** chỉ nói policy diagnostics và trust context, không định nghĩa governance process.
- **Quá security-heavy:** giữ ở mức required controls và trust boundaries, không viết threat model đầy đủ.
- **False positive secret scanning:** RFC-0800 chỉ định nghĩa hook/behavior; heuristic cụ thể để validator/policy implementation sở hữu sau.