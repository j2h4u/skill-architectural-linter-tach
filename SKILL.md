---
name: architectural-linter-tach
license: LICENSE.md
description: Audit, classify, design, introduce, migrate, enforce, and continuously improve Python architecture governance with Tach and focused complementary checks. Use when assessing an unfamiliar Python repository, choosing a greenfield or brownfield adoption strategy, evaluating or replacing import-linter coverage, configuring Tach, turning architecture rules into required CI gates, reviewing feature boundaries, or incrementally refactoring a codebase toward a clean modular monolith.
---

# Govern Python Architecture with Tach

Use Tach as the primary executable architecture spine, not as a one-time
dependency diagram. The goal is architecture governance: keep the protected
branch green, block architectural regression, and make each cleanup slice
shrink explicit debt.

Tach is often the best core tool, but it is not the whole governance system.
When an invariant is about product ownership, side effects, external technology
ownership, or runtime semantics, evaluate whether Tach can express it precisely
before adding a focused complementary check.

## Refresh Tach Knowledge First

At every invocation, attempt a lightweight live documentation refresh before
relying on remembered Tach behavior:

1. Read [references/tach-live-reference.md](references/tach-live-reference.md).
2. If Tach is installed in the target repository, run `tach --version` and read
   the help for commands relevant to the task.
3. If web access is available, open the official overview, current
   configuration or command page relevant to the task, and the canonical
   repository or release page when version drift matters.
4. Compare current documentation with the installed version. Configure for the
   installed version unless the user also authorizes an upgrade.
5. Note newly discovered capabilities or changed semantics that materially
   affect the work.

If Tach is not installed or the user asks how to install it, read
[references/installing-tach.md](references/installing-tach.md) and adapt its
commands to the repository's package manager.

Do not browse every documentation page mechanically. Read the overview plus the
few pages needed for the current decision. If live documentation is
unavailable, proceed from installed CLI help and the bundled reference, and
state the limitation when it affects confidence.

## Preserve the Truth Boundary

Treat Tach as an import-architecture checker. It can enforce module membership,
dependency direction, cycles, public interfaces, visibility, and selected
third-party dependency rules.

Do not claim that green Tach proves the whole architecture is clean. Static
import analysis does not prove:

- runtime wiring or dependency injection;
- ownership of database writes, network clients, processes, or mutable state;
- behavior hidden behind dynamic imports or reflection;
- configuration provenance and policy placement;
- semantic cohesion inside a module.

Add focused tests or repository-specific checks for non-import invariants and
keep them beside Tach in the quality gate.

When the task involves external libraries, standard-library capabilities, raw
SQL, environment access, process boundaries, or ownership of side effects, read
[references/complementary-architecture-tools.md](references/complementary-architecture-tools.md)
before deciding whether Tach alone is sufficient.

## Lifecycle

First audit and classify the repository so the target boundaries come from
product and runtime ownership. Then adopt Tach as a green enforced gate using
the least permissive migration strategy the codebase can sustain. After
adoption, use the gate during every structural change and continuously shrink
interfaces, dependency edges, and explicit architecture debt.

## Select the Operating Mode

Choose the mode before changing files.

### Audit

Use audit mode when the user asks what an unfamiliar repository is, what to do
next, or whether its architecture is healthy.

- Keep the audit read-only unless implementation is also requested.
- Read [references/audit-checklist.md](references/audit-checklist.md) fully.
- Use [assets/architecture-audit.template.md](assets/architecture-audit.template.md)
  for the result.
- Do not run `tach init`, `tach mod`, or `tach sync`; they write or normalize
  configuration.
- Classify lifecycle, deployment topology, internal organization, and
  governance maturity independently.
- Finish with a target direction, gate strategy, and three bounded actions.

### Adopt or Migrate

Use this mode when installing Tach, replacing another architecture linter, or
making Tach a hard gate. Audit enough of the repository to choose one of the
adoption strategies below, then implement and validate the full gate.

### Guide Feature Development

