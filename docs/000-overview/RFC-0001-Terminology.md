# RFC-0001 — Terminology

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0100

---

## Abstract
Định nghĩa từ vựng dùng chung trong mọi RFC của ARPS. Mỗi thuật ngữ trỏ tới RFC sở hữu định nghĩa đầy đủ.

## 1. Conventions
Thuật ngữ đã định nghĩa được Viết Hoa khi dùng trong các RFC (vd Resource, Kind). Định nghĩa
chi tiết và normative nằm ở RFC sở hữu (cột "Owning RFC").

## 2. Glossary

### Core model
| Term | Định nghĩa ngắn | Owning RFC |
|---|---|---|
| Resource | Đơn vị chuẩn hóa cốt lõi; mọi platform object là một Resource | RFC-0100 |
| Kind | Loại resource (Plugin, Skill, Prompt, Workflow, Policy, Adapter, Package, Registry) | RFC-0100 |
| Manifest | Khai báo resource/package/registry | RFC-0105 |
| Metadata | Trường định danh/sở hữu/discovery/governance/search/compat | RFC-0102 |
| Spec | Phần đặc tả nội dung riêng theo kind | RFC-0100 |
| Status | Trạng thái resource, gồm lifecycle | RFC-0103 |

### Runtime roles
| Term | Định nghĩa ngắn | Owning RFC |
|---|---|---|
| Discovery | Khám phá resource từ fs/package/registry | RFC-0201 |
| Validation | Kiểm tra resource theo các layer | RFC-0203 |
| Registry | Lưu + lookup metadata resource đã khám phá | RFC-0202 |
| Resolver | Giải phụ thuộc, chọn version, sinh lock | RFC-0204 |
| Planner | Chuyển resolved graph thành execution plan | RFC-0205 |
| Executor | Thực thi plan, không tự resolve | RFC-0206 |

### Outputs & graph
| Term | Định nghĩa ngắn | Owning RFC |
|---|---|---|
| Artifact | Output bất biến do execution/packaging sinh ra | RFC-0207 |
| Package | Gói bất biến do platform sinh ra | RFC-0107 |
| Lock | Snapshot phụ thuộc đã resolve | RFC-0401 |
| Dependency | Cạnh phụ thuộc giữa hai resource | RFC-0104 |
| DAG | Đồ thị phụ thuộc có hướng, không chu trình | RFC-0104 |

### Distribution & integration
| Term | Định nghĩa ngắn | Owning RFC |
|---|---|---|
| Registry (distribution) | Lưu package + metadata để phân phối | RFC-0403 |
| Marketplace | Index/search/present package | RFC-0403 |
| Adapter | Biến đổi canonical resource thành output đích | RFC-0500 |

## References
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
