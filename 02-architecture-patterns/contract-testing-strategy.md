# Contract Testing Strategy

> **Deep technical + strategic — 02-architecture-patterns**

---

## What Contract Testing Is

Contract testing verifies that the interface between two services matches an agreed specification — without requiring both services to run simultaneously.

A contract is a formal description of what one service promises to provide and what another service depends on receiving. Contract tests verify that both sides of this promise hold.

---

## Why Contract Testing Is Critical at Scale

In a monolith, if you change the shape of a function's return value, the compiler tells you immediately. Every caller is visible.

In a microservices architecture, services communicate over HTTP. The "compiler" cannot see across service boundaries. You can change a response field name in Service A and not discover that Service B was depending on the old name until it fails in production — or until an E2E test catches it, hours later.

Contract testing moves this discovery to the earliest possible moment: when the change is made.

---

## Schema Contract Testing (What We Built)

The simplest form of contract testing uses schemas to validate API responses against documented contracts.

```typescript
// Define the contract — this is the promise
export const UserSchema = z.object({
  id:        z.number(),
  firstName: z.string(),
  lastName:  z.string(),
  email:     z.string().email(),
  role:      z.enum(['admin', 'moderator', 'user']),
});

// Test the contract
test('GET /users/:id response matches UserSchema contract', async ({ apiClient }) => {
  const response = await apiClient.get('/users/1');
  await ResponseValidator.expectSchemaMatch(response, UserSchema, 'contract:GET /users/:id');
});
```

When the API changes and removes the `role` field, this test fails immediately. The contract violation is caught before it reaches any consumer.

---

## Consumer-Driven Contract Testing (Advanced)

For service-to-service contracts in microservices, consumer-driven contract testing (CDCT) is the gold standard. Tools: **Pact**, **Spring Cloud Contract**.

The pattern:
1. **Consumer** defines what it needs from the provider (the contract)
2. **Provider** verifies it can fulfill the contract
3. A broker (Pact Broker) stores contracts and tracks compatibility

```
Consumer Service          Pact Broker           Provider Service
────────────────          ───────────           ────────────────
1. Write consumer    →    2. Publish       →    3. Verify provider
   test (contract)           contract             fulfills contract
4. Deploy if                                  4. Deploy if
   contract verified                             contract satisfied
```

**When to use CDCT:** When you have multiple services with independently deployed consumers and providers. When you need to know, before deploying Service A, whether its changes will break Service B.

**When schema contracts are sufficient:** When you own both sides, when services are deployed together, or when the overhead of a full CDCT setup is not yet warranted.

---

## The Two Types of Contract Violations

**Type 1: Structural violation** — a field is missing, renamed, or has a different type.
```
• accessToken: Invalid input: expected string, received undefined
```
Caught immediately by schema validation.

**Type 2: Behavioral violation** — the field is present with the right type, but the value semantics changed.
```
role: "superuser"  // schema expects "admin" | "moderator" | "user"
```
Caught by enum validation in the schema.

---

## Endpoint-Specific Schema Design

A critical insight from real-world usage: **different endpoints return different shapes**, even for the same resource.

```typescript
// POST /auth/login → returns user profile WITH tokens
export const LoginResponseSchema = z.object({
  id:           z.number(),
  username:     z.string(),
  accessToken:  z.string().min(1),   // only on login
  refreshToken: z.string().min(1),   // only on login
});

// GET /auth/me → returns user profile WITHOUT tokens
export const AuthUserSchema = z.object({
  id:       z.number(),
  username: z.string(),
  // NO accessToken, NO refreshToken
});
```

Using `LoginResponseSchema` to validate a `/auth/me` response will always fail — not because the API is broken, but because the schema is wrong for that endpoint. Every endpoint needs the schema that matches what it *actually* returns.

---

## Contract Test Structure

Contract tests are a distinct test tier with a distinct purpose. They do not test business logic — they test interface stability.

```typescript
test.describe('@contract — User API schema contracts', () => {

  // GET operations — test with stable seed data
  test('GET /users response matches UsersListSchema', async ({ apiClient }) => {
    const response = await apiClient.get('/users?limit=3');
    await ResponseValidator.expectSchemaMatch(response, UsersListSchema, 'contract:GET /users');
  });

  // POST operations — test with generated data
  test('POST /users/add response matches UserSchema', async ({ apiClient, authService }) => {
    await authService.login();
    const response = await apiClient.post('/users/add', UserFactory.create());
    await ResponseValidator.expectSchemaMatch(response, UserSchema, 'contract:POST /users/add');
  });
});
```

**Key characteristics of contract tests:**
- One test per endpoint per HTTP method
- No business logic assertions — only schema validation
- Run on every PR and on every API deployment
- Failures block deployment — a contract violation is a breaking change

---

*Next: [Testing by Architecture Type](./testing-by-architecture-type.md)*
