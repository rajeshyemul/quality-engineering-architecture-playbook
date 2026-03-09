# Test Health Metrics

> **Deep technical + strategic — 05-observability**

---

## What to Measure

The goal of test health metrics is to answer one question at any moment: **"Can we trust our test suite to tell us the truth about our system?"**

If the answer is yes, the suite is healthy. If the answer is uncertain, you have a measurement problem.

---

## The Core Metric Set

### 1. Test Reliability Rate
Percentage of test runs that produce a deterministic result (not flaky).

```
Reliability Rate = (Passing runs + Failing runs) / Total runs
                   where Failing runs = failed all retries (real failure)
                         Passing runs = passed first attempt
                         NOT counted: failed then passed (flaky)

Target: > 99%
Alert at: < 97%
```

### 2. Defect Escape Rate
Percentage of defects that reached production vs were caught by the test suite.

```
Escape Rate = Production defects / (Production defects + QE-caught defects)

Target: < 5%
Trending up = test coverage gaps or wrong tier coverage
```

### 3. Mean Time to Feedback (MTTF)
Average time from code commit to test result available to the developer.

```
MTTF = avg(test_result_time - commit_time) across all PRs

Target: < 5 minutes for PR gate
Alert at: > 10 minutes
```

### 4. Contract Coverage
Percentage of public API endpoints covered by contract tests.

```
Contract Coverage = Contract-tested endpoints / Total public endpoints

Target: 100%
Alert at: < 90%
```

### 5. Test Suite Growth Rate
Rate at which new tests are added vs tests are deleted.

```
Growth Rate = New tests added per sprint - Tests deleted per sprint

Healthy: Positive but controlled (net new coverage)
Warning: Rapid growth without corresponding runtime management
Alert: Negative growth (coverage shrinking)
```

---

## The Test Health Dashboard

A test health dashboard should be visible to the entire engineering team — not just QE. It communicates the state of quality confidence in a way that informs deployment and release decisions.

**Dashboard sections:**

```
┌──────────────────────────────────────────────────────┐
│  TEST SUITE HEALTH — Last 30 Days                   │
├──────────────┬───────────────────────────────────────┤
│ Reliability  │  ████████████████░░ 98.2%            │
│ MTTF         │  4.2 min average                     │
│ Escape Rate  │  2.1%                                │
│ Contract Cov │  ████████████████████ 100%            │
├──────────────┴───────────────────────────────────────┤
│  Flaky tests: 2 currently quarantined               │
│  Suite runtime trend: ↑ 8% (action needed at 10%)  │
│  Last full suite: ✓ PASSED  08:47 today            │
└──────────────────────────────────────────────────────┘
```

---

## Connecting Metrics to Business Outcomes

QE metrics must connect to outcomes that matter to engineering leadership. The translation:

| QE Metric | Business meaning |
|-----------|-----------------|
| Low defect escape rate | Fewer production incidents. Lower MTTR. Happier customers. |
| Low MTTF | Faster developer feedback → higher delivery velocity |
| High reliability rate | Engineers trust the suite → fewer manual checks → faster releases |
| High contract coverage | Fewer integration failures at deployment → higher deployment frequency |

This framing is what makes QE budget conversations productive.

---

*Next: [Failure Analysis](./failure-analysis.md)*
