# Parallel Test Execution

> **Deep technical + strategic — 04-scaling-quality**

---

## Why Parallel Execution

A test suite that takes 30 minutes to run will not be run before every commit. A suite that takes 3 minutes will be. Parallelism is the primary lever for keeping feedback loops fast as test suites grow.

---

## Playwright's Worker Model

Playwright runs test *files* in parallel across workers. Each worker is an isolated process with its own memory space.

```
playwright.config.ts: workers: 4

Worker 1          Worker 2          Worker 3          Worker 4
─────────         ─────────         ─────────         ─────────
auth.smoke        users.smoke       auth.contract     user.contract
auth-flow         user-crud         getUsers
```

Tests *within* a file run sequentially by default. To run tests within a file in parallel, use `test.describe.parallel()`.

---

## Prerequisites for Safe Parallelism

**1. Independent test data**

```typescript
// ❌ Unsafe — shared email causes conflict when workers collide
const testUser = { email: 'test@example.com', firstName: 'Test' };

// ✅ Safe — unique per invocation
const testUser = UserFactory.create(); // faker generates unique email
```

**2. Isolated auth state**

Each worker has its own `AuthManager` instance because each worker is a separate process. But within a worker, tests share the `authManager` singleton. The fixture teardown must clear it:

```typescript
authService: async ({ apiClient }, use) => {
  const svc = new AuthService(apiClient);
  await use(svc);
  svc.logout(); // clears authManager state — critical for isolation
},
```

**3. No test ordering dependencies**

Tests must never assume another test has run before them. Every test that needs authentication must call `authService.login()` in its setup — not rely on a previous test having logged in.

---

## Worker Count Tuning

The right worker count depends on:
- Number of available CPU cores
- API rate limiting thresholds
- Test suite size and distribution

```
workers: 2   → Conservative. Use for APIs with strict rate limits (DummyJSON, public APIs)
workers: 4   → Default for most CI environments
workers: 8+  → Only for large suites against robust internal APIs with no rate limits
```

**The rate limiting trap:** Increasing workers to speed up the suite can trigger 429 rate limiting, making the suite *slower* due to retries and failures. Always test with the target API's rate limits in mind.

---

## Shard-Based Parallelism (Large Suites)

For very large suites, Playwright supports sharding — splitting the suite across multiple CI machines:

```bash
# Machine 1 of 4
npx playwright test --shard=1/4

# Machine 2 of 4
npx playwright test --shard=2/4

# ... and so on
```

Each machine runs a quarter of the test files. The full suite runtime is reduced to ~1/4. This approach requires merging reports after all shards complete.

---

## Measuring Parallelism Effectiveness

| Metric | How to measure | Target |
|--------|---------------|--------|
| Wall clock time reduction | time(parallel) / time(sequential) | > 60% reduction with 4 workers |
| Worker utilization | % time workers are active | > 80% |
| Interference failures | Failures only in parallel runs | 0 |
| Rate limit errors | 429 status codes in logs | 0 |

---

*Next: [Flaky Test Management](./flaky-test-management.md)*
