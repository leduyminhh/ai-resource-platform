# RFC-0800 Security Model Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `RFC-0800 — Security Model` into the normative security principles, trust boundaries and required controls for ARPS.

**Architecture:** Keep `RFC-0800` focused on security contract, trust boundaries, controls and validation hooks only. Extend `tools/check-rfc-links` with RFC-0800 guardrails first, then rewrite the RFC until the guard passes and close the pending delegation.

**Tech Stack:** Markdown RFC documents, Bash validation script, Git Bash/PowerShell on Windows.

---

## File Structure

- Modify: `tools/check-rfc-links` — add RFC-0800-specific assertions for security model ownership.
- Modify: `docs/900-security-governance/RFC-0800-Security-Model.md` — replace boilerplate with security model content.
- Modify: `TODO.md` — mark the RFC-0800 pending delegation complete.
- Verify only: `docs/superpowers/specs/2026-06-29-rfc-0800-security-model-design.md` — source design for this plan.
- Verify only: `SUMMARY.md` — RFC title/path should remain unchanged.
- No changes: `docs/900-security-governance/RFC-0801-Signing-and-Integrity.md`, `docs/900-security-governance/RFC-0802-Governance.md`, `docs/300-runtime/RFC-0206-Execution-Engine.md`.

---

## Task 1: Add RFC-0800 Validator Guard

**Files:**
- Modify: `tools/check-rfc-links`
- Verify: `docs/900-security-governance/RFC-0800-Security-Model.md`

- [ ] **Step 1: Inspect current validator**

Run:

```bash
sed -n '1,520p' tools/check-rfc-links
```

Expected: script includes overview checks plus existing guards for RFC-0007, RFC-0100, RFC-0106 and RFC-0200.

- [ ] **Step 2: Add RFC-0800 checks**

Append this block before the final success/exit lines:

```bash
# --- Check 9: RFC-0800 Security Model ownership ---
SECURITY="$ROOT/docs/900-security-governance/RFC-0800-Security-Model.md"
if [ -f "$SECURITY" ]; then
  for field in '\*\*Status:\*\*' '\*\*Category:\*\*' '\*\*Specification:\*\*' '\*\*Version:\*\*'; do
    grep -qE "$field" "$SECURITY" || err "missing header field ${field//\\/} in RFC-0800"
  done

  for bad in 'Motivation' 'Goals' 'Non-Goals' 'Canonical Model' 'Required Behavior' 'Runtime Flow' 'Validation Rules' 'Error Model' 'Compatibility' 'Migration Guidance' 'Future Work' 'BUILD_ERROR'; do
    grep -qE "^## ([0-9]+\. )?$bad$|$bad" "$SECURITY" && err "boilerplate or out-of-scope content '$bad' still in RFC-0800"
  done

  for required in 'Security Principles' 'Trust Boundaries' 'Required Controls' 'Validation Hooks' 'Failure Behavior' 'Examples' 'References'; do
    grep -qE "^#+ .*$required" "$SECURITY" || err "missing RFC-0800 section '$required'"
  done

  for boundary in 'Local Workspace' 'Registry / Marketplace' 'Package / Artifact' 'Adapter / Execution' 'Secret Boundary'; do
    grep -q "$boundary" "$SECURITY" || err "missing RFC-0800 trust boundary '$boundary'"
  done

  grep -q 'MUST NOT contain plaintext secrets' "$SECURITY" || err "RFC-0800 must forbid plaintext secrets in manifests"
  grep -q 'not trusted by default\|not trust.*default' "$SECURITY" || err "RFC-0800 must state remote/trust targets are not trusted by default"
  grep -q 'Execution MUST use declared adapters and capabilities' "$SECURITY" || err "RFC-0800 must require declared adapters/capabilities for execution"
  grep -q 'POLICY_VIOLATION' "$SECURITY" || err "RFC-0800 must map security failures to POLICY_VIOLATION diagnostics"
fi
```

- [ ] **Step 3: Run validator and confirm it fails first**

Run:

```bash
bash tools/check-rfc-links
```

Expected: exit code is non-zero and output includes failures for current RFC-0800, including boilerplate headings and missing sections such as `Security Principles`, `Trust Boundaries`, `Validation Hooks` and `Failure Behavior`.

- [ ] **Step 4: Commit validator guard**

Run:

