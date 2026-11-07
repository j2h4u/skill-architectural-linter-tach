# Complementary Architecture Tools

Use this reference when Tach is already present or under consideration, but the
architecture concern is broader than first-party module dependencies.

## Goal

The goal is not to install one favorite linter. The goal is executable
architecture governance: module boundaries, ownership of side effects,
technology isolation, public contracts, explicit debt, and hard CI enforcement.

Tach should usually be the spine of this system. Complement it only when the
invariant cannot be expressed precisely with Tach's current installed version.

## Coverage Matrix

Before adding or removing a tool, classify each important invariant:

| Invariant | Preferred enforcement |
|---|---|
| First-party module graph, cycles, layers, public interfaces | Tach |
| Stale declared internal dependencies | Tach `--exact` |
| Package dependency declarations for external imports | Tach `check-external`, deptry, package smoke tests |
| Forbidden external package imports by internal owner | Tach if current docs support it precisely; otherwise import-linter, pytest-archon, or a focused custom gate |
| Exact brownfield owner ratchet with stale-entry failure | Focused custom AST or Grimp-based gate |
| Raw SQL, environment reads, file writes, subprocess use, network-client ownership | Focused AST/semantic gate or architecture tests |
| Runtime wiring, process ownership, deployment behavior | Runtime smoke tests, integration tests, deployment checks |
| Product semantics and data ownership | Domain tests, contract tests, ADRs plus code gates where observable |

Do not call the architecture green until the important invariants are either
covered, explicitly out of scope, or recorded as owned debt.

## Tool Roles

### Tach

Use Tach for the central first-party architecture contract: source roots,
modules, dependencies, cycles, layers, public interfaces, visibility,
deprecated dependencies, unchecked scopes, and domain-owned configuration.

Use `tach report --external` for inventory and `tach check-external` for
packaging correctness. These commands show or validate external imports, but do
not by themselves prove that the correct internal layer owns a specific
external technology.

### Import Linter

Import-linter overlaps with Tach for dependency governance and layers. It can
still be useful when an existing repository relies on contracts Tach does not
yet express with the same precision.

Important retained use cases:

- forbidden imports involving external root packages;
- custom contracts;
- migration audits where existing contracts encode business-specific
  constraints not yet modelled in Tach;
- graph exploration during a replacement migration.

When replacing import-linter, do not remove it merely because Tach is green.
Inventory every useful contract and replace the value it provided. Remove
import-linter only after Tach or complementary checks cover the useful
semantics.

### Grimp

Use Grimp when a custom graph query is justified and raw AST scanning is too
low-level. It can build a queryable import graph, optionally including external
packages and type-checking imports.

Prefer Grimp over a custom AST scanner when the rule needs graph semantics,
transitive queries, or module-expression matching. Prefer a small AST scanner
when the rule is a simple local import or call-site ownership check and the
repository benefits from no new dependency.

### Architecture-Test Libraries

Libraries such as PyTestArch, ArchUnitPython, and pytest-archon express
architecture rules as tests. They can be useful for teams that prefer
pytest-native fitness functions or need rules that combine graph assertions
with repository-specific test fixtures.

Evaluate maturity, maintenance activity, diagnostics, and CI cost before
adopting one as a hard gate. Avoid scattering the source of architectural
truth across many test files without a clear ownership model.

### Flake8 Import Plugins

Plugins such as flake8-import-graph or flake8-import-rules can block selected
imports, including external technology leaks. Use them only when the repository
already standardizes on Flake8 and the plugin is actively maintained enough for
the team's risk tolerance.

## Focused Semantic Gates

Add a repository-specific gate when all of the following are true:

1. the invariant is important enough to block merges;
2. Tach cannot currently express it precisely;
3. an off-the-shelf tool would add more surface area than the rule needs;
4. the rule can be stated narrowly and tested cheaply.

Good candidates:

- "Only these modules may import `sqlalchemy`, `sqlite3`, `telethon`, `boto3`,
  `openai`, or another concrete adapter technology."
- "No module may import `subprocess`, `socket`, or another high-risk escape
  hatch until the architecture explicitly assigns an owner."
- "Only configuration/composition modules may read environment variables."
- "Only persistence modules may execute SQL."
- "Only deployment/runtime modules may start subprocesses or open sockets."
- "Generated files or migration modules may use special APIs, but ordinary
  application code may not."

For import-ownership rules, prefer:

- AST parsing, not grep;
- exact module-owner allowlists, not counts;
- failure on both unexpected owners and stale allowlist entries;
- diagnostics with path, line, dependency, and category;
- tests for aliases, dotted imports, nested imports, type-checking imports,
  comments, and strings;
- explicit non-claims in the script docstring or nearby documentation.

Avoid numeric ratchets when exact ownership matters. A count can stay unchanged
while one violation disappears and another appears. Exact owner sets make drift
visible.

### Staged Adoption

Introduce semantic gates in small, reviewable stages:

1. Start with high-signal external technology ownership, such as direct imports
   of database drivers, framework clients, SDKs, or Telegram/API clients.
2. Add zero-owner rules for high-risk capabilities that have no legitimate
   production owner today, such as direct process spawning or raw socket use.
3. Defer broad semantic rules until the repository has a precise contract for
   them. Raw SQL execution, environment access, and file IO often need a clearer
   call-site model before they make good hard gates.
4. Prefer tightening one important invariant per change over building a large
   advisory scanner nobody understands.

A zero-owner rule is an explicit empty allowlist. It is useful when the current
architecture should have no direct owner for a capability and any future use
should be a deliberate design decision. This is stronger than omitting the
capability from the gate because CI will fail on the first accidental direct
import.

Add or expand a semantic gate only when:

- current owners are reviewed and exact, or the intended owner set is empty;
- false positives are expected to be rare;
- the capability is a real architecture escape hatch;
- the diagnostic tells the developer where to move the code or which owner to
  consult;
- tests include adjacent legitimate patterns that must not trigger the rule.

Do not expand a hard gate yet when the invariant is still vague. Common warning
signs:

- broad `os` or `pathlib` imports where most usage is benign;
- file APIs without a read/write or state-directory ownership distinction;
- raw SQL scans that cannot distinguish migration/bootstrap code from
  persistence code;
- network or process words in comments, strings, or documentation;
- runtime concepts that share names with low-level modules but are not the same
  architectural dependency, for example Unix-socket IPC through an approved
  async transport rather than direct `socket` imports.

## Non-Claims

A complementary gate should state what it does not prove. For example, an
external import owner ratchet does not prove:

- the allowlisted modules are architecturally clean;
- the allowed modules are not accumulating more behavior;
- indirect type leakage has been eliminated;
- raw SQL or network calls are fully isolated;
- runtime wiring respects the intended ownership;
- dynamic imports or reflection are covered.

These limitations are acceptable when the gate is presented as one narrow
fitness function rather than as proof of the whole architecture.

## Decision Procedure

1. Refresh Tach docs and installed CLI help.
2. Check whether Tach can express the invariant directly.
3. If replacing import-linter, inspect the existing contract's actual value.
4. Consider import-linter, Grimp, or an architecture-test library for the gap.
5. Choose the smallest hard gate that produces clear diagnostics and can live
   in CI.
6. Record the invariant, owner, exceptions, non-claims, and removal condition.
7. Keep the gate beside Tach in the same quality workflow.

The default answer should be "Tach plus the smallest precise complement", not
"one tool must prove everything."
