# Domain Context

> Tổng hợp từ RFC-0001 (Terminology), RFC-0100/0102/0103/0104 (resource model), RFC-0000/0106
> (validation + error model). Nguồn sự thật đầy đủ: các RFC tương ứng trong `docs/`.

## Bối cảnh nghiệp vụ
ARPS chuẩn hóa cách **định nghĩa → khám phá → đăng ký → validate → resolve → plan → thực thi →
đóng gói → phân phối** các tài nguyên AI một cách độc lập vendor. Mọi tạo phẩm là một **Resource**
chuẩn hóa; runtime thao tác trên resource + dependency graph (RFC-0000, RFC-0200).

## Glossary thuật ngữ (RFC-0001)
| Thuật ngữ | Nghĩa |
|---|---|
| Resource | Đơn vị chuẩn hóa cốt lõi; mọi platform object là một Resource |
| Kind | Loại resource (Plugin, Skill, Prompt, Workflow, Policy, Adapter, Package, Registry) |
| Manifest | Khai báo resource/package/registry (RFC-0105) |
| Metadata | Trường định danh/sở hữu/discovery/governance/search/compat (RFC-0102) |
| Spec | Phần đặc tả nội dung riêng của từng kind |
| Status | Trạng thái resource, gồm `lifecycle` (RFC-0103) |
| Registry | Lưu metadata resource đã khám phá; lookup theo id/kind/version/label/checksum (RFC-0202) |
| Resolver | Giải phụ thuộc, chọn version, xử lý xung đột, sinh lock (RFC-0204) |
| Planner | Chuyển resolved graph → execution plan deterministic (RFC-0205) |
| Executor | Thực thi plan, KHÔNG tự resolve phụ thuộc (RFC-0206) |
| Adapter | Biến đổi canonical resource → output đích (RFC-0500) |
| Artifact | Output bất biến do execution/packaging sinh ra (RFC-0207) |
| Package | Gói bất biến do platform sinh ra (RFC-0107) |

## Canonical resource (RFC-0100, schemas/resource.schema.json)
```yaml
apiVersion: platform/v1
kind: ResourceKind
metadata: { id: namespace/name, name: name, version: 1.0.0, labels: {}, annotations: {} }
spec: {}
status: { lifecycle: Draft }
```
- Bắt buộc: `apiVersion`, `kind`, `metadata{id,name,version}`, `spec`. Serialize bằng YAML/JSON (RFC-0101).

## Vòng đời resource (RFC-0103)
`Draft → Review → Approved → Published → Deprecated → Archived`.

## Quy tắc & ràng buộc domain
- Resource ID **duy nhất** trong một registry (RFC-0000 §8).
- Version **SHOULD** theo Semantic Versioning (RFC-0005); breaking change ⇒ major mới (RFC-0007).
- Dependency graph **MUST** acyclic — DAG (RFC-0104).
- Read-only phase KHÔNG mutate resource nguồn (RFC-0000 §6).
- Secrets KHÔNG lưu trong manifest dạng plain (RFC-0000 §10, RFC-0800).

## Error model (RFC-0000 §9)
`SCHEMA_ERROR` · `METADATA_ERROR` · `DEPENDENCY_ERROR` · `COMPATIBILITY_ERROR` · `POLICY_VIOLATION` · `BUILD_ERROR`.

## Validation layers (RFC-0106)
syntax · schema · metadata · semantic · compatibility · policy.
