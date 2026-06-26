# Design — Hoàn thiện cụm RFC `000-overview`

- **Ngày:** 2026-06-26
- **Phạm vi:** Phase A (hoàn thiện đặc tả), sub-project #1 — cụm `docs/000-overview/`
- **Approach:** A — De-dup + delegate + ADR convention
- **Trạng thái:** Draft (chờ review)

## Bối cảnh

ARPS hiện là spec-only. Toàn bộ 44 RFC trong `docs/` dùng **chung một body boilerplate**
(mục §2–§14 giống hệt nhau); nội dung RIÊNG của mỗi RFC chỉ nằm ở `## 1. Abstract`. Hệ quả:
nội dung normative (Error Model, Validation Rules, Security…) bị **nhân bản** khắp mọi RFC.

Mục tiêu tổng (do người dùng chốt): **hoàn thiện đặc tả RFC trước (Phase A), rồi hiện thực
loader + validator (Phase B)**. Sub-project đầu tiên của Phase A là cụm `000-overview` gồm 5 RFC:
Vision, Terminology, Guiding Principles, Layered Architecture, Repository Layout.

Quyết định brainstorming:
- Cấu trúc: **riêng theo từng RFC** (bỏ boilerplate lặp), giữ header chuẩn + Abstract.
- Phạm vi vòng này: **viết đầy đủ cả 5 RFC overview**.
- Approach: **A** — gỡ normative trùng lặp khỏi overview, delegate qua cross-ref, chốt convention bằng ADR.

## Mục tiêu & Non-Goals

**Mục tiêu**
- Viết nội dung thật cho 5 RFC overview theo cấu trúc riêng từng RFC.
- Lập một convention authoring nhất quán (ADR-0003) áp cho mọi RFC về sau.
- Gỡ tận gốc trùng lặp normative trong cụm overview qua quy tắc delegation.

**Non-Goals**
- KHÔNG sửa 39 RFC còn lại (các vòng Phase A sau).
- KHÔNG viết code hiện thực (Phase B).
- KHÔNG đổi `schemas/`, `examples/`, `src/`.
- KHÔNG chốt stack/ngôn ngữ (vẫn là Non-Goal theo RFC-0000 §4).

## Phần 1 — Convention authoring (→ ADR-0003)

**a) Header block chuẩn** (markdown, giữ phong cách hiện tại):
```
# RFC-XXXX — <Title>

**Status:** Draft            (Draft → Accepted → Final; hoặc Deprecated / Superseded)
**Category:** <NNN-tên>
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0           (SemVer của RIÊNG tài liệu RFC này)
**Requires:** RFC-XXXX, …    (cross-ref RFC phụ thuộc; bỏ nếu không có)
**Supersedes:** RFC-ZZZZ     (tùy chọn)

---
```

**b) Status tài liệu RFC** — TÁCH BẠCH với lifecycle resource (RFC-0103):
- RFC document: `Draft / Accepted / Final / Deprecated / Superseded`.
- RFC-0103 `Draft→Review→Approved→Published→Deprecated→Archived` là cho **resource**, không phải văn bản RFC.

**c) Ngôn ngữ normative** — RFC 2119. RFC nào có normative thật chèn 1 dòng chuẩn:
*"The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119."*
Cụm overview phần lớn informational; chỉ đặt MUST/SHOULD ở chỗ thực sự ràng buộc.

**d) Quy tắc delegation (cốt lõi)** — mỗi quy tắc cross-cutting có **đúng một RFC sở hữu**;
RFC khác *trỏ tới*, KHÔNG phát biểu lại dạng normative. Cross-ref dạng link tương đối:
`[RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)`.

**e) Kết RFC** bằng `## References` (liệt kê cross-ref) + tùy chọn `## Changelog`.

## Phần 2 — Cấu trúc & nội dung riêng từng RFC

### RFC-0000 — Vision (informational)
1. The Problem — vendor lock-in, asset AI không tái dùng, mỗi IDE một model riêng
2. The Vision — nền tảng vendor-neutral, resource-oriented; "everything is a canonical Resource"
3. Goals — 4. Non-Goals
5. The Resource Thesis — narrative, delegate chi tiết → RFC-0100
6. Lifecycle at a glance — luồng Repository→Discovery→…→Marketplace, delegate → RFC-0200
7. Roadmap / Future Work — References

### RFC-0001 — Terminology (definitional)
1. Conventions (viết hoa thuật ngữ đã định nghĩa)
2. Glossary theo nhóm, mỗi mục `term → definition → owning RFC`:
   - Core model: Resource, Kind, Manifest, Metadata, Spec, Status
   - Runtime roles: Discovery, Validation, Registry, Resolver, Planner, Executor
   - Outputs: Artifact, Package, Lock
   - Graph: Dependency, DAG
   - Distribution: Registry, Marketplace
   - Integration: Adapter
3. References

### RFC-0002 — Guiding Principles (informational + vài normative)
- Mỗi nguyên tắc = `Statement → Rationale → Implications`:
  resource-first · schema-first · deterministic builds · vendor neutrality · non-invasive migration · extensibility
- "Resolving tensions" (vd extensibility ⇄ determinism) — References

