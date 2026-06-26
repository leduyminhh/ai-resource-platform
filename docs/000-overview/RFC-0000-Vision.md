# RFC-0000 — Vision

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0100, RFC-0200

---

## Abstract
Tầm nhìn dài hạn cho một nền tảng vendor-neutral, resource-oriented dành cho năng lực phát triển AI tái sử dụng.

## 1. The Problem
- Asset phát triển AI (plugin, skill, prompt, workflow…) bị khóa theo từng IDE/vendor, khó tái dùng.
- Mỗi công cụ một model riêng → không có contract chung, không tương tác chéo.

## 2. The Vision
ARPS chuẩn hóa mọi platform object thành một **Resource** chuẩn (canonical). Runtime engine thao
tác trên Resource + dependency graph, độc lập vendor. "Everything is a canonical Resource."

## 3. Goals
- Định nghĩa contract ổn định.
- Hỗ trợ migration không xâm lấn.
- Giữ nền tảng resource-oriented.
- Bảo toàn hành vi deterministic.
- Cho phép mở rộng mà không đổi kiến trúc lõi.

## 4. Non-Goals
- Không bắt buộc một ngôn ngữ lập trình cụ thể.
- Không phụ thuộc một AI assistant / IDE / vendor cụ thể.
- Không ép tái cấu trúc mã nguồn nghiệp vụ sẵn có.

## 5. The Resource Thesis
Mỗi đối tượng được biểu diễn bằng một canonical resource (apiVersion/kind/metadata/spec/status).
Chi tiết mô hình: xem [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md).

## 6. Lifecycle at a glance
Repository → Discovery → Registry → Validation → Dependency Resolver → Planning → Execution →
Packaging → Publishing → Registry/Marketplace. Chi tiết engine + ranh giới: xem
[RFC-0200](../300-runtime/RFC-0200-Runtime-Architecture.md); sơ đồ: `docs/diagrams/runtime-flow.mmd`.

## 7. Roadmap / Future Work
- Conformance tests.
- Reference runtime implementation.
- Registry interoperability suite.
- Extended JSON/YAML schema definitions.

## References
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
- [RFC-0002 — Guiding Principles](RFC-0002-Guiding-Principles.md)
