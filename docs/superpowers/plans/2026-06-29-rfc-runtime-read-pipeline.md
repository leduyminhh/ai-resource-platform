# RFC Runtime Read-Pipeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Viết nội dung thật cho 4 RFC nửa đọc/resolve của cụm `docs/300-runtime/` (RFC-0201 Discovery, RFC-0202 Registry, RFC-0203 Validation Engine, RFC-0204 Dependency Resolver), theo per-RFC ownership + delegation, neo bởi khung RFC-0200.

**Architecture:** Mỗi engine sở hữu hành vi của nó, tham chiếu đúng handoff artifact của RFC-0200 (ResourceCandidate/CanonicalResourceSet/ValidationResult/DependencyGraph) và delegate hợp đồng cross-cutting (0100/0102/0104/0106/0403/0401) qua cross-ref. Mở rộng `tools/check-rfc-links` bằng "Check 13" làm test.

**Tech Stack:** Markdown (RFC docs), Bash (guard).

**Nguồn nội dung:** Design doc [docs/superpowers/specs/2026-06-29-rfc-runtime-read-pipeline-design.md](../specs/2026-06-29-rfc-runtime-read-pipeline-design.md); style mẫu theo RFC-0106/RFC-0200 đã hoàn thiện.

## Global Constraints

- Header block chuẩn (2 trailing-space ở 3 dòng đầu):
  ```
  # RFC-XXXX — <Title>

  **Status:** Draft  
  **Category:** 300-runtime  
  **Specification:** AI Resource Platform Specification (ARPS)  
  **Version:** 1.0.0

  ---
  ```
- Sau header: `## Abstract` (1 câu + 1 đoạn ownership/delegation), `## 1. Conventions` (RFC 2119), section đánh số `## N.`, kết `## References`.
- Cross-ref tương đối từ `docs/300-runtime/`: cùng thư mục `RFC-0200..0204-...md`; `../200-resource-model/...`, `../100-foundation/...`, `../500-build-distribution/...`.
- KHÔNG đặt heading trùng boilerplate: Motivation, Goals, Non-Goals, Canonical Model, Required Behavior, Runtime Flow, Validation Rules, Error Model, Security Considerations, Migration Guidance, Future Work.
- KHÔNG đụng: RFC ngoài 4 cái này, `schemas/`, `examples/`, `src/`, RFC đã REAL.
- Commit: Conventional Commits, header tiếng Việt KHÔNG dấu, **KHÔNG trailer Co-Authored-By**. PATHSPEC TƯỜNG MINH: `git add <path> && git commit -m "..." -- <path>`; KHÔNG `git add -A`/`.`/`commit -a`.
- An toàn: 1 task = 1 commit; trên master (trunk-based); KHÔNG tự push.

---

## File Structure

| File | Trách nhiệm |
|---|---|
| `tools/check-rfc-links` | Thêm Check 13 (read-pipeline guard) cho 4 RFC |
| `docs/300-runtime/RFC-0201-Discovery-Engine.md` | Discovery: nguồn + ResourceCandidate + precedence |
| `docs/300-runtime/RFC-0202-Registry-Engine.md` | Runtime registry: store + lookup |
| `docs/300-runtime/RFC-0203-Validation-Engine.md` | Thực thi validation model |
| `docs/300-runtime/RFC-0204-Dependency-Resolver.md` | Resolve + version select + lock |

---

## Task 1: Read-pipeline validator guard (Check 13)

**Files:**
- Modify: `tools/check-rfc-links` (chèn Check 13 ngay TRƯỚC 2 dòng cuối `[ "$fail" -eq 0 ] && echo ...` / `exit "$fail"`)

**Interfaces:**
- Consumes: helper `check_foundation_header`, `check_no_foundation_boilerplate` (đã có ở Check 11, trong scope).
- Produces: guard FAIL khi 4 RFC còn boilerplate; PASS khi viết xong.

- [ ] **Step 1: Chèn khối Check 13** vào `tools/check-rfc-links`, ngay trước dòng `[ "$fail" -eq 0 ] && echo "OK: all RFC overview checks passed"`:

