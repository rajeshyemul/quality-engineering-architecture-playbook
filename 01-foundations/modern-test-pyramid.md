# Modern Test Pyramid

> **Strategic depth — 01-foundations**

---

## The Original Pyramid and Why It Still Holds

Mike Cohn's test pyramid (2009) proposed a simple model: write many unit tests, fewer integration tests, and even fewer end-to-end tests. The rationale was speed and cost, unit tests are fast and cheap; E2E tests are slow and brittle.

The core insight remains valid. What has changed is the vocabulary, the layers, and how the pyramid applies to modern distributed systems.

---

## The Classic Model

```
          ▲
         /E2E\          — Few, slow, high confidence, high cost
        /─────\
       / Integ  \       — Some, medium speed, service-level confidence
      /──────────\
     /   Unit     \     — Many, fast, component-level confidence
    ──────────────────
```

**The problem with applying this naively in 2024:** Modern systems are not monoliths. A "unit" in a microservices architecture is often a service, not a function. An "integration test" may span three services, a message queue, and a database. E2E tests may be the only way to verify certain user-facing behaviors at all.

The shape of the pyramid is correct. The definition of each layer needs updating.

---

## The Modern Test Pyramid

```
                    ▲
                   /E2E\              Critical user journeys only
                  /─────\             Playwright, Cypress — sparingly
                 /Contract\           Service boundary validation
                /──────────\          Pact, Zod schema contracts
               / Integration \        Service + infrastructure tests
              /────────────────\      API tests, DB tests, queue tests
             /   Component      \     Isolated service tests
            /────────────────────\    Mocked dependencies
           /       Unit           \   Pure function tests
          ────────────────────────────
```

### Layer Definitions for Modern Systems

**Unit Tests**
Pure function testing. No I/O, no network, no database. Tests a single function or class method in complete isolation. Should be hundreds or thousands of these. Runtime: milliseconds.

*Example: Test that a price calculation function returns the correct value for given inputs.*

**Component Tests**
A single service tested in isolation with all external dependencies mocked or stubbed. Verifies the service's internal behavior, routing, business logic, error handling without touching real infrastructure. Runtime: seconds.

*Example: Test that the UserService correctly handles a 404 from the upstream users API, returning an appropriate error response.*

**Integration Tests**
A service tested with its real dependencies as real database, real cache, real message queue but other services still mocked. Verifies that the service integrates correctly with its infrastructure. Runtime: seconds to low minutes.

*Example: Test that creating a user via the API correctly persists to the database and publishes an event to the queue.*

**Contract Tests**
Schema and behavioral contracts at service boundaries. Verify that the interface between two services remains stable, without requiring both services to be running simultaneously. This is the layer that enables independent deployability. Runtime: fast (no network calls to the real service).

*Example: Verify that the response from GET /users/:id matches the UserSchema contract that downstream services depend on.*

**End-to-End Tests**
Full stack tests that exercise the system as a real user. Using real services, real infrastructure, real browser or API client. Should cover critical paths only. Runtime: minutes.

*Example: A user can register, verify their email, log in, and complete a purchase.*

---

## The Trophy Model (Alternative)

Kent C. Dodds proposed the Testing Trophy as a counterpoint to the pyramid, suggesting that integration tests provide the best return on investment for modern web applications.

```
        ___
       /E2E \
      /──────\
     /  Integ  \     ← widest layer (most tests here)
    /────────────\
   /   Unit       \
  /────────────────\
 /    Static        \    TypeScript, ESLint, type checking
───────────────────────
```

**When the trophy model applies:** Frontend-heavy applications where component interactions are the primary source of bugs, and where unit tests of individual functions provide limited real-world confidence.

**When the pyramid still applies:** Backend services, APIs, data pipelines, and systems where the business logic lives in discrete functions and classes.

**The practical answer:** Use the pyramid shape as a thinking model, not a rule. The right distribution depends on your system's architecture and where defects actually originate.

---

## Common Pyramid Anti-Patterns

### The Ice Cream Cone
```
     Unit  ← tiny
    ────────
   Integration
  ────────────
 E2E, E2E, E2E, E2E, E2E    ← most tests are here
─────────────────────────────
```
Symptoms: Test suite takes 45 minutes. Tests fail for environment reasons. Nobody runs the full suite locally. Confidence is low despite high test count.

**Root cause:** Teams default to E2E because it feels like "real" testing. No investment in unit or integration layer.

### The Hourglass
```
     E2E       ← many
    ──────
    (empty)    ← nothing here
    ──────
     Unit      ← many
```
Symptoms: Unit tests pass, E2E tests pass, production bugs at service boundaries. Contract and integration layer completely missing.

**Root cause:** Teams wrote unit tests first, then added E2E later, skipped the middle.

### The Flat Line
Nothing but unit tests. Fast suite, no confidence in system behavior.

---

## Applying This to Distributed Systems

In a microservices architecture, every service has its own pyramid. The challenge is ensuring the pyramids connect at the seams, this is where contract testing becomes critical.

```
Service A pyramid     Service B pyramid
──────────────────    ──────────────────
E2E (shared)    ←──────────→    E2E (shared)
Contract        ←── boundary ──→ Contract
Integration          tests       Integration
Component                        Component
Unit                             Unit
```

The contract layer is the handshake between pyramids. Without it, teams either over-test with shared E2E suites (slow, brittle) or under-test and discover integration bugs in production.

---

## Metrics That Matter

| Metric | Target | Anti-pattern |
|--------|--------|-------------|
| Unit test suite runtime | < 60 seconds | > 5 minutes |
| Integration test runtime | < 5 minutes | > 20 minutes |
| E2E test count | < 50 critical paths | > 500 tests |
| Contract coverage | All public API endpoints | Zero contract tests |
| Flaky test rate | < 1% | > 5% |

---

*Next: [Shift-Left Quality](./shift-left-quality.md)*
