# Risk-Based Testing

> **Strategic depth — 07-future-of-qe**

---

## What Risk-Based Testing Is

Risk-based testing is the practice of allocating test effort proportionally to risk — spending more time and depth on areas that are most likely to fail and most costly if they do.

It is not new. What is new is the ability to apply risk analysis systematically and dynamically using data and AI, rather than relying on manual risk assessment that is slow and inconsistent.

---

## The Risk Model

Risk = Probability of Failure × Impact of Failure

```
                High Impact
                     │
    High Risk        │        High Impact
    Low Priority     │        High Priority
    (will fail but   │        (will fail AND
     low impact)     │         costly)
─────────────────────┼──────────────────────
    Low Risk         │        High Impact
    Deprioritize     │        Low Probability
    (stable,         │        (monitor but
     low impact)     │         don't over-test)
                     │
                Low Impact
```

**Probability inputs:** Change frequency, code complexity, defect history, test coverage gaps.

**Impact inputs:** Revenue path criticality, user impact scope, regulatory exposure, data integrity risk.

---

## Applying Risk-Based Testing to a Real System

Given the DummyJSON API framework used in this playbook:

| Endpoint | Probability | Impact | Risk Level | Test Depth |
|----------|-------------|--------|------------|-----------|
| POST /auth/login | High (auth changes often) | Critical (blocks everything) | HIGH | Deep — schema, status, token validity |
| GET /users/:id | Low (stable read) | Medium | MEDIUM | Contract + happy path |
| POST /users/add | Medium (write path) | Medium | MEDIUM | Schema + ID assignment |
| GET /users/search | Low | Low | LOW | Basic smoke test |

This analysis directly informs which tests get Tier 1 coverage and which get Tier 3.

---

## Dynamic Risk Assessment (AI-Enhanced)

Manual risk assessment is a point-in-time exercise. AI-enhanced risk assessment is continuous:

- **Change frequency** is measured from git history automatically
- **Defect density** is measured from production incident data
- **Coverage gaps** are identified from test-to-code coverage maps
- **Risk scores** are updated on every commit

The result: a risk score for every file, every service, and every endpoint — updated continuously. Test selection uses these scores to prioritize which tests run first and how deeply.

---

*Next: [Autonomous Quality Systems](./autonomous-quality-systems.md)*
