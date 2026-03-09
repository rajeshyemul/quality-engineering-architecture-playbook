# QE Center of Excellence

> **Strategic depth — 06-org-and-team-models**

---

## What a QE CoE Is

A Quality Engineering Center of Excellence (CoE) is a cross-functional group that owns QE standards, tooling, and best practices for the entire engineering organization. It is not a team that tests products — it is a team that enables the organization to test products well.

The CoE model sits between the fully centralized department and the fully embedded model. It provides central governance and infrastructure while allowing product teams to maintain ownership of their quality practices.

---

## CoE Responsibilities

**Standards:** Define what "good" looks like for automation architecture, test design, coverage thresholds, and flakiness budgets. Document it (this playbook is an example).

**Framework:** Maintain the shared automation framework. Version it. Provide migration guides when it changes.

**Tooling:** Own the CI/CD quality gate templates, the contract registry, the test health dashboards.

**Enablement:** Run QE guild sessions, training, and architecture reviews. Help product teams implement QE patterns.

**Governance:** Review and approve automation architectures for new services. Audit test health metrics quarterly.

---

## CoE vs Platform Team

The CoE has a governance and standards mandate that a pure platform team does not. A platform team builds things. A CoE builds things *and* sets the rules for how those things are used.

In smaller organizations, one team does both. In larger organizations, they are often separate — a platform engineering team builds the infrastructure, the CoE sets the standards.

---

## Starting a QE CoE

A CoE does not require a large team to start. The minimum viable CoE:

1. **One QE principal/architect** who owns the standards and framework
2. **A documented standard** (a version of this playbook adapted for your context)
3. **A regular QE guild** — monthly cross-team session where QE practices are shared
4. **One shared artifact** — a framework, a CI template, or a contract registry that all teams use

Start with this. Expand as adoption grows.

---

*Next: [Engineering Partnerships](./engineering-partnerships.md)*
