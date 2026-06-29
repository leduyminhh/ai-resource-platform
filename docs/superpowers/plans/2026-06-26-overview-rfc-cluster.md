# Overview RFC Cluster — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Viết nội dung thật cho 5 RFC cụm `docs/000-overview/` (Vision, Terminology, Guiding Principles, Layered Architecture, Repository Layout), bỏ boilerplate trùng lặp và delegate qua cross-ref, theo convention chốt ở ADR-0003.

**Architecture:** Approach A (de-dup + delegate + ADR). Mỗi RFC có cấu trúc riêng + header chuẩn; quy tắc normative cross-cutting chỉ phát biểu ở RFC sở hữu, RFC overview trỏ tới. Một script shell `tools/check-rfc-links` đóng vai "test": link integrity, header conformance, de-dup, terminology coverage.

**Tech Stack:** Markdown (RFC docs), Bash (script kiểm tra). KHÔNG chốt ngôn ngữ hiện thực (Non-Goal RFC-0000 §4).

**Nguồn nội dung:** Design doc [docs/superpowers/specs/2026-06-26-overview-rfc-cluster-design.md](../specs/2026-06-26-overview-rfc-cluster-design.md); abstract hiện có của từng RFC; `project-knowledge/` (đã điền từ docs).

## Global Constraints

- Header block chuẩn cho mọi RFC (verbatim, thay `<…>`):
  ```
  # RFC-XXXX — <Title>

  **Status:** Draft
  **Category:** 000-overview
  **Specification:** AI Resource Platform Specification (ARPS)
  **Version:** 1.0.0
  **Requires:** <RFC-XXXX, …>   (bỏ dòng nếu không có)

  ---
  ```
- Status tài liệu RFC ∈ {Draft, Accepted, Final, Deprecated, Superseded} — TÁCH khỏi resource lifecycle RFC-0103.
- Cross-ref dạng link tương đối, ví dụ: `[RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)`.
- RFC có normative thật chèn dòng: *"The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119."*
- Quy tắc delegation: mỗi quy tắc cross-cutting có ĐÚNG một RFC sở hữu; overview trỏ tới, KHÔNG phát biểu lại normative.
- Mỗi RFC kết bằng `## References`.
- KHÔNG đụng: 39 RFC còn lại, `schemas/`, `examples/`, `src/`.
- An toàn: 1 task = 1 commit; dừng cho người duyệt diff; KHÔNG push master.
- Commit theo Conventional Commits, message tiếng Việt không dấu ở header; trailer:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.

---

## File Structure

| File | Trách nhiệm |
|---|---|
| `docs/decisions/0003-rfc-authoring-convention.md` | ADR chốt convention authoring (header, status vocab, delegation) |
| `tools/check-rfc-links` | Script bash kiểm tra: link integrity, header, de-dup, terminology |
| `docs/000-overview/RFC-0001-Terminology.md` | Glossary (nền vocabulary cho 4 RFC kia) |
| `docs/000-overview/RFC-0000-Vision.md` | Vision narrative + delegate RFC-0100/0200 |
| `docs/000-overview/RFC-0002-Guiding-Principles.md` | 6 nguyên tắc (statement/rationale/implications) |
| `docs/000-overview/RFC-0003-Layered-Architecture.md` | 7 tầng + dependency rule (normative, sở hữu) |
| `docs/000-overview/RFC-0004-Repository-Layout.md` | 2 cây layout + thư mục quy trình |
| `TODO.md` | Ghi forward-ref pending |

---

## Task 1: ADR-0003 — Convention authoring

**Files:**
- Create: `docs/decisions/0003-rfc-authoring-convention.md`

**Interfaces:**
- Produces: convention mà Task 3–7 tuân theo (header fields, status vocab, delegation rule, cross-ref style).

- [x] **Step 1: Viết ADR-0003** với nội dung:

```markdown
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
```

- [x] **Step 2: Commit**

```bash
git add docs/decisions/0003-rfc-authoring-convention.md
git commit -m "docs(adr): them ADR-0003 convention authoring RFC" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 2: `tools/check-rfc-links` — verification harness

**Files:**
- Create: `tools/check-rfc-links`

**Interfaces:**
- Produces: lệnh `bash tools/check-rfc-links` trả exit 0 nếu PASS, ≠0 nếu FAIL. Task 3–8 chạy để verify.

- [x] **Step 1: Viết script** (bash thuần, POSIX-ish; chạy được trên Git Bash Windows):

```bash
#!/usr/bin/env bash
# Kiem tra tinh nhat quan cum RFC 000-overview.
set -u
ROOT="$(cd "$(dirname "$0")/.." && pwd)"
OV="$ROOT/docs/000-overview"
fail=0
err() { echo "FAIL: $*"; fail=1; }

