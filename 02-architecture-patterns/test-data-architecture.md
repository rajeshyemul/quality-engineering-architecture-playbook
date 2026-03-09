# Test Data Architecture

> **Deep technical + strategic — 02-architecture-patterns**

---

## The Core Problem

Test data is the most underestimated architectural concern in automation. Teams spend months building beautiful service abstractions and then watch their test suite become unreliable because of test data collisions, environment pollution, and hard-coded assumptions about shared data.

A test data architecture answers four questions:
1. Where does test data come from?
2. How do we ensure tests do not interfere with each other's data?
3. How do we handle data that must exist before tests run?
4. How do we clean up after tests?

---

## The Four Test Data Strategies

### Strategy 1: Generated Data (Preferred for most tests)
Tests generate their own unique data at runtime using factories.

```typescript
// Each call produces a unique object — parallel-safe
const user = UserFactory.create();
const created = await userService.createUser(user);
```

**Pros:** No shared state. Tests are always independent. Safe for parallel execution.
**Cons:** Requires the API to support create operations. Data does not persist across runs.
**Use for:** Create, update, delete operations. Any test that writes data.

### Strategy 2: Stable Seed Data (For read-only tests)
A known dataset exists in the environment that tests can read but never modify.

```typescript
// User ID 1 always exists in the DummyJSON environment
const user = await userService.getUser(1);
expect(user.id).toBe(1);
```

**Pros:** Fast. No setup required.
**Cons:** Fragile if seed data changes. Creates implicit dependency on environment state.
**Use for:** Read-only GET tests against stable records.

### Strategy 3: Test-scoped Data (For complex scenarios)
Data is created in test setup and torn down after the test completes.

```typescript
test.beforeEach(async ({ userService }) => {
  testUser = await userService.createUser(UserFactory.create());
});

test.afterEach(async ({ userService }) => {
  await userService.deleteUser(testUser.id); // cleanup
});
```

**Pros:** Tests are fully self-contained.
**Cons:** Slower (setup/teardown on every test). Cleanup can fail if test crashes.
**Use for:** Tests that need a specific pre-existing user/record state.

### Strategy 4: Database Seeding (For environment setup)
Scripts populate the environment with known datasets before the test suite runs.

**Use for:** E2E tests that require complex relational data. Not for unit or API tests.

---

## The Factory Pattern in Depth

Factories are the implementation of Strategy 1. A well-designed factory system has three layers:

```typescript
// Layer 1: Core factory — random valid data
class UserFactory {
  static create(overrides?: Partial<CreateUserRequest>): CreateUserRequest {
    return {
      firstName: faker.person.firstName(),
      lastName:  faker.person.lastName(),
      email:     faker.internet.email(),
      age:       faker.number.int({ min: 18, max: 65 }),
      ...overrides,  // override specific fields for specific scenarios
    };
  }
}

// Layer 2: Scenario factories — named, intent-expressing variants
class UserScenarios {
  static minorUser():   CreateUserRequest { return UserFactory.create({ age: 16 }); }
  static seniorUser():  CreateUserRequest { return UserFactory.create({ age: 75 }); }
  static adminUser():   CreateUserRequest { return UserFactory.create({ role: 'admin' }); }
}

// Layer 3: Bulk factories — for scale tests
class UserBulkFactory {
  static createMany(count: number): CreateUserRequest[] {
    return Array.from({ length: count }, () => UserFactory.create());
  }
}
```

---

## Data Isolation Principles

**Principle 1: Tests own the data they create.**
If a test creates a user, that test is responsible for the lifecycle of that user. Tests should not assume other tests will clean up after them.

**Principle 2: Tests must not depend on data created by other tests.**
Test execution order is not guaranteed in parallel execution. Tests that depend on a previous test's data will fail unpredictably.

**Principle 3: Use unique identifiers, not fixed values.**
`faker.internet.email()` generates a unique email each time. Hard-coding `test@example.com` creates conflicts when tests run in parallel.

**Principle 4: Read-only tests are safe to share stable data.**
A test that reads `/users/1` without modifying it can safely run in parallel with other tests reading the same record.

---

## Handling Nullable Fields in Schema Contracts

A common data architecture challenge: APIs return `null` for fields that were not provided during creation. The schema must handle this gracefully.

```typescript
// DummyJSON returns null for height/weight when a user is created
// without those fields — the schema must reflect reality
export const UserSchema = z.object({
  id:        z.number(),
  firstName: z.string(),
  height:    z.number().positive().nullable().optional(), // null for created users
  weight:    z.number().positive().nullable().optional(), // null for created users
  image:     z.string().nullable().optional(),            // null for created users
});
```

The schema is the contract between your tests and the API. It must accurately represent what the API *actually* returns, not what you wish it returned.

---

*Next: [Contract Testing Strategy](./contract-testing-strategy.md)*
