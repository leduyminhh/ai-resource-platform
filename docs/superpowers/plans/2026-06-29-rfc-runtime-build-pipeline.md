# RFC Runtime Build-Pipeline Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Viết nội dung thật cho 5 RFC nửa build/execute của cụm `docs/300-runtime/` (RFC-0205 Planning, RFC-0206 Execution, RFC-0207 Artifact Model, RFC-0208 Caching, RFC-0209 Observability), theo per-RFC ownership + delegation, đóng trọn cụm 300 (0200–0209).

**Architecture:** Mỗi engine/concern sở hữu phần của nó, dùng đúng handoff artifact của RFC-0200 (ExecutionPlan/ExecutionArtifact) và delegate hợp đồng cross-cutting (0104/0106/0107/0101/0500/0800) qua cross-ref. Mở rộng `tools/check-rfc-links` bằng "Check 14".

**Tech Stack:** Markdown (RFC docs), Bash (guard).

**Nguồn nội dung:** Design doc [docs/superpowers/specs/2026-06-29-rfc-runtime-build-pipeline-design.md](../specs/2026-06-29-rfc-runtime-build-pipeline-design.md); style mẫu RFC-0200/0204.

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
- Sau header: `## Abstract` (1 câu + đoạn ownership/delegation), `## 1. Conventions` (RFC 2119), section đánh số, kết `## References`.
- Cross-ref tương đối từ `docs/300-runtime/`: cùng thư mục `RFC-0200/0204/0205..0209-...md`; `../000-overview/...`, `../200-resource-model/...`, `../500-build-distribution/...`, `../600-sdk/...`, `../900-security-governance/...`.
- KHÔNG heading trùng boilerplate: Motivation, Goals, Non-Goals, Canonical Model, Required Behavior, Runtime Flow, Validation Rules, Error Model, Security Considerations, Migration Guidance, Future Work.
- KHÔNG đụng: RFC ngoài 5 cái này, `schemas/`, `examples/`, `src/`, RFC đã REAL.
- Commit: Conventional Commits, header tiếng Việt KHÔNG dấu, **KHÔNG trailer**. PATHSPEC TƯỜNG MINH; KHÔNG `git add -A`/`.`/`commit -a`.
- An toàn: 1 task = 1 commit; trên master; KHÔNG tự push.

---

## File Structure

| File | Trách nhiệm |
|---|---|
| `tools/check-rfc-links` | Thêm Check 14 cho 5 RFC |
| `docs/300-runtime/RFC-0205-Planning-Engine.md` | resolved-set → ExecutionPlan |
| `docs/300-runtime/RFC-0206-Execution-Engine.md` | thực thi plan → ExecutionArtifact |
| `docs/300-runtime/RFC-0207-Artifact-Model.md` | mô hình ExecutionArtifact bất biến |
| `docs/300-runtime/RFC-0208-Caching-Strategy.md` | cache keys/invalidation/reproducible |
| `docs/300-runtime/RFC-0209-Observability.md` | logs/metrics/traces/reports |

---

## Task 1: Build-pipeline validator guard (Check 14)

**Files:**
- Modify: `tools/check-rfc-links` (chèn Check 14 ngay TRƯỚC 2 dòng cuối `[ "$fail" -eq 0 ] && echo ...` / `exit "$fail"`)

**Interfaces:**
- Consumes: helper `check_foundation_header`, `check_no_foundation_boilerplate` (đã có, trong scope).

- [ ] **Step 1: Chèn khối Check 14** vào `tools/check-rfc-links`, ngay trước dòng `[ "$fail" -eq 0 ] && echo "OK: all RFC overview checks passed"`:

```bash
# --- Check 14: Runtime build-pipeline (RFC-0205/0206/0207/0208/0209) ---
PLANNING="$ROOT/docs/300-runtime/RFC-0205-Planning-Engine.md"
EXECUTION="$ROOT/docs/300-runtime/RFC-0206-Execution-Engine.md"
ARTIFACT="$ROOT/docs/300-runtime/RFC-0207-Artifact-Model.md"
CACHING="$ROOT/docs/300-runtime/RFC-0208-Caching-Strategy.md"
OBSERVABILITY="$ROOT/docs/300-runtime/RFC-0209-Observability.md"

if [ -f "$PLANNING" ]; then
  check_foundation_header "$PLANNING" "RFC-0205"
  check_no_foundation_boilerplate "$PLANNING" "RFC-0205"
  for required in 'Planning Inputs' 'Execution Plan' 'Plan Determinism' 'Parallelism and Cache Reuse' 'Planning Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$PLANNING" || err "missing RFC-0205 section '$required'"
  done
  grep -q 'ExecutionPlan' "$PLANNING" || err "RFC-0205 must produce an ExecutionPlan"
  grep -q 'MUST NOT mutate' "$PLANNING" || err "RFC-0205 must state planning does not mutate sources"
  grep -q 'RFC-0204' "$PLANNING" || err "RFC-0205 must consume the resolved graph from RFC-0204"
fi

if [ -f "$EXECUTION" ]; then
  check_foundation_header "$EXECUTION" "RFC-0206"
  check_no_foundation_boilerplate "$EXECUTION" "RFC-0206"
  for required in 'Execution Inputs' 'Adapter Invocation' 'Execution Artifacts' 'Failure Handling' 'Execution Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$EXECUTION" || err "missing RFC-0206 section '$required'"
  done
  grep -q 'ExecutionArtifact' "$EXECUTION" || err "RFC-0206 must produce ExecutionArtifact"
  grep -q 'RFC-0204' "$EXECUTION" || err "RFC-0206 must delegate resolution to RFC-0204"
  grep -q 'RFC-0500' "$EXECUTION" || err "RFC-0206 must delegate adapter contract to RFC-0500"
fi

if [ -f "$ARTIFACT" ]; then
  check_foundation_header "$ARTIFACT" "RFC-0207"
  check_no_foundation_boilerplate "$ARTIFACT" "RFC-0207"
  for required in 'Artifact Concept' 'Artifact Identity' 'Immutability' 'Provenance' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$ARTIFACT" || err "missing RFC-0207 section '$required'"
  done
  grep -q 'ExecutionArtifact' "$ARTIFACT" || err "RFC-0207 must define ExecutionArtifact"
  grep -q 'immutable' "$ARTIFACT" || err "RFC-0207 must state artifacts are immutable"
  grep -q 'RFC-0107' "$ARTIFACT" || err "RFC-0207 must delegate bundling to RFC-0107"
fi

if [ -f "$CACHING" ]; then
  check_foundation_header "$CACHING" "RFC-0208"
  check_no_foundation_boilerplate "$CACHING" "RFC-0208"
  for required in 'Cache Keys' 'Invalidation' 'Reproducible Builds' 'Caching Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$CACHING" || err "missing RFC-0208 section '$required'"
  done
  grep -q 'reproducible' "$CACHING" || err "RFC-0208 must define reproducible builds"
  grep -q 'invalidation' "$CACHING" || err "RFC-0208 must define cache invalidation"
  grep -q 'RFC-0101' "$CACHING" || err "RFC-0208 must delegate serialization determinism to RFC-0101"
fi

if [ -f "$OBSERVABILITY" ]; then
  check_foundation_header "$OBSERVABILITY" "RFC-0209"
  check_no_foundation_boilerplate "$OBSERVABILITY" "RFC-0209"
  for required in 'Signals' 'Logs Metrics Traces' 'Reports' 'Diagnostics vs Validation' 'Observability Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$OBSERVABILITY" || err "missing RFC-0209 section '$required'"
  done
  grep -q 'RFC-0106' "$OBSERVABILITY" || err "RFC-0209 must distinguish telemetry from RFC-0106 validation diagnostics"
  for sig in logs metrics traces; do
    grep -q "$sig" "$OBSERVABILITY" || err "RFC-0209 must define signal '$sig'"
  done
fi
```

- [ ] **Step 2: Chạy guard (kỳ vọng FAIL)** — Run: `bash tools/check-rfc-links`; Expected: nhiều `FAIL:` cho RFC-0205/0206/0207/0208/0209; exit ≠ 0.

- [ ] **Step 3: Commit**

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho cum RFC runtime build-pipeline" -- tools/check-rfc-links
```

---

## Task 2: RFC-0205 — Planning Engine

**Files:**
- Modify: `docs/300-runtime/RFC-0205-Planning-Engine.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0205 — Planning Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines conversion from a resolved graph into a deterministic execution plan optimized for cache reuse and safe parallelism.

