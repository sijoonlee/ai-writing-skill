# Frontier Engineering: How Amazon Teams Got Step-Function Gains

**Source:** https://www.youtube.com/watch?v=pqlWNihgdjI
**Speaker:** Clare Liguori, Senior Principal Engineer, AWS (works on Kiro)
**Length:** ~20 minutes

---

## Three earlier phases of AI coding added maybe 20%; the pilots show 4.5x

Clare Liguori has worked on agentic AI for over three years and has
watched coding assistance move through three phases: inline completion
that wrote the next line or function, chat that answered questions about
code, and vibe coding, which everyone took up last year. By her own
account, entirely anecdotal, all three together made her feel only 10 to
20 percent more productive.

The fourth phase, which Amazon calls frontier development, is different
in kind. Across pilots run with teams throughout the company, Amazon has
measured a median productivity improvement of 4.5x, and sometimes more
than 10x.

## Frontier developers are defined by three behaviors

Liguori defines a frontier developer by how the person works, not by
what tools the person holds.

The first behavior is hands-off coding: a frontier developer writes
perhaps one to two percent of the code that ships under their name, and
agents write the rest. The second is infrequent interaction, aiming to
let an agent run for hours at a stretch without intervention. The third
is minimizing idle time by running several agents in parallel against a
backlog of tasks.

## Bedrock Mantle built in 76 days what was scoped at 30 people for 18 months

The first frontier team Liguori saw was Bedrock Mantle. Bedrock is
Amazon's model hosting service, serving LLMs such as Claude and GPT, and
the team knew it needed a new inference data plane.

The estimate for that new data plane was 30 people over 18 months,
covering the build plus migrating customers and models across. The team
stepped back and instead put six people on it with Kiro. They finished
in 76 days. Measured by commits, the result was up to a 20x improvement,
and it made Bedrock Mantle the Pathfinder team that proved the ceiling
was much higher than anyone assumed.

## Talent explains too much of the Mantle result to reproduce it

The Mantle story spread across Amazon like wildfire, and it also left
most teams with no way to act on it.

Those six people were some of the top engineers in the company,
including two distinguished engineers, expert in distributed systems and
in LLM architecture. Six people did the work, but not six ordinary
people. The obvious question followed: can any other team reproduce
this?

## A Prime Video sprint got close with a different set of engineers

The Prime Video organization ran a 10-day experimental sprint that put
six engineers in a room and let them go wild with Kiro.

The 10 days moved the project's delivery estimate from 90 weeks down to
24. The team compared its commit history before the sprint against the
commits produced during it. So a different set of engineers reached
something close to what Bedrock Mantle had reached.

## The sprint removed everything that makes an ordinary week ordinary

Liguori is candid that the sprint was not real life either.

The six engineers had no on-call duties, few meetings, and very few
distractions, none of which describes a normal engineering week. On top
of that, the team's senior engineer had spent the previous three weeks
writing detailed, small, well-scoped tasks with full requirements for
the six to churn through. The sprint was a structured point in time, so
the question about day-to-day work on real teams stayed open.

## Fifty ordinary Stores teams split in half on deployment velocity

Amazon Stores, which covers Amazon.com, the retail sites, and the
physical stores, ran the more structured pilot that answers that
question. It followed 50 entirely normal teams for the better part of
last year: a normal mix of early-career, mid-career, and senior
engineers, all working on existing systems and existing codebases rather
than anything greenfield.

The metric was deployment velocity to production, not commit count, so
it measured how fast changes reached customers. The 50 teams split
almost evenly. Half saw less than a 3x increase. The other half saw a
median of 4.5x, and in some cases more than 10x.

## The split tracked how teams worked, not which tools they used

Tooling cannot explain the gap, because 90 percent of the 50 teams used
Kiro alongside Amazon's other internal tools.

What separated the two halves was working style. The teams with
step-function gains deliberately changed how they worked. The rest
sprinkled Kiro and the other tools on top of the way they already
worked. For Liguori this was the aha moment that explained why she had
never felt the gains AI kept promising.

Interviews with the pilot teams, plus Bedrock Mantle and Prime Video,
surfaced five habits. She uses the word habits deliberately: the point
is not one sprint but daily practice, and these habits are slow and hard
to build.

## Habit one: write down the context that lives in your head

Engineers carry a great deal of knowledge in their heads and pass it on
through Slack threads, onboarding mentors, code reviews, standups, and
sprint planning. Frontier teams had to write all of that knowledge down
instead.

The habit is triggered by failure. Every time the agent makes a mistake
or does something differently than you would have, ask what is missing
from your skills files and steering files.

Pruning matters as much as adding, because the models keep improving.
Sonnet 3.7 had quirks that demanded a long list of "do nots" in steering
files; Opus 4.5 needed far fewer, and more than six months of model
releases have followed. So the newer habit is to ask whether a given
instruction is still needed, or is just bloating context.