```bash
# --- Check 13: Runtime read-pipeline (RFC-0201/0202/0203/0204) ---
DISCOVERY="$ROOT/docs/300-runtime/RFC-0201-Discovery-Engine.md"
REGISTRY="$ROOT/docs/300-runtime/RFC-0202-Registry-Engine.md"
VAL_ENGINE="$ROOT/docs/300-runtime/RFC-0203-Validation-Engine.md"
RESOLVER="$ROOT/docs/300-runtime/RFC-0204-Dependency-Resolver.md"

if [ -f "$DISCOVERY" ]; then
  check_foundation_header "$DISCOVERY" "RFC-0201"
  check_no_foundation_boilerplate "$DISCOVERY" "RFC-0201"
  for required in 'Discovery Sources' 'Resource Candidate' 'Source Precedence' 'Discovery Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$DISCOVERY" || err "missing RFC-0201 section '$required'"
  done
  grep -q 'ResourceCandidate' "$DISCOVERY" || err "RFC-0201 must define the ResourceCandidate artifact"
  grep -q 'MUST NOT mutate' "$DISCOVERY" || err "RFC-0201 must state discovery does not mutate sources"
fi

if [ -f "$REGISTRY" ]; then
  check_foundation_header "$REGISTRY" "RFC-0202"
  check_no_foundation_boilerplate "$REGISTRY" "RFC-0202"
  for required in 'Registry Purpose' 'Stored Metadata' 'Lookup API' 'Registry vs Distribution' 'Registry Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$REGISTRY" || err "missing RFC-0202 section '$required'"
  done
  grep -q 'CanonicalResourceSet' "$REGISTRY" || err "RFC-0202 must hold the CanonicalResourceSet"
  grep -q 'RFC-0403' "$REGISTRY" || err "RFC-0202 must distinguish the distribution registry RFC-0403"
  for key in id kind version label checksum; do
    grep -q "$key" "$REGISTRY" || err "RFC-0202 lookup must support '$key'"
  done
fi

if [ -f "$VAL_ENGINE" ]; then
  check_foundation_header "$VAL_ENGINE" "RFC-0203"
  check_no_foundation_boilerplate "$VAL_ENGINE" "RFC-0203"
  for required in 'Engine Responsibility' 'Execution Strategy' 'Layer Ordering' 'Caching and Incremental' 'Result Handoff' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$VAL_ENGINE" || err "missing RFC-0203 section '$required'"
  done
  grep -q 'ValidationResult' "$VAL_ENGINE" || err "RFC-0203 must produce a ValidationResult"
  grep -q 'RFC-0106' "$VAL_ENGINE" || err "RFC-0203 must delegate the validation model to RFC-0106"
fi

if [ -f "$RESOLVER" ]; then
  check_foundation_header "$RESOLVER" "RFC-0204"
  check_no_foundation_boilerplate "$RESOLVER" "RFC-0204"
  for required in 'Resolution Inputs' 'Version Selection' 'Conflict Handling' 'Lock Generation' 'Resolver Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$RESOLVER" || err "missing RFC-0204 section '$required'"
  done
  grep -q 'DependencyGraph' "$RESOLVER" || err "RFC-0204 must produce a resolved DependencyGraph"
  grep -q 'RFC-0104' "$RESOLVER" || err "RFC-0204 must delegate graph structure to RFC-0104"
  grep -q 'RFC-0401' "$RESOLVER" || err "RFC-0204 must delegate lock format to RFC-0401"
  grep -q 'DEPENDENCY_ERROR' "$RESOLVER" || err "RFC-0204 must surface DEPENDENCY_ERROR on conflicts"
fi
```

- [ ] **Step 2: Chạy guard trên trạng thái HIỆN TẠI (kỳ vọng FAIL)**

Run: `bash tools/check-rfc-links`
Expected: nhiều dòng `FAIL:` cho RFC-0201/0202/0203/0204 (boilerplate heading + thiếu section + thiếu key-phrase). Exit ≠ 0.

- [ ] **Step 3: Commit**

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho cum RFC runtime read-pipeline" -- tools/check-rfc-links
```

---

## Task 2: RFC-0201 — Discovery Engine

**Files:**
- Modify: `docs/300-runtime/RFC-0201-Discovery-Engine.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0201 — Discovery Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines how resources are discovered from filesystem, package and registry sources.

This RFC owns discovery behavior and the `ResourceCandidate` model. Canonical parsing and the envelope are owned by [RFC-0100](../200-resource-model/RFC-0100-Canonical-Resource-Model.md); storage and lookup of discovered resources by [RFC-0202](RFC-0202-Registry-Engine.md); validation by [RFC-0203](RFC-0203-Validation-Engine.md); overall pipeline orchestration by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Discovery Sources

| Source | Description |
|---|---|
| filesystem | Resource files (YAML/JSON) under a workspace path. |
| package | Resources bundled inside a package. |
| registry | Resources exposed by a runtime registry source. |

Discovery reads from filesystem, package and registry sources and emits candidates without interpreting business logic.

## 3. Resource Candidate

A `ResourceCandidate` is a discovered, not-yet-validated resource together with its origin.

| Field | Semantics |
|---|---|
| `source` | Origin kind (`filesystem`, `package` or `registry`). |
| `location` | Path, package reference or registry coordinate. |
| `document` | Raw serialized document for later parsing. |

- Discovery emits one `ResourceCandidate` per discovered document.
- Discovery MUST NOT mutate source resources.

## 4. Source Precedence

