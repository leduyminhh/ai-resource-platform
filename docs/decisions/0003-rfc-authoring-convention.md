# ADR-0003: Convention authoring cho RFC

- Trạng thái: Accepted
- Ngày: 2026-06-26

## Bối cảnh
44 RFC hiện dùng chung body boilerplate (§2–§14 giống hệt); nội dung normative bị nhân bản.
Cần một convention authoring để mỗi RFC có cấu trúc riêng nhưng vẫn nhất quán, và để quy tắc
normative không bị lặp.

## Quyết định
1. **Header block chuẩn** (markdown): Status, Category, Specification, Version, Requires (tùy chọn), Supersedes (tùy chọn).
2. **Status tài liệu RFC**: Draft → Accepted → Final; hoặc Deprecated / Superseded. TÁCH khỏi
   resource lifecycle (RFC-0103 dành cho resource, không phải văn bản RFC).
3. **Cấu trúc riêng theo từng RFC** — bỏ boilerplate 14 mục; mỗi RFC cấu trúc theo chủ đề, kết bằng `## References`.
4. **Ngôn ngữ normative**: RFC 2119; chèn dòng diễn giải chuẩn ở RFC có MUST/SHOULD.
5. **Delegation rule**: mỗi quy tắc cross-cutting có đúng một RFC sở hữu; RFC khác trỏ tới
   (cross-ref link tương đối), KHÔNG phát biểu lại dạng normative.

## Hệ quả
- (+) Hết trùng lặp normative; mỗi quy tắc một nguồn sự thật.
- (+) RFC dễ đọc, đúng chủ đề.
- (−) Tạo forward-reference tới RFC chưa hoàn thiện (chấp nhận; ghi TODO).
- Áp dụng dần: cụm 000-overview làm trước (xem docs/superpowers/plans/2026-06-26-overview-rfc-cluster.md).
