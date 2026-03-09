# ROI and Reporting

> **Strategic depth — 05-observability**

---

## The ROI Problem

Quality Engineering investment is hard to justify with traditional ROI models because the primary value is in things that *did not happen* — production incidents prevented, customer impact avoided, rollbacks that were not needed.

This document provides a framework for measuring and communicating QE ROI in terms that resonate with engineering leadership and business stakeholders.

---

## The Cost of a Production Defect

Establish a baseline cost for production defects in your organization. This number varies widely but a reasonable model for a typical SaaS product:

```
Cost of one production defect =
  Engineering investigation time    (avg 4 hours × $150/hr)  = $600
+ On-call engineer time             (avg 2 hours × $200/hr)  = $400
+ Customer support overhead         (avg 10 tickets × $50)   = $500
+ Customer impact (churn risk)      (varies widely)          = $500–$5,000
+ Reputation cost                   (hard to quantify)       = ?
─────────────────────────────────────────────────────────────────────
Minimum cost per incident:                                   = $2,000–$6,500+
```

Once you have this number, every defect caught by the test suite before production has a calculable value.

---

## The QE ROI Model

```
Annual QE ROI =
  (Defects prevented × Cost per defect)
+ (Delivery velocity gain × Engineer cost)
─────────────────────────────────────────
  Annual QE investment

Example:
  50 prevented production defects × $3,000 = $150,000
+ 20% velocity gain for 10 engineers × $200k/yr × 0.2 = $400,000
─────────────────────────────────────────────────────────────
  Total value:  $550,000
  QE investment: $300,000 (3 QE engineers + tooling)
  ROI: 83%
```

---

## What to Report and to Whom

**To Engineering Leadership (weekly/monthly):**
- Test suite health score (reliability rate, MTTF)
- Defects caught in QE before production (this sprint/month)
- Production incidents that trace back to QE gaps
- Pipeline health (gate pass rate, suite runtime trend)

**To Product Leadership (quarterly):**
- Defect escape rate trend
- Release confidence score
- Test coverage vs risk areas

**To Executive Leadership (annually):**
- QE ROI calculation
- Maturity level progress
- Investment roadmap and expected outcomes

---

## The One-Page QE Health Report

```
QE HEALTH — Q1 2026
────────────────────────────────────────
Defects caught before production:  47
Defects escaped to production:      3    (Escape rate: 6%)
Production incidents prevented:     ~$140,000 in incident cost
Pipeline reliability:               98.4%
Mean time to feedback:              4.8 minutes
Contract coverage:                  94% (target: 100%)

Top risk: 3 services without contract tests — remediation in Q2

Team maturity level: 3 → targeting Level 4 by end of year
Investment request: 1 additional QE engineer for platform work
Expected outcome: Contract coverage to 100%, MTTF to < 3 minutes
────────────────────────────────────────
```

This format gives leaders what they need to make investment decisions in two minutes.

---

*Next: [Embedded vs Centralized QE](../06-org-and-team-models/embedded-vs-centralized-qe.md)*