- When the same `metadata.id` appears in multiple sources, default precedence is `filesystem` > `package` > `registry` unless configured otherwise.
- Duplicate candidates MUST be de-duplicated deterministically by id, version and source precedence.
- Discovery MUST produce deterministic ordering for identical inputs.

## 5. Discovery Behavior

- Discovery is a read-only phase; it MUST NOT write, move or modify sources.
- Unreadable or malformed sources SHOULD be surfaced as candidates carrying an error marker for the validation engine, not silently dropped.
- Discovered candidates are handed to the runtime registry ([RFC-0202](RFC-0202-Registry-Engine.md)).

## 6. Examples

```yaml
candidate:
  source: filesystem
  location: examples/resources/skill.yaml
  document: |
    apiVersion: platform/v1
    kind: Skill
    metadata:
      id: core/clean-code
      name: clean-code
      version: 1.0.0
```

## References

- [RFC-0100 — Canonical Resource Model](../200-resource-model/RFC-0100-Canonical-Resource-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0202 — Registry Engine](RFC-0202-Registry-Engine.md)
- [RFC-0203 — Validation Engine](RFC-0203-Validation-Engine.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0201; vẫn FAIL cho 0202/0203/0204.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0201-Discovery-Engine.md
git commit -m "docs(rfc): viet lai RFC-0201 Discovery Engine" -- docs/300-runtime/RFC-0201-Discovery-Engine.md
```

---

## Task 3: RFC-0202 — Registry Engine

**Files:**
- Modify: `docs/300-runtime/RFC-0202-Registry-Engine.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0202 — Registry Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines the local runtime registry that stores discovered resource metadata and supports lookup by id, kind, version, label and checksum.

This RFC owns the runtime registry store and lookup. It is distinct from the distribution registry and marketplace owned by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md). Metadata field semantics are owned by [RFC-0102](../200-resource-model/RFC-0102-Metadata-Model.md); naming by [RFC-0006](../100-foundation/RFC-0006-Naming-Convention.md); discovery by [RFC-0201](RFC-0201-Discovery-Engine.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Registry Purpose

- The runtime registry is an in-process store of discovered resources that holds the `CanonicalResourceSet` for a run.
- It enables lookup and cross-resource checks without re-reading sources.

## 3. Stored Metadata

| Field | Use |
|---|---|
| `id` | Identity in `namespace/name` form. |
| `kind` | Resource kind. |
| `version` | Resource version. |
| `labels` | Selection and grouping. |
| `checksum` | Content integrity and de-duplication. |

`metadata.id` MUST be unique within the registry.

## 4. Lookup API

- Lookup MUST support `id`, `kind`, `version`, `label` and `checksum`.
- Lookup by id returns at most one resource per version.
- Lookup results MUST be deterministic for identical registry state.

## 5. Registry vs Distribution

- The runtime registry (this RFC) is local and ephemeral to a run.
- The distribution registry and marketplace — remote, persistent package storage and indexing — are owned by [RFC-0403](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md).
- These are different concepts and MUST NOT be conflated.

## 6. Registry Behavior

- The registry stores discovered resources as a `CanonicalResourceSet`.
- Registration MUST reject a second resource with the same id and version but a differing checksum (conflict).
- The registry MUST NOT mutate stored source documents.

## 7. Examples

```yaml
lookup:
  by: id
  value: core/clean-code
  result:
    id: core/clean-code
    kind: Skill
    version: 1.0.0
    checksum: sha256:abc123
```

## References

- [RFC-0006 — Naming Convention](../100-foundation/RFC-0006-Naming-Convention.md)
- [RFC-0102 — Metadata Model](../200-resource-model/RFC-0102-Metadata-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0201 — Discovery Engine](RFC-0201-Discovery-Engine.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0201/0202; vẫn FAIL cho 0203/0204.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0202-Registry-Engine.md
git commit -m "docs(rfc): viet lai RFC-0202 Registry Engine" -- docs/300-runtime/RFC-0202-Registry-Engine.md
```

---

## Task 4: RFC-0203 — Validation Engine

**Files:**
- Modify: `docs/300-runtime/RFC-0203-Validation-Engine.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0203 — Validation Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines validation execution over individual resources and registries.

This RFC owns validation execution strategy. The validation model — layers, result envelope and error codes — is owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md); the runtime pipeline by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Engine Responsibility

- The engine executes the validation model defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- It MUST NOT redefine validation layers, the result envelope, or error codes.
- It produces a `ValidationResult` using the [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md) envelope.

## 3. Execution Strategy

- The engine owns execution order, short-circuiting and parallelism across layers.
- The engine MAY validate a single resource or an entire registry (`CanonicalResourceSet`).
- Validation MUST NOT mutate source resources.

## 4. Layer Ordering

- Layers run in the order syntax, schema, metadata, semantic, dependency, compatibility, policy unless a layer's required context is unavailable.
- A layer MAY be skipped only when its required context is absent, producing a diagnostic per [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 5. Caching and Incremental

- The engine MAY cache layer results keyed by resource content plus context; cache keys MUST be reproducible.
- Incremental validation MAY re-run only layers whose inputs changed.
- Caching MUST NOT change the produced `ValidationResult` for identical inputs.

## 6. Result Handoff

- The engine emits a `ValidationResult` consumed by downstream engines.
- A resource with any `error`-severity diagnostic MUST be treated as invalid downstream.

## 7. Examples

```yaml
run:
  target: core/clean-code
  layers: [syntax, schema, metadata, semantic, dependency]
  result:
    valid: true
    diagnostics: []
```

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0202 — Registry Engine](RFC-0202-Registry-Engine.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0201/0202/0203; vẫn FAIL cho 0204.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0203-Validation-Engine.md
git commit -m "docs(rfc): viet lai RFC-0203 Validation Engine" -- docs/300-runtime/RFC-0203-Validation-Engine.md
```

---

## Task 5: RFC-0204 — Dependency Resolver + full verify

**Files:**
- Modify: `docs/300-runtime/RFC-0204-Dependency-Resolver.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0204 — Dependency Resolver

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines dependency resolution, version selection, conflict handling and lock generation.

This RFC owns the resolution algorithm. The dependency graph structure and invariants are owned by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md); version range grammar by [RFC-0005](../100-foundation/RFC-0005-Versioning.md); lock file format by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md); diagnostics by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Resolution Inputs

