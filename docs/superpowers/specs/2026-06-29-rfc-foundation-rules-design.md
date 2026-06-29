# RFC Foundation Rules Design

**Date:** 2026-06-29  
**Status:** Approved for implementation planning  
**Scope:** `RFC-0005 — Versioning`, `RFC-0006 — Naming Convention`, `RFC-0104 — Dependency Graph`

---

## 1. Context

The overview RFCs already forward-reference three rule-level RFCs that still contain generic boilerplate:

- `RFC-0005 — Versioning`
- `RFC-0006 — Naming Convention`
- `RFC-0104 — Dependency Graph`

These RFCs are foundational rules used by compatibility, repository layout, canonical resource metadata, validation, dependency resolution and packaging. They should be completed as one coherent design because version ranges, stable names and dependency edges are tightly coupled.

## 2. Design Goal

Rewrite the three RFCs as a focused foundation-rules cluster:

- `RFC-0005` owns version syntax, SemVer semantics and range vocabulary.
- `RFC-0006` owns naming grammar and stable identity rules.
- `RFC-0104` owns resource-level dependency graph structure and graph invariants.

The cluster should remove boilerplate sections and avoid redefining canonical resource envelopes, validation diagnostics, resolver algorithms, lock files or package formats.

## 3. Approved Approach

Use the **Foundation Split** approach.

### 3.1 RFC-0005 — Versioning ownership

`RFC-0005` should define:

- SemVer-like version format for ARPS specifications, resources, schemas, adapters, packages and runtime components.
- Stable interpretation of `MAJOR.MINOR.PATCH`.
- Optional prerelease and build metadata vocabulary when present.
- Version range grammar sufficient for dependency declarations.
- Change classification mapping for when authors SHOULD bump major, minor or patch.
- Relationship to `RFC-0007`: compatibility classification is owned by `RFC-0007`; versioning consumes that classification for bump decisions.

Required sections:

- `Version Format`
- `Versioned Subjects`
- `Version Ranges`
- `Change Rules`
- `Pre-release and Build Metadata`
- `Examples`
- `References`

Required rules:

- Breaking changes require a major version bump.
- Compatible additive changes should use a minor version bump.
- Patch changes must preserve compatibility and meaning.
- Version ranges must not be resolved by this RFC; selection belongs to dependency resolver RFCs.

### 3.2 RFC-0006 — Naming Convention ownership

`RFC-0006` should define:

- Stable grammar for namespaces, resource names, `metadata.id`, resource kinds, file slugs and package slugs.
- Case and character constraints for portable IDs.
- Relationship between `metadata.id`, `metadata.name`, `kind` and repository paths.
- Rename behavior: changing `metadata.id` changes resource identity and is compatibility-sensitive.
- Alias/deprecation guidance without defining registry mechanics.

Required sections:

- `Naming Principles`
- `Identifier Grammar`
- `Resource Identity`
- `Kinds and Slugs`
- `Rename and Alias Rules`
- `Examples`
- `References`

Required rules:

- `metadata.id` should be globally stable inside a registry namespace.
- Resource IDs should use `namespace/name` form.
- Names and slugs should be lowercase and portable.
- Changing `metadata.id` is a breaking identity change unless an alias or migration path preserves consumers.
- File paths may mirror names but must not be the source of resource identity.

### 3.3 RFC-0104 — Dependency Graph ownership

`RFC-0104` should define resource-level dependency graph semantics only.

It should define:

- Graph nodes as canonical resources identified by `metadata.id` and version constraints.
- Directed dependency edges from a resource to the resources it requires.
- Dependency declaration shape at the conceptual level.
- Graph invariants: acyclic, deterministic, no unresolved required dependency.
- Validation expectations and diagnostics delegated to `RFC-0106`.

Required sections:

- `Graph Model`
- `Dependency Edge`
- `Dependency Constraints`
- `Graph Invariants`
- `Validation Behavior`
- `Examples`
- `References`

Required rules:

- Dependency graphs must be directed and acyclic.
- Required dependencies must identify a target resource ID and may constrain kind and version range.
- Missing, ambiguous or cyclic dependencies should produce `DEPENDENCY_ERROR` diagnostics through `RFC-0106`.
- Version selection and conflict resolution are out of scope and belong to `RFC-0204`.
- Lock snapshots are out of scope and belong to `RFC-0401`.

## 4. Out of Scope

Do not define these topics in this cluster:

- Canonical resource envelope fields; owned by `RFC-0100`.
- Metadata field schema details beyond naming implications; owned by `RFC-0102`.
- Diagnostic envelope shape; owned by `RFC-0106`.
- Resolver algorithm, version selection and conflict handling; owned by `RFC-0204`.
- Lock file snapshots and checksums; owned by `RFC-0401`.
- Package format and publishing mechanics; owned by `RFC-0402` and `RFC-0403`.
- Runtime flow; owned by `RFC-0200`.

## 5. Validator Design

Extend `tools/check-rfc-links` with a foundation-rules guard after the existing completed RFC guards.

The guard should:

- Verify header fields in `RFC-0005`, `RFC-0006` and `RFC-0104`.
- Reject boilerplate headings such as `Motivation`, `Goals`, `Non-Goals`, `Canonical Model`, `Required Behavior`, `Runtime Flow`, `Validation Rules`, `Error Model`, `Security Considerations`, `Migration Guidance` and `Future Work`.
- Require the section sets listed above for each RFC.
- Require key phrases or rules that prove ownership boundaries:
  - `Breaking changes require a major version bump`
  - `metadata.id` and `namespace/name`
  - `directed and acyclic`
  - `DEPENDENCY_ERROR`
  - `version selection` delegated to `RFC-0204`
  - `lock` delegated to `RFC-0401`

The guard should use exact heading checks where possible to avoid false positives in prose and references.

## 6. Implementation Plan Shape

The implementation plan should split work into four commits:

1. Add foundation-rules validator guard and verify it fails against current boilerplate.
2. Rewrite `RFC-0005 — Versioning` and verify the new guard still fails only for remaining RFCs.
3. Rewrite `RFC-0006 — Naming Convention` and verify the new guard still fails only for `RFC-0104`.
4. Rewrite `RFC-0104 — Dependency Graph`, tick the pending delegation in `TODO.md`, and run full validation.

This sequencing preserves fail-first evidence and keeps each RFC reviewable.

## 7. Acceptance Criteria

- `RFC-0005`, `RFC-0006` and `RFC-0104` no longer contain boilerplate ownership sections.
- Each RFC has its required sections and references related owner RFCs instead of duplicating their scope.
- `RFC-0104` stays resource-level only and does not define resolver algorithms.
- `tools/check-rfc-links` passes after the cluster is implemented.
- `TODO.md` marks the foundation-rules pending delegation complete.
- `SUMMARY.md` links remain unchanged.

## 8. Recommendation

Proceed with this cluster as the next implementation cycle. This closes the last pending forward-reference group from the overview RFC phase while keeping runtime resolver and build distribution details available for later RFC-specific cycles.
