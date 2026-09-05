# Engineering Requirements Document (ERD) — Template

> Source: [Confluence — DEV / DRAFT - Engineering Requirements Document (ERD) Template](https://ratehub.atlassian.net/wiki/spaces/DEV/pages/4365680645/DRAFT+-+Engineering+Requirements+Document+ERD+Template)
> Space: DEV · Status: Draft · Page version: 9 · Last updated: 2026-08-19

> **What this is.** The ERD is the umbrella planning doc for a piece of engineering work. It explains **how** we'll build something and **why we chose this approach** over the alternatives. The product _why_ — the user/business case — lives in the PRD (owned by product); the ERD links to it rather than restating it. It's written by the engineering team.
>
> **How much to write.** Right-size it. A small, well-understood change might be a few paragraphs. A complex or risky one earns a fuller Technical Vision, a spun-out **System Design** page, and a co-owned **Test Plan**. Let the principles below guide where the effort goes.
>
> **Only one person owns and writes the ERD**. It’s very difficult to have a cohesive readable ERD when multiple people work on it. You can seek input from others, but only one person should own it. You can transfer ownership in the case of vacations, time off, emergencies, etc.

---

## Metadata

|  |  |
| --- | --- |
| **Status** | `Draft` · `In Review` · `Approved` · `Implemented` |
| **Author** | @you |
| **Reviewers** | @tech-lead, @pm, @qa, @design (as relevant) |
| **Last updated** | YYYY-MM-DD |
| **Related** | PRD · Jira epic · System Design · Test Plan · Figma · prior ERDs |

---

## Guiding principles

* **Plans aren't valuable; the alignment they generate is valuable.** Only plan as much as necessary to align the team and stakeholders.
* We prefer **rapid iteration, feedback, and adjustment** over detailed, rigorous plans — agile as much as possible.
* These are **guidelines, not laws.** You can break them, but understand _why_ you're making the exception.
* _"It is easier to refactor an approach than code."_
* **Every significant design choice should show its alternatives.** A decision stated without the options it beat isn't a decision a reviewer can evaluate — it's an assertion. (See _Design Decisions_ below.)
* **Test at the lowest level that gives you confidence.** Plan testing up front, with QA, not after the code is written. (See the Test Plan.)

---

## Context

_Link the PRD in the header — product owns the problem case (who has the problem, why it matters, what success looks like for users/the business), so don't restate it here. Summarise in two or three sentences so this doc reads on its own, then move into the engineering framing below. If there's no PRD for this work, capture the brief problem context here instead._

## Goals & non-goals

_Don't copy the PRD's goals — translate them. Take the product goals and derive the engineering goals, non-functional requirements (performance, scalability, reliability), and the technical non-goals that follow._

_Goals: the specific technical outcomes this work should achieve — measurable where possible._

_Non-goals: what this work is deliberately **not** doing. This section prevents scope creep and saves review cycles; don't skip it._

## Technical vision

_How is this going to work? Words, diagrams, or both — whatever conveys the idea best. Highlight:_

* _Architecture — the major components and how they fit together_
* _Inter-system communication — event schemas, request/response contracts, public interfaces_
* _For UI components: draw component boundaries and show props on the design mocks_
* _For services: draw component boundaries and how they communicate with the rest of the system_

## Design decisions

> **This is the section we've historically been weakest on. It is required for any non-obvious choice.**
> For each significant decision — an architecture pattern, a datastore, a library, a build-vs-buy, a sync-vs-async — record the options you weighed and _why_ you landed where you did. Reviewers: if a decision appears here with no alternatives, push back.

**Decision: _[name the choice, e.g. "Event-driven vs. synchronous calls between rates and quotes"]_**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — _(chosen)_** | … | … |
| B | … | … |
| C / do nothing | … | … |

* **Decision:** _what we chose_
* **Rationale & tradeoff accepted:** _the deciding factor, and what we're knowingly giving up_
* _(For weighty, long-lived choices, promote this to a standalone ADR and link it.)_

_(Repeat per significant decision.)_

## Milestone list

_A milestone is something that can be **demoed** and/or **deployed** — including, e.g., hitting an API with_ `curl`. This is **not** an exhaustive task list.

_Break the project into small, deliverable bits of value over time. Each milestone should land in a couple of weeks at most — ideally shorter. Don't spend months building something to deliver it all at once; we need to keep checking the solution against stakeholder needs and our integration assumptions._

1. _Milestone — what's demoable/deployable_

## Test plan

_Summarise the testing approach, or link to a full **Test Plan** for larger work. Either way this is **co-owned with QA** and decided up front. Capture: what risks we're testing for, what's covered at which level (unit / integration / E2E), what stays manual/exploratory, and who owns each. See the Test Plan template for the test-level decision guide._

## Deployment plan

_How do we finish and ship this?_

* _What must be validated prior to deployment? (**QA heavily involved** in defining this.)_
* _Do multiple teams need to be involved?_
* _If replacing an existing system: how do we cut over?_
* _If upgrading one: how do we introduce the change gradually?_
* _Given the move toward daily / multiple-per-day deploys: feature flags, phased rollout, backwards compatibility, and the rollback plan._

## Open questions

_Writing the ERD always surfaces questions about requirements. Write them down for traceability; assign owners; resolve before Approved._

| Question | Assignee | Answer |
| --- | --- | --- |
| … | @… |  |

## Appendix

_Links to any related source of information. Decision log / ADRs. Reference material._
