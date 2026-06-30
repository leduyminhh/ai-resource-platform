# RFC Resource-Model Completion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Viết nội dung thật cho 5 RFC còn boilerplate của cụm `docs/200-resource-model/` (RFC-0101 Serialization, RFC-0102 Metadata Model, RFC-0103 Resource Lifecycle, RFC-0105 Manifest, RFC-0107 Packaging Model), theo per-RFC ownership + delegation, đóng trọn cụm 200 (8/8).

**Architecture:** Mỗi RFC sở hữu một facet của canonical resource và delegate phần cross-cutting qua cross-ref (không định nghĩa lại). Mở rộng `tools/check-rfc-links` bằng một "resource-model guard" (Check 12) đóng vai test: header conformance, no-boilerplate, required sections, ownership key-phrases.

**Tech Stack:** Markdown (RFC docs), Bash (guard). KHÔNG chốt ngôn ngữ hiện thực.

**Nguồn nội dung:** Design doc [docs/superpowers/specs/2026-06-29-rfc-resource-model-completion-design.md](../specs/2026-06-29-rfc-resource-model-completion-design.md); style mẫu theo RFC-0106 đã hoàn thiện.

## Global Constraints

- Header block chuẩn (giữ 2 trailing-space ở 3 dòng đầu như house style RFC-0106):
  ```
  # RFC-XXXX — <Title>

  **Status:** Draft  
  **Category:** 200-resource-model  
  **Specification:** AI Resource Platform Specification (ARPS)  
  **Version:** 1.0.0

  ---
  ```
- Sau header: `## Abstract` (1 câu tóm tắt + 1 đoạn nêu RFC này sở hữu gì và delegate gì qua cross-ref), rồi `## 1. Conventions` (RFC 2119), các section đánh số `## N. <Title>`, kết bằng `## References`.
- Cross-ref link tương đối từ `docs/200-resource-model/`: cùng thư mục `RFC-0100-...md`; khác thư mục `../100-foundation/...`, `../300-runtime/...`, `../500-build-distribution/...`, `../900-security-governance/...`.
- KHÔNG đặt heading trùng boilerplate: Motivation, Goals, Non-Goals, Canonical Model, Required Behavior, Runtime Flow, Validation Rules, Error Model, Security Considerations, Migration Guidance, Future Work.
- KHÔNG đụng: RFC ngoài 5 cái này, `schemas/`, `examples/`, `src/`, các RFC đã REAL.
- Commit: Conventional Commits, header tiếng Việt KHÔNG dấu, **KHÔNG thêm trailer Co-Authored-By** (khớp convention commit cụm 2026-06-29). COMMIT BẰNG PATHSPEC TƯỜNG MINH: `git add <path> && git commit -m "..." -- <path>`; KHÔNG `git add -A`/`git add .`/`git commit -a`.
- An toàn: 1 task = 1 commit; làm trên master (repo trunk-based); KHÔNG tự push (controller/người dùng push ở cuối).

---

## File Structure

| File | Trách nhiệm |
|---|---|
| `tools/check-rfc-links` | Thêm Check 12 (resource-model guard) cho 5 RFC |
| `docs/200-resource-model/RFC-0101-Serialization.md` | YAML/JSON mapping của canonical model |
| `docs/200-resource-model/RFC-0102-Metadata-Model.md` | Ngữ nghĩa field metadata |
| `docs/200-resource-model/RFC-0103-Resource-Lifecycle.md` | Trạng thái + chuyển trạng thái |
| `docs/200-resource-model/RFC-0105-Manifest.md` | Cấu trúc manifest resource/package/registry |
| `docs/200-resource-model/RFC-0107-Packaging-Model.md` | Mô hình logic của package bất biến |

---

## Task 1: Resource-model validator guard (Check 12)

**Files:**
- Modify: `tools/check-rfc-links` (chèn Check 12 ngay TRƯỚC 2 dòng cuối `[ "$fail" -eq 0 ] && echo ...` / `exit "$fail"`)

**Interfaces:**
- Consumes: helper `check_foundation_header` và `check_no_foundation_boilerplate` đã định nghĩa ở Check 11 (cùng file, đã trong scope).
- Produces: guard FAIL khi 5 RFC còn boilerplate; PASS khi cả 5 viết xong.

