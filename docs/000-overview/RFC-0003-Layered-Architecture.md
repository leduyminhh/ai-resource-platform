# RFC-0003 — Layered Architecture

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0100, RFC-0200

---

## Abstract
Định nghĩa các tầng nền tảng: Specification, Metadata, Resource, Runtime, Adapter, Distribution, Governance.

The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119.

## 1. Layers
```text
Governance
Distribution
Adapter
Runtime
Resource
Metadata
Specification
```

## 2. Layer responsibilities
| Tầng | Trách nhiệm | RFC phủ | Phụ thuộc cho phép |
|---|---|---|---|
| Specification | Định nghĩa contract/đặc tả, từ vựng, nguyên tắc | RFC-0000..0007 | (đáy) |
| Metadata | Mô hình metadata: identity, ownership, discovery, compat | RFC-0102 | Specification |
| Resource | Canonical model, manifest, lifecycle, dependency graph | RFC-0100..0107 | Metadata |
| Runtime | Engine: discovery→…→packaging | RFC-0200..0209 | Resource |
| Adapter | Biến đổi resource → output đích | RFC-0500, RFC-0700 | Runtime |
| Distribution | Build, package, registry, marketplace | RFC-0400..0403 | Runtime |
| Governance | Ownership, review, policy, security | RFC-0800..0802 | tất cả tầng dưới |

## 3. Dependency rule (normative)
- Một tầng MUST chỉ phụ thuộc các tầng BÊN DƯỚI nó.
- Một tầng MUST NOT phụ thuộc (reach up) tầng bên trên.
- Đây là nguồn sự thật cho quy tắc phân tầng; RFC khác trỏ tới mục này.

## 4. Mapping tầng ↔ engine ↔ category
- Runtime ↔ các engine [RFC-0200](../300-runtime/RFC-0200-Runtime-Architecture.md).
- Distribution ↔ build/pipeline [RFC-0400](../500-build-distribution/RFC-0400-Build-Pipeline.md).
- Governance ↔ [RFC-0802](../900-security-governance/RFC-0802-Governance.md).

## References
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
