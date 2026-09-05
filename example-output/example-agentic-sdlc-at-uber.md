# Agentic SDLC at Uber

**Source:** https://www.youtube.com/watch?v=17-YSUHo6Lk
**Speakers:** Udai and Adam, engineers at Uber
**Length:** ~18 minutes, recorded conference talk

Uber did not get agents writing most of its pull requests by handing engineers
a better model. In Udai and Adam's telling, everything an agent needs — a
model, tools, a machine to run on, reusable skills, knowledge of how Uber works
— sat scattered across dozens of systems, and each had to be pulled behind one
door. Once those six pieces exist, a feature can ride through all of them: a
Slack thread becomes market research, then mockups, then a draft pull request
that stops short of CI on purpose, then a cleanup job scheduled for Sunday
night. And when shipping gets that cheap, the speakers say, what slows you down
is no longer whether you can build the thing.

Udai covers the building blocks, Adam the feature. Their numbers: a few
thousand engineers across 12 global tech sites, more than 70% of pull requests
now opened by local or cloud agents, twice the lines of code per engineer year
over year, 250-plus automated migrations across 9 million lines. The six blocks
are at various stages of rollout.

## The six things an agent needs

**A model gateway.** Every use case, coding harnesses included, goes through
one OpenAI- and Anthropic-compatible endpoint, behind SPIRE identity checks, an
anonymizer redacting 20-plus types of personal data, and an AI guard of five
specialized safety models — all inside a strict 100-millisecond budget. It
carries 800-plus projects and over 100 million requests a day.

**An MCP gateway.** Uber's thousands of internal APIs were unreachable by
agents. A crawler now projects internal APIs into MCP servers with one config
change, and Google, Slack, and Jira come through the same door. Then there's
the token tax: direct MCP servers gave way to one "Omni MCP" that discovers and
invokes the rest, then a CLI pattern keeping responses out of context, then a
code-mode skill writing Python for the heaviest consumers — over 40% fleetwide
savings across 1,000-plus tools.

**Somewhere to run.** Uber's cloud dev pods, built for engineers in huge
monorepos, got what Udai calls agentified: pre-provisioned Kubernetes balloon
pods with repositories snapshotted and the search index built, so an agent
starts in seconds, and one mega dev pod holds every repository at once.

**A skills marketplace.** Engineers were rebuilding the same skills in
different repos, and quality was uneven. Skills now flow through a managed
marketplace — 2,500 of them, over 20,000 executions a day — gated by lint
checks and automated reviews, auto-installed by engineer persona.

**A context graph.** Traces showed agents burning tokens just working out where
a service lives, who owns it, what patterns to follow — facts scattered across
20 to 30 systems. One graph now holds them: 150 unique node and edge types, 40
million entries, from mobile builds to design docs to incident bugs. Udai's
example: how many mobility trips in India are paid in cash — a question the
graph answers in far fewer tokens, turns, and seconds.

**An assistant on top.** Cortana, the internal AI assistant, plugs skills, MCP
servers, and the graph into Slack, CLI, and web. Employees can park a
personalized copy in a team channel, where it works like a teammate: 300
personas last month, over 20,000 sessions a day.

## One feature, end to end

Adam's example is a World Cup feature: a better pickup spot for a rider leaving
a packed stadium. It starts in a Slack thread where someone tags Cortana, which
uses the graph to weigh the opportunity, then moves to the web for requirements
— North America only — and two Figma mockups with different button strings, for
an A/B test. That used to take weeks.

Building hands off to Minion, Uber's cloud coding agent, running interactively
or autonomously on a dev pod with every repo available. It deliberately stops
at a draft PR without pushing to CI: autonomous runs were fine for toil, Adam
says, but real features need validating first, and the team would rather not
pile that load onto CI.

So checks shift into the inner loop. Static analysis moves in, and so does
visual validation: a skill launches a simulator, screenshots it, compares that
against the Figma spec, then brings up staging to test front end against back
end. The outer loop keeps self-healing CI and a deep review by a powerful
reasoning model; a smaller, faster one reviews inside. For the human reviewer,
the PR arrives with a table of every check the diff survived, screenshots
included.

Maintenance gets the same treatment: a service is enrolled into maintenance
skills — cleaning up the losing feature flag variant, say — on what Adam
stresses is a *managed* loop, not thousands of unbounded ones. It runs Sunday
when CI is free, capped so engineers aren't buried Monday morning, and whether
those diffs land is labeled data for improving the skill.

What breaks next is capacity. More code strains CI, and only so many
experiments can run. But the bottleneck Adam ends on is the decision — no
longer really whether Uber can build something, but whether it should.
