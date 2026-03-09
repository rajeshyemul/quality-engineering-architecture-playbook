# Environment Strategy

> **Deep technical + strategic — 03-platform-and-pipeline**

---

## Environment Tiers

```
Local Dev        CI / Staging        Pre-production      Production
──────────       ────────────        ──────────────      ──────────
Developer        Shared, auto-       Mirrors prod.       Real users.
machine.         deployed.           Final gate.         Synthetic
Mocked or        API + contract      Full E2E +          monitors only.
real external    tests run here.     performance.
APIs.
Fast feedback.
```

---

## Environment Requirements

**Stable:** Failures must mean application problems, not environment problems.

**Isolated:** Runs in one environment must not pollute another. Shared mutable state between test runs is a reliability killer.

**Representative:** Must reflect production behavior closely enough to provide real confidence.

**Accessible:** CI/CD pipelines must reach the environment without manual intervention (VPN, firewall rules managed as infrastructure).

---

## Credential Strategy Across Environments

```
Environment     .env file              CI secret names
───────────     ─────────              ───────────────
local           .env.local             n/a (file only)
ci/staging      .env.ci                CI_TEST_USERNAME / CI_TEST_PASSWORD
pre-prod        .env.preprod           PREPROD_TEST_USERNAME / etc.
```

`ConfigManager` reads environment-specific variables. The same test code runs everywhere — only the config differs.

**Never commit `.env` files with real credentials.** The `.env` file in the repo should contain only placeholder values (`TEST_USERNAME=YOUR_USERNAME_HERE`). Real values live in CI secrets or local `.env.local` files that are gitignored.

---

## Ephemeral Environments (Gold Standard)

Spin up a complete, isolated environment for each PR. Tear it down after merge.

**Benefits:** Zero cross-PR data interference. Every PR has its own clean state. Contract and integration tests run against real services.

**Requirement:** Platform engineering support (Kubernetes, Docker Compose, or a cloud platform with preview environments).

**When to invest:** When the team exceeds ~15 engineers and shared environment conflicts become a regular source of test failures.

---

*Next: [Quality as a Platform](./quality-as-a-platform.md)*
