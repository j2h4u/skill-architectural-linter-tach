# Architecture Audit Checklist

Use this reference only for read-only architecture audits and initial adoption
assessment. Use `../assets/architecture-audit.template.md` for the final
artifact.

## Contents

- Audit discipline
- Evidence checklist
- Classification
- Interpretation rules
- Recommendation rules

## Audit Discipline

- Keep source, configuration, CI, and external systems unchanged unless the
  user also requests implementation.
- Do not run `tach init`, `tach mod`, or `tach sync`.
- If Tach already exists, run `tach check`, `tach report`, and `tach map`
  read-only.
- If Tach is absent, inspect imports and runtime boundaries directly. Use a
  temporary copy only when Tach analysis adds material evidence.
- Treat source, tests, and live runtime as stronger evidence than aspirational
  architecture documents.
- Separate observed facts from inference and assign confidence.
- Prefer a prioritized diagnosis over an exhaustive file inventory.

## Evidence Checklist

Mark each item `confirmed`, `unknown`, or `not applicable`. Cite important
evidence.

### Repository and Runtime

- [ ] Identify source roots, packages, namespace packages, generated code, and
      test roots.
- [ ] Identify deployable units, entry points, processes, jobs, libraries, and
      CLI surfaces.
- [ ] Identify composition roots and lifecycle owners.
- [ ] Identify owners of network clients, database writes, files, queues,
      caches, and mutable state.
- [ ] Identify inbound transports, external systems, and downstream consumers.
- [ ] Verify whether one repository contains one or several independently
      releasable distributions.

### Product and Capability Boundaries

- [ ] Identify user-facing capabilities or domains that change independently.
- [ ] Identify the vocabulary and data owned by each candidate capability.
- [ ] Find operations that cross several candidate capabilities.
- [ ] Distinguish orchestration from domain behavior.
- [ ] Identify compatibility-sensitive public APIs.

### Dependency Structure

- [ ] Map imports between the largest dependency clusters.
- [ ] Find circular dependencies.
- [ ] Find upward or layer-bypassing dependencies.
- [ ] Find cross-capability imports.
- [ ] Find deep imports and imports of private members.
- [ ] Find framework, transport, or database types leaking into neutral code.
- [ ] Find type-only imports that create concrete coupling.
- [ ] Identify high fan-in and high fan-out modules.

### Ownership Smells

- [ ] Find god modules and composition roots containing business behavior.
- [ ] Find generic `utils`, `common`, `shared`, or `helpers` areas with mixed
      ownership.
- [ ] Find duplicated contracts and speculative abstractions.
- [ ] Find concrete adapters imported outside composition or application seams.
- [ ] Find multiple writers for state that should have one owner.
- [ ] Find operator-controlled policy or defaults outside configuration
      ownership.

### Existing Governance

- [ ] Inspect `tach.toml` and all `tach.domain.toml` files.
- [ ] Inspect import-linter or other architecture contracts.
- [ ] Inspect custom architecture checks.
- [ ] Inspect CI, pre-commit, task-runner commands, and local hooks.
- [ ] Inspect branch protection or repository rulesets and required checks.
- [ ] Inspect `CODEOWNERS`, ADRs, and architecture-debt records.
- [ ] Determine whether the architecture gate is green, red advisory, partial,
      bypassable, or absent.
- [ ] Inventory unchecked modules, deprecated dependencies, utilities,
      exclusions, ignores, and permissive missing `depends_on` declarations.
- [ ] Identify important invariants that import analysis cannot enforce.

### Evolution Pressure

- [ ] Identify areas with frequent changes or repeated regressions.
- [ ] Identify boundaries whose violation risks remote side effects, state
      corruption, security, or deployment coupling.
- [ ] Identify a small candidate for the first fully protected module.
- [ ] Identify cleanup slices that reduce coupling without a broad rewrite.
- [ ] Check whether proposed new features are likely to deepen existing debt.

## Classification

Classify on independent axes.

### Lifecycle

- `greenfield`: little or no production or compatibility history.
- `brownfield`: existing behavior and migration constraints.
- `blackfield`: temporary intake label for opaque brownfield code. Replace it
  after collecting evidence.

### Deployment Topology

- library;
- CLI;
- single-process service;
- multi-process service;
- one deployable monolith;
- multi-package monorepo;
- several independently deployable services.

Do not infer "microservice" from repository size, a Dockerfile, or naming.
Verify deployment, lifecycle, ownership, and data independence.

### Internal Organization

- coherent small application;
- layered monolith;
- vertical-slice monolith;
- modular monolith;
- tangled monolith;
- hybrid.

Deployment topology does not determine internal modularity.

### Governance Maturity

- absent;
- documented-only;
- red advisory;
- partially enforced frontier;
- green hard gate;
- green hard gate with owned, shrinking debt.

### Adoption Readiness

- strict greenfield gate;
- import-linter replacement migration;
- brownfield enforced frontier;
- current-state ratchet;
- short target-design spike before a green frontier.

## Interpretation Rules

- Do not call a dependency wrong merely because it crosses a directory.
- Do not call an architecture modular merely because folders have domain names.
- Do not treat a green linter as proof of runtime ownership.
- Do not treat every cycle as equivalent; prioritize cycles that mix ownership
  or prevent isolated change.
- Do not recommend microservice extraction to hide internal tangling.
- Preserve useful existing seams even when naming or layout is imperfect.
- Prefer product and side-effect boundaries over aesthetic layering.
- Mark missing runtime or product-owner evidence as unknown.

## Recommendation Rules

The audit must finish with:

1. a concise verdict;
2. the four-axis classification with evidence and confidence;
3. strengths worth preserving;
4. hard-boundary risks;
5. structural and governance debt;
6. the smallest credible target direction;
7. the appropriate Tach adoption strategy;
8. three bounded actions, each with a completion test;
9. unresolved questions that materially affect the direction.

Keep the first action executable. It should establish or tighten one valuable
boundary, not "refactor architecture."
