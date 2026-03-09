# Flaky Test Management

> **Deep technical + strategic — 04-scaling-quality**

---

## The Flakiness Problem

A flaky test is a test that produces different results on the same code without any code change. It passes sometimes, fails sometimes, for reasons unrelated to the behavior being tested.

Flaky tests are more dangerous than no tests. A no-test gap is visible. Flaky tests are invisible — they train engineers to ignore failures, which means *real* failures get ignored too.

---

## Root Causes of Test Flakiness

| Root Cause | Description | Fix |
|------------|-------------|-----|
| **Shared mutable state** | Tests interfere with each other's data | Factory pattern, test isolation |
| **Timing dependencies** | Async operations without proper awaiting | Explicit await, polling helpers |
| **Rate limiting** | Too many requests trigger 429 | Reduce workers, add retries |
| **Environment instability** | Test environment is unreliable | Environment health monitoring |
| **Test data collisions** | Hard-coded values conflict in parallel | Unique factory-generated data |
| **Token state bleed** | Auth state not cleared between tests | Fixture teardown calls `logout()` |
| **External dependencies** | Third-party APIs are unreliable | Mock external dependencies |
| **Order dependencies** | Tests depend on prior test results | Each test sets up its own state |

---

## The Flakiness Detection Pipeline

Flaky tests must be detected systematically, not noticed randomly.

```typescript
// playwright.config.ts
retries: 2, // Run each failed test up to 2 more times

// A test that fails then passes = flaky
// A test that fails all 3 times = real failure
```

Playwright's HTML report labels tests as "flaky" (passed on retry) vs "failed" (failed all retries). Track these separately.

**Monitoring setup:**
- Export test results to JSON on every CI run
- Parse for flaky vs failed status
- Track flakiness rate per test over 30 days
- Alert when any test's flakiness rate exceeds 5%

---

## The Flakiness Budget

Define an explicit flakiness budget — a maximum acceptable flakiness rate for the entire test suite. When the budget is exceeded, flaky test remediation takes priority over new feature work.

**Recommended budget:** < 1% flaky tests in the blocking gate.

If 1 in 100 tests is flaky, on average every ~5 CI runs will show a false failure. That is disruptive but manageable. Above 2%, the suite loses credibility.

---

## Investigation Protocol

When a flaky test is identified:

1. **Reproduce locally** — run the test 10 times in a loop: `for i in $(seq 1 10); do npx playwright test <file>; done`
2. **Isolate** — does it flake alone or only in parallel? Alone = timing issue. Parallel only = shared state issue
3. **Log the actual behavior** — what is the test asserting? What did it actually receive when it failed?
4. **Root cause** — identify which category from the table above
5. **Fix or delete** — a flaky test that cannot be fixed within 72 hours is deleted and replaced with a story to rewrite it

---

## Real Example: The Token State Bleed

From the `api-framework-architect-pw-ts` project — a real flakiness pattern:

```
Test: "Auth service reports authenticated state after login"
Expected: isAuthenticated() starts as false
Actual: isAuthenticated() starts as true (FLAKY)
```

**Root cause:** `authManager` is a module-level singleton. A previous test in the same worker called `login()` and the token was still present when the next test started.

**Fix:**
```typescript
test('Auth service reports authenticated state after login', async ({ authService }) => {
  authService.logout(); // explicitly clear state — do not assume clean start
  expect(authService.isAuthenticated()).toBe(false);
  await authService.login();
  expect(authService.isAuthenticated()).toBe(true);
});
```

**Lesson:** Any test that asserts on initial state must explicitly establish that state. Never assume the previous test left a clean environment.

---

*Next: [Regression Strategy](./regression-strategy.md)*
