# CI/CD Quality Gates

> **Deep technical + strategic — 03-platform-and-pipeline**

---

## The Core Concept

A quality gate is a mandatory checkpoint in the CI/CD pipeline that must pass before code can advance to the next stage. Gates encode your quality standards into the delivery process — they are not optional and they are not skippable.

Quality gates are how "shift-left" becomes structural rather than aspirational.

---

## Gate Design Principles

**Principle 1: Every gate has a clear owner.**
The team that owns the code owns the gate. QE owns the gate definitions and standards; the delivery team owns passing them.

**Principle 2: Gates are fast or they are bypassed.**
A gate that takes 20 minutes will be worked around. Every gate must justify its position based on its speed and the feedback value it provides.

**Principle 3: Gates fail loudly with actionable messages.**
"Tests failed" is not useful. "[contract:GET /auth/me] Schema violation: accessToken missing from response — see AuthUserSchema" is.

**Principle 4: Gates block, warnings advise.**
A gate either passes or fails. If something is important enough to check, it is important enough to block on.

---

## The Pipeline Quality Gate Architecture

```
Developer commits
       │
       ▼
┌─────────────────────┐
│  Gate 1: Static     │  < 30 seconds
│  TypeScript compile │  Blocks on: type errors
│  ESLint             │  Blocks on: lint violations
│  Secret scanning    │  Blocks on: credentials in code
└──────────┬──────────┘
           │ PASS
           ▼
┌─────────────────────┐
│  Gate 2: Unit       │  < 2 minutes
│  Unit tests         │  Blocks on: test failure
│  Coverage threshold │  Blocks on: coverage drop > 5%
└──────────┬──────────┘
           │ PASS
           ▼
┌─────────────────────┐
│  Gate 3: Contract   │  < 3 minutes
│  Schema contracts   │  Blocks on: contract violation
│  API compatibility  │  Blocks on: breaking change detected
└──────────┬──────────┘
           │ PASS
           ▼
┌─────────────────────┐
│  Gate 4: Integration│  < 10 minutes
│  API test suite     │  Blocks on: test failure
│  Service tests      │  Blocks on: flaky rate > 1%
└──────────┬──────────┘
           │ PASS
           ▼
┌─────────────────────┐
│  Gate 5: Pre-release│  < 30 minutes
│  Full regression    │  Blocks on: critical path failure
│  E2E critical paths │  Blocks on: smoke suite failure
│  Performance check  │  Blocks on: p95 latency > threshold
└──────────┬──────────┘
           │ PASS
           ▼
       Deploy
```

---

## Playwright Configuration as a Quality Gate

```typescript
export default defineConfig({
  workers: process.env.CI ? 2 : 4,   // limit concurrency in CI to avoid rate-limiting
  retries: process.env.CI ? 1 : 0,   // 1 retry catches transient network failures
  timeout: 30_000,                    // tests that hang are as bad as tests that fail

  reporter: [
    ['html', { open: 'never' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['list'],
  ],

  projects: [
    { name: 'smoke',       testMatch: '**/smoke/**/*.spec.ts' },
    { name: 'contract',    testMatch: '**/contract/**/*.spec.ts' },
    { name: 'integration', testMatch: '**/integration/**/*.spec.ts' },
    { name: 'all',         testMatch: '**/*.spec.ts' },
  ],
});
```

---

## Rate Limiting — The Hidden Gate Failure Mode

Parallel tests against real APIs (especially shared/public environments) trigger rate limiting. This produces 429 errors that look like flaky tests but are actually a throughput problem.

**Solution:**
- `workers: 2` reduces concurrency
- `retries: 1` catches the occasional 429 that slips through
- Long-term: rate-limit-aware `ApiClient` that respects `Retry-After` headers

---

## Environment Variable Safety in CI

```yaml
# GitHub Actions — never store credentials in config files
env:
  BASE_URL:       ${{ secrets.TEST_BASE_URL }}
  TEST_USERNAME:  ${{ secrets.TEST_USERNAME }}   # NOT "USERNAME" — that is reserved by the OS
  TEST_PASSWORD:  ${{ secrets.TEST_PASSWORD }}
```

`USERNAME` is a reserved OS environment variable (set to the system user running the process). Always use `TEST_USERNAME` for test credentials.

---

*Next: [Test Orchestration](./test-orchestration.md)*