```bash
git add tools/check-rfc-links
git commit -m "test(docs): them guard cho RFC-0800 security model"
```

Expected: one commit containing only `tools/check-rfc-links`.

---

## Task 2: Rewrite RFC-0800 Security Model

**Files:**
- Modify: `docs/900-security-governance/RFC-0800-Security-Model.md`
- Verify: `docs/superpowers/specs/2026-06-29-rfc-0800-security-model-design.md`

- [ ] **Step 1: Replace RFC-0800 content**

Replace the full content of `docs/900-security-governance/RFC-0800-Security-Model.md` with:

```markdown
# RFC-0800 — Security Model

**Status:** Draft  
**Category:** 900-security-governance  
**Specification:** AI Resource Platform Specification (ARPS)  
**Version:** 1.0.0

---

## Abstract

Defines ARPS security principles, trust boundaries, required controls and validation hooks.

This RFC owns the security model for trust, secrets, remote resources, package verification and execution boundaries. Signing and integrity mechanisms are defined by [RFC-0801](RFC-0801-Signing-and-Integrity.md); governance workflow is defined by [RFC-0802](RFC-0802-Governance.md); diagnostics are defined by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 1. Conventions

The key words MUST, SHOULD, MAY, MUST NOT and SHOULD NOT are to be interpreted as described in RFC 2119.

Security controls in this RFC define requirements and boundaries. Specific algorithms, approval workflows, execution internals and diagnostic envelope shapes are owned by their respective RFCs.

## 2. Security Principles

- Remote registries, packages and execution targets are not trusted by default.
- Implementations SHOULD use least privilege for adapters, tools and runtime context.
- Resource manifests MUST NOT contain plaintext secrets.
- Remote resources and packages SHOULD be verified before use.
- Security-relevant decisions SHOULD be deterministic and auditable.
- Unknown trust context SHOULD fail closed for high-risk actions such as execution and publishing.

## 3. Trust Boundaries

| Boundary | Meaning | Main risk |
|---|---|---|
| Local Workspace | Repository files and local resource manifests. | unreviewed local changes or accidental secret inclusion |
| Registry / Marketplace | Remote metadata, resources and packages. | dependency confusion, untrusted source or malicious metadata |
| Package / Artifact | Distributable bundles and generated artifacts. | tampering, missing integrity metadata or artifact substitution |
| Adapter / Execution | Tool, model or vendor execution boundary. | undeclared capability use, unsafe side effects or data exfiltration |
| Secret Boundary | External secret provider or runtime secret injection. | plaintext secret leakage, logging secrets or manifest persistence |

## 4. Required Controls

- Remote registries and marketplace sources SHOULD be explicitly trusted before use.
- Remote resources SHOULD be verified before validation, execution or publishing decisions.
- High-risk actions MAY require policy to make verification mandatory.
- Packages SHOULD include integrity metadata; signing and checksum mechanism details are owned by [RFC-0801](RFC-0801-Signing-and-Integrity.md).
- Resource manifests MUST NOT contain plaintext secrets.
- Execution MUST use declared adapters and capabilities.
- Tools MUST NOT log secrets or write them into generated manifests or artifacts.
- Security policy failures SHOULD surface as `POLICY_VIOLATION` diagnostics using [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md).

## 5. Validation Hooks

| Hook | Layer | Example diagnostic |
|---|---|---|
| Secret scan | policy or semantic | plaintext token-like value found in manifest |
| Registry trust check | policy | registry source is not in trusted set |
| Package integrity check | policy | package is missing required integrity metadata |
| Adapter capability check | policy or semantic | execution requires undeclared adapter capability |
| Remote source check | policy | remote resource source cannot be verified |

Validation hooks define what SHOULD be checked. Validator execution strategy and diagnostic envelope shape are owned by [RFC-0106](../200-resource-model/RFC-0106-Validation-Model.md) and validation engine RFCs.

## 6. Failure Behavior

- Unknown trust context SHOULD produce warning or error based on policy risk level.
- High-risk execution or publishing SHOULD fail closed when trust cannot be established.
- Plaintext secret detection MUST be an error.
- Missing required package integrity metadata MUST produce `POLICY_VIOLATION` when policy requires integrity metadata.
- Untrusted registry, package or execution target MUST NOT be silently allowed.
- Security diagnostics SHOULD include reference, path and location information when available.

## 7. Examples

### 7.1 Trusted registry accepted

A registry source configured as trusted by policy can be used for discovery and dependency resolution. The trust decision should be auditable through policy or registry configuration.

### 7.2 Package missing integrity metadata

A package without required integrity metadata produces a warning or `POLICY_VIOLATION` depending on the active policy. Signing and checksum details are delegated to [RFC-0801](RFC-0801-Signing-and-Integrity.md).

### 7.3 Plaintext secret rejected

A manifest containing `spec.token: abc123` is invalid when that value is a plaintext secret. The secret must be referenced through an external secret provider or runtime injection mechanism.

### 7.4 Undeclared adapter capability blocked

An execution plan requiring filesystem write access is blocked when the selected adapter does not declare that capability.

## References

- [RFC-0106 — Validation Model](../200-resource-model/RFC-0106-Validation-Model.md)
- [RFC-0200 — Runtime Architecture](../300-runtime/RFC-0200-Runtime-Architecture.md)
- [RFC-0206 — Execution Engine](../300-runtime/RFC-0206-Execution-Engine.md)
- [RFC-0402 — Packaging](../500-build-distribution/RFC-0402-Packaging.md)
- [RFC-0403 — Registry and Marketplace](../500-build-distribution/RFC-0403-Registry-and-Marketplace.md)
- [RFC-0801 — Signing and Integrity](RFC-0801-Signing-and-Integrity.md)
- [RFC-0802 — Governance](RFC-0802-Governance.md)
```

