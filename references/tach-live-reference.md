# Tach Live Documentation Reference

This file is a navigation and interpretation aid, not a frozen substitute for
the official documentation.

Last manually reviewed: 2026-07-26.

## Contents

- Live refresh protocol
- Canonical sources
- Capability map
- Complementary tool checkpoints
- Important semantics
- Command map
- Version and documentation drift

## Live Refresh Protocol

At each skill invocation:

1. Inspect the target repository's installed version with `tach --version`
   when available.
2. Inspect `tach <command> --help` for commands that will be used.
3. With web access, open:
   - the official overview;
   - configuration or command documentation relevant to the task;
   - the canonical GitHub repository or releases when version drift matters.
4. Check whether documentation describes capabilities or options absent from
   the installed version.
5. Design for installed behavior unless an upgrade is in scope.
6. Record only changes that materially alter the audit, configuration, or gate.

If the documentation site moves, follow the documentation link from the
canonical repository README. If browsing is unavailable, use CLI help and this
reference, then state that live documentation was not refreshed when relevant.

## Canonical Sources

| Topic | URL |
|---|---|
| Documentation home | https://docs.gauge.sh/ |
| Overview and capability list | https://docs.gauge.sh/getting-started/introduction/ |
| Why Tach | https://docs.gauge.sh/getting-started/why-tach/ |
| Getting started | https://docs.gauge.sh/getting-started/getting-started/ |
| Commands | https://docs.gauge.sh/usage/commands/ |
| Configuration | https://docs.gauge.sh/usage/configuration/ |
| Layers and closed layers | https://docs.gauge.sh/usage/layers/ |
| Public interfaces | https://docs.gauge.sh/usage/interfaces/ |
| Incremental unchecked modules | https://docs.gauge.sh/usage/unchecked-modules/ |
| Deprecated dependencies | https://docs.gauge.sh/usage/deprecate/ |
| FAQ and analysis limitations | https://docs.gauge.sh/usage/faq/ |
| Caching | https://docs.gauge.sh/usage/caching/ |
| VS Code integration | https://docs.gauge.sh/usage/vscode/ |
| Canonical source repository | https://github.com/tach-org/tach |
| Releases | https://github.com/tach-org/tach/releases |
| PyPI package | https://pypi.org/project/tach/ |
| Pre-commit integration | https://github.com/gauge-sh/tach-pre-commit |

Prefer official documentation, CLI help, source, and releases over third-party
articles.

## Capability Map

Re-check current documentation before relying on this list.

### Architecture Enforcement

- Declare first-party modules and allowed dependencies.
- Forbid dependencies with `cannot_depend_on`.
- Require explicit public interfaces and prevent deep imports.
- Restrict module and interface visibility.
- Define ordered and optionally closed layers.
- Require explicit cross-layer dependencies.
- Detect circular dependencies.
- Treat type-checking imports as visible coupling when configured.
- Require all source files to belong to explicit modules through root-module
  policy.
- Reject unused declared dependencies with exact checking.

### Incremental Adoption

- Mark narrow modules unchecked.
- Deprecate individual dependencies while surfacing their usages.
- Keep some code under a permissive root policy during early adoption.
- Split stable domains into local `tach.domain.toml` configuration.
- Assign domain configuration ownership through `CODEOWNERS`.
- Require reasons for inline ignores and detect unused ignores.

### Discovery and Visualization

- `tach report` shows dependencies, usages, and external imports for a path.
- `tach map` emits file dependency or dependent maps and closures.
- `tach show` visualizes the declared module graph.
- `tach mod` interactively selects modules and source roots.
- `tach sync` reconciles declared dependencies with observed imports.

### Packaging and Monorepos

- Multiple source roots support workspace and namespace-package layouts.
- `tach check-external` compares imports with package dependency declarations.
- External import renames and exclusions handle distribution/import-name
  mismatches.

## Complementary Tool Checkpoints

As of the last manual review, Tach can report external imports and validate
that they are declared as package dependencies, but the public docs do not show
a precise policy for "only these internal modules may import this external or
stdlib technology." Re-check current docs before assuming this gap still
exists.

If the target invariant is external technology ownership rather than package
dependency hygiene, evaluate:

- import-linter forbidden contracts with external packages;
- Grimp-based custom graph checks;
- pytest-native architecture test libraries;
- small repository-specific AST checks.

Keep these checks complementary to Tach. Prefer Tach when it can express the
invariant directly.

### Workflow Integrations

- `tach check` fits CI and pre-commit.
- A dedicated pre-commit repository is available.
- `tach test` and the pytest plugin support impact-based test selection.
- Computation caching can include file and environment dependencies.
- A VS Code extension provides editor integration.

## Important Semantics

- `tach sync` mutates configuration to match observed dependencies. Never use
  it as an automatic CI validator or to approve feature imports.
- Omitting `depends_on` permits imports from any module but still respects the
  imported module's public interface.
- `unchecked = true` permits the module to import anything, including private
  members hidden by another module's interface.
- A deprecated dependency remains allowed; Tach surfaces its usage without
  failing the dependency check.
- `utility = true` allows all modules to import the utility without declaring
  the edge.
- `root_module = "forbid"` requires every source file to belong to an explicit
  module.
- Public interfaces constrain imports into a module; visibility constrains who
  may use the module or interface.
- Static analysis cannot reliably catch dynamic imports or dynamic symbol
  lookup.
- Installing Tach also exposes a pytest plugin. Evaluate or disable it
  explicitly rather than accepting test behavior changes accidentally.
- Run Tach from the target repository root. Selecting another project's Python
  environment does not necessarily change Tach's working directory or which
  `tach.toml` it reads.
- If strict cycle detection stops before showing useful dependency or interface
  details, use a temporary copy of the configuration with cycle checks disabled
  for inventory only. Keep the real project configuration strict.
- When another module imports a leading-underscore symbol, move or rename that
  symbol into an intentional public contract rather than adding an ignore.

## Command Map

Confirm flags with installed CLI help.

```bash
tach --version
tach init
tach mod
tach sync
tach check
tach check --output json
tach check-external
tach report path/to/module.py --dependencies --usages --raw
tach map --direction dependents
tach map --closure path/to/module.py
tach show --mermaid -o tach-module-graph.mmd
tach test
tach install pre-commit
```

During a read-only audit, do not run `init`, `mod`, or `sync`.

## Version and Documentation Drift

When installed CLI and current docs disagree:

1. Verify the installed binary provenance and version.
2. Read the release notes between installed and current versions.
3. Inspect source or CLI help for exact installed semantics.
4. Separate "available after upgrade" recommendations from changes valid now.
5. Do not silently upgrade dependencies or rewrite configuration.

Include the installed Tach version and documentation refresh date in substantial
audit or adoption reports when version drift could affect the result.