This RFC owns plan construction and the `ExecutionPlan`. The resolved dependency graph is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md) and [RFC-0104](../200-resource-model/RFC-0104-Dependency-Graph.md); cache-reuse strategy by [RFC-0208](RFC-0208-Caching-Strategy.md); execution by [RFC-0206](RFC-0206-Execution-Engine.md); pipeline orchestration by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Planning Inputs

- Planning consumes the resolved dependency graph produced by [RFC-0204](RFC-0204-Dependency-Resolver.md).
- Planning MAY consume user intent (targets, goals) selecting which resources to build.
- Planning MUST NOT mutate source resource files.

## 3. Execution Plan

- Planning produces an `ExecutionPlan`: an ordered set of steps to execute.
- Each step names the resource, the action and its declared dependencies within the plan.
- The `ExecutionPlan` is handed to the execution engine ([RFC-0206](RFC-0206-Execution-Engine.md)).

## 4. Plan Determinism

- The `ExecutionPlan` MUST be deterministic: identical resolved inputs and intent produce an identical plan.
- Step ordering MUST be stable and derived only from the graph and declared inputs.

## 5. Parallelism and Cache Reuse

- The plan MAY mark independent steps as safe to run in parallel.
- The plan SHOULD expose cache keys so unchanged steps can be skipped; cache-key semantics are owned by [RFC-0208](RFC-0208-Caching-Strategy.md).

## 6. Planning Behavior

- Planning is read-only with respect to source resources.
- Planning MUST NOT perform dependency resolution; that is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md).

## 7. Examples

```yaml
plan:
  steps:
    - id: core/clean-code
      action: build
      dependsOn: []
      parallelSafe: true
    - id: plugins/backend
      action: build
      dependsOn: [core/clean-code]
```

## References

- [RFC-0104 — Dependency Graph](../200-resource-model/RFC-0104-Dependency-Graph.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0204 — Dependency Resolver](RFC-0204-Dependency-Resolver.md)
- [RFC-0206 — Execution Engine](RFC-0206-Execution-Engine.md)
- [RFC-0208 — Caching Strategy](RFC-0208-Caching-Strategy.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0205; vẫn FAIL cho 0206/0207/0208/0209.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0205-Planning-Engine.md
git commit -m "docs(rfc): viet lai RFC-0205 Planning Engine" -- docs/300-runtime/RFC-0205-Planning-Engine.md
```

---

## Task 3: RFC-0206 — Execution Engine

**Files:**
- Modify: `docs/300-runtime/RFC-0206-Execution-Engine.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0206 — Execution Engine

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines how execution plans are executed without performing dependency resolution.

This RFC owns plan execution. Dependency resolution is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md); the adapter contract by [RFC-0500](../600-sdk/RFC-0500-Adapter-SDK.md); capability and security constraints by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md); the artifact model by [RFC-0207](RFC-0207-Artifact-Model.md); validation by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Execution Inputs

- Execution consumes an `ExecutionPlan` produced by [RFC-0205](RFC-0205-Planning-Engine.md).
- Execution MUST NOT perform dependency resolution; resolution is owned by [RFC-0204](RFC-0204-Dependency-Resolver.md).
- Execution MUST NOT start before required validation has passed ([RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md)).

## 3. Adapter Invocation

- Execution MUST use declared adapters and capabilities, not implicit vendor-specific behavior.
- The adapter contract is owned by [RFC-0500](../600-sdk/RFC-0500-Adapter-SDK.md); capability grants and security limits by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md).

## 4. Execution Artifacts

- Executing a step produces one or more `ExecutionArtifact` outputs.
- The artifact model (identity, immutability, provenance) is owned by [RFC-0207](RFC-0207-Artifact-Model.md).

## 5. Failure Handling

- A failed step MUST stop dependents that require its output.
- Execution MUST surface failures with enough context for observability ([RFC-0209](RFC-0209-Observability.md)).

## 6. Execution Behavior

- Execution MUST be deterministic given the same plan, adapters and inputs.
- Independent steps marked parallel-safe by the plan MAY run concurrently.

## 7. Examples

