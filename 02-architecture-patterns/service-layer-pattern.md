# Service Layer Pattern

> **Deep technical + strategic — 02-architecture-patterns**

---

## The Problem This Pattern Solves

Without a service layer, tests make raw HTTP calls. This creates a direct coupling between every test and every implementation detail of the API — the URL structure, the authentication method, the request format, the response parsing logic.

When the API changes — and it always changes — every coupled test breaks.

The service layer is the insulation that prevents API changes from cascading into test failures across the entire suite.

---

## Pattern Definition

The Service Layer pattern introduces a typed, domain-oriented abstraction between tests and the HTTP client. Tests call service methods that express *what* they want to do. The service handles *how* to do it.

```
Test                    Service                 ApiClient
─────                   ───────                 ─────────
userService          →  GET /users/:id      →   fetch(url, headers)
  .getUser(1)           validate schema         parse response
                        return typed User
```

The test knows about users. The service knows about the API. The client knows about HTTP. Each layer knows exactly one thing.

---

## Anatomy of a Well-Designed Service

```typescript
export class UserService {
  constructor(private readonly api: ApiClient) {}

  // ── Read operations ────────────────────────────────────────────

  async getUser(id: number): Promise<User> {
    const response = await this.api.get(`/users/${id}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UserSchema, 'getUser');
  }

  async getUsers(limit = 10, skip = 0): Promise<UsersList> {
    const response = await this.api.get(`/users?limit=${limit}&skip=${skip}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UsersListSchema, 'getUsers');
  }

  async searchUsers(query: string): Promise<UsersList> {
    const response = await this.api.get(`/users/search?q=${encodeURIComponent(query)}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UsersListSchema, 'searchUsers');
  }

  // ── Write operations ───────────────────────────────────────────

  async createUser(data: CreateUserRequest): Promise<User> {
    const response = await this.api.post('/users/add', data);
    await ResponseValidator.expectStatus(response, 201);
    return ResponseValidator.validateSchema(response, UserSchema, 'createUser');
  }

  async updateUser(id: number, data: Partial<User>): Promise<User> {
    const response = await this.api.put(`/users/${id}`, data);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UserSchema, 'updateUser');
  }

  async patchUser(id: number, data: Partial<User>): Promise<User> {
    const response = await this.api.patch(`/users/${id}`, data);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, UserSchema, 'patchUser');
  }

  async deleteUser(id: number): Promise<DeleteUserResponse> {
    const response = await this.api.delete(`/users/${id}`);
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, DeleteUserResponseSchema, 'deleteUser');
  }

  // ── Negative path helpers ──────────────────────────────────────

  async getUserExpectingNotFound(id: number): Promise<Response> {
    const response = await this.api.get(`/users/${id}`);
    await ResponseValidator.expectStatus(response, 404);
    return response; // return raw response for test-specific assertions
  }
}
```

### Design Rules for Services

**Rule 1: Services return typed domain objects, not raw responses.**
`getUser()` returns `User`. Tests work with `User`. Tests do not call `.json()`.

**Rule 2: Services own status assertion.**
If `GET /users/1` should return 200, `UserService.getUser()` asserts that. Tests should not re-assert what the service already guarantees.

**Rule 3: Services own schema validation.**
The service validates the response against its contract before returning. Tests receive data that is already guaranteed to match the schema.

**Rule 4: Negative paths are explicit methods.**
`getUserExpectingNotFound()` is cleaner than expecting tests to know to skip status assertion. Makes test intent clear.

**Rule 5: One service per domain.**
`UserService`, `AuthService`, `OrderService` — not `TestHelperService`. Domain boundaries in services should mirror domain boundaries in the system under test.

---

## The AuthService Pattern

Authentication is a cross-cutting concern that nearly every test needs. Its architecture requires special consideration.

```typescript
export class AuthService {
  constructor(private readonly api: ApiClient) {}

  // Login and store token in AuthManager (singleton)
  async login(expiresInMins = 30): Promise<string> {
    return authManager.login(this.api, expiresInMins);
  }

  // Login and return the full profile — for tests that need to verify the profile
  async loginAndGetProfile(): Promise<LoginResponse> {
    const response = await this.api.post('/auth/login', {
      username:      configManager.getUsername(),
      password:      configManager.getPassword(),
      expiresInMins: 30,
    });
    await ResponseValidator.expectStatus(response, 200);
    return ResponseValidator.validateSchema(response, LoginResponseSchema, 'loginAndGetProfile');
  }

