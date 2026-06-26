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
