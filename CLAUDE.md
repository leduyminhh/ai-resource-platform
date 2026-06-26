# CLAUDE.md

Baseline chung (cách hành xử, an toàn, Definition of Done, ngôn ngữ): xem [AGENTS.md](AGENTS.md).
File này lưu rule RIÊNG của dự án — điền theo project.

## Đọc trước khi code (nguyên tắc nền tảng)
Trước khi chạy bất kỳ giai đoạn nào, đọc skill nguyên tắc nền tảng: `principles`
(bản cài dạng plugin: `core:principles`). Nguyên tắc nền tảng + ranh giới tầng tài liệu
là bắt buộc cho cả Cowork lẫn Code.

## Tổng quan
AI Resource Platform Specification (ARPS) — kho đặc tả vendor-neutral, resource-oriented để xây
nền tảng phát triển AI tái sử dụng. Ý tưởng cốt lõi: mọi thứ (Plugin, Skill, Prompt, Workflow,
Policy, Adapter, Package, Registry) là một Resource chuẩn hóa; các runtime engine thao tác trên
resource + graph chứ không phụ thuộc model IDE của một vendor cụ thể.

## Tài liệu cần đọc trước khi code
- project-knowledge/  ← context nền (đọc TRƯỚC khi code)
- project-knowledge/code-convention.md  ← CHUẨN CODE BẮT BUỘC (đọc TRƯỚC khi viết code)
- docs/requests/<...>/plan.md  ← kế hoạch của yêu cầu hiện tại
- docs/decisions/  ← lý do các quyết định
- docs/ (RFC-0000..RFC-0901)  ← đặc tả nền tảng; SUMMARY.md là mục lục

## Quy ước code (RÀNG BUỘC — không phải gợi ý)
- Nguồn chuẩn duy nhất: `project-knowledge/code-convention.md`. ĐỌC TRƯỚC khi viết code.
- Mọi code sinh ra PHẢI tuân thủ convention đó: đặt tên, format, cấu trúc, xử lý lỗi, test.
- Lệch chuẩn = LỖI: phải sửa cho pass lint/format TRƯỚC khi commit; không commit code fail lint.
- Đổi convention chỉ qua ADR mới (cập nhật code-convention.md + ADR), không tự ý lệch.

## Lệnh
- Build: <lệnh>
- Test:  <lệnh>
- Lint:  <lệnh>
- Run:   <lệnh>

## RANH GIỚI AN TOÀN (Code KHÔNG được tự làm)
- Không push thẳng lên nhánh main/master.
- Không chạy migration hay lệnh phá hủy dữ liệu khi chưa được xác nhận.
- Không sửa file bí mật (.env, secrets, credentials).
- Không commit code lệch convention hoặc fail lint/format.
- Luôn dừng cho người duyệt diff trước khi commit.

## Nguồn sự thật khi tài liệu lệch nhau
- Schema thực trong code  >  project-knowledge/data-model.md
- contract.md (REST API)  >  mock / sample
- project-knowledge/code-convention.md + lint config  >  thói quen / style cá nhân
- docs/requests/<...>/plan.md  >  TODO.md

## Onboarding skill (cho người mới mở repo)
Repo này KHÔNG chứa skill workflow; skill sống ở KIT nguồn riêng và được cài theo máy.
Khi mở repo mà các skill workflow chưa khả dụng, HÃY HỎI người dùng có muốn cài từ kit không
(xem README, mục "Onboarding cho người mới clone"). KHÔNG tự cài khi chưa được đồng ý.
