# Design — RFC-0007 Compatibility Policy

- **Ngày:** 2026-06-29
- **Phạm vi:** Phase A (hoàn thiện đặc tả), sub-project — `docs/100-foundation/RFC-0007-Compatibility-Policy.md`
- **Approach:** B — Policy + compatibility matrix
- **Trạng thái:** Draft (chờ review)

## Bối cảnh

Cụm `000-overview` đã hoàn tất và cross-ref `RFC-0007` như nguồn sở hữu compatibility policy. Hiện `RFC-0007` vẫn còn cấu trúc boilerplate giống các RFC chưa xử lý: phần Canonical Model, Runtime Flow, Error Model, Security, Migration và Future Work đang lặp nội dung chung thay vì mô tả chính sách tương thích riêng.

Các tài liệu hiện đang kỳ vọng `RFC-0007` trả lời các câu hỏi như:
- Breaking change khi nào phải nâng major version.
- Field bổ sung, unknown labels/annotations, schema extension có được xem là tương thích không.
- Compatibility được xét ra sao giữa Resource, Schema, Adapter, Package và Registry.

## Mục tiêu

- Biến `RFC-0007` thành nguồn sự thật cho compatibility classification trong ARPS.
- Đưa ra compatibility matrix đủ rõ để các RFC khác cross-ref mà không phải lặp lại policy.
- Giữ ranh giới với `RFC-0005` (versioning), `RFC-0106` (validation/error reporting) và runtime engine RFC.
- Loại bỏ boilerplate không thuộc trách nhiệm của `RFC-0007`.

## Không Làm

- Không định nghĩa chi tiết SemVer hoặc version range syntax; phần này thuộc `RFC-0005`.
- Không định nghĩa schema đầy đủ của canonical resource; phần này thuộc `RFC-0100`.
- Không định nghĩa taxonomy lỗi hoặc output chính thức của validator; phần này thuộc `RFC-0106`.
- Không định nghĩa runtime flow hay implementation algorithm cho validation engine.

## Thiết Kế Đề Xuất

`RFC-0007` sẽ được viết lại thành một RFC policy/matrix, giữ header chuẩn đã chốt ở ADR-0003 và dùng các section riêng theo trách nhiệm.

### 1. Abstract

Tóm tắt rằng RFC này định nghĩa cách phân loại và đánh giá compatibility cho resource-oriented platform objects: Resource, Schema, Adapter, Package và Registry.

### 2. Conventions

Nêu quy ước RFC 2119 và quan hệ nguồn sự thật:
- Versioning rule chi tiết: `RFC-0005`.
- Canonical resource fields: `RFC-0100`.
- Validation/error reporting: `RFC-0106`.
- Packaging/registry implications: `RFC-0107`, `RFC-0402`, `RFC-0403`.

### 3. Compatibility Classes

Định nghĩa bốn lớp phân loại:

| Class | Ý nghĩa |
|---|---|
| `Compatible` | Consumer hiện tại có thể tiếp tục hoạt động mà không cần đổi cấu hình hoặc code. |
| `Conditionally Compatible` | Tương thích nếu consumer hỗ trợ capability/schema policy cụ thể hoặc bật migration/alias đã khai báo. |
| `Breaking` | Consumer hợp lệ trước đó có thể fail, hiểu sai dữ liệu hoặc tạo artifact khác nghĩa. |
| `Unknown` | Không đủ thông tin để kết luận; implementation không được tự coi là compatible. |

### 4. General Rules

Các rule chung áp dụng trước matrix:
- Additive optional field thường là `Compatible` nếu schema policy cho phép extension.
- Thêm required field, đổi type, đổi semantic của field hiện hữu là `Breaking`.
- Rename/remove field là `Breaking` nếu không có alias hoặc migration path.
- Tighten constraint thường là `Breaking`; loosen constraint có thể `Compatible` hoặc `Conditionally Compatible`.
- Deprecation tự thân không breaking; removal sau deprecation vẫn phải xét theo migration/alias/version policy.
- Khi nhiều rule áp dụng, chọn classification nghiêm hơn.

