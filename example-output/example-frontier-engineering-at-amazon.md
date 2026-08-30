# Frontier Engineering at Amazon

**Source:** https://www.youtube.com/watch?v=pqlWNihgdjI
**Speaker:** Clare Liguori, Senior Principal Engineer, AWS (works on Kiro)
**Length:** ~20 minutes

---

## Every prior phase of AI coding delivered only 10–20%

The industry moved through inline completion, then chat about your code,
then vibe coding. Across all of it, the speaker reports feeling maybe
10–20% more productive — anecdotally, from her own experience over three
years working on agentic AI.

Amazon's internal pilots are showing something different: a median 4.5x
productivity improvement, sometimes more than 10x. That discontinuity is
what the talk sets out to explain.

## Frontier developers are defined by three behaviors, not by their tools

Amazon defines the pattern by what these developers do:

- **Hands-off coding** — they write maybe 1–2% of the code they produce;
  agents write the rest.
- **Infrequent interaction** — they aim to get an agent running up to
  hours at a time without intervention.
- **Minimized idle time** — they run multiple agents in parallel against a
  backlog of tasks.

## The Bedrock Mantle team collapsed 30-people-18-months into 6-people-76-days

Bedrock, Amazon's model hosting service, needed a new inference data
plane. The estimate was 30 people over 18 months — a large service, with
customers and models to migrate. The team stepped back, put six people on
it, and built it in 76 days with Kiro, measured on commits.

This was the Pathfinder result. It proved roughly 20x was possible, and
the story spread across Amazon.

## Both showcase results came with conditions ordinary teams don't have

The Mantle team was six people, but they were among the top engineers in
the company, including two distinguished engineers — experts in
distributed systems and in LLM architecture. Impressive, and unachievable
for most teams. The obvious question was whether it reproduced.

A Prime Video experiment tried. Six engineers, a 10-day sprint, told to go
wild with Kiro. They cut a delivery estimate from 90 weeks to 24, measured
by comparing commits during the sprint against their prior history.

But that sprint also had conditions: no on-call duties, limited meetings,
few distractions — none of which describe an engineer's normal week. And
the team's senior engineer had spent the previous three weeks writing
detailed, small, well-scoped tasks with full requirements for the other
six to churn through.

So neither result answers the question. Both are structured points in
time, not day-to-day work.

## Across 50 ordinary teams, the split came from how they worked, not what they used

Amazon Stores — retail websites and physical stores — ran a more
structured pilot: 50 normal teams, a normal distribution of early-career
through senior engineers, all working on existing brownfield codebases
rather than greenfield. They watched for the better part of a year, using
deployment velocity to production as the metric rather than commits — how
fast changes actually reach customers.

The teams split in half. One half saw less than 3x. The other saw a median
of 4.5x, and sometimes more than 10x.

90% of the teams used Kiro among other internal tools, so the tooling
didn't explain the difference. What did: the teams that saw step-function
gains **intentionally changed how they worked**. The others sprinkled the
tools on top of their existing way of working.

## Five habits separate the two halves

Interviews across the pilot teams, Mantle, and Prime Video surfaced five.
The word *habits* is deliberate — not what one sprint did, but what teams
built day to day, which is hard and takes time.

### Invest in agent context, then prune it

Engineers carry a great deal in their heads and transfer it through Slack,
onboarding, mentoring, code reviews, standups, and sprint planning. All of
it has to be written down. The habit: every time the agent makes a mistake
or does something you wouldn't have, ask what's missing from your skills or
steering files.

The pruning half matters just as much, because models keep improving.
Sonnet 3.7 had quirks that required many "do nots" in steering files;
those became unnecessary with later models. So the second question is
whether a given instruction is still needed, or is now just bloating
context.

### Slow down to speed up

Nearly every team interviewed reported productivity *going down* as they
adopted the new way of working. Counterintuitive, but the hockey stick
comes after real engineering investment — especially in brownfield
codebases.