Use this mode when a feature must fit an existing Tach-governed architecture.
Determine ownership and intended dependency edges before implementation. Treat
Tach failures as architecture feedback, not configuration chores.

### Improve Existing Architecture

Use this mode when removing cycles, deep imports, broad interfaces, god
modules, unchecked scopes, or other structural debt. Work in small,
behavior-preserving slices and tighten Tach in the same change.

## Derive Architecture from Product Ownership

Derive modules from product capabilities, ownership, change cadence, and stable
APIs rather than copying the current directory tree.

Distinguish these responsibilities where they genuinely exist:

- inbound delivery or transports;
- process composition and concrete wiring;
- application use cases and orchestration;
- product capabilities or domains;
- technology-facing adapters;
- persistence adapters and state ownership;
- small neutral contracts and foundational primitives.

Do not force these exact layers onto every repository. Prefer the simplest
model that expresses real dependency direction and ownership.

Treat deployment topology and internal modularity as orthogonal. A microservice
can be internally tangled; a monolith can be cleanly modular. Verify
independent deployment, lifecycle, ownership, and data boundaries before
calling something a microservice. Do not recommend distributed extraction as a
substitute for establishing internal module boundaries.

## Choose an Adoption Strategy

### Greenfield

Write the intended module graph before feature code spreads. Enable strict
source coverage, cycle checks, interfaces, and CI immediately. Keep
`tach check` green from the first protected commit.

### Brownfield with Import-Linter

Treat the work as a replacement migration:

1. Inventory each existing contract and the boundary it protects.
2. Discard obsolete scaffolding and tool-specific workarounds.
3. Express useful coverage with Tach modules, layers, interfaces, visibility,
   and dependency rules.
4. Identify any useful import-linter coverage Tach does not express precisely,
   especially forbidden external-package ownership or custom contracts.
5. Replace those gaps with Tach if current docs support it, or with a focused
   complementary check.
6. Run both tools only while replacement coverage is incomplete.
7. Remove import-linter, its dependency, configuration, cache handling, and CI
   commands once useful coverage is equivalent or stronger.

Do not assume Tach completely supersedes import-linter. They overlap heavily on
first-party dependency governance, but import-linter can still be the better
fit for some forbidden external-package rules until Tach grows equivalent
semantics or a smaller repository-specific check replaces the contract.

Do not maintain both tools indefinitely without a named coverage gap and a
removal condition.

### Legacy Without Architecture Linting

Prefer a green enforced frontier:

1. Model the most valuable runtime and product boundaries first.
2. Configure legacy debt as the smallest practical modules.
3. Mark only narrow debt scopes `unchecked` when they cannot yet satisfy the
   target graph.
4. Keep cleaned and new modules fully checked.
5. Shrink or remove unchecked scopes in each cleanup slice.
6. Require `tach check` in CI from the first adoption merge.

Avoid broad unchecked packages because new files silently inherit the waiver.
Prefer explicit module coverage and `root_module = "forbid"` once practical.

### Current-State Ratchet

Use `tach sync` for a green current-state baseline only when an enforced
frontier is impractical. Mark known-wrong dependencies as deprecated and record
their removal plan.

Recognize that a deprecated edge remains allowed and does not prevent new uses
of the same dependency. Prefer the enforced frontier when possible.

### Target Exploration

Use a deliberately red ideal graph only as a short-lived design artifact.
Never leave it red on the protected branch and call it a quality gate. Convert
it into a green enforced frontier before adoption.

Never run `tach sync` against an ideal target graph merely to make it green.
That approves accidental dependencies instead of governing them.

## Configure for Governance

Verify exact semantics in current documentation, then choose deliberately:

- Set exact `source_roots`.
- Prefer `root_module = "forbid"` when first-party files are explicitly
  modelled.
- Set `forbid_circular_dependencies = true`.
- Set `ignore_type_checking_imports = false`; type-only imports still couple
  modules.
- Require reasons for ignore directives and reject unused ignores in a mature
  gate.