### RFC-0003 — Layered Architecture (normative về phân tầng)
1. Sơ đồ 7 tầng: Specification · Metadata · Resource · Runtime · Adapter · Distribution · Governance
2. Mỗi tầng: trách nhiệm · cái gì nằm trong · RFC nào phủ · phụ thuộc cho phép
3. **Dependency rule** (tầng chỉ phụ thuộc xuống, MUST NOT reach up) — normative, **sở hữu tại đây**
4. Ánh xạ tầng ↔ runtime engine ↔ category RFC — References

### RFC-0004 — Repository Layout (phải khớp repo thật)
1. Hai cây: *specification* vs *reference implementation*
2. Spec layout: `docs/<NNN-category>/RFC-*.md`, `schemas/`, `examples/`, `reference/`, `SUMMARY.md`
3. Impl layout: `src/` (module phân tầng api→service→repository), `tests/`, `tools/`
4. Thư mục quy trình Cowork→Code: `project-knowledge/`, `docs/requests/`, `docs/decisions/`, `docs/contracts/` — đối chiếu hiện trạng
5. Quy ước đánh số file RFC (sở hữu tại đây); naming resource delegate → RFC-0006 — References

## Phần 3 — Bản đồ sở hữu nội dung (de-dup)

| Khối boilerplate hiện tại | RFC sở hữu (canonical home) | Hành động trong overview |
|---|---|---|
| Canonical Model (§5) | RFC-0100 | RFC-0000 narrative + cross-ref; gỡ khỏi 0001–0004 |
| Required Behavior (§6) | RFC-0100 / RFC-0200 | Gỡ, cross-ref |
| Runtime Flow (§7) | RFC-0200 (+ `docs/diagrams/runtime-flow.mmd`) | RFC-0000 vision-level + cross-ref; gỡ chỗ khác |
| Validation Rules (§8) | RFC-0106; con: id-unique→RFC-0202, SemVer→RFC-0005, DAG→RFC-0104 | Gỡ, cross-ref |
| Error Model (§9) | RFC-0106 (taxonomy lỗi; phạm vi chính thức + mã build/policy/compat chốt khi viết RFC-0106, có thể liên đới RFC-0007/0400/0802) | Gỡ khỏi overview, cross-ref |
| Security (§10) | RFC-0800 | Gỡ, cross-ref |
| Compatibility (§11) | RFC-0007 | Gỡ, cross-ref |
| Example (§12) | `examples/` | Gỡ; link tới example thật |
| Migration Guidance (§13) | RFC-0901 | Gỡ, cross-ref |
| Future Work (§14) | — | Chỉ giữ ở RFC-0000 (Roadmap); gỡ khỏi 0001–0004 |

**Hệ quả — forward-reference:** hoàn thiện overview tạo cross-ref tới các RFC còn boilerplate
(0100, 0106, 0200, 0800…). Chấp nhận được: RFC đích đã tồn tại, sẽ hoàn thiện ở vòng sau.
Cross-ref theo số + tiêu đề; ghi các forward-ref pending vào `TODO.md`.

## Phần 4 — Kiểm chứng (doc-level, deterministic)

Script shell thuần `tools/check-rfc-links` (không chốt stack sớm):
1. **Link integrity** — mọi cross-ref `RFC-XXXX` và link `.md` tương đối resolve tới file có thật.
2. **De-dup check** — grep `RFC-0001..0004` xác nhận heading boilerplate đã gỡ (`## 9. Error Model`,
   `## 7. Runtime Flow`, `Migration Guidance`, `Future Work`…); `Future Work` chỉ còn ở RFC-0000.
3. **Terminology coverage** — term viết-hoa-đã-định-nghĩa dùng ở 0000/0002/0003/0004 có mục trong glossary RFC-0001 (spot-check core terms).
4. **Header conformance** — mỗi RFC đủ trường header chuẩn.
5. **SUMMARY.md** — vẫn liệt kê đủ 5 RFC (kỳ vọng không đổi).

## Definition of Done

- 5 RFC overview + ADR-0003 viết xong theo convention Phần 1.
- Check 1–5 PASS.
- Forward-ref pending đã ghi `TODO.md`.
- Dừng cho người duyệt diff trước khi commit (ranh giới an toàn CLAUDE.md/AGENTS.md).

## Sản phẩm bàn giao

| File | Loại |
|---|---|
| `docs/000-overview/RFC-0000-Vision.md` | viết lại |
| `docs/000-overview/RFC-0001-Terminology.md` | viết lại |
| `docs/000-overview/RFC-0002-Guiding-Principles.md` | viết lại |
| `docs/000-overview/RFC-0003-Layered-Architecture.md` | viết lại |
| `docs/000-overview/RFC-0004-Repository-Layout.md` | viết lại |
| `docs/decisions/0003-rfc-authoring-convention.md` | mới (ADR) |
| `tools/check-rfc-links` | mới (shell) |
| `TODO.md` | sửa (pending delegations) |
| `SUMMARY.md` | kiểm tra (kỳ vọng không đổi) |

**Không đụng:** 39 RFC còn lại, `schemas/`, `examples/`, `src/`.

## Phase tiếp theo (ngoài phạm vi vòng này)
- Các vòng Phase A kế: hoàn thiện 100-foundation → 200-resource-model → 300-runtime…
- Phase B: hiện thực loader đọc resource + validator theo schema.