- [ ] **Step 2: Run validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Confirm SUMMARY still points to RFC-0800**

Run:

```bash
grep -n 'RFC-0800 — Security Model' SUMMARY.md
```

Expected: output includes `docs/900-security-governance/RFC-0800-Security-Model.md`.

- [ ] **Step 4: Commit RFC rewrite**

Run:

```bash
git add docs/900-security-governance/RFC-0800-Security-Model.md
git commit -m "docs(rfc): viet lai RFC-0800 Security Model"
```

Expected: one commit containing only `docs/900-security-governance/RFC-0800-Security-Model.md`.

---

## Task 3: Close Pending Reference and Final Verification

**Files:**
- Modify: `TODO.md`
- Verify: `SUMMARY.md`
- Verify: `tools/check-rfc-links`

- [ ] **Step 1: Mark RFC-0800 pending delegation complete**

In `TODO.md`, change this line:

```markdown
- [ ] RFC-0800 Security Model — nhà của Security Considerations
```

to:

```markdown
- [x] RFC-0800 Security Model — nhà của Security Considerations
```

- [ ] **Step 2: Run full docs validator**

Run:

```bash
bash tools/check-rfc-links
```

Expected: `OK: all RFC overview checks passed`, exit code `0`.

- [ ] **Step 3: Verify only intended files changed**

Run:

```bash
git status --short
```

Expected: only `TODO.md` is modified after Task 1 and Task 2 commits.

- [ ] **Step 4: Commit closure note**

Run:

```bash
git add TODO.md
git commit -m "docs: dong pending RFC-0800 security model"
```

Expected: one commit containing only `TODO.md`.

---

## Definition of Done

- `tools/check-rfc-links` includes RFC-0800-specific guardrails.
- `docs/900-security-governance/RFC-0800-Security-Model.md` contains `Security Principles`, `Trust Boundaries`, `Required Controls`, `Validation Hooks`, `Failure Behavior`, `Examples` and `References`.
- `docs/900-security-governance/RFC-0800-Security-Model.md` no longer contains boilerplate headings `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Required Behavior`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Compatibility`, `Migration Guidance` or `Future Work`.
- RFC-0800 states manifests MUST NOT contain plaintext secrets.
- RFC-0800 states remote registries, packages and execution targets are not trusted by default.
- RFC-0800 states execution MUST use declared adapters and capabilities.
- RFC-0800 maps security policy failures to `POLICY_VIOLATION` diagnostics.
- `TODO.md` marks the RFC-0800 pending delegation complete.
- `bash tools/check-rfc-links` passes with exit code `0`.
- `SUMMARY.md` still references `docs/900-security-governance/RFC-0800-Security-Model.md`.
- Related RFCs such as `RFC-0801`, `RFC-0802` and `RFC-0206` remain unchanged in this plan.