# Automation Framework Architecture

> **Deep technical + strategic — 02-architecture-patterns**

---

## Core Principle

An automation framework is a software product. It has users (the engineers who write tests), reliability requirements (it must not introduce flakiness), performance requirements (it must not slow delivery), and a maintenance burden (it will need to evolve).

Most automation frameworks fail not because the tests are wrong, but because the framework was not designed as a product. It was assembled from scripts.

This document describes the architectural patterns that make automation frameworks maintainable, scalable, and trustworthy.

---

## The Layered Architecture Model

A well-designed automation framework has clearly separated concerns across layers. Each layer has one responsibility. Lower layers know nothing about upper layers.

```
┌───────────────────────────────────────────────────────────┐
│                     TEST LAYER                            │
│   Tests express intent. No HTTP, no JSON parsing here.   │
│   userService.getUser(1)  →  expect(user.id).toBe(1)     │
├───────────────────────────────────────────────────────────┤
│                   SERVICE LAYER                           │
│   Domain-specific operations. Orchestrates API calls.    │
│   UserService, AuthService, OrderService                  │
├───────────────────────────────────────────────────────────┤
│                  VALIDATION LAYER                         │
│   Schema validation, status assertions, error messages.   │
│   ResponseValidator, ContractValidator                    │
├───────────────────────────────────────────────────────────┤
│                    CLIENT LAYER                           │
│   Raw HTTP. Retry logic, logging, timing, auth headers.   │
│   ApiClient                                               │
├───────────────────────────────────────────────────────────┤
│                INFRASTRUCTURE LAYER                       │
│   Configuration, secrets, fixture injection, factories.  │
│   ConfigManager, ApiFixture, UserFactory, AuthFactory    │
└───────────────────────────────────────────────────────────┘
```

### Why This Layering Matters

**Change isolation.** When the API adds a new required header, you change `ApiClient`. When a schema field is renamed, you change the contract. When a service endpoint changes, you change `UserService`. No test file needs to change for any of these.

**Readability.** A test that reads `await userService.createUser(UserFactory.create())` communicates intent. A test that reads raw `fetch()` calls with JSON manipulation communicates nothing.

**Maintainability at scale.** As the test suite grows from 50 to 500 tests, the layered architecture ensures that common patterns are in one place — not repeated in every test file.

---

## Layer 1: Infrastructure Layer

This is the foundation everything else builds on. It should be the most stable layer — changes here are rare.

### ConfigManager

Centralizes all configuration. No test or service reads `process.env` directly.

```typescript
// ❌ Wrong — scattered env reads
const baseUrl = process.env.BASE_URL;
const username = process.env.TEST_USERNAME; // in test file

// ✅ Correct — centralized config
class ConfigManager {
  getBaseUrl():  string { return this.require('BASE_URL'); }
  getUsername(): string { return this.require('TEST_USERNAME'); }
  getPassword(): string { return this.require('TEST_PASSWORD'); }

  private require(key: string): string {
    const val = process.env[key];
    if (!val) throw new Error(`Missing required env var: ${key}`);
    return val;
  }
}
export const configManager = new ConfigManager();
```

**Why it matters:** When you need to support multiple environments (dev, staging, production), you change `ConfigManager` once. Nothing else changes.

**Critical note:** Never use `process.env.USERNAME` — it is a reserved OS variable on macOS/Linux that returns the system login name, not your test credential.

### Factory Pattern

Factories generate test data. They are the single source of truth for what valid test inputs look like.

```typescript
class UserFactory {
  // Random valid user — safe for parallel tests
  static create(overrides?: Partial<CreateUserRequest>): CreateUserRequest {
    return {
      firstName: faker.person.firstName(),
      lastName:  faker.person.lastName(),
      email:     faker.internet.email(),
      age:       faker.number.int({ min: 18, max: 65 }),
      ...overrides,
    };
  }

  // Intentionally invalid — for negative tests
  static createInvalid(): Record<string, unknown> {
    return { firstName: '', lastName: null, email: 'not-an-email', age: -5 };
  }
}
```

