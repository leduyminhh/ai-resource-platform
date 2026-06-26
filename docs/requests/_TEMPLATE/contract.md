# API Contract: <tên yêu cầu>

> Nguồn sự thật cho mock và implement: file này.
> Chuẩn chung (định dạng lỗi, phân trang, auth) tham chiếu docs/contracts/.

## Phạm vi
<những endpoint thuộc yêu cầu này>

## Endpoints
### <METHOD> <path>  — <mục đích>
- Auth: <yêu cầu token/role nếu có>
- Path params: `<tên>` (<type>) — <mô tả>
- Query params: `<tên>` (<type>, optional/required) — <mô tả>
- Request body:
```json
{ "<field>": "<type> — <ràng buộc>" }
```
- Response 200/201:
```json
{ "<field>": "<type>" }
```
- Lỗi: 400 <khi nào>, 401/403 <khi nào>, 404 <khi nào>, 409 <khi nào>

## Quy tắc validate
- <field>: <ràng buộc nghiệp vụ>

## Mock data
- File khớp endpoint trong `mocks/` (ví dụ `mocks/<resource>.json`).
- Mock phải PASS đúng schema response ở trên.
