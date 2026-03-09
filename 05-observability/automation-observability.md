# Automation Observability

> **Deep technical + strategic — 05-observability**

---

## The Core Insight

Production systems have monitoring, alerting, and dashboards. Automation systems — which are also software products — usually have none of these. The result is that the health of the test suite is invisible until something goes badly wrong.

Automation observability means applying the same principles of observability to your test infrastructure that you apply to production services.

---

## The Three Pillars Applied to Test Systems

**Logs** — structured, timestamped records of every test action
**Metrics** — quantitative measurements of test system behavior over time
**Traces** — the path of a test execution from trigger to result

---

## The Logger Pattern

Every meaningful action in the test framework should be logged with structure:

```typescript
// src/core/Logger.ts
export class Logger {
  info(message: string, data?: Record<string, unknown>): void {
    const timestamp = new Date().toISOString().replace('T', ' ').slice(0, -1);
    const dataStr = data ? ` ${JSON.stringify(data)}` : '';
    console.log(`[${timestamp}] [INFO ] ${message}${dataStr}`);
  }

  error(message: string, error?: unknown): void {
    const timestamp = new Date().toISOString().replace('T', ' ').slice(0, -1);
    console.error(`[${timestamp}] [ERROR] ${message}`, error ?? '');
  }
}
```

What to log:
- Every HTTP request: method, path, status, duration
- Authentication events: login attempts, token refresh, failures
- Schema validation failures: which fields failed, what was expected vs received
- Retry events: which request was retried, how many times

This is exactly what the `api-framework-architect-pw-ts` project produces:
```
[2026-03-08 08:00:25.397] [INFO ] POST /auth/login {"status":200,"durationMs":533}
[2026-03-08 08:00:25.440] [INFO ] Authenticating user... {"username":"emilys"}
```

---

## The MetricsCollector Pattern

Track quantitative data about test execution:

```typescript
export class MetricsCollector {
  private metrics: Map<string, number[]> = new Map();

  record(key: string, value: number): void {
    if (!this.metrics.has(key)) this.metrics.set(key, []);
    this.metrics.get(key)!.push(value);
  }

  summary(key: string): { count: number; mean: number; p95: number; max: number } {
    const values = (this.metrics.get(key) ?? []).sort((a, b) => a - b);
    if (values.length === 0) return { count: 0, mean: 0, p95: 0, max: 0 };

    return {
      count: values.length,
      mean:  values.reduce((a, b) => a + b, 0) / values.length,
      p95:   values[Math.floor(values.length * 0.95)],
      max:   values[values.length - 1],
    };
  }
}
```

Record: request duration per endpoint, schema validation time, test execution time per suite.

---

## Test Execution Visibility

The most important observability output is the test result report. Playwright's HTML reporter provides:
- Pass/fail status per test
- Retry history (distinguishes flaky from failed)
- Duration per test
- Error messages with stack traces
- Test timeline showing parallel execution

**Export to CI:** Always configure JSON output alongside HTML so CI pipelines can parse results programmatically:

```typescript
reporter: [
  ['html',   { open: 'never' }],
  ['json',   { outputFile: 'test-results/results.json' }],
  ['junit',  { outputFile: 'test-results/junit.xml' }], // for CI dashboards
],
```

---

*Next: [Test Health Metrics](./test-health-metrics.md)*