- [ ] **Step 1: Chèn khối Check 12** vào `tools/check-rfc-links`, ngay trước dòng `[ "$fail" -eq 0 ] && echo "OK: all RFC overview checks passed"`:

```bash
# --- Check 12: Resource-model completion (RFC-0101/0102/0103/0105/0107) ---
SERIALIZATION="$ROOT/docs/200-resource-model/RFC-0101-Serialization.md"
METADATA="$ROOT/docs/200-resource-model/RFC-0102-Metadata-Model.md"
LIFECYCLE="$ROOT/docs/200-resource-model/RFC-0103-Resource-Lifecycle.md"
MANIFEST="$ROOT/docs/200-resource-model/RFC-0105-Manifest.md"
PACKAGING="$ROOT/docs/200-resource-model/RFC-0107-Packaging-Model.md"

if [ -f "$SERIALIZATION" ]; then
  check_foundation_header "$SERIALIZATION" "RFC-0101"
  check_no_foundation_boilerplate "$SERIALIZATION" "RFC-0101"
  for required in 'Format Policy' 'Encoding' 'Serialization Rules' 'Round-trip' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$SERIALIZATION" || err "missing RFC-0101 section '$required'"
  done
  grep -q 'parsed canonical' "$SERIALIZATION" || err "RFC-0101 must require engines operate on parsed canonical objects"
  grep -q 'YAML and JSON' "$SERIALIZATION" || err "RFC-0101 must define YAML and JSON mappings"
fi

if [ -f "$METADATA" ]; then
  check_foundation_header "$METADATA" "RFC-0102"
  check_no_foundation_boilerplate "$METADATA" "RFC-0102"
  for required in 'Metadata Fields' 'Identity' 'Discovery and Governance' 'Compatibility Fields' 'Extensibility' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$METADATA" || err "missing RFC-0102 section '$required'"
  done
  grep -q 'metadata.id' "$METADATA" || err "RFC-0102 must define metadata.id"
  grep -q 'RFC-0006' "$METADATA" || err "RFC-0102 must delegate identifier grammar to RFC-0006"
  grep -q 'RFC-0005' "$METADATA" || err "RFC-0102 must delegate version format to RFC-0005"
fi

if [ -f "$LIFECYCLE" ]; then
  check_foundation_header "$LIFECYCLE" "RFC-0103"
  check_no_foundation_boilerplate "$LIFECYCLE" "RFC-0103"
  for required in 'Lifecycle States' 'State Transitions' 'Transition Rules' 'Status Field' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$LIFECYCLE" || err "missing RFC-0103 section '$required'"
  done
  for state in Draft Review Approved Published Deprecated Archived; do
    grep -q "$state" "$LIFECYCLE" || err "missing RFC-0103 lifecycle state '$state'"
  done
  grep -q 'RFC-0802' "$LIFECYCLE" || err "RFC-0103 must delegate governance to RFC-0802"
fi

if [ -f "$MANIFEST" ]; then
  check_foundation_header "$MANIFEST" "RFC-0105"
  check_no_foundation_boilerplate "$MANIFEST" "RFC-0105"
  for required in 'Manifest Purpose' 'Resource Manifest' 'Package Manifest' 'Registry Manifest' 'Manifest Rules' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$MANIFEST" || err "missing RFC-0105 section '$required'"
  done
  grep -q 'metadata.id' "$MANIFEST" || err "RFC-0105 must reference resources by metadata.id"
  grep -q 'RFC-0100' "$MANIFEST" || err "RFC-0105 must delegate canonical envelope to RFC-0100"
fi

if [ -f "$PACKAGING" ]; then
  check_foundation_header "$PACKAGING" "RFC-0107"
  check_no_foundation_boilerplate "$PACKAGING" "RFC-0107"
  for required in 'Package Concept' 'Package Contents' 'Package Identity' 'Immutability' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$PACKAGING" || err "missing RFC-0107 section '$required'"
  done
  grep -q 'immutable' "$PACKAGING" || err "RFC-0107 must state packages are immutable"
  grep -q 'RFC-0402' "$PACKAGING" || err "RFC-0107 must delegate packaging mechanics to RFC-0402"
  grep -q 'RFC-0801' "$PACKAGING" || err "RFC-0107 must delegate signing/integrity to RFC-0801"
fi
```