## Habit two: slow down to speed up

Nearly every team interviewed reported that productivity dropped at
first, while they were deliberately changing how they worked.

The drop is the point. Real engineering work has to happen in the
codebase before agents succeed there, especially in brownfield code.
Teams built up agent context, improved error messages in existing tools
so the model could tell what had failed, and wrote new tools and MCP
servers. Many restructured their codebases so agents could navigate
them.

Some went further and changed languages. Liguori has watched teams
struggle with Python and JavaScript, where the lack of types and
compiler errors leaves the model guessing, and move to TypeScript. Rust
has become popular inside Amazon for its error messages. No team has to
go that far, but some chose to for the gains.

## Habit three: feed agents instead of babysitting them

Conversational back-and-forth with an agent all day caps your gains,
because you stay in the loop the whole time.

Waiting 30 seconds to a minute for generated code, then reviewing it,
keeps you at the keyboard. You cannot go do something else, which makes
running agents in parallel very difficult; you cannot clone yourself
into several agents.

Feeding an agent means telling it what to do and how to check its own
work, so it self-corrects and returns only when it clears a quality bar:
it compiles, it runs, the tests pass, coverage is high. The next level
is moving all of that into the steering file so the agent does it every
time without being asked.

## Habit four: fix the intent before the agent writes code

Amazon practices spec-driven development, which is built into Kiro, so
Amazon engineers adopt it naturally.

Vibe coding starts from a high-level prompt, lets the agent generate a
great deal of code, and then argues with the result: that isn't what I
meant, the requirements are wrong, I wanted a different design.
Iterating on code is the wrong place to fix a wrong intent.

For ambiguous, complex features, Amazon engineers write the
specification first. Kiro can generate the specification rather than
making you write it, and either way it is much easier to argue with a
model about a document than about changes scattered across a codebase.

## Habit five: shift testing left so agents can self-correct

A fast feedback loop is what lets an agent run for hours on its own. The
agent will make mistakes, which is fine, as long as the signals it gets
let it correct them.

Teams have added linters and unit, integration, performance, and
security tests. Liguori notes these are practices everyone already knew
they should have, and argues the return on investment is finally high
enough to actually pay for them.

Mocking services is the change she sees most. Integration tests used to
run end to end against live services; teams now invest in mock services
that run locally with deterministic responses. Keeping everything on the
laptop, with no cloud services to spin up and connect to, is faster, and
faster feedback means more loops for the agent.

## The habits do not make the burnout risk go away

Liguori declines to promise that adopting all five habits delivers
nirvana. Frontier engineering is still an early-adopter phase, and teams
are still working it out.

Burnout is the organizational risk she sees. FOMO, a term she credits to
someone else, is real: engineers stay up late chasing the perfect prompt
that will keep an agent running overnight so a change is waiting in the
morning. Cognitive load rises with parallel agents and constant shifting
between terminal tabs.

Reviewing AI output is also harder than writing code for some people,
particularly early in a career. Senior engineers have spent years
reviewing other people's code; early-career engineers have not built
that muscle, so review costs them more than writing would.

## Organizations have to change as much as engineers do

Changing how engineers work is hard on its own, and frontier engineering
changes the entire shape of the day. Organizations have to change too.

Accepting the slowdown is the first change, and Liguori includes herself
and her fellow leaders among those who have failed at it, asking why
teams are not faster now that the models are so good. Teams need those
two months to invest in the codebase, find practices that suit them, and
build hard new habits, which is impossible under a monthly shipping
expectation set by posts on X about 20 PRs a day.

Rolling out too broadly too fast is the second. Had Amazon expected
every team to be a frontier team immediately, it would have lost the
learnings from the Pathfinder team, the sprint experiment, and the
pilot. Too fast a rollout leaves many teams with no idea what they are
doing and no time to find the practices and context their own
organization needs. Scaling is the 2026 problem for Amazon: how to reach
the next 2,000 teams rather than 50.

## Decision speed becomes the new bottleneck

Writing code by hand used to be the constraint. Inside Amazon, the speed
of deciding has replaced it.

When a product took 9 to 12 months to build, two months to decide to
build it and two more to approve the launch barely registered. Now the
code takes one to two months, so the decision and the launch review
processes are the long pole. Liguori finds that frontier teams spend
more time making decisions than writing code, which is an argument for
deciding fast, especially where the decision is easy to reverse.

## The one takeaway: changing how you work is the hard part

Frontier engineering is about intentionally changing how you work, and
that change is difficult and slow. It means forming new habits and a new
way of working, across the engineering team and across the organization
around it.

Liguori's closing ask is to look at how you interact with AI tools today
and ask how that interaction could change to get you out of the loop.