That investment: building up agent context, improving existing tools' error
messages so the model can tell what failed, building new tools and MCP
servers, and restructuring codebases so agents can navigate them. Some
teams went as far as changing language — untyped languages like Python and
JavaScript leave the model guessing without compiler errors, so teams moved
to TypeScript, and Rust became popular internally for its error messages.
Not required, but a change teams made deliberately for the gains.

### Feed agents instead of babysitting them

If you're in a back-and-forth conversation with your agent all day, you
can't see 4–5x, because you're in the loop the whole time — sitting for 30
seconds to a minute waiting for code to review. Waiting means you can't run
agents in parallel; you can't clone yourself.

The alternative is giving the agent what it needs *and how to validate
itself*, so it self-corrects and only returns when it clears a quality bar:
it compiles, tests pass, coverage is real. The next level is putting all of
that in your steering file so it happens every time without prompting.

### Make intent explicit before code exists

Amazon practices spec-driven development, built into Kiro. The vibe-coding
failure mode is a high-level prompt, a pile of generated code, then a long
conversation about how that wasn't what you meant and the requirements
weren't right.

Iterating on code when the *intent* was wrong is the expensive path. For
ambiguous or complex features, write the specification first — the model
can generate it — because iterating with the model on a document is far
easier than on code changes spread across a codebase.

### Shift testing left so agents can self-correct

The fast feedback loop is what lets an agent run for hours. It will make
mistakes; that's fine, provided the signals let it recover. So teams added
linters, unit tests, integration tests, performance tests, security tests —
hygiene everyone always knew they should have, where the ROI is finally
high enough to justify.

One specific pattern: mocking out services. Instead of end-to-end tests
against live services, deterministic mocks that run entirely locally, so
the agent can do everything on a laptop without spinning up cloud
dependencies. Faster feedback means more loops, and more loops means a more
productive agent.

## The habits carry a real burnout cost

No nirvana is promised — this is still an early-adopter phase and teams are
figuring it out. Three costs observed:

- **FOMO-driven overwork** — engineers staying up late chasing the perfect
  prompt that will run their agent overnight so a change is waiting in the
  morning.
- **Cognitive load** — running parallel agents means constantly switching
  between terminal tabs.
- **Review difficulty** — reviewing AI output is harder than writing code
  for some, especially early-career engineers. Senior engineers have spent
  years reviewing others' code; that muscle isn't there yet for everyone.

## Organizations block frontier teams unless they change too

Changing how engineers work is hard enough; the organization around them has
to change as well.

**Accept slowing down to speed up.** The speaker names herself and fellow
leaders as guilty of asking why teams aren't faster now that the tools and
models are good. But teams need those two months to invest in the codebase,
find their practices, and change habits — which doesn't survive an
expectation of monthly feature shipping fueled by posts about 20 PRs a day.

**Don't go too broad too fast.** Had Amazon expected every team to be a
frontier team immediately, the learnings from Pathfinder, the sprint, and
the pilot wouldn't exist. Rolling out too quickly leaves many teams not
knowing what they're doing, before you've found the practices and context
your organization needs. Scaling from 50 teams to the next 2,000 is what
2026 is for.

## Removing the coding bottleneck exposes decision speed as the next one

Writing code manually used to be the constraint. Once it isn't, decision-
making becomes the bottleneck: if code takes one to two months, then two
months to decide to build the product and two more to approve the launch
become the long pole. At 9–12 months of build time, that overhead didn't
matter much in the wash. Now it dominates.

Frontier engineering teams often spend more time making decisions than
writing code. The response is to make decisions fast, especially the ones
that are easy to reverse.

## The change is behavioral, which is why it's slow

The single takeaway: frontier engineering is about intentionally changing
how you work. That's difficult, it takes time, and it means forming new
habits — for individuals, teams, and the organization. The question to ask
is how your interaction with AI tools could change to get you out of the
loop.
