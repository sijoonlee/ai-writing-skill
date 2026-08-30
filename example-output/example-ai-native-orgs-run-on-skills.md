# AI-Native Organizations Run on Skills

**Source:** https://www.youtube.com/watch?v=M05vON8i0aI
**Speaker:** Distinguished engineer at QuantumBlack
**Length:** ~20 minutes

---

## Everyone is building skills; almost nobody governs them

The talk opens with a show of hands. Many in the room had created and
used skills. Fewer were sharing them across their team. Only a handful
had governed and maintained skills across their organization. That gap —
between individual use and organizational practice — is the subject of
the talk.

## The four-step coding loop is one step in a much larger lifecycle

Most coding agents are shaped around specify → design/plan → break into
tasks → implement. That is a familiar picture, and it describes building
one product increment.

The actual enterprise lifecycle is much wider: product strategy and
success metrics, market and competitive research, customer interviews,
discovery and problem framing, solution validation and experiments, user
stories, then data preparation — cleaning the catalog, adjusting endpoint
integrations — then data product delivery with pipelines and quality
validation. Only then does the four-step increment loop begin. After it
come platform engineering and ops, infrastructure provisioning, launch,
performance optimization, incident response, and back around.

And there isn't one such lifecycle per organization. In an 18-year career
the speaker has consistently found several running in parallel — mobile,
internal platforms, customer-facing systems, different departments — each
shaped differently. The diagram shown, he estimates, captures maybe
10–20% of the real landscape.

## Of the four workflow components, only skills carry your know-how

The agentic stack has two loops. The inner loop is the code agent
harness: context manager, tools and MCPs, memory and state, skills
loader. The outer loop is workflows, built from four components — hooks,
MCP servers, sub-agents, and skills. Underneath sit enablement
components: environment sandbox, MCP gateway, model gateway, a knowledge
graph over IT systems and codebase and skills registry, and a workflow
marketplace. A context layer assembles project instructions (the
CLAUDE.md / agents file), tool and MCP schemas, memory and conversation
history, and retrieved content.

Of the four workflow components, three don't carry organizational
knowledge. Hooks fire on events. MCP servers are mostly consumed, not
authored — few teams build their own. Sub-agents exist to keep the
context window small by delegating a task. The know-how ends up in the
skills. Without well-structured skills, the workflow isn't
deterministic. Workflows themselves are best understood as harness
blueprints that shape agent behavior at runtime.

Two pieces of evidence that this matters now: eight months from
Anthropic's first skills article to an open standard adopted across agent
harnesses (by roughly February, most agents pull skills mid-task, visible
in their thinking if you watch); and a snapshot of public GitHub repos
and skill registries showing rapid growth in skills created. On
skills-bench, the latest models did well on auto-engineering and
cybersecurity tasks without skills — as expected, and improving — but
scored clearly higher with skills applied.

## Skills are the microservices problem again, and the design principles transfer

This isn't a new class of problem. The microservices era already produced
the design principles, and they carry over. Skills should be **reusable**;
**modular**; **discoverable**, so a team that needs one can find it rather
than rebuild it; **portable** across both workflows and harnesses, since a
skill written for Claude Code moves to Cursor unchanged; **specialized** to
one task rather than monolithic; **composable**, so skills don't duplicate
or conflict; **consistent and deterministic**; and **cost-efficient**.

Cost is where the progressive disclosure pattern earns its place — the
right skills, in the right amount, at the right time, which is what keeps
token usage down. Together these make organizational know-how executable,
portable, and cheap.

The composability payoff, in one example: a regulatory disclosure review
workflow pulls a data retention policy skill, disclosure standards, GDPR
rules, and fill-in templates at runtime. Any feature built across web or
mobile respects the same rules, and the output is deterministic — an
audit report you can store, plus identified improvements that loop back
into the codebase.

## Ungoverned skills become a new class of technical debt

Left ungoverned, skills accumulate the failures you would predict:

- **Duplication** — teams on the same stack and infrastructure build the
  same skills repeatedly without sharing them.
- **Quality decay** — skills must be validated not only against their
  task but against each new model release, or they degrade over time.
- **Undiscoverable** — the same problem internal developer portals solved
  for services: you should be able to consult a catalog instead of asking
  around.
- **No ownership** — unowned skills don't get maintained or scaled.
- **No composability** — it doesn't arrive by default; it takes a
  governed, domain-driven approach to shaping the catalog.
- **Security** — public skills can carry prompt injection, and skills
  contain scripts by design (that's the deterministic part), so without a
  security pipeline you may be pulling something unsafe.
- **Permissions** — some skills encode sensitive business logic and
  shouldn't be open to everyone.

## Fixing it takes a central platform plus named owners

Adoption runs in three stages. Individuals create, test, improve, and use
skills — in a structured way, on an agreed tool, not randomly. Teams then
share and co-evolve them, which happens fast since they build the same
products on the same stack. The critical stage is a centralized platform.

That platform needs: a catalog with metadata, searchable, with an MCP for
search and a CLI to pull skills into an IDE or sandbox; dependency
tracking between skills; versioning and lifecycle, so an agent can detect
and pull the latest version mid-build; access control; and evaluation plus
observability.

Then technology stops solving the problem. Someone must govern it, and the
shape depends on how your organization is already structured — architects,
engineering leads, infrastructure leads, and cyber leads owning their
domains and keeping skills aligned to policy. Done right, every team pulls
high-quality skills from one place and pushes improvements back.

## A 15-team simulation shows what governance actually changes

To make it concrete, the speaker simulated an organization: 15 teams of
5–12 people, tracking skills contributed per engineer, average skills
utilization per day, cross-team duplication ratio, and skills quality and
security ratios. Run over six months, teams create and use skills — which
is already happening in your organization — but with no visibility into
any of it.

The costs are coupled. Without a regulation skill, someone vibe-codes back
and forth trying to steer the agent into compliance: more tokens burned
*and* more time spent than a single correct shot. Quality and security
follow the same pattern — undefined, unmaintained skills leave the
judgment call to the human, so implementation quality drops, and maturity
varies visibly team to team. In the simulation one team lands at medium
productivity, another at low-medium quality and security with high cost.

Under governance the picture isn't perfect — some teams still diverge —
but common ground appears across teams. Once one skill is published, the
next engineer's harness finds it and pulls it instead of rebuilding, which
resolves most of the failures above.

## Skills are the first unit, not the last — workflows come next

Getting skills right isn't the finish line. The same approach applies to
whole workflows: a central platform holding workflows (which contain the
skills), so an engineer who needs to provision infrastructure can pull a
workflow, build with the required skills, run and test it, and push
improvements back.

## Auto-evolving skills make governance more urgent, not less

Three things to explore next. **Skills registries** — you should have one;
the internal developer portal vendors are already centralizing this
capability, and other tools address it directly. **Skills evaluation** —
the right approach is still being debated; the most valuable thing found
so far is statically testing skills against Anthropic's best practices,
since a skill that isn't invoked or structured properly is unlikely to be
high quality. **Auto-evolving skills** — the current hype, a closed loop
that improves skills automatically.

The closing warning is about that last one. Auto-evolution amplifies
whatever is already happening, so the impact will be far larger than
today's. Without governance providing the guardrails, you're automating
the maintenance of an ungoverned catalog.
