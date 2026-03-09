# Testing by Architecture Type

> **Deep technical + strategic — 02-architecture-patterns**

---

## Why Architecture Type Matters

The right testing strategy depends on the architecture of the system under test. A monolith, a microservices system, an event-driven system, and a serverless system each have different integration points, different failure modes, and different testing challenges.

---

## Monolithic Architecture

**Testing profile:** Simpler integration, harder to isolate, database-heavy.

```
Test strategy:
├─ Unit tests: individual classes/functions
├─ Integration tests: against real database (test containers)
├─ API tests: end-to-end through the monolith
└─ E2E: browser tests for UI layer
```

**Key pattern:** Test containers spin up a real database instance for integration tests, eliminating the need to mock database interactions.

---

## Microservices Architecture

**Testing profile:** Complex service mesh, independent deployability, contract boundaries critical.

```
Service A tests:        Service B tests:        Shared:
├─ Unit                 ├─ Unit                 ├─ Contract tests
├─ Component            ├─ Component            ├─ E2E (critical paths)
├─ Integration          ├─ Integration          └─ Performance tests
└─ Contract (consumer)  └─ Contract (provider)
```

**The critical layer:** Contract tests at service boundaries. Without them, you discover integration failures in E2E tests (slow) or production (worse).

**Practical rule:** Every service owns its own test pyramid. The contract layer is the handshake between pyramids.

---

## Event-Driven Architecture

**Testing profile:** Asynchronous, time-delayed, hard to assert on.

```
Challenges:
- Cannot assert on immediate response (events are async)
- Must verify event was published AND consumed correctly
- Event ordering and idempotency must be tested
```

**Testing patterns:**

```typescript
// Pattern: Publish and poll
await eventProducer.publish('user.created', userPayload);
await waitUntil(() => eventConsumer.hasProcessed('user.created'), { timeout: 5000 });
const result = await resultStore.get(userId);
expect(result.status).toBe('processed');

// Pattern: Stub the consumer, verify event shape
const capturedEvent = await eventCapture.waitFor('user.created');
expect(capturedEvent.payload).toMatchSchema(UserCreatedEventSchema);
```

---

## API-First Architecture

**Testing profile:** Contract-heavy, multiple consumers, versioning critical.

This is the architecture of the `api-framework-architect-pw-ts` project. The full testing approach is documented across this playbook — service layer, contract testing, schema validation, and fixture patterns are all optimized for API-first systems.

**Additional considerations for API-first:**
- OpenAPI/Swagger spec should be the source of truth for contracts
- Version your schemas alongside your API versions
- Consumer-driven contract testing becomes essential when external consumers depend on your API

---

## Serverless Architecture

**Testing profile:** Function-level isolation, cold start latency, infrastructure-coupled.

```
Test at three levels:
1. Function unit tests (pure logic, no infrastructure)
2. Integration tests (function + real cloud services via localstack or real dev env)
3. E2E tests (invoke via API Gateway, verify downstream effects)
```

**Key challenge:** Serverless functions are often tightly coupled to cloud provider services. The test strategy must decide early whether to mock cloud services (fast, less realistic) or use real services in a test environment (slower, more realistic).

---

*Next: [CI/CD Quality Gates](../03-platform-and-pipeline/ci-cd-quality-gates.md)*
