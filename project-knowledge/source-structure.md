# Source Structure

> Thư mục ROOT của mã nguồn: `src/` — mọi code chạy được nằm trong đây.
> NGUỒN SỰ THẬT về cây thư mục: CÂY THỰC TẾ trong `src/` > file này khi lệch.
> Quyết định chọn root + quy ước module: xem docs/decisions/0001-*.md.

## Root & quy ước module
- Root mã nguồn: `src/`.
- Mỗi module = một thư mục con TỰ CHỨA: `src/<ten-module>/`, tách theo nghiệp vụ.
- Phân tầng trong mỗi module (backend): `api/` → `service/` → `repository/`. Tầng trên gọi
  tầng dưới, KHÔNG ngược lại.
- Code dùng chung: `src/shared/`.
- Test: gom ở `tests/` (đã có sẵn ở root). Ghi rõ nếu chuyển sang đặt cạnh module.

## Cây thư mục mã nguồn

    src/
    ├── shared/                # type, util, config dùng chung
    └── <module-a>/            # <vai trò nghiệp vụ module A>
        ├── api/               # controller / route / handler (tầng vào)
        ├── service/           # nghiệp vụ
        └── repository/        # read/write dữ liệu

## Danh mục module (CẬP NHẬT mỗi khi thêm module)
> Hiện chỉ `src/shared/` tồn tại. Các module dưới đây là **ứng viên theo runtime engine của ARPS**
> (RFC-0200..0209) — CHƯA tạo; tạo khi yêu cầu cụ thể được phân tích ở `backend-analysis`.

| Module (ứng viên) | Thư mục | Trách nhiệm | RFC | Phụ thuộc |
|---|---|---|---|---|
| shared | src/shared/ | type/util/config dùng chung (canonical resource type, error model) | RFC-0100, RFC-0000 §9 | — |
| discovery | src/discovery/ | khám phá resource từ fs/package/registry | RFC-0201 | shared |
| registry | src/registry/ | lưu + lookup metadata resource | RFC-0202 | shared |
| validation | src/validation/ | validate resource/registry theo các layer | RFC-0203, RFC-0106 | shared, registry |
| resolver | src/resolver/ | resolve phụ thuộc, chọn version, sinh lock | RFC-0204, RFC-0401 | shared, registry |
| planning | src/planning/ | resolved graph → execution plan | RFC-0205 | shared, resolver |
| execution | src/execution/ | thực thi plan | RFC-0206 | shared, planning |
| packaging | src/packaging/ | sinh package bất biến (nén/ký/verify) | RFC-0107, RFC-0402 | shared, execution |

> Mỗi module backend giữ phân tầng `api/` → `service/` → `repository/`. KHÔNG tạo trước khi
> yêu cầu cụ thể được phân tích (tránh code chết).

## Quy ước đặt tên
<đặt tên thư mục/file/symbol theo convention của stack — xem code-convention.md>

## Ranh giới module (bắt buộc)
- Module CHỈ giao tiếp qua API công khai của module khác hoặc qua `src/shared/`.
- KHÔNG import trực tiếp file nội bộ của module khác (no reach-in).
- Thêm module mới: tạo `src/<ten>/` với `api/`+`service/`+`repository/`, thêm dòng vào bảng trên,
  khai báo phụ thuộc.