- [ ] **Step 2: Chạy guard trên trạng thái HIỆN TẠI (kỳ vọng FAIL)**

Run: `bash tools/check-rfc-links`
Expected: nhiều dòng `FAIL:` cho RFC-0101/0102/0103/0105/0107 (boilerplate heading còn đó + thiếu required section + thiếu key-phrase). Exit code ≠ 0. Đây là fail-first evidence.

- [ ] **Step 3: Commit**

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho cum RFC resource-model" -- tools/check-rfc-links
```

---

## Task 2: RFC-0101 — Serialization

**Files:**
- Modify: `docs/200-resource-model/RFC-0101-Serialization.md` (thay TOÀN BỘ nội dung)

**Interfaces:**
- Produces: RFC-0101 REAL; guard hết FAIL cho RFC-0101.

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0101 — Serialization

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines YAML and JSON as serialization mappings of the canonical resource model.

This RFC owns serialization format policy, encoding and round-trip rules. The resource envelope and top-level fields are defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); metadata field semantics by [RFC-0102](RFC-0102-Metadata-Model.md); validation of serialized documents by [RFC-0106](RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Format Policy

YAML and JSON are supported serialization mappings of the same canonical object model. The canonical object model is the source of truth; YAML and JSON are interchangeable encodings of it.

- A resource MAY be authored in YAML or JSON.
- Both encodings MUST represent the same canonical object for the same resource.
- Runtime engines MUST operate on parsed canonical objects, not raw YAML or JSON text.

## 3. Encoding

- Serialized documents MUST be UTF-8 encoded.
- Map keys MUST be strings; map key ordering is not semantically significant.
- Documents SHOULD NOT rely on encoding-specific escapes that do not survive a round-trip.

## 4. Serialization Rules

- Serialization SHOULD be deterministic: identical canonical objects SHOULD produce byte-identical output under the same serializer settings.
- Tools SHOULD emit map keys in a stable order to support reproducible diffs and checksums.
- Empty optional fields SHOULD be omitted rather than serialized as null, unless null is semantically meaningful.
- Scalar types (string, number, boolean, null) MUST be preserved across YAML and JSON.

## 5. Round-trip

- Parsing a serialized document and re-serializing it MUST preserve the canonical object (round-trip fidelity).
- Converting YAML to JSON and back MUST preserve the canonical object.
- A round-trip MUST NOT silently drop unknown fields permitted by the active schema policy.

## 6. Examples

### 6.1 YAML

```yaml
apiVersion: platform/v1
kind: Skill
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.0.0
spec:
  dependencies: []
status:
  lifecycle: Published
```

### 6.2 Equivalent JSON

```json
{
  "apiVersion": "platform/v1",
  "kind": "Skill",
  "metadata": { "id": "core/clean-code", "name": "clean-code", "version": "1.0.0" },
  "spec": { "dependencies": [] },
  "status": { "lifecycle": "Published" }
}
```

## References

- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0102 — Metadata Model](RFC-0102-Metadata-Model.md)
- [RFC-0106 — Validation Model](RFC-0106-Validation-Model.md)
```

- [ ] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn dòng FAIL nhắc RFC-0101; vẫn FAIL cho 0102/0103/0105/0107.

- [ ] **Step 3: Commit**

```bash
git add docs/200-resource-model/RFC-0101-Serialization.md
git commit -m "docs(rfc): viet lai RFC-0101 Serialization" -- docs/200-resource-model/RFC-0101-Serialization.md
```

---

## Task 3: RFC-0102 — Metadata Model

**Files:**
- Modify: `docs/200-resource-model/RFC-0102-Metadata-Model.md` (thay TOÀN BỘ nội dung)

**Interfaces:**
- Produces: RFC-0102 REAL; guard hết FAIL cho RFC-0102.

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0102 — Metadata Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines metadata fields used for identity, ownership, discovery, governance, search and compatibility.

