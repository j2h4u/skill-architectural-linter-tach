# Continuous Architecture Maintenance

Use this reference after Tach is installed, green, and enforced. The goal is to
keep ordinary development aligned with the architecture while continuously
improving existing boundaries.

## Contents

- Steady-state rule
- Every pull request
- Adding or changing modules
- Changing dependencies
- Maintaining public interfaces
- Maintaining semantic gates
- Responding to failures
- Continuous cleanup
- Governance maintenance

## Steady-State Rule

Keep `tach check` green on the protected branch. Change feature code and its
architecture declaration together. Never run `tach sync` as an automatic repair
for a feature branch.

Treat `tach.toml`, `tach.domain.toml`, public interfaces, and architecture
waivers as governed code. Review their semantic effect, not only syntax.

Treat complementary architecture checks the same way. A custom AST gate,
import-linter contract retained during migration, Grimp query, or architecture
test is part of governance when it blocks merges.

## Every Pull Request

Before implementation, answer:

1. Which capability owns the behavior?
2. Which module owns its data and side effects?
3. What is the inbound use case or public API?
4. Which existing modules may it depend on?
5. Which concrete adapters are wired in composition?
6. Does the change add, remove, or expose a dependency edge?
7. Does the change introduce or move direct use of a framework, external
   client, database API, file/process API, or other side-effect technology?

During implementation:

- keep delivery and framework adapters thin;
- keep concrete wiring in composition roots;
- expose only API required by real consumers;
- introduce a port only at a genuine variation, ownership, testing, or
  side-effect boundary;
- run `tach check` before broad tests;
- run any complementary semantic gates before broad tests;
- use `tach report` or `tach map` when the blast radius is unclear.

In the PR description, state architecture impact when non-trivial:

- module ownership;
- edges added or removed;
- public interface changes;
- waivers added or removed;
- nearby debt intentionally left.
- complementary gate changes, if any.

## Adding or Changing Modules

Create a module boundary when the code has an independently explainable
responsibility, ownership, change cadence, or public contract. Do not create a
module merely because a new file or directory exists.

When adding a module:

1. Write its one-sentence responsibility.
2. Select its layer or domain ownership.
3. Declare only required dependencies.
4. Define a narrow public interface when it has external consumers.
5. Set visibility when only selected modules should consume it.
6. Verify it does not create a cycle or bypass a closed layer.
7. Add ownership in `CODEOWNERS` when responsibility is team-specific.

Split a module when it contains responsibilities that need different dependency
directions or public surfaces. Merge modules when the boundary creates ceremony
without independent ownership or change.

## Changing Dependencies

Treat a new dependency edge as an architecture decision.

Before declaring it, ask:

- Is the direction consistent with product and runtime ownership?
- Does the caller need a public use case rather than an implementation detail?
- Does a neutral contract belong with the lower-level owner?
- Should orchestration move upward?
- Is dependency inversion justified by a real volatile or side-effect boundary?

Declare the edge only when the dependency is intentional. Otherwise move the
abstraction, extract a neutral contract, add a facade, invert through a port, or
move concrete wiring to composition.

Remove unused or obsolete edges as soon as they become visible. Enable exact
checking when this signal is actionable for the repository.

## Maintaining Public Interfaces

Treat a public interface as a compatibility promise.

- Design it from consumer use cases, not implementation contents.
- Prefer a stable facade over deep imports.
- Keep framework, storage, and transport types out of neutral contracts.
- Expand the interface only for a real consumer.
- Narrow it when consumers no longer require a symbol.
- Use visibility or exclusive interfaces for privileged consumers.
- Move externally imported leading-underscore symbols into intentional public
  contracts instead of ignoring the violation.

Review interface expansion as potential architecture weakening even when Tach
passes.

## Maintaining Semantic Gates

Use Tach for the main first-party dependency contract. Use focused
complementary gates for important invariants Tach cannot currently observe,
such as external technology ownership, environment reads, raw SQL placement,
process spawning, file writes, or runtime composition rules.

Before changing a semantic gate:

- check whether current Tach capabilities now cover the rule;
- prefer removing exceptions over adding them;
- treat new allowlist entries as architecture decisions;
- fail on stale allowlist entries when the gate uses exact ownership;
- keep zero-owner rules explicit when a capability should remain unowned;
- state what the gate does and does not prove.

Prefer exact owner lists for brownfield technology ownership. Avoid count-based
ratchets when the identity of the owning module matters.

Do not widen a semantic gate just to make CI pass. If a new direct owner is
legitimate, record why that module owns the capability, add focused tests for
the accepted shape, and update the rule in the same change. If the new owner is
not legitimate, move the dependency behind the existing port, adapter, or
composition boundary instead.

For zero-owner policies, a failure usually means the feature is crossing a
boundary that has not been designed yet. The fix is normally to introduce or
reuse an owned adapter seam, not to add the first allowlist entry casually.

## Responding to Failures

Do not immediately edit `depends_on`, broaden an interface, add `unchecked`, or
write an ignore.

Classify the failure first:

- legitimate missing dependency;
- wrong dependency direction;
- private implementation import;
- abstraction owned by the wrong module;
- concrete adapter leaking upward;
- orchestration placed too low;
- module too broad;
- cycle exposing mixed ownership.

Fix the ownership or seam when the edge is wrong. Update configuration only
when the model was incomplete and the dependency is intended.

## Continuous Cleanup

Prefer opportunistic cleanup near touched code plus occasional focused slices.

For one bounded slice:

1. Select a cycle, deep-import cluster, broad interface, deprecated edge,
   unchecked scope, or high fan-in/fan-out hotspot.
2. Inspect dependencies and dependents.
3. Preserve behavior with focused tests.
4. Apply the smallest structural correction.
5. Tighten Tach in the same change.
6. Remove the corresponding waiver or debt record.

Moving files without changing responsibility, dependency direction, or public
surface is not an architecture improvement.

Record temporary debt with exact scope, reason, target boundary, owner, and
removal condition. Keep waiver scope monotonically shrinking.

## Governance Maintenance

On Tach upgrades:

1. Refresh official documentation and release notes.
2. Compare new behavior with the installed configuration.
3. Run the gate before and after the upgrade.
4. Evaluate new enforcement capabilities before adding custom checks.
5. Keep repository-specific checks for invariants Tach still cannot observe.
6. Remove or simplify complementary checks when Tach can express the invariant
   with equivalent or stronger semantics.

Periodically review:

- whether every first-party file remains covered;
- whether utilities or unchecked scopes have grown;
- whether deprecated dependencies or ignores can be removed;
- whether public interfaces are broader than current consumers need;
- whether domain ownership belongs in `tach.domain.toml`;
- whether CI remains required and branch protection still applies.
- whether complementary semantic gates still cover real risks or have become
  obsolete after architecture cleanup or Tach upgrades.

Require an ADR or equivalent explicit approval for changes to product
ownership, dependency direction, layer semantics, public module APIs, or
composition roots. Routine edges that already fit the accepted model need
normal review, not an ADR.
