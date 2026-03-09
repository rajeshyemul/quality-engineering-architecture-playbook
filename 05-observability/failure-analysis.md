# Failure Analysis

> **Deep technical + strategic — 05-observability**

---

## Why Failure Analysis Matters

Test failures are signals. A single failure is an incident. A pattern of failures is a system-level insight. Failure analysis is the practice of extracting the system-level insight from the pattern.

Without failure analysis, QE teams spend their time fixing individual tests. With it, they identify and fix the root conditions that cause categories of failures.

---

## The FailureAnalyzer Pattern

```typescript
export class FailureAnalyzer {
  private failures: FailureRecord[] = [];

  record(test: string, error: string, category: FailureCategory): void {
    this.failures.push({
      test,
      error,
      category,
      timestamp: new Date(),
    });
  }

  topFailureCategories(): CategorySummary[] {
    const counts = new Map<FailureCategory, number>();
    for (const f of this.failures) {
      counts.set(f.category, (counts.get(f.category) ?? 0) + 1);
    }
    return [...counts.entries()]
      .sort((a, b) => b[1] - a[1])
      .map(([category, count]) => ({ category, count, percentage: count / this.failures.length }));
  }
}

type FailureCategory =
  | 'schema_violation'    // API response does not match contract
  | 'auth_failure'        // Login or token failure
  | 'rate_limited'        // 429 from API
  | 'environment'         // Environment unreachable or misconfigured
  | 'data_collision'      // Test data conflict
  | 'timing'              // Async operation timeout
  | 'assertion'           // Real behavior regression
  | 'unknown';
```

---

## Reading the Failure Patterns

Different failure distributions indicate different systemic problems:

**High schema_violation rate** → API is changing faster than contracts are updated. Need better contract coverage and change management.

**High auth_failure rate** → Credential rotation, token expiry logic, or environment credential management issues. Check `TEST_USERNAME`/`TEST_PASSWORD` configuration.

**High rate_limited rate** → Worker concurrency too high for the target API. Reduce `workers` setting. Add backoff logic to `ApiClient`.

**High environment rate** → Test infrastructure is unstable. Invest in environment reliability before adding more tests.

**High timing rate** → Async operations are not being properly awaited. Review `await` usage. Add explicit polling for async operations.

**High assertion rate** → Real regressions being caught. This is the failure category you *want* to be high. Investigate each one.

---

## Post-Incident Analysis for QE

When a production incident occurs that was not caught by the test suite, run a structured post-incident analysis:

**Questions to answer:**
1. Which test should have caught this defect?
2. Why did that test not exist, not run, or not catch it?
3. Was this a coverage gap, a wrong assertion, or a contract that was not validated?
4. What test or contract change would prevent recurrence?
5. Was this a known risk that was accepted? If so, was the acceptance still valid?

**Output:** A new test or contract that encodes the production failure as a test case. This is how the test suite learns from production.

---

## ROI and Reporting

See [ROI and Reporting](./roi-and-reporting.md) for how to translate failure analysis into business-facing metrics.

---

*Next: [ROI and Reporting](./roi-and-reporting.md)*
