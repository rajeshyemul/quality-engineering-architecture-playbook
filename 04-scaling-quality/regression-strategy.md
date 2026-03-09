# Regression Strategy

> **Deep technical + strategic — 04-scaling-quality**

---

## What Regression Strategy Means

A regression strategy is not "run all tests before release." That is a tactic, not a strategy. A regression strategy answers:

- Which tests constitute the regression suite?
- When does the regression suite run?
- What is the acceptable runtime for the regression suite?
- What is the escalation path when the regression suite fails?
- How does the regression suite evolve as the system changes?

---

## Regression Suite Composition

Not all tests belong in the regression suite. The regression suite should contain tests that verify behavior that must not regress — not every test ever written.

```
Regression suite = Tier 1 (critical paths) + Tier 2 (core features) + All contract tests

Does NOT include:
- Tests for deprecated features
- Exploratory tests that were automated without clear value
- Tests for third-party behavior outside your control
- Performance tests (separate suite)
```

---

## Risk-Based Regression

The most effective regression strategies are risk-based. Not every area of the system has equal regression risk.

**High regression risk:** Recently changed code, code with high cyclomatic complexity, code with a history of regressions, code near critical business paths.

**Low regression risk:** Stable code with no recent changes, code with existing high-confidence test coverage, read-only operations on stable data.

**Application:** In CI, run the full regression suite on high-risk areas (recently changed files and their dependents) on every PR. Run the full suite only on merge to main.

---

## The Regression Anti-Pattern: Suite Bloat

Regression suites accumulate tests. They rarely lose them. Over time, the suite bloats with:
- Tests for removed features that still pass because the code was never deleted
- Redundant tests covering the same behavior from multiple angles
- Tests that were written as exploratory checks and never had clear regression value

**Annual regression audit:** Once a year, review the regression suite. Remove tests for features that no longer exist. Identify and consolidate redundant tests. Measure the runtime impact.

---

## Regression Failure Escalation

When the regression suite fails, the team needs a clear protocol:

```
Regression failure detected
         │
         ▼
Is this a known flaky test?
│                  │
YES                NO
│                  │
Skip (quarantine   Is this a real regression?
already tagged)    │                    │
                   YES                  UNSURE
                   │                    │
            Block deployment     Investigate
            Notify team lead     within 2 hours
            Create P1 ticket     Escalate if
                                 unresolved
```

---

## QE Maturity Model

Use this model to assess your current regression strategy maturity:

| Level | Regression approach |
|-------|-------------------|
| 1 | Manual regression testing before each release |
| 2 | Automated smoke tests only in CI |
| 3 | Automated regression suite, run manually pre-release |
| 4 | Automated regression in CI, tiered execution by risk |
| 5 | Risk-based automated regression with impact analysis and continuous monitoring |

---

*Next: [Automation Observability](../05-observability/automation-observability.md)*