- The resolver consumes the dependency graph defined by [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md) plus the runtime registry ([RFC-0202](RFC-0202-Registry-Engine.md)).
- It MUST NOT redefine graph structure or the acyclic invariant.

## 3. Version Selection

- The resolver selects one concrete version per dependency from the versions satisfying the declared range.
- Version range grammar is owned by [RFC-0005](../100-foundation/RFC-0005-Versioning.md).
- Selection MUST be deterministic for identical inputs and registry state.

## 4. Conflict Handling

- When no version satisfies all constraints, the resolver MUST surface a `DEPENDENCY_ERROR` diagnostic (via [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md)).
- Cyclic or missing required dependencies MUST also surface as `DEPENDENCY_ERROR`.

## 5. Lock Generation

- The resolver produces a resolved `DependencyGraph` and a lock describing the exact resolved versions.
- The lock file format is owned by [RFC-0401](../500-build-distribution/RFC-0401-Lock-File.md); this RFC owns only the act of producing the resolved set.

## 6. Resolver Behavior

- Resolution is read-only with respect to source resources.
- Given the same registry state and inputs, the resolved `DependencyGraph` and lock MUST be identical.

## 7. Examples

```yaml
resolve:
  root: plugins/backend
  selected:
    - id: plugins/backend
      version: 1.0.0
    - id: core/clean-code
      version: 1.0.0
  lock: generated
```

## References

- [RFC-0005 — Versioning](../100-foundation/RFC-0005-Versioning.md)
- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0401 — Lock File](../500-build-distribution/RFC-0401-Lock-File.md)
```

- [ ] **Step 2: Full verify (kỳ vọng PASS)** — Run: `bash tools/check-rfc-links`; Expected: `OK: all RFC overview checks passed`, exit 0.

- [ ] **Step 3: SUMMARY check** — Run: `grep -c '300-runtime/RFC-020[1-4]' SUMMARY.md`; Expected: `4`. Nếu ≠4, đồng bộ SUMMARY.md.

- [ ] **Step 4: TODO check** — Run: `grep -nE '0201|0202|0203|0204' TODO.md || echo "no entry"`. Nếu có `- [ ]` cho các RFC này, đổi `- [x]` và thêm TODO.md vào pathspec; nếu không, bỏ qua.

- [ ] **Step 5: Commit**

```bash
git add docs/300-runtime/RFC-0204-Dependency-Resolver.md
git commit -m "docs(rfc): viet lai RFC-0204 Dependency Resolver" -- docs/300-runtime/RFC-0204-Dependency-Resolver.md
```

---

## Definition of Done

- RFC-0201/0202/0203/0204 hết boilerplate + đủ required sections + tôn trọng ranh giới (0202≠0403; 0203 không tả lại 0106; 0204 không tả lại 0104).
- Mỗi engine tham chiếu đúng handoff artifact của RFC-0200 (ResourceCandidate/CanonicalResourceSet/ValidationResult/DependencyGraph).
- `tools/check-rfc-links` PASS (exit 0); `SUMMARY.md` không đổi.
- Pipeline đọc/resolve `300-runtime` (0201–0204) REAL.
- Mỗi task 1 commit (pathspec, không trailer); chưa push.
