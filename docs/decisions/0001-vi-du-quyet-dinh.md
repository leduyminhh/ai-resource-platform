# ADR-0001: Chốt root mã nguồn + quy ước module + phân tầng backend

- Trạng thái: Accepted
- Ngày: 2026-06-26

## Bối cảnh
Cần chốt thư mục ROOT của mã nguồn và quy ước tổ chức module để git diff sạch, scope
build/test/lint rõ, và tách tầng tài liệu khỏi tầng mã nguồn.

## Các lựa chọn đã cân nhắc
1. Một root `src/` + mỗi module một thư mục con tự chứa — rõ ràng, dễ scope.
2. Monorepo nhiều app (`apps/` + `packages/`) — phù hợp khi nhiều ứng dụng chung lib.
3. Trộn mã nguồn với tài liệu — loại (diff bẩn, khó scope).

## Quyết định
- Root mã nguồn: `src/` (đã tồn tại `src/shared/`).
- Mỗi module là thư mục con tự chứa trong `src/<ten-module>/`; code dùng chung ở `src/shared/`.
- Module chỉ giao tiếp qua API công khai hoặc shared (no reach-in). Chi tiết: project-knowledge/source-structure.md.
- **Phân tầng backend (bắt buộc):** mỗi module gồm `api/` (tầng vào: controller/route/handler)
  → `service/` (nghiệp vụ) → `repository/` (read/write dữ liệu). Tầng trên gọi tầng dưới,
  KHÔNG gọi ngược.
- Tài liệu nằm ở `docs/`, đặc tả ở `docs/` (RFC); KHÔNG trộn tài liệu với `src/`.

## Hệ quả
- (+) Diff sạch, scope rõ, dễ thêm module mới; ranh giới tầng rõ ràng cho test/lint.
- (−) Cần kỷ luật ranh giới module + tầng ngay từ đầu (no reach-in, không gọi ngược tầng).
