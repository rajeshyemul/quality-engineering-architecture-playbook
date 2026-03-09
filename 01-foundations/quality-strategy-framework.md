# Quality Strategy Framework

> **Strategic depth — 01-foundations**

---

## What a Quality Strategy Is (and Is Not)

A quality strategy is not a test plan. A test plan describes what will be tested and how. A quality strategy answers a different set of questions:

- What level of quality risk is acceptable for this system and this business?
- Where in the engineering lifecycle does quality work happen?
- What is the right investment in automation, tooling, and QE headcount?
- How do we measure whether our quality system is working?
- How does our quality approach change as the system scales?

A quality strategy is a leadership artifact. It connects engineering decisions to business risk.

---

## The Four Dimensions of a Quality Strategy

Every quality strategy must address four dimensions. Missing any one of them produces a strategy that will eventually fail.

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   RISK          ←→        COVERAGE                 │
│   What can         What do we validate              │
│   go wrong?        and how deeply?                 │
│                                                     │
│       ↕                       ↕                    │
│                                                     │
│   SPEED         ←→        CONFIDENCE               │
│   How fast do      How certain are we               │
│   we need to       in what we ship?                │
│   move?                                             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

These four dimensions are in constant tension. Maximizing one often constrains another. The strategy is about finding the right balance for your context — not maximizing all four simultaneously.

---

## Dimension 1: Risk Assessment

Quality strategy starts with risk, not with tests.

**Questions to answer:**
- What are the highest-consequence failure modes in this system?
- What is the blast radius of a defect in each area (financial, reputational, safety, regulatory)?
- What areas change most frequently (highest defect probability)?
- What areas are most complex (highest defect density)?

**Risk matrix:**

```
              Low Change Frequency    High Change Frequency
              ───────────────────    ──────────────────────
High Impact │   Deep coverage,      │  Deep coverage,
            │   run infrequently    │  run on every commit
            │                       │
Low Impact  │   Light coverage,     │  Moderate coverage,
            │   or skip             │  spot check
```

This matrix directly informs where you invest in automation depth and where you accept lighter coverage.

**Practical application:** Before writing a single test, map your system's features and services against this matrix. Your automation investment should correlate with the upper-right quadrant.

---

## Dimension 2: Coverage Model

Coverage is not a percentage. "80% code coverage" is a metric that tells you what was executed, not what was validated. A better coverage model answers:

**Functional coverage:** Are all specified behaviors validated?
**Boundary coverage:** Are edge cases, error paths, and limits tested?
**Contract coverage:** Are all service interfaces validated against their published contracts?
**Integration coverage:** Are all critical service-to-service interactions verified?
**Journey coverage:** Are the end-to-end user paths that drive business value validated?

**Coverage tiers — a practical model:**

| Tier | What it covers | Automation approach |
|------|---------------|---------------------|
| **Tier 1 — Critical** | Revenue paths, data integrity, security boundaries | 100% automated, run on every commit |
| **Tier 2 — Core** | Primary feature functionality | Automated, run on every PR |
| **Tier 3 — Extended** | Secondary features, edge cases | Automated, run on schedule or pre-release |
| **Tier 4 — Exploratory** | New features, unusual paths | Manual exploratory, risk-based |

This tiering model directly informs your regression strategy and CI pipeline design.

---

## Dimension 3: Speed Requirements

The feedback loop requirement should drive your test architecture, not the other way around.

**Feedback loop targets by phase:**

| Phase | Target feedback time | What runs |
|-------|---------------------|-----------|
| Pre-commit (local) | < 30 seconds | Static analysis, unit tests |
| PR pipeline | < 5 minutes | Unit + contract + fast integration |
| Merge pipeline | < 15 minutes | Full integration + API tests |
| Pre-release | < 45 minutes | Full suite including E2E |
| Production monitoring | Continuous | Synthetic monitors |

If your current suite does not fit these targets, you have an architecture problem — not a test count problem. The answer is parallelization, test selection, and layer restructuring — not "run fewer tests."

---

## Dimension 4: Confidence Model

Confidence is what you are actually selling to the business. Not test counts, not coverage percentages — confidence that what ships will work.

**Confidence is built from:**
- Test reliability (low flakiness rate)
- Test relevance (tests verify real behavior, not implementation details)
- Test depth (tests validate the right things at the right level)
- Test freshness (tests stay in sync with the system as it evolves)

**Confidence is destroyed by:**
- Flaky tests that are ignored
- Tests that pass despite being wrong
- Coverage theater (high numbers, low signal)
- Test suites nobody runs because they are too slow

A quality strategy must explicitly address how confidence will be maintained over time — not just built initially.

---

## Building Your Quality Strategy: A Framework

### Step 1: Define the Risk Profile
Map your system against the risk matrix. Identify your Tier 1 critical paths. Be explicit about what you are *not* going to test deeply.

### Step 2: Define the Coverage Model
Assign tiers to system areas. Define what "done" means for each tier. Make this visible to product and engineering leadership.

### Step 3: Set Feedback Loop Targets
Define the acceptable feedback loop time for each pipeline phase. Use this to drive architecture decisions about parallelization and test selection.

### Step 4: Define the Confidence Metrics
Choose the metrics that will tell you whether your quality system is working. Recommended starting set:
- Defect escape rate (defects reaching production vs caught in QE)
- Test reliability rate (passing tests / total runs over 30 days)
- Mean time to feedback (commit to test result)
- Deployment confidence score (subjective team rating, 1–5)

### Step 5: Set the Investment Model
Translate the strategy into resource requirements: automation engineering time, tooling budget, and headcount. This is the step most teams skip — and it is the step that makes strategies real.

---

## The Strategy Document

A quality strategy should be a one-page document that any engineer or engineering manager can read and understand. It is not a test plan. It does not list test cases.

**Template:**

```
Quality Strategy — [System/Product Name]
───────────────────────────────────────

Risk profile:     [High / Medium / Low] — [rationale in one sentence]

Critical paths:   [List the 3-5 things that absolutely cannot fail]

Coverage model:   Tier 1: [areas]
                  Tier 2: [areas]
                  Tier 3: [areas]
                  Explicitly out of scope: [areas and why]

Feedback targets: Pre-commit: [X sec]   PR: [X min]   Pre-release: [X min]

Confidence metrics: [3-4 metrics with targets]

Investment:       [Automation effort %]  [Tooling budget]  [QE headcount]

Review cadence:   [When this strategy is reviewed and updated]
```

---

## When to Update Your Quality Strategy

A quality strategy is not set-and-forget. Trigger a review when:
- The system architecture changes significantly (new services, new integrations)
- Team size changes by more than 50%
- A major production incident reveals a gap in coverage
- Delivery velocity is being constrained by the test suite
- A new technology layer is introduced (new database, new message broker, new protocol)

---

*This document is the strategic anchor for everything that follows in this playbook.*

*Next: [Automation Framework Architecture](../02-architecture-patterns/automation-framework-architecture.md)*