```yaml
execution:
  plan: build
  step: plugins/backend
  adapter: platform/build-adapter
  result:
    status: success
    artifacts:
      - id: plugins/backend@1.0.0
        checksum: sha256:def456
```

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0204 — Dependency Resolver](RFC-0204-Dependency-Resolver.md)
- [RFC-0205 — Planning Engine](RFC-0205-Planning-Engine.md)
- [RFC-0207 — Artifact Model](RFC-0207-Artifact-Model.md)
- [RFC-0500 — Adapter SDK](../600-sdk/RFC-0500-Adapter-SDK.md)
- [RFC-0800 — Security Model](../900-security-governance/RFC-0800-Security-Model.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0205/0206; vẫn FAIL cho 0207/0208/0209.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0206-Execution-Engine.md
git commit -m "docs(rfc): viet lai RFC-0206 Execution Engine" -- docs/300-runtime/RFC-0206-Execution-Engine.md
```

---

## Task 4: RFC-0207 — Artifact Model

**Files:**
- Modify: `docs/300-runtime/RFC-0207-Artifact-Model.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0207 — Artifact Model

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines immutable outputs produced by execution and packaging.

This RFC owns the `ExecutionArtifact` model. Artifacts are produced by execution ([RFC-0206](RFC-0206-Execution-Engine.md)); bundling artifacts into packages is owned by [RFC-0107](../200-resource-model/RFC-0107-Packaging-Model.md) and [RFC-0402](../500-build-distribution/RFC-0402-Packaging.md); signing and integrity by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Artifact Concept

- An `ExecutionArtifact` is an output produced by executing a plan step.
- An artifact is data plus metadata describing what produced it.

## 3. Artifact Identity

- An artifact is identified by a content checksum plus the producing resource id and version.
- Checksum and signing mechanics are owned by [RFC-0801](../900-security-governance/RFC-0801-Signing-and-Integrity.md).

## 4. Immutability

- An `ExecutionArtifact` is immutable once produced; any change MUST yield a new artifact with a new checksum.
- Consumers MUST reference artifacts by identity, not by mutable location.

## 5. Provenance

- An artifact MUST record provenance: the plan step, resource id/version and input checksums that produced it.
- Provenance enables reproducibility checks ([RFC-0208](RFC-0208-Caching-Strategy.md)).

## 6. Examples

```yaml
artifact:
  id: plugins/backend@1.0.0
  checksum: sha256:def456
  producedBy:
    step: plugins/backend
    inputs:
      - core/clean-code@1.0.0
```

## References

- [RFC-0107 — Packaging Model](../200-resource-model/RFC-0107-Packaging-Model.md)
- [RFC-0206 — Execution Engine](RFC-0206-Execution-Engine.md)
- [RFC-0208 — Caching Strategy](RFC-0208-Caching-Strategy.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0801 — Signing and Integrity](../900-security-governance/RFC-0801-Signing-and-Integrity.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0205/0206/0207; vẫn FAIL cho 0208/0209.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0207-Artifact-Model.md
git commit -m "docs(rfc): viet lai RFC-0207 Artifact Model" -- docs/300-runtime/RFC-0207-Artifact-Model.md
```

---

## Task 5: RFC-0208 — Caching Strategy

**Files:**
- Modify: `docs/300-runtime/RFC-0208-Caching-Strategy.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0208 — Caching Strategy

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines cache keys, invalidation and reproducible build behavior.

This RFC owns caching semantics. Serialization determinism is owned by [RFC-0101](../200-resource-model/RFC-0101-Serialization.md); the deterministic-builds principle by [RFC-0002](../000-overview/RFC-0002-Guiding-Principles.md); plans that expose cache keys by [RFC-0205](RFC-0205-Planning-Engine.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Cache Keys

- A cache key MUST be derived from the content and context that determine a step's output (resource content, resolved inputs, adapter identity).
- Cache keys MUST be reproducible: identical inputs produce identical keys.

## 3. Invalidation

- A cache entry MUST be invalidated when any input contributing to its key changes.
- Cache invalidation MUST be conservative: when in doubt, recompute rather than serve a stale entry.

## 4. Reproducible Builds

- Reproducible builds: identical inputs MUST yield identical outputs.
- Reproducibility relies on deterministic serialization ([RFC-0101](../200-resource-model/RFC-0101-Serialization.md)) and the deterministic-builds principle ([RFC-0002](../000-overview/RFC-0002-Guiding-Principles.md)).

## 5. Caching Behavior

- Caching MUST NOT change the produced output for identical inputs; it only skips recomputation.
- A cache hit MUST be equivalent to recomputation.

## 6. Examples

```yaml
cache:
  key: sha256:aa11bb
  inputs:
    - core/clean-code@1.0.0:sha256:abc123
    - adapter: platform/build-adapter@0.1.0
  state: hit
```

## References

- [RFC-0002 — Guiding Principles](../000-overview/RFC-0002-Guiding-Principles.md)
- [RFC-0101 — Serialization](../200-resource-model/RFC-0101-Serialization.md)
- [RFC-0205 — Planning Engine](RFC-0205-Planning-Engine.md)
```

- [ ] **Step 2: Verify** — `bash tools/check-rfc-links`; không còn FAIL nhắc RFC-0205/0206/0207/0208; vẫn FAIL cho 0209.

- [ ] **Step 3: Commit**

```bash
git add docs/300-runtime/RFC-0208-Caching-Strategy.md
git commit -m "docs(rfc): viet lai RFC-0208 Caching Strategy" -- docs/300-runtime/RFC-0208-Caching-Strategy.md
```

---

## Task 6: RFC-0209 — Observability + full verify

**Files:**
- Modify: `docs/300-runtime/RFC-0209-Observability.md` (thay TOÀN BỘ nội dung)

- [ ] **Step 1: Thay toàn bộ file** bằng:

```markdown
# RFC-0209 — Observability

**Status:** Draft  
**Category:** 300-runtime  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines logs, metrics, traces, reports and diagnostic outputs for runtime engines.

This RFC owns runtime telemetry. It is distinct from the validation result envelope owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md); secret handling is owned by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md); pipeline orchestration by [RFC-0200](RFC-0200-Runtime-Architecture.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

## 2. Signals

- Runtime engines emit four signal kinds: logs, metrics, traces and reports.
- Signals describe engine behavior; they do not change resource content or outputs.

## 3. Logs Metrics Traces

- `logs`: time-ordered messages about engine activity.
- `metrics`: numeric measurements (counts, durations) for engine steps.
- `traces`: causal spans linking steps across a run.

## 4. Reports

- A report summarizes a run: steps executed, cache hits, artifacts produced and failures.
- Reports SHOULD be machine-readable and reproducible for identical runs.

## 5. Diagnostics vs Validation

- Observability telemetry is distinct from validation diagnostics; the validation result envelope and error codes are owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).
- This RFC MUST NOT redefine validation diagnostics.

## 6. Observability Behavior

- Telemetry MUST NOT contain plaintext secrets; secret handling is owned by [RFC-0800](../900-security-governance/RFC-0800-Security-Model.md).
- Telemetry SHOULD be optional to consumers and MUST NOT alter execution results.

## 7. Examples

```yaml
report:
  run: build
  steps: 2
  cacheHits: 1
  artifacts: 1
  failures: 0
```

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](RFC-0200-Runtime-Architecture.md)
- [RFC-0800 — Security Model](../900-security-governance/RFC-0800-Security-Model.md)
```

- [ ] **Step 2: Full verify (kỳ vọng PASS)** — Run: `bash tools/check-rfc-links`; Expected: `OK: all RFC overview checks passed`, exit 0.

- [ ] **Step 3: SUMMARY check** — Run: `grep -c '300-runtime/RFC-020[5-9]' SUMMARY.md`; Expected: `5`. Nếu ≠5, đồng bộ SUMMARY.md.

- [ ] **Step 4: TODO check** — Run: `grep -nE '0205|0206|0207|0208|0209' TODO.md || echo "no entry"`. Nếu có `- [ ]` cho các RFC này, đổi `- [x]` + thêm TODO.md vào pathspec; nếu không, bỏ qua.

- [ ] **Step 5: Commit**

```bash
git add docs/300-runtime/RFC-0209-Observability.md
git commit -m "docs(rfc): viet lai RFC-0209 Observability" -- docs/300-runtime/RFC-0209-Observability.md
```

---

## Definition of Done

- RFC-0205/0206/0207/0208/0209 hết boilerplate + đủ required sections + tôn trọng ranh giới (0206≠0204, dùng 0500/0800; 0207≠0107; 0209≠0106).
- Planning/Execution tham chiếu đúng handoff artifact (`ExecutionPlan`, `ExecutionArtifact`).
- `tools/check-rfc-links` PASS (exit 0); `SUMMARY.md` không đổi.
- Cụm `300-runtime` REAL trọn vẹn (0200–0209).
- Mỗi task 1 commit (pathspec, không trailer); chưa push.
