# QE Maturity Model

> **Strategic depth — 04-scaling-quality**

---

## Purpose

A maturity model gives your organization a shared vocabulary for where you are and where you are going. It creates a roadmap for QE investment and provides a communication tool for explaining QE capability gaps to engineering leadership.

---

## The Five Levels

### Level 1: Reactive Quality
**Characteristics:**
- Testing happens after development is complete
- Primarily manual test execution
- No automated regression
- Defects found in QA phase or production
- Test coverage is undefined

**What it feels like:** Every release is a fire drill. QA is the bottleneck before every deployment. Production incidents are common.

---

### Level 2: Automated Basics
**Characteristics:**
- Some automated tests exist
- CI pipeline runs automated tests on commit
- Coverage is ad-hoc — tests exist where engineers chose to write them
- No architecture or standards for automation
- Flaky tests exist and are tolerated

**What it feels like:** Some automation exists but it is not trusted. The suite is slow, flaky, and incomplete. Teams debate whether to run it.

---

### Level 3: Structured QE
**Characteristics:**
- Layered framework architecture in place
- Service layer, schema contracts, factory pattern adopted
- Tiered CI execution (smoke, integration, contract)
- Flaky test policy exists and is enforced
- QE standards documented and followed

**What it feels like:** The test suite is trusted. Engineers use it to make deployment decisions. Test failures have clear meaning.

---

### Level 4: QE Platform
**Characteristics:**
- QE operates as an internal platform team
- Shared frameworks, libraries, and CI templates
- Contract testing across all service boundaries
- Test health metrics tracked and visible
- QE enabling product teams, not writing all tests

**What it feels like:** Product teams ship with confidence. QE is a multiplier, not a gatekeeper. Quality metrics are part of engineering dashboards.

---

### Level 5: Intelligent Quality
**Characteristics:**
- AI-assisted test generation for routine coverage
- Self-healing tests that adapt to UI/API changes
- Risk-based test selection — CI only runs relevant tests
- Predictive quality — analytics surface risk before release
- Autonomous monitoring with AI-driven failure triage

**What it feels like:** Quality is proactive, not reactive. The system surfaces risk before engineers ask. Test maintenance overhead is low despite large coverage.

---

## Using This Model

**For self-assessment:** Score your organization honestly against each level. Be specific about which characteristics are present vs absent.

**For communication:** Present the model to engineering leadership as a roadmap. "We are at Level 2. Here is what Level 3 requires and what it will deliver."

**For hiring:** Use the target maturity level to define the skills you need. Level 3 needs strong automation engineers. Level 4 needs platform engineers. Level 5 needs ML/AI expertise.

**For investment:** Each level transition requires specific tooling, process, and headcount investments. Make those explicit in the QE budget discussion.

---

*Next: [Automation Observability](../05-observability/automation-observability.md)*