This RFC owns metadata field semantics. The canonical envelope that contains metadata is defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); identifier grammar by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); version format by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); compatibility classification by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md); lifecycle state by [RFC-0103](RFC-0103-Resource-Lifecycle.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Metadata Fields

| Field | Required | Semantics |
|---|---|---|
| `id` | Yes | Stable resource identity in `namespace/name` form. |
| `name` | Yes | Short human-readable name. |
| `version` | Yes | Resource version. |
| `owner` | No | Owning team or person. |
| `labels` | No | Key/value pairs for selection and grouping. |
| `annotations` | No | Non-identifying auxiliary metadata. |

## 3. Identity

- `metadata.id` MUST be present and stable within a registry namespace.
- Identifier grammar (namespace/name form, case and characters) is defined by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md).
- `metadata.version` format is defined by [RFC-0005](../100-foundation/RFC-0005-Versioning.md).
- `metadata.name` is a display name and MUST NOT be used as resource identity.

## 4. Discovery and Governance

- `labels` SHOULD be used for discovery, selection and grouping.
- `owner` SHOULD identify the party responsible for governance.
- `annotations` MAY carry tooling, audit or provenance information that does not affect identity.

## 5. Compatibility Fields

- Compatibility classification of metadata changes is defined by [RFC-0007](../100-foundation/RFC-0007-Compatibility-Policy.md).
- Changing `metadata.id` is an identity change and is compatibility-sensitive.
- Adding optional labels or annotations is a compatible change.

## 6. Extensibility

- `labels` and `annotations` are open string maps.
- Unknown metadata fields MUST follow the active schema policy.
- Unknown labels and annotations SHOULD be ignored by tools that do not understand them.

## 7. Examples

```yaml
metadata:
  id: core/clean-code
  name: clean-code
  version: 1.2.0
  owner: platform-team
  labels:
    tier: core
  annotations:
    source: legacy-import
```

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0007 — Compatibility Policy](../100-foundation/RFC-0007-Compatibility-Policy.md)
- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0103 — Resource Lifecycle](RFC-0103-Resource-Lifecycle.md)
```

- [ ] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0101/0102; vẫn FAIL cho 0103/0105/0107.

- [ ] **Step 3: Commit**

```bash
git add docs/200-resource-model/RFC-0102-Metadata-Model.md
git commit -m "docs(rfc): viet lai RFC-0102 Metadata Model" -- docs/200-resource-model/RFC-0102-Metadata-Model.md
```

---

## Task 4: RFC-0103 — Resource Lifecycle

**Files:**
- Modify: `docs/200-resource-model/RFC-0103-Resource-Lifecycle.md` (thay TOÀN BỘ nội dung)

**Interfaces:**
- Produces: RFC-0103 REAL; guard hết FAIL cho RFC-0103.

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0103 — Resource Lifecycle

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines lifecycle states for all resources: Draft, Review, Approved, Published, Deprecated and Archived.

This RFC owns lifecycle states and valid transitions. The `status` field location on the envelope is defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); governance and approval processes are defined by [RFC-0802](../900-security-governance/RFC-0802-Governance.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Lifecycle States

| State | Meaning |
|---|---|
| `Draft` | Authored, not yet reviewed; unstable. |
| `Review` | Submitted for review; under evaluation. |
| `Approved` | Accepted but not yet published to consumers. |
| `Published` | Available for resolution and use by dependents. |
| `Deprecated` | Discouraged; still resolvable for existing dependents. |
| `Archived` | Retired; SHOULD NOT be used by new dependents. |

## 3. State Transitions

```text
Draft -> Review -> Approved -> Published -> Deprecated -> Archived
```

The normal path is `Draft → Review → Approved → Published → Deprecated → Archived`.

## 4. Transition Rules

- Transitions are directed; a resource MUST NOT jump from `Draft` directly to `Published`.
- A resource MAY return from `Review` to `Draft` for rework.
- `Deprecated` resources MAY remain resolvable to avoid breaking existing dependents.
- `Archived` resources SHOULD NOT be selected by new dependents.
- Transition gates (who may approve/publish) are owned by [RFC-0802](../900-security-governance/RFC-0802-Governance.md).

## 5. Status Field

- The lifecycle state is carried in `status.lifecycle` on the canonical envelope ([RFC-0100](RFC-0100-Canonical-Resource-Model.md)).
- `status.lifecycle` SHOULD be generated by tooling and MUST NOT define desired configuration.
- Authors MUST NOT use `status.lifecycle` to request a transition; transitions are governed actions.

## 6. Examples

```yaml
status:
  lifecycle: Published
