# Install Tach

Use this guide when Tach is not yet installed in a Python repository. Prefer a
project development dependency so local development and CI use the same
version.

Before copying commands, check the current official
[installation guide](https://docs.gauge.sh/getting-started/getting-started/),
[PyPI package](https://pypi.org/project/tach/), and installed CLI help when
available.

## 1. Add the Development Dependency

Use the package manager already owned by the repository.

With uv:

```bash
uv add --dev tach
uv run tach --version
```

With pip in an activated project virtual environment:

```bash
python -m pip install tach
tach --version
```

For Poetry, PDM, Hatch, or another manager, add `tach` to the equivalent
development or quality-assurance dependency group. Commit the changed manifest
and lockfile when the repository tracks them.

Avoid relying only on an unpinned global installation for a required CI gate.

## 2. Check Installation Side Effects

Installing Tach also registers a pytest plugin. By default it reports
impact-analysis information while still running the full suite.

If the repository is not adopting Tach test selection, verify normal pytest
behavior and disable the plugin explicitly when desired:

```toml
[tool.pytest.ini_options]
addopts = ["-p", "no:tach"]
```

Preserve existing pytest options when adding this entry.

## 3. Choose Setup Mode Before Generating Configuration

Run commands from the repository root.

For a greenfield repository:

```bash
uv run tach init
uv run tach check
```

Use the repository's normal runner instead of `uv run` when it uses another
package manager.

For an existing repository, audit first. Do not run `tach sync` blindly:
`sync` changes `tach.toml` to approve dependencies already present in the
source. Choose an enforced frontier, import-linter migration, or current-state
ratchet deliberately.

## 4. Add Fast Local Feedback

If the repository already uses the pre-commit framework, integrate Tach with
that existing configuration and pin the current supported revision.

If it does not use a hook manager, current Tach CLI versions expose:

```bash
tach install pre-commit
```

Check `tach install --help` before running it. The command writes a Git hook and
may require manual integration when a hook already exists. Some documentation
may still show the older `tach install --pre-commit` spelling; trust the
installed CLI for the installed version.

Local hooks are convenience, not the authoritative gate.

## 5. Add the CI Gate

First make the intended enforcement surface green. Then add the repository's
equivalent of:

```bash
tach check
```

Run it through the same locked project environment used to install Tach. Make
the CI job required in branch protection or repository rulesets.

Never run `tach sync` automatically in CI. CI must validate architecture, not
rewrite the policy to accept new imports.

## 6. Verify the Installation

Confirm:

- the installed version is visible in the project environment;
- `tach check` is green for the intended enforcement surface;
- `tach check-external` has been evaluated when package dependency hygiene
  matters;
- pytest behavior did not change accidentally;
- the check runs in CI;
- the CI result is required before merge;
- configuration changes receive appropriate ownership review.

When replacing import-linter, keep it until Tach or a complementary gate covers
each useful contract. Tach and import-linter overlap heavily, but Tach does not
automatically replace every forbidden external-package or custom-contract use.
After useful coverage is replaced, remove the old dependency, configuration,
commands, and cache handling.