# --- Check 1: Link integrity (moi link .md tuong doi phai ton tai) ---
while IFS= read -r line; do
  file="${line%%::*}"; link="${line#*::}"
  dir="$(dirname "$file")"
  target="$(cd "$dir" 2>/dev/null && cd "$(dirname "$link")" 2>/dev/null && echo "$(pwd)/$(basename "$link")")"
  [ -f "$target" ] || err "broken link in $file -> $link"
done < <(grep -rEo '\]\([^)]+\.md[^)]*\)' "$OV" "$ROOT/docs/decisions/0003-rfc-authoring-convention.md" 2>/dev/null \
          | sed -E 's/^([^:]+):.*\(([^)#]+)(#[^)]*)?\)/\1::\2/')

# --- Check 4: Header conformance ---
for f in "$OV"/RFC-*.md; do
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$f" || err "missing header field ${field//\\/} in $(basename "$f")"
  done
done

# --- Check 2: De-dup (heading boilerplate da go khoi 0001..0004) ---
for n in 0001 0002 0003 0004; do
  f="$(ls "$OV"/RFC-$n-*.md 2>/dev/null)"
  [ -z "$f" ] && continue
  for bad in 'Runtime Flow' 'Error Model' 'Migration Guidance' 'Future Work' 'Required Behavior'; do
    grep -qE "^#+ .*$bad" "$f" && err "boilerplate heading '$bad' still in $(basename "$f")"
  done
done
# Future Work CHI duoc phep o RFC-0000
grep -qE '^#+ .*Future Work' "$OV"/RFC-0000-*.md || err "RFC-0000 should keep a Future Work/Roadmap section"

# --- Check 3: Terminology coverage (core term phai co trong RFC-0001) ---
TERM="$(ls "$OV"/RFC-0001-*.md 2>/dev/null)"
if [ -n "$TERM" ]; then
  for t in Resource Kind Manifest Metadata Registry Resolver Planner Executor Adapter Artifact Package; do
    grep -qE "\b$t\b" "$TERM" || err "term '$t' missing from RFC-0001 glossary"
  done
fi

[ "$fail" -eq 0 ] && echo "OK: all RFC overview checks passed"
exit "$fail"
```

- [x] **Step 2: Chạy script trên trạng thái HIỆN TẠI (kỳ vọng FAIL)**

Run: `bash tools/check-rfc-links`
Expected: in ra một số dòng `FAIL:` (RFC hiện còn boilerplate "Runtime Flow"/"Error Model"/"Future Work" ở 0001–0004; thiếu phần Future Work hợp lệ ở 0000 có thể vẫn pass do còn boilerplate). Exit code ≠ 0. Đây là "failing test" xác nhận script bắt được vấn đề.

- [x] **Step 3: Commit**

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them tools/check-rfc-links kiem tra cum RFC overview" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 3: RFC-0001 — Terminology

**Files:**
- Modify: `docs/000-overview/RFC-0001-Terminology.md` (thay toàn bộ nội dung)

**Interfaces:**
- Produces: glossary các term mà RFC-0000/0002/0003/0004 viết hoa khi tham chiếu.

- [x] **Step 1: Thay toàn bộ file** bằng:

```markdown
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
```

- [x] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: KHÔNG còn dòng FAIL nào nhắc RFC-0001 (terminology coverage + header pass; có thể vẫn FAIL cho 0000/0002/0003/0004 chưa làm).

- [x] **Step 3: Commit**

```bash
git add docs/000-overview/RFC-0001-Terminology.md
git commit -m "docs(rfc): viet lai RFC-0001 Terminology theo convention" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 4: RFC-0000 — Vision

**Files:**
- Modify: `docs/000-overview/RFC-0000-Vision.md` (thay toàn bộ nội dung)

**Interfaces:**
- Consumes: term viết hoa định nghĩa ở RFC-0001.
- Produces: RFC duy nhất giữ phần "Future Work / Roadmap".

- [x] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0000 — Vision

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0100, RFC-0200

---

## Abstract
Tầm nhìn dài hạn cho một nền tảng vendor-neutral, resource-oriented dành cho năng lực phát triển AI tái sử dụng.

## 1. The Problem
- Asset phát triển AI (plugin, skill, prompt, workflow…) bị khóa theo từng IDE/vendor, khó tái dùng.
- Mỗi công cụ một model riêng → không có contract chung, không tương tác chéo.

## 2. The Vision
ARPS chuẩn hóa mọi platform object thành một **Resource** chuẩn (canonical). Runtime engine thao
tác trên Resource + dependency graph, độc lập vendor. "Everything is a canonical Resource."

## 3. Goals
- Định nghĩa contract ổn định.
- Hỗ trợ migration không xâm lấn.
- Giữ nền tảng resource-oriented.
- Bảo toàn hành vi deterministic.
- Cho phép mở rộng mà không đổi kiến trúc lõi.

## 4. Non-Goals
- Không bắt buộc một ngôn ngữ lập trình cụ thể.
- Không phụ thuộc một AI assistant / IDE / vendor cụ thể.
- Không ép tái cấu trúc mã nguồn nghiệp vụ sẵn có.

## 5. The Resource Thesis
Mỗi đối tượng được biểu diễn bằng một canonical resource (apiVersion/kind/metadata/spec/status).
Chi tiết mô hình: xem [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md).

## 6. Lifecycle at a glance
Repository → Discovery → Registry → Validation → Dependency Resolver → Planning → Execution →
Packaging → Publishing → Registry/Marketplace. Chi tiết engine + ranh giới: xem
[RFC-0200](../300-runtime/RFC-0200-Runtime-Architecture.md); sơ đồ: `docs/diagrams/runtime-flow.mmd`.

## 7. Roadmap / Future Work
- Conformance tests.
- Reference runtime implementation.
- Registry interoperability suite.
- Extended JSON/YAML schema definitions.

## References
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
- [RFC-0002 — Guiding Principles](RFC-0002-Guiding-Principles.md)
```

- [x] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0000 (giữ "Future Work"; header + link pass).

- [x] **Step 3: Commit**

```bash
git add docs/000-overview/RFC-0000-Vision.md
git commit -m "docs(rfc): viet lai RFC-0000 Vision theo convention" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 5: RFC-0002 — Guiding Principles

**Files:**
- Modify: `docs/000-overview/RFC-0002-Guiding-Principles.md` (thay toàn bộ nội dung)

- [x] **Step 1: Thay toàn bộ file** bằng:

```markdown
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
```

- [x] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0002.

- [x] **Step 3: Commit**

```bash
git add docs/000-overview/RFC-0002-Guiding-Principles.md
git commit -m "docs(rfc): viet lai RFC-0002 Guiding Principles theo convention" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 6: RFC-0003 — Layered Architecture

**Files:**
- Modify: `docs/000-overview/RFC-0003-Layered-Architecture.md` (thay toàn bộ nội dung)

**Interfaces:**
- Produces: **Dependency rule** (normative) — RFC khác trỏ tới đây, không phát biểu lại.

- [x] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0003 — Layered Architecture

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0100, RFC-0200

---

## Abstract
Định nghĩa các tầng nền tảng: Specification, Metadata, Resource, Runtime, Adapter, Distribution, Governance.

The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119.

## 1. Layers
```text
Governance
Distribution
Adapter
Runtime
Resource
Metadata
Specification
```

## 2. Layer responsibilities
| Tầng | Trách nhiệm | RFC phủ | Phụ thuộc cho phép |
|---|---|---|---|
| Specification | Định nghĩa contract/đặc tả, từ vựng, nguyên tắc | RFC-0000..0007 | (đáy) |
| Metadata | Mô hình metadata: identity, ownership, discovery, compat | RFC-0102 | Specification |
| Resource | Canonical model, manifest, lifecycle, dependency graph | RFC-0100..0107 | Metadata |
| Runtime | Engine: discovery→…→packaging | RFC-0200..0209 | Resource |
| Adapter | Biến đổi resource → output đích | RFC-0500, RFC-0700 | Runtime |
| Distribution | Build, package, registry, marketplace | RFC-0400..0403 | Runtime |
| Governance | Ownership, review, policy, security | RFC-0800..0802 | tất cả tầng dưới |

## 3. Dependency rule (normative)
- Một tầng MUST chỉ phụ thuộc các tầng BÊN DƯỚI nó.
- Một tầng MUST NOT phụ thuộc (reach up) tầng bên trên.
- Đây là nguồn sự thật cho quy tắc phân tầng; RFC khác trỏ tới mục này.

## 4. Mapping tầng ↔ engine ↔ category
- Runtime ↔ các engine [RFC-0200](../300-runtime/RFC-0200-Runtime-Architecture.md).
- Distribution ↔ build/pipeline [RFC-0400](../500-build-distribution/RFC-0400-Build-Pipeline.md).
- Governance ↔ [RFC-0802](../900-security-governance/RFC-0802-Governance.md).

## References
- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
```

- [x] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0003.

- [x] **Step 3: Commit**

```bash
git add docs/000-overview/RFC-0003-Layered-Architecture.md
git commit -m "docs(rfc): viet lai RFC-0003 Layered Architecture theo convention" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 7: RFC-0004 — Repository Layout

**Files:**
- Modify: `docs/000-overview/RFC-0004-Repository-Layout.md` (thay toàn bộ nội dung)

**Interfaces:**
- Produces: quy ước đánh số file RFC (sở hữu tại đây).

- [x] **Step 1: Đối chiếu hiện trạng repo trước khi viết**

Run: `ls -1 docs && echo '---' && ls -1 && echo '---' && ls -1 src`
Mục đích: xác nhận layout thật để mô tả đúng (docs/<category>/, schemas/, examples/, reference/, tools/, src/, tests/, project-knowledge/, docs/requests, docs/decisions, docs/contracts).

- [x] **Step 2: Thay toàn bộ file** bằng (chỉnh lại nếu Step 1 lệch):

```markdown
# RFC-0004 — Repository Layout

**Status:** Draft
**Category:** 000-overview
**Specification:** AI Resource Platform Specification (ARPS)
**Version:** 1.0.0
**Requires:** RFC-0006

---

## Abstract
Định nghĩa cấu trúc repo khuyến nghị cho phần đặc tả và phần hiện thực tham chiếu.

The key words MUST, SHOULD, MAY… are to be interpreted as described in RFC 2119.

## 1. Hai cây
- **Specification tree** — đặc tả, schema, ví dụ (đọc-để-hiểu, không build).
- **Reference implementation tree** — mã nguồn hiện thực engine/validator (build/test được).

## 2. Specification layout
```text
docs/<NNN-category>/RFC-XXXX-<Title>.md   # RFC theo category đánh số
docs/diagrams/                            # sơ đồ (mmd)
schemas/                                  # JSON/YAML schema chuẩn
examples/                                 # manifest resource mẫu
reference/                                # ghi chú tham chiếu
SUMMARY.md                                # mục lục RFC
```

## 3. Reference implementation layout
```text
src/<module>/{api,service,repository}/    # module engine phân tầng
src/shared/                               # type/util/config dùng chung
tests/                                    # test
tools/                                    # tooling (vd validator, check-rfc-links)
```

## 4. Working-process directories (Cowork → Code)
```text
project-knowledge/        # kiến thức nền (đọc trước khi code)
docs/requests/<...>/      # tiến trình theo từng yêu cầu
docs/decisions/           # ADR đánh số tăng dần
docs/contracts/           # contract đã công bố (handoff)
```

## 5. RFC file numbering (normative, sở hữu tại đây)
- File RFC: `RFC-XXXX-<Title>.md`, đặt trong `docs/<NNN-category>/`.
- `XXXX` MUST duy nhất toàn repo; `<NNN-category>` là prefix nhóm (000, 100, …, 1000).
- Quy ước naming cho resource ID/kind (khác file RFC): xem [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md).

## References
- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [SUMMARY.md](../../SUMMARY.md)
```

- [x] **Step 3: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0004.

- [x] **Step 4: Commit**

```bash
git add docs/000-overview/RFC-0004-Repository-Layout.md
git commit -m "docs(rfc): viet lai RFC-0004 Repository Layout theo convention" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Task 8: Integration — TODO pending + full verify + SUMMARY check

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md` (kỳ vọng không đổi)

- [x] **Step 1: Thêm mục forward-ref pending vào `TODO.md`** (append, giữ nội dung cũ):

```markdown
## Pending delegations (forward-ref từ cụm 000-overview)
> Cross-ref đã trỏ tới các RFC sau (còn boilerplate) — hoàn thiện ở vòng Phase A sau:
- [ ] RFC-0100 Canonical Resource Model — nhà của Canonical Model, Required Behavior
- [ ] RFC-0106 Validation Model — nhà của Validation Rules + Error Model
- [ ] RFC-0200 Runtime Architecture — nhà của Runtime Flow
- [ ] RFC-0800 Security Model — nhà của Security Considerations
- [ ] RFC-0007 Compatibility Policy — nhà của Compatibility
- [ ] RFC-0901 Migration Guide — nhà của Migration Guidance
- [ ] RFC-0005 Versioning, RFC-0006 Naming, RFC-0104 Dependency Graph — rule con được trỏ tới
```

- [x] **Step 2: Kiểm tra SUMMARY.md vẫn đủ 5 RFC overview**

Run: `grep -c '000-overview/RFC-000' SUMMARY.md`
Expected: `5` (RFC-0000..0004 vẫn được liệt kê; tiêu đề không đổi → không cần sửa). Nếu ≠5, cập nhật SUMMARY.md cho khớp tiêu đề mới.

- [x] **Step 3: Chạy verify toàn cụm (kỳ vọng PASS)**

Run: `bash tools/check-rfc-links`
Expected: `OK: all RFC overview checks passed`, exit 0.

- [x] **Step 4: Commit**

```bash
git add TODO.md SUMMARY.md
git commit -m "docs: ghi pending delegations va xac nhan verify cum overview" -m "Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

## Definition of Done
- 5 RFC overview + ADR-0003 viết theo convention; `tools/check-rfc-links` PASS (exit 0).
- Forward-ref pending ghi trong `TODO.md`.
- Mỗi task 1 commit; dừng cho người duyệt diff; không push master.