**Critical property:** Every call to `UserFactory.create()` returns a unique object. Tests do not share data. This makes them safe for parallel execution.

### Fixture Injection (Playwright)

Playwright's `test.extend()` enables dependency injection — tests declare what they need, the fixture wires it up.

```typescript
// ApiFixture.ts
export const test = base.extend<{
  apiClient:    ApiClient;
  authService:  AuthService;
  userService:  UserService;
}>({
  apiClient: async ({}, use) => {
    await use(new ApiClient(configManager.getBaseUrl()));
  },
  authService: async ({ apiClient }, use) => {
    const svc = new AuthService(apiClient);
    await use(svc);
    svc.logout(); // cleanup — critical for test isolation
  },
  userService: async ({ apiClient, authService }, use) => {
    await authService.login();
    await use(new UserService(apiClient));
  },
});
```

**What this gives you:**
- Tests never instantiate dependencies directly
- Cleanup runs automatically after each test
- Changing how a service is constructed means changing the fixture, not 200 tests

---

## Layer 2: Client Layer

The `ApiClient` is the only place that makes HTTP requests. Everything above it uses services.

### Responsibilities of ApiClient

1. **Base URL management** — prepend the base URL to all paths
2. **Auth headers** — inject the Bearer token on authenticated requests
3. **Retry logic** — retry transient failures (network timeouts, 429s)
4. **Request logging** — log every request with method, path, status, and duration
5. **Metrics collection** — record timing data for observability

```typescript
class ApiClient {
  async get(path: string, options?: RequestOptions): Promise<Response> {
    return this.request('GET', path, undefined, options);
  }

  async post(path: string, body: unknown): Promise<Response> {
    return this.request('POST', path, body);
  }

  private async request(
    method: string,
    path:   string,
    body?:  unknown,
    opts?:  RequestOptions,
    attempt = 1
  ): Promise<Response> {
    const url     = `${this.baseUrl}${path}`;
    const headers = this.buildHeaders(opts);
    const start   = Date.now();

    try {
      const response = await fetch(url, {
        method,
        headers,
        body: body ? JSON.stringify(body) : undefined,
      });

      this.logger.info(`${method} ${path}`, {
        status:     response.status,
        durationMs: Date.now() - start,
      });

      return response;
    } catch (err) {
      if (attempt < this.maxRetries) {
        await this.delay(attempt * 1000); // exponential backoff
        return this.request(method, path, body, opts, attempt + 1);
      }
      throw err;
    }
  }
}
```

**Key architectural decision:** `ApiClient` returns raw `Response` objects. It does not parse JSON or validate schemas. That is the job of layers above it. This separation means the client can be used for both valid and invalid response testing without the validation layer getting in the way.

---

## Layer 3: Validation Layer

The `ResponseValidator` is responsible for two things: status assertion and schema validation.

### Schema Validation with Zod

Zod is the validation engine. The key architectural principle is **schema as single source of truth**.

```typescript
// The schema defines the shape
export const UserSchema = z.object({
  id:        z.number(),
  firstName: z.string(),
  lastName:  z.string(),
  email:     z.string().email(),
  // ...
});

// The TypeScript type is derived from the schema — never written manually
export type User = z.infer<typeof UserSchema>;
```

**Why this matters:** If you write the interface manually and separately from the schema, they will drift. One will be updated and the other won't. Using `z.infer<>` ensures they are always identical by construction.

### validateSchema vs expectSchemaMatch

Two methods with distinct purposes:

```typescript
class ResponseValidator {
  // Use when you need the typed data back (most service methods)
  static async validateSchema<T>(
    response: Response,
    schema:   ZodSchema<T>,
    label:    string
  ): Promise<T> {
    const raw = await response.json();
    const result = schema.safeParse(raw);
    if (!result.success) {
      throw new Error(`[${label}] Schema validation failed:\n${formatErrors(result.error)}`);
    }
    return result.data; // typed, validated data
  }

  // Use for pure assertion in contract tests (returns void)
  static async expectSchemaMatch<T>(
    response: Response,
    schema:   ZodSchema<T>,
    label:    string
  ): Promise<void> {
    const raw = await response.json();
    const result = schema.safeParse(raw);
    expect(result.success, formatContractViolation(result, label)).toBe(true);
  }
}
```