- Consider `layers_explicit_depends_on = true` when every cross-layer edge must
  be visible in review.
- Use closed layers when callers must not bypass an application or mediation
  layer.
- Keep `exact = false` during coarse brownfield modelling; enable it when
  unused declared edges are actionable.
- Use `utility = true` rarely.
- Define narrow public interfaces for modules with stable consumers.
- Use interface visibility or exclusivity for consumer-specific APIs.
- Move mature independently owned domains into `tach.domain.toml` and protect
  their configuration with `CODEOWNERS`.

Omitting `depends_on` is permissive. `unchecked = true` is broader because it
also bypasses interface enforcement for imports made by that module. Use
neither as an unexplained convenience.

For monorepos, declare every real source root. Use `tach check` for internal
module boundaries and evaluate `tach check-external` for dependencies between
distributions. Do not confuse packaging boundaries with product domains.

Use `tach check-external` for packaging correctness: imported external modules
must be satisfied by declared package dependencies. Do not treat it as proof
that the right internal layer owns each external technology.

## Make Tach a Hard Quality Gate

Require all of the following:

1. `tach check` passes on the protected branch and every pull request.
2. Its CI job is a required status check in branch protection or repository
   rulesets.
3. Tach configuration and governance scripts have explicit review ownership.
4. `tach sync` never runs automatically in CI; CI validates rather than
   approving discovered imports.
5. Changes that widen an exception are visible and reviewed as architecture
   changes.

Treat these edits as potential weakening:

- adding or widening `unchecked`;
- adding deprecated dependencies;
- adding or broadening utilities;
- adding inline ignores or exclusions;
- expanding public interfaces or visibility;
- removing `cannot_depend_on`;
- weakening layers, root coverage, exactness, or cycle checks.

Where review discipline is insufficient, add a repository-specific policy
check that compares the base and proposed Tach configuration and rejects
unapproved weakening. Evaluate semantics rather than a raw waiver count.

Where Tach cannot express a hard invariant, add a narrow complementary gate
instead of broadening Tach configuration until it passes. Prefer exact owner
ratchets over numeric thresholds when the question is "which module is allowed
to own this dependency or side effect?"

Use pre-commit for fast local feedback, but keep CI authoritative. A command in
a task runner is not governance without required checks and protected merge
rules.

## Maintain Architecture After Adoption

Once Tach is installed and enforced, read
[references/continuous-architecture-maintenance.md](references/continuous-architecture-maintenance.md)
for feature work, new modules, dependency or public-interface changes, Tach
violations, and continuous cleanup.

Keep feature behavior and its architecture declaration in the same change.
Run Tach before broad tests. Do not approve a new edge merely to clear a
failure; first justify its ownership and direction.

Prefer small behavior-preserving cleanup slices. Keep every temporary exception
owned, bounded, and moving toward removal.

## Validate Completion

Do not report an adoption or migration complete until:

- the protected Tach check is green and required in CI;
- new first-party files are covered by the intended module model;
- cycles are forbidden;
- interfaces and dependency directions reflect accepted architecture;
- exceptions are explicit and bounded;
- import-linter is removed only once useful coverage is replaced;
- non-import invariants retain appropriate enforcement;
- the cleanup queue names the next structural slice.

For a feature or cleanup, report intended ownership, edges added and removed,
interfaces or waivers changed, remaining debt, and validation commands.

## Avoid Failure Modes

- Keeping a permanently red advisory graph on the protected branch.
- Running `tach sync` to bless imports introduced by a feature.
- Translating every import-linter rule without reviewing its meaning.
- Going green through broad unchecked, utility, interface, or ignore expansion.
- Treating layers as folder labels rather than dependency rules.
- Modelling every file before identifying product boundaries.
- Adding speculative ports, repositories, or service classes.
- Treating Tach as proof of runtime ownership or semantic cohesion.
- Removing import-linter before replacing its useful non-overlapping coverage.
- Leaving import-linter beside Tach after migration coverage is complete.
