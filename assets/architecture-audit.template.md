# Architecture Audit

## Verdict

<!-- State the current shape, main risk, and recommended direction in 3-6 sentences. -->

## Classification

| Axis | Classification | Evidence | Confidence |
|---|---|---|---|
| Lifecycle |  |  |  |
| Deployment topology |  |  |  |
| Internal organization |  |  |  |
| Governance maturity |  |  |  |
| Adoption readiness |  |  |  |

## Scope and Evidence

<!-- Name the repository/subsystem, sources inspected, runtime access, installed
Tach version, and live documentation refresh date. -->

## Existing Strengths

<!-- Record useful seams and invariants worth preserving. -->

## Hard-Boundary Risks

<!-- Record runtime ownership, state, side-effect, security, dependency-direction,
or gate-bypass risks. Separate facts from inference. -->

## Structural Debt

<!-- Record important cycles, coupling, deep imports, god modules, broad public
interfaces, adapter leakage, and mixed ownership. -->

## Governance Gaps

<!-- Record missing or bypassable gates, unowned waivers, stale rules, branch
protection gaps, and non-import invariants without enforcement. -->

## Target Direction

<!-- Describe the smallest credible target architecture. Explain why it fits the
product and runtime ownership model. -->

## Gate Plan

<!-- Select strict greenfield, import-linter migration, enforced frontier,
current-state ratchet, or a short target-design spike followed by a green
frontier. Name any complementary semantic gates needed for invariants Tach
cannot express precisely, including zero-owner policies for unowned high-risk
capabilities. -->

## Architecture Debt

| Scope or edge | Reason | Target boundary | Owner | Removal condition |
|---|---|---|---|---|
|  |  |  |  |  |

## Action Queue

1. **First protected boundary**
   - Change:
   - Completion test:
2. **Second cleanup slice**
   - Change:
   - Completion test:
3. **Third cleanup slice**
   - Change:
   - Completion test:

## Unknowns

<!-- Include only questions that materially affect architecture direction or
gate design. -->