  // Get authenticated user profile — NOTE: different schema than login response
  async getAuthenticatedUser(): Promise<AuthUser> {
    const response = await this.api.get('/auth/me');
    await ResponseValidator.expectStatus(response, 200);
    // /auth/me does NOT return accessToken/refreshToken — use AuthUserSchema
    return ResponseValidator.validateSchema(response, AuthUserSchema, 'getAuthenticatedUser');
  }

  logout(): void {
    authManager.clearTokens();
  }

  isAuthenticated(): boolean {
    return authManager.getAccessToken() !== null && !authManager.isTokenExpired();
  }
}
```

### The AuthManager Singleton

`AuthManager` handles token state. It is a singleton because the token is shared across all service instances in a test.

```typescript
class AuthManager {
  private accessToken:  string | null = null;
  private refreshToken: string | null = null;
  private tokenExpiry:  Date | null   = null;

  async login(api: ApiClient, expiresInMins: number): Promise<string> {
    // Reuse unexpired token — avoids redundant logins in the same test
    if (this.accessToken && !this.isTokenExpired()) {
      return this.accessToken;
    }

    const response = await api.post('/auth/login', {
      username:      configManager.getUsername(),
      password:      configManager.getPassword(),
      expiresInMins,
    });

    const profile = await ResponseValidator.validateSchema(
      response, LoginResponseSchema, 'authManager:login'
    );

    this.accessToken  = profile.accessToken;
    this.refreshToken = profile.refreshToken;
    this.tokenExpiry  = new Date(Date.now() + expiresInMins * 60 * 1000 - 30_000);

    return this.accessToken;
  }

  getAccessToken():  string | null { return this.accessToken; }
  isTokenExpired():  boolean { return !this.tokenExpiry || new Date() > this.tokenExpiry; }
  clearTokens():     void { this.accessToken = null; this.refreshToken = null; this.tokenExpiry = null; }
}

export const authManager = new AuthManager();
```

**Critical note on test isolation:** Because `authManager` is a singleton, its state persists between tests within the same worker. Tests that assert `isAuthenticated() === false` must call `authService.logout()` first to guarantee a clean state. The fixture teardown handles this automatically when configured correctly.

---

## Service Composition Patterns

### Pattern A: Standalone Services (Most Common)
Each service is independent. Tests use whatever services they need.

```typescript
test('user can be found after creation', async ({ userService }) => {
  const created = await userService.createUser(UserFactory.create());
  const fetched  = await userService.getUser(created.id);
  expect(fetched.firstName).toBe(created.firstName);
});
```

### Pattern B: Coordinated Services (Integration Tests)
Multiple services interact in a single test to verify cross-domain behavior.

```typescript
test('auth token gives access to user data', async ({ authService, userService }) => {
  await authService.login();
  const users = await userService.getUsers();
  expect(users.total).toBeGreaterThan(0);
});
```

### Pattern C: Pre-authenticated Services (Fixture Pattern)
The fixture handles authentication before the test starts, reducing boilerplate.

```typescript
// In ApiFixture.ts
authenticatedUserService: async ({ apiClient, authService }, use) => {
  await authService.login(); // done before test runs
  await use(new UserService(apiClient));
},

// In test
test('search returns results', async ({ authenticatedUserService }) => {
  const results = await authenticatedUserService.searchUsers('John');
  expect(results.users.length).toBeGreaterThan(0);
});
```

---

## What Services Should NOT Do

**Services should not contain test assertions (except status/schema validation).**
`expect(user.age).toBeGreaterThan(0)` does not belong in a service. That is a test-specific assertion.

**Services should not generate test data.**
`UserService.createUser(UserFactory.create())` — the factory call belongs in the test, not the service.

**Services should not be stateful (except AuthService).**
Each service method should be self-contained. The exception is `AuthService`, which intentionally manages token state.

**Services should not implement retry logic.**
Retry belongs in `ApiClient`. Services assume the client handles transient failures.

---

## Scaling the Service Layer

As the system under test grows, the service layer grows with it. Architectural principles for managing this growth:

**One file per service domain.** `UserService.ts`, `AuthService.ts`, `OrderService.ts`. Never combine unrelated operations.

**Export from a barrel index.** `src/services/index.ts` re-exports all services. Tests import from `../services`, not from individual files.

**Version services when APIs version.** `UserServiceV2.ts` that wraps `UserService` with V2-specific behavior. Do not modify existing service classes for breaking changes.

**Use service composition for complex scenarios.** `OrderService` might internally use `UserService` to validate that a user exists. This is fine — services can use services.

---

*Next: [Test Data Architecture](./test-data-architecture.md)*
