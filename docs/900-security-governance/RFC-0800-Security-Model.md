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