# Project Overview

> Nội dung dưới đây tổng hợp từ các RFC trong `docs/` (nguồn có thật). Trích dẫn RFC ở mỗi mục.

## Mục tiêu
Xây dựng **AI Resource Platform Specification (ARPS)** — bộ đặc tả vendor-neutral,
resource-oriented, độc-lập-hiện-thực để xây nền tảng phát triển AI tái sử dụng (RFC-0000).
Mục tiêu cốt lõi (RFC-0000 §3): định nghĩa **contract ổn định**, hỗ trợ **migration không xâm lấn**,
giữ nền tảng **resource-oriented**, bảo toàn **hành vi deterministic**, và cho phép **mở rộng**
mà không đổi kiến trúc lõi.

## Nguyên tắc dẫn đường (RFC-0002)
resource-first · schema-first · deterministic builds · vendor neutrality · non-invasive migration · extensibility.

## Phạm vi
- **In-scope (theo SUMMARY.md / docs/):**
  - Overview & foundation: vision, terminology, guiding principles, layered architecture, repo layout,
    versioning, naming, compatibility (RFC-0000..0007).
  - Resource model: canonical model, serialization, metadata, lifecycle, dependency graph, manifest,
    validation model, packaging model (RFC-0100..0107).
  - Runtime engines: discovery, registry, validation, dependency resolver, planning, execution,
    artifact, caching, observability (RFC-0200..0209).
  - Plugin system, build & distribution, SDK (adapter/plugin/validator), CLI, IDE integration,
    security & governance (RFC-0300..0802).
  - `schemas/` (JSON Schema chuẩn), `examples/` (manifest mẫu), `tools/validator` (placeholder).
- **Out-of-scope (Non-Goals, RFC-0000 §4):** KHÔNG bắt buộc một ngôn ngữ lập trình cụ thể;
  KHÔNG phụ thuộc một AI assistant/IDE/vendor cụ thể; KHÔNG ép tái cấu trúc mã nguồn nghiệp vụ
  sẵn có. Hiện thực runtime tham chiếu là "Future Work" (RFC-0000 §14).

## Người dùng
[Inference — dựa trên các tầng SDK/CLI/IDE/Governance trong docs, không nêu tường minh trong RFC]:
tác giả resource (plugin/skill/prompt/workflow), đội nền tảng vận hành runtime/registry, nhà tích
hợp IDE/AI assistant qua adapter (RFC-0700), và đội governance/bảo mật (RFC-0800..0802).

## Tiêu chí thành công
- Mọi resource hợp lệ PARSE được và PASS validate trước khi resolve (RFC-0000 §6, RFC-0106).
- Resource ID duy nhất trong một registry; version theo SemVer; dependency graph **acyclic** (RFC-0000 §8, RFC-0104).
- Output deterministic; read-only phase KHÔNG mutate resource nguồn (RFC-0000 §6).
- "Future Work" (RFC-0000 §14): conformance test, reference runtime, registry interoperability suite.
