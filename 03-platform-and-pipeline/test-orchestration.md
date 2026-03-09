# Test Orchestration

> **Deep technical + strategic — 03-platform-and-pipeline**

---

## What Test Orchestration Means

Test orchestration is deciding *which tests run*, *when*, *in what order*, and *on what infrastructure* at the pipeline level. It is the key lever for keeping feedback loops fast as test suites grow to hundreds or thousands of tests.

---

## Orchestration Strategies

### Tiered Execution (Recommended)

```
On every commit:   Smoke tests            < 2 minutes
On every PR:       + Contract tests       < 5 minutes total
On merge to main:  + Integration suite   < 15 minutes total
Nightly:           + Full regression     < 60 minutes total
Pre-release:       + E2E + performance   < 90 minutes total
```

```bash
# CI commit check — smoke only
npx playwright test --project=smoke

# PR gate — smoke + contract
npx playwright test --project=smoke --project=contract

# Merge gate — everything
npx playwright test
```

### Impact-Based Selection (Advanced)

Only run tests affected by the changes in a given commit. Requires test-to-code dependency mapping.

- **Benefit:** Dramatically faster feedback on small changes
- **Risk:** May miss cross-cutting failures
- **Tools:** Playwright `--last-failed`, custom change-detection scripts

---

## Parallel Execution Safety Requirements

Playwright workers run test files in parallel. For parallel execution to work correctly, every test must:

1. Generate its own unique test data (`UserFactory.create()` — never hard-coded values)
2. Own its own auth state (fixture teardown calls `authService.logout()`)
3. Not depend on execution order of other tests
4. Not assume shared mutable state in the system under test

---

## Flaky Test Quarantine Protocol

```
Test fails intermittently (< 20% failure rate)
              │
              ▼
    Tag with @quarantine
    Remove from blocking gate
    Route to non-blocking parallel job
              │
              ▼
    Assign owner — 72-hour investigation SLA
              │
         ┌────┴────┐
       Fixed     Cannot fix
         │           │
    Remove tag    Delete test
    Restore gate  Open rewrite story
```

**The rule:** Flaky tests in the blocking gate train engineers to ignore pipeline failures. That is more damaging than having no test at all.

---

*Next: [Environment Strategy](./environment-strategy.md)*
