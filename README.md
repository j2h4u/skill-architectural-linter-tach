# Tach Architecture Governance

An agent skill for auditing and governing Python module architecture with [Tach](https://github.com/tach-org/tach). It helps turn intended boundaries into checks that remain useful as a codebase changes.

## What it does

The skill audits a repository's product and runtime boundaries, then helps choose an adoption strategy for new and existing projects. It covers incremental adoption, migration from import-linter, and focused complementary checks when Tach cannot express a rule. For an enforced setup, it guides a green `tach check` and a required CI status check, with review of changes that weaken the configuration.

After adoption, it guides feature and cleanup work through the same boundaries and encourages removal of explicit architecture debt. Each invocation attempts to refresh relevant official Tach documentation and compares it with the installed version; if live documentation is unavailable, it uses available CLI help and bundled references and notes any material limitation.

## Install

Clone the skill into your agent's skills directory. The destination must not already exist; if it does, update that existing checkout instead of cloning over it.

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/j2h4u/skill-architectural-linter-tach.git ~/.agents/skills/architectural-linter-tach
```

Use an agent host that supports skills and provides the tools needed to inspect the target repository and, when available, browse documentation. To implement a Tach gate, install Tach in the target project's development environment using its package manager. The skill includes an installation guide.

## Update

```bash
git -C ~/.agents/skills/architectural-linter-tach pull --ff-only
```

The host controls skill discovery and whether a reload or new session is needed.

## Use

For a read-only assessment, ask your agent:

> Use $architectural-linter-tach to audit this Python repository. Report its boundaries, architecture risks, and a proposed adoption strategy; do not change files.

To implement a reviewed strategy, ask:

> Use $architectural-linter-tach to adopt Tach in this repository. Audit first, migrate useful import-linter contracts where applicable, and establish a green required CI gate.

See [SKILL.md](SKILL.md) for the workflow and linked references. Tach primarily analyzes static imports: a green check does not establish runtime wiring, dynamic behavior, or product correctness. Use suitable tests and focused checks for those concerns.

## License

[PolyForm Noncommercial License 1.0.0](LICENSE.md). Commercial use requires separate written permission from the owner; request it through this repository's issues or by contacting the GitHub owner.
