# RFC-0002 — Guiding Principles

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0

---

## Abstract
Định nghĩa các nguyên tắc thiết kế: resource-first, schema-first, deterministic builds, vendor neutrality, non-invasive migration, extensibility.

The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119.

## 1. Resource-first
- **Statement:** Mọi platform object là một Resource chuẩn.
- **Rationale:** Một mô hình chung cho phép engine thao tác đồng nhất, không phụ thuộc kind.
- **Implications:** Tính năng mới nên thêm như một kind/spec, không phải cơ chế ngoài luồng.

## 2. Schema-first
- **Statement:** Hình dạng resource do schema định nghĩa; resource SHOULD validate trước khi xử lý.
- **Rationale:** Bắt lỗi sớm, contract rõ ràng.
- **Implications:** Thay đổi schema đi qua versioning ([RFC-0005](../100-foundation/RFC-0005-Versioning.md)).

## 3. Deterministic builds
- **Statement:** Cùng input MUST cho cùng output.
- **Rationale:** Tái lập, cache an toàn, tin cậy.
- **Implications:** Tránh phụ thuộc thời gian/ngẫu nhiên ngầm; xem [RFC-0208](../300-runtime/RFC-0208-Caching-Strategy.md).

## 4. Vendor neutrality
- **Statement:** Lõi không phụ thuộc IDE/assistant/vendor cụ thể.
- **Rationale:** Tương tác chéo, tránh khóa.
- **Implications:** Tích hợp vendor đi qua Adapter ([RFC-0500](../600-sdk/RFC-0500-Adapter-SDK.md)).

## 5. Non-invasive migration
- **Statement:** Đưa asset sẵn có vào ARPS không cần dời/đổi cấu trúc nguồn.
- **Rationale:** Giảm rào cản áp dụng.
- **Implications:** Thêm metadata tại chỗ; xem [RFC-0901](../1000-enterprise-reference/RFC-0901-Migration-Guide.md).

## 6. Extensibility
- **Statement:** Mở rộng được mà không đổi kiến trúc lõi.
- **Rationale:** Tiến hóa lâu dài.
- **Implications:** Field bổ sung theo schema policy; xem [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md).

## 7. Resolving tensions
- Extensibility ⇄ determinism: phần mở rộng phải giữ tính deterministic; field tùy ý không được đổi output đã build.
- Vendor neutrality ⇄ tích hợp thực tế: đặc thù vendor sống ở tầng Adapter, không rò vào lõi.

## References
- [RFC-0000 — Vision](RFC-0000-Vision.md)
- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
