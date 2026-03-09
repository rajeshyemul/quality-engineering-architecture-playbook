# Framework Selection Guide

> **Deep technical + strategic — 02-architecture-patterns**

---

## The Decision

Choosing an automation framework is a 3–5 year architectural commitment. It affects hiring, tooling, CI/CD pipeline design, and the skills your team needs to develop. It should not be made based on what engineers are personally comfortable with or what was trending at the last conference.

---

## The Selection Matrix

Score each candidate framework 1–5 on each criterion. Weight by organizational priority.

| Criterion | Weight | Playwright | Cypress | RestAssured | pytest + requests |
|-----------|--------|-----------|---------|-------------|------------------|
| TypeScript-first | High | 5 | 4 | 1 | 2 |
| API testing capability | High | 5 | 3 | 5 | 5 |
| Parallelization | High | 5 | 3 | 4 | 4 |
| Fixture / DI support | High | 5 | 2 | 3 | 3 |
| CI/CD integration | Medium | 5 | 4 | 4 | 4 |
| Reporting | Medium | 5 | 4 | 3 | 3 |
| Learning curve | Medium | 3 | 4 | 3 | 4 |
| Community maturity | Medium | 4 | 5 | 5 | 5 |
| Browser testing (if needed) | Variable | 5 | 5 | 1 | 1 |

---

## Decision Tree

```
Do you need browser/UI testing?
│
├─ YES → Do you use TypeScript?
│        ├─ YES → Playwright
│        └─ NO  → Playwright or Cypress (both work)
│
└─ NO (API only) → What is your team's primary language?
                   ├─ TypeScript/JavaScript → Playwright
                   ├─ Java                 → RestAssured + JUnit 5
                   └─ Python               → pytest + httpx/requests
```

---

## When to Build vs Buy

**Build a custom framework when:**
- You have highly specialized protocols (gRPC, GraphQL, WebSockets at scale)
- Existing frameworks cannot meet your performance requirements
- Your team has 3+ dedicated QE engineers who can own it

**Use an existing framework when:**
- Team size is under 20 engineers
- You need to move quickly
- The framework covers 80%+ of your needs

**Hybrid (most common):** Use an existing framework as the foundation. Build custom layers (services, factories, validators) on top of it. This is exactly what the api-framework-architect-pw-ts project does — Playwright as the runner, custom service and validation layers on top.

---

## Avoiding Lock-in

The service layer pattern protects you from framework lock-in. Because tests call `userService.getUser()` rather than Playwright-specific APIs, migrating the runner in the future requires changing `ApiClient` — not every test file.

Design for replaceability at the runner layer, even if you never end up replacing it.

---

*Next: [CI/CD Quality Gates](../03-platform-and-pipeline/ci-cd-quality-gates.md)*