**The rule:** If you need to use the data after validation, use `validateSchema`. If you only need to assert the shape, use `expectSchemaMatch`.

---

## Layer 4: Service Layer

Services expose domain operations. They are the API of your test framework — what test authors actually call.

```typescript
class UserService {
  constructor(private readonly api: ApiClient) {}

  async getUser(id: number): Promise<User> {
    const response = await this.api.get(`/users/${id}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UserSchema, 'getUser');
  }

  async createUser(data: CreateUserRequest): Promise<User> {
    const response = await this.api.post('/users/add', data);
    await ResponseValidator.expectStatus(response, 201);
    return ResponseValidator.validateSchema(response, UserSchema, 'createUser');
  }

  async searchUsers(query: string): Promise<UsersList> {
    const response = await this.api.get(`/users/search?q=${query}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UsersListSchema, 'searchUsers');
  }
}
```

**What the service layer buys you:**

Every test that needs to create a user calls `userService.createUser()`. When the endpoint changes from `/users/add` to `/users/create`, you change one line in `UserService`. Zero test files change.

---

## Layer 5: Test Layer

Tests should read like specifications. They should express intent, not mechanics.

```typescript
// ❌ A test that knows too much about implementation
test('create user', async () => {
  const response = await fetch('https://dummyjson.com/users/add', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Authorization: `Bearer ${token}` },
    body: JSON.stringify({ firstName: 'John', lastName: 'Doe', email: 'john@test.com' }),
  });
  const data = await response.json();
  expect(response.status).toBe(201);
  expect(data.id).toBeDefined();
  expect(data.firstName).toBe('John');
});

// ✅ A test that expresses intent
test('POST /users/add creates a user and returns it with an assigned ID', async ({ userService }) => {
  const payload = UserFactory.create();
  const user    = await userService.createUser(payload);

  expect(user.id).toBeGreaterThan(0);
  expect(user.firstName).toBe(payload.firstName);
  expect(user.email).toBe(payload.email);
});
```

The second test is shorter, clearer, and immune to implementation changes in auth, headers, URLs, and JSON parsing.

---

## Framework Selection Guide

When choosing or evaluating an automation framework for API testing, apply this decision matrix:

| Criteria | Weight | Questions to ask |
|----------|--------|-----------------|
| **Language alignment** | High | Does the team already know this language? Learning a new language adds risk. |
| **TypeScript support** | High | Type safety in tests catches schema drift at compile time. |
| **Parallelization** | High | Can it run tests across multiple workers? Is worker count configurable? |
| **Fixture/DI support** | Medium | Does it support setup/teardown patterns that enforce test isolation? |
| **Reporting** | Medium | Does it produce reports that CI/CD pipelines and stakeholders can consume? |
| **Community + maintenance** | Medium | Is it actively maintained? Are security patches applied? |
| **Extension model** | Low | Can you add custom reporters, fixtures, or plugins? |

**For API-focused test suites in TypeScript environments:** Playwright is the current strong recommendation. It was designed for browser testing but its fixture injection model, worker parallelization, retry logic, and reporting are excellent for API testing.

---

## Anti-Patterns to Avoid

**God Test File:** One test file with 200 tests and all the setup code inline. Split by domain.

**Shared Mutable State:** Tests that modify global variables or share service instances. Each test needs its own state.

**Magic Numbers:** `expect(users.length).toBe(209)` — tests that depend on exact data counts in a shared environment will fail unpredictably.

**Nested Await Hell:** Tests with deeply nested async calls with no error context. Wrap in services with clear labels.

**Test-as-Documentation Lies:** Test names that do not match what the test actually checks. The test name is the spec.

---

*Next: [Service Layer Pattern](./service-layer-pattern.md)*