```

```yaml
status:
  lifecycle: Deprecated
```

## References

- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0802 — Governance](../900-security-governance/RFC-0802-Governance.md)
```

- [ ] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0101/0102/0103; vẫn FAIL cho 0105/0107.

- [ ] **Step 3: Commit**

```bash
git add docs/200-resource-model/RFC-0103-Resource-Lifecycle.md
git commit -m "docs(rfc): viet lai RFC-0103 Resource Lifecycle" -- docs/200-resource-model/RFC-0103-Resource-Lifecycle.md
```

---

## Task 5: RFC-0105 — Manifest

**Files:**
- Modify: `docs/200-resource-model/RFC-0105-Manifest.md` (thay TOÀN BỘ nội dung)

**Interfaces:**
- Produces: RFC-0105 REAL; guard hết FAIL cho RFC-0105.

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0105 — Manifest

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines manifest structure for resources, packages and registries.

This RFC owns manifest structure: how resources are declared and collected. The canonical resource envelope is defined by [RFC-0100](RFC-0100-Canonical-Resource-Model.md); the logical package model by [RFC-0107](RFC-0107-Packaging-Model.md) and packaging mechanics by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); registry mechanics by [RFC-0202](../300-runtime/RFC-0202-Registry-Engine.md) and [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Manifest Purpose

A manifest declares or collects resources by `metadata.id` and version so tooling can discover, group and distribute them.

- A manifest MUST reference resources by `metadata.id`.
- A manifest MUST NOT redefine the canonical resource envelope (owned by [RFC-0100](RFC-0100-Canonical-Resource-Model.md)).
- A manifest is itself a canonical resource and follows the envelope rules.

## 3. Resource Manifest

A resource manifest lists resources that belong together.

```yaml
apiVersion: platform/v1
kind: Manifest
metadata:
  id: core/manifest
  name: core
  version: 1.0.0
spec:
  resources:
    - id: core/clean-code
      version: ^1.0.0
```

## 4. Package Manifest

A package manifest declares the resources contained in a package. The package format and mechanics are owned by [RFC-0107](RFC-0107-Packaging-Model.md) and [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md).

- A package manifest MUST list contained resources by `metadata.id` and resolved version.

## 5. Registry Manifest

A registry manifest declares which packages or resources a registry exposes. Registry mechanics are owned by [RFC-0202](../300-runtime/RFC-0202-Registry-Engine.md) and [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).

## 6. Manifest Rules

- Referenced resource IDs MUST be unique within a manifest.
- A manifest SHOULD declare version ranges for resource references; range resolution is owned by [RFC-0204](../300-runtime/RFC-0204-Dependency-Resolver.md).
- A manifest MUST NOT embed full resource bodies in place of references unless the format explicitly allows inlining.

## 7. Examples

```yaml
apiVersion: platform/v1
kind: Manifest
metadata:
  id: plugins/backend-manifest
  name: backend-manifest
  version: 1.0.0
spec:
  resources:
    - id: plugins/backend
      version: ^1.0.0
    - id: core/clean-code
      version: ^1.0.0
```

## References

- [RFC-0100 — Canonical Resource Model](RFC-0100-Canonical-Resource-Model.md)
- [RFC-0107 — Packaging Model](RFC-0107-Packaging-Model.md)
- [RFC-0202 — Registry Engine](../300-runtime/RFC-0202-Registry-Engine.md)
- [RFC-0204 — Dependency Resolver](../300-runtime/RFC-0204-Dependency-Resolver.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
```

- [ ] **Step 2: Chạy verify**

Run: `bash tools/check-rfc-links`
Expected: không còn FAIL nhắc RFC-0101/0102/0103/0105; vẫn FAIL cho 0107.

- [ ] **Step 3: Commit**

```bash
git add docs/200-resource-model/RFC-0105-Manifest.md
git commit -m "docs(rfc): viet lai RFC-0105 Manifest" -- docs/200-resource-model/RFC-0105-Manifest.md
```

---

## Task 6: RFC-0107 — Packaging Model + full verify

**Files:**
- Modify: `docs/200-resource-model/RFC-0107-Packaging-Model.md` (thay TOÀN BỘ nội dung)

**Interfaces:**
- Produces: RFC-0107 REAL; toàn bộ `check-rfc-links` PASS (exit 0); cụm 200 REAL 8/8.

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0107 — Packaging Model

**Status:** Draft  
**Category:** 200-resource-model  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the logical model of immutable packages produced by the platform.

This RFC owns the logical package model: what a package is, what it contains and how it is identified. Packaging mechanics (compression, signing, verification) are defined by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); publishing by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md); the artifact relationship by [RFC-0207](../300-runtime/RFC-0207-Artifact-Model.md); signing and integrity by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Package Concept

A package is an immutable, distributable unit that bundles one or more canonical resources together with a manifest describing its contents.

- A package is produced by the build pipeline and is not edited in place.
- A package is described by a package manifest ([RFC-0105](RFC-0105-Manifest.md)).

## 3. Package Contents

- A package MUST contain a package manifest listing the resources it bundles by `metadata.id` and resolved version.
- A package MAY contain resolved dependency snapshots; lock semantics are owned by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md).
- A package MUST NOT contain plaintext secrets.

## 4. Package Identity

- A package is identified by name, version and content checksum.
- The checksum and signing mechanics are defined by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).
- Two packages with the same name and version MUST have identical content (same checksum).

## 5. Immutability

- Packages are immutable once produced; any change MUST produce a new version.
- Republishing the same name and version with different content MUST be rejected.
- Compression, signing and verification mechanics are owned by [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md).

## 6. Examples

```yaml
package:
  name: plugins/backend
  version: 1.0.0
  checksum: sha256:0d1f...c3
  contents:
    - id: plugins/backend
      version: 1.0.0
    - id: core/clean-code
      version: 1.0.0
```

## References

- [RFC-0105 — Manifest](RFC-0105-Manifest.md)
- [RFC-0207 — Artifact Model](../300-runtime/RFC-0207-Artifact-Model.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
- [RFC-0801 — Signing and Integrity](../900-security-governance/RFC-0801-Signing-and-Integrity.md)
```

- [ ] **Step 2: Chạy full verify (kỳ vọng PASS)**

Run: `bash tools/check-rfc-links`
Expected: `OK: all RFC overview checks passed`, exit 0.

- [ ] **Step 3: Xác nhận SUMMARY.md không cần đổi**

Run: `grep -c '200-resource-model/RFC-01' SUMMARY.md`
Expected: `8` (cả 8 RFC cụm 200 vẫn được liệt kê; tiêu đề không đổi → không sửa SUMMARY). Nếu ≠ 8, cập nhật SUMMARY.md cho khớp tiêu đề.

- [ ] **Step 4: Kiểm TODO pending delegation**

Run: `grep -nE '0101|0102|0103|0105|0107' TODO.md || echo "khong co entry pending cho cum 200"`
- Nếu CÓ entry `- [ ]` cho các RFC này: đổi sang `- [x]` và thêm vào pathspec commit ở Step 5.
- Nếu KHÔNG có: bỏ qua (cụm 200 là target của forward-ref từ RFC-0100, không có entry pending riêng).

- [ ] **Step 5: Commit**

```bash
git add docs/200-resource-model/RFC-0107-Packaging-Model.md
git commit -m "docs(rfc): viet lai RFC-0107 Packaging Model" -- docs/200-resource-model/RFC-0107-Packaging-Model.md
```

---

## Definition of Done

- RFC-0101/0102/0103/0105/0107 hết boilerplate + đủ required sections + tôn trọng ranh giới ownership.
- `tools/check-rfc-links` PASS (exit 0).
- `SUMMARY.md` không đổi (hoặc đã đồng bộ nếu lệch).
- Cụm `200-resource-model` REAL trọn vẹn (8/8).
- Mỗi task 1 commit (pathspec tường minh, không trailer); chưa push — chờ duyệt nhánh/đẩy ở cuối.
