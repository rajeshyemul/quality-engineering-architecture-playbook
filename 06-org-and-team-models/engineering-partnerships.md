# Engineering Partnerships

> **Strategic depth — 06-org-and-team-models**

---

## QE Does Not Work Alone

Quality Engineering's effectiveness is directly proportional to how well it partners with other engineering functions. The highest-impact QE teams are deeply integrated with Platform Engineering, Development, and Product Management.

---

## QE × Platform Engineering

Platform Engineering owns the CI/CD infrastructure, the Kubernetes clusters, the observability stack, and the deployment pipelines. QE depends on all of these.

**Joint responsibilities:**
- Quality gates are defined by QE but implemented in pipelines owned by Platform
- Test environments are provisioned by Platform, configured by QE
- Test result storage and dashboards require observability infrastructure owned by Platform

**How to make it work:** QE should have a named partner on the Platform team. Quality infrastructure work should appear in Platform's roadmap alongside reliability and deployment work.

---

## QE × Development

The most critical partnership. QE's value is delivered through the developer experience — tests that provide useful feedback, contracts that prevent integration failures, frameworks that make it easy to write good tests.

**Developer experience principles for QE:**
- Tests must be easy to run locally (one command, documented)
- Test failures must have clear, actionable error messages
- QE frameworks must be documented and accompanied by examples
- QE engineers must be available to pair with developers on testing hard problems

**Friction points to avoid:**
- Tests that only work in CI, not locally
- Test failures with cryptic error messages that require QE interpretation
- Mandatory test coverage thresholds without framework support to meet them

---

## QE × Product Management

Product Managers need to understand quality risk at the feature level. QE should provide:

**Feature risk assessment:** Before a feature is built, flag high-risk areas that require extra test coverage or design review.

**Release confidence signal:** Before a release, provide a clear signal — "this is safe to ship" or "these specific risks exist."

**Production quality data:** Share defect escape rates and production quality trends in product review meetings.

The goal is making QE a business partner, not just a technical function. When PM understands quality risk, they make better prioritization decisions.

---

*Next: [AI in Quality Engineering](../07-future-of-qe/ai-in-quality-engineering.md)*
