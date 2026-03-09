# Embedded vs Centralized QE

> **Strategic depth — 06-org-and-team-models**

---

## The Core Tradeoff

Every engineering organization must make a fundamental decision about where QE engineers sit: embedded in product teams, or centralized in a dedicated QE function. This decision has cascading effects on speed, consistency, specialization, and scale.

There is no universally correct answer. The right model depends on your organization's size, maturity, and strategic priorities.

---

## The Embedded Model

QE engineers are members of product squads, reporting to engineering managers alongside developers.

```
Product Team A          Product Team B          Product Team C
──────────────          ──────────────          ──────────────
Dev, Dev, Dev           Dev, Dev, Dev           Dev, Dev, Dev
QE Engineer ←           QE Engineer ←           QE Engineer ←
                  (each QE reports to their squad's EM)
```

**Strengths:**
- Deep domain knowledge — the QE engineer knows the product area intimately
- Fast feedback loops — QE is in every standup, every sprint planning, every design review
- High trust — developers and QE engineers build strong working relationships
- Shift-left by default — QE is involved from the beginning of every feature

**Weaknesses:**
- Inconsistency — each QE engineer develops their own patterns. Standards drift across teams.
- Isolation — QE engineers have no peer community. Career growth is limited.
- Framework fragmentation — each team builds their own automation approach
- Scaling cost — every new product team requires a new QE hire

**Best fit:** Organizations under ~50 engineers. Teams where domain knowledge is the primary QE value. Early-stage products where speed of feedback matters most.

---

## The Centralized Model

QE engineers are in a dedicated QE department, typically serving multiple product teams.

```
QE Department
──────────────────────────────────────
QE Director
├── QE Lead (Platform)
│   ├── QE Engineer (Framework)
│   └── QE Engineer (CI/CD)
├── QE Lead (Product Area A)
│   └── QE Engineers x2
└── QE Lead (Product Area B)
    └── QE Engineers x2
```

**Strengths:**
- Consistency — shared standards, shared frameworks, shared tooling
- Specialization — QE engineers can develop deep expertise
- Career path — clear growth trajectory within the QE function
- Efficiency — shared infrastructure benefits all teams

**Weaknesses:**
- Slower feedback — QE is not in the room when decisions are made
- Context switching — QE engineers serve multiple teams, domain knowledge is shallower
- "Throwing over the wall" — risk of reverting to QA-phase thinking
- Distance from delivery — QE is not part of the product team's sprint rhythm

**Best fit:** Organizations with 50+ engineers. Mature products with stable domains. Organizations that have invested in QE platform capabilities.

---

## The Hybrid Model (Most Common at Scale)

A platform QE team provides shared infrastructure. QE leads are embedded in major product areas.

```
QE Platform Team (centralized)
├── Framework maintenance
├── CI/CD quality gates
├── Contract registry
└── Standards and training

   ↕ (consumed by)

Product Area QE Leads (embedded)
├── Lead for Platform/Infrastructure
├── Lead for Core Product
└── Lead for Growth/Acquisition
```

**The division of responsibility:**
- Platform QE: Build the tools. Set the standards. Measure quality health across the org.
- Embedded QE leads: Apply the tools. Enforce the standards. Drive quality within their area.

This model scales from ~30 engineers to large organizations because the platform work centralizes efficiency while the embedded work maintains domain depth.

---

## Making the Decision

Ask these questions to determine the right model for your current stage:

1. **How many engineers?** Under 30: embedded. 30–100: hybrid. 100+: platform-heavy hybrid.
2. **How mature is your QE practice?** Early: embedded (build domain knowledge first). Mature: centralize platform work.
3. **How often do teams share components?** High sharing: centralize. Low sharing: embed.
4. **What is your hiring reality?** Specialized QE engineers are easier to retain in a centralized function with a career path.

---

*Next: [QE Center of Excellence](./qe-center-of-excellence.md)*