### 5. Compatibility Matrix

Matrix là phần lõi của RFC. Mỗi subject có bảng các change type và classification mặc định.

| Subject | Nội dung cần bao phủ |
|---|---|
| Resource | `metadata`, `spec`, `status`, lifecycle, labels/annotations, kind/id changes. |
| Schema | required/optional fields, enum values, type changes, constraints, unknown fields. |
| Adapter | declared capabilities, supported resource kinds, output shape, vendor mapping. |
| Package | dependency range, included resources, lock/integrity metadata, package metadata. |
| Registry | ID uniqueness, namespace movement, alias, deprecation, replacement policy. |

### 6. Deprecation Policy

Chuẩn hóa vòng đời tương thích:
- Deprecated item phải chỉ ra replacement hoặc lý do không có replacement.
- Deprecated item vẫn phải được consumer hiểu nếu còn trong supported compatibility window.
- Removal/rename cần major version hoặc declared migration/alias tùy rule `RFC-0005`.

### 7. Decision Rules

Khi implementation đánh giá compatibility:
- Không đủ metadata thì trả `Unknown` thay vì `Compatible`.
- Rule cụ thể theo subject thắng rule chung.
- Nếu hai rule mâu thuẫn, chọn classification nghiêm hơn theo thứ tự: `Breaking` > `Unknown` > `Conditionally Compatible` > `Compatible`.
- Error code/reporting chính thức forward-ref sang `RFC-0106`.

### 8. Examples

RFC nên có ví dụ ngắn, không quá dài:
- Thêm optional field vào `spec`.
- Thêm required field vào schema.
- Thêm enum value.
- Deprecate adapter capability và khai báo replacement.
- Di chuyển namespace registry kèm alias.

### 9. References

Tối thiểu gồm:
- `RFC-0005 — Versioning`
- `RFC-0100 — Canonical Resource Model`
- `RFC-0106 — Validation Model`
- `RFC-0107 — Packaging Model`
- `RFC-0401 — Lock File`
- `RFC-0402 — Packaging`
- `RFC-0403 — Registry and Marketplace`

## Ranh Giới Với Các RFC Khác

| Chủ đề | RFC sở hữu | Cách `RFC-0007` xử lý |
|---|---|---|
| SemVer, major/minor/patch | `RFC-0005` | Chỉ nói breaking change cần version boundary phù hợp. |
| Canonical resource model | `RFC-0100` | Chỉ phân loại compatibility khi field thay đổi. |
| Validation errors | `RFC-0106` | Chỉ forward-ref cho error/reporting. |
| Runtime validation engine | `RFC-0203` | Không mô tả flow engine. |
| Package/registry mechanics | `RFC-0402`, `RFC-0403` | Chỉ nêu compatibility implication. |

## Acceptance Criteria Cho Vòng Implementation Sau

- `docs/100-foundation/RFC-0007-Compatibility-Policy.md` không còn boilerplate chung §2–§14.
- RFC có compatibility classes và matrix theo 5 subject: Resource, Schema, Adapter, Package, Registry.
- RFC cross-ref đúng các RFC sở hữu phần versioning, validation, packaging, registry.
- Không sửa `RFC-0005` hoặc `RFC-0006` trong cùng task, trừ khi chỉ cần cross-ref tối thiểu do link/summary.
- Validator hiện có `bash tools/check-rfc-links` vẫn PASS; nếu cần rule riêng cho RFC-0007 thì thêm ở plan implementation, không tự làm trong design này.

## Rủi Ro Và Giảm Thiểu

- **Lấn sang RFC-0106:** giới hạn `RFC-0007` ở classification; error reporting chỉ là forward-ref.
- **Lấn sang RFC-0005:** không định nghĩa SemVer syntax; chỉ dùng policy “breaking cần version boundary”.
- **Matrix quá chi tiết:** ưu tiên classification mặc định và ví dụ tiêu biểu, tránh thuật toán implementation.
- **Forward-ref tới RFC còn boilerplate:** chấp nhận trong Phase A; ghi rõ ownership để xử lý các vòng sau.
