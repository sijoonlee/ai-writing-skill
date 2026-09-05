Source: "Building an Autonomous Engineering Org," a recorded conference talk
https://www.youtube.com/watch?v=whue9_YquGA
Speaker: unnamed in the transcript; led AI enablement for Block's 12,000
employees before the CTO tasked them with making engineering agentic.

Nine out of ten Block engineers were using coding agents, and the CEO still
insisted engineering wasn't using AI, because nothing shipped faster. The speaker
had the token bills to prove adoption and decided the CEO was right anyway:
engineers were asking questions inside the IDE, not handing work off. Changing
that for 3,500 people one at a time was hopeless, so they picked about 50 and
pointed them at the repositories, because a repo lifts everyone who commits to
it. Once agents behaved inside a repo, delegation could move to where work
already arrives — a Jira ticket, a Slack thread. Agents then multiplied until the
bottleneck became code review and engineers' laptops; clearing those got Block to
what they call an autonomous engineering org. The layoffs came the day after they
felt proudest of it.

## Adoption that didn't move anything

Block started early, they say: it built goose, its internal coding agent, before
LLMs supported tool calling, and goose became the reference MCP client
implementation while Block was an Anthropic design partner. Within months roughly
90% of engineers used goose or Claude Code regularly — the end of what they call
the experimentation phase: boilerplate and questions, not shipped features.

So they defined the target — engineers whose default is decomposing a problem,
delegating it, and verifying what comes back — and scored it on a six-stage
ladder, crediting Steve Yegge's Gastown article. Stage five is an agent producing
shippable results without handholding; most Block engineers sat at one or two.

## Betting on the 1%

In online communities 1% create, 9% interact and 90% consume, and engineers
adopt AI the same way, they say, so a plan needing everyone to level themselves up
reaches nobody. They handpicked about 50 AI champions, not volunteers, from Square,
Cash App, Afterpay and TIDAL, each willing to give 30% of their time and
unbothered when agents failed out of the box.

Their job was making repos AI-ready, not making themselves faster. Monorepos got
shared context and rules at the root with service-level layers beneath, and
Android sometimes needed a different approach from web. Teams converged on a
common kit, chosen per team rather than mandated: an AGENTS.md or CLAUDE.md,
rules files, slash commands and later agent skills, an AI co-reviewer told what
matters, AI attribution on PRs.

## Delegation where the work already lands

**Slack.** An engineer reported a bug in a channel; a third asked goose to check.
It pulled the repo, confirmed the bug, offered three fixes in snippets, the group
picked one, and goose returned a PR link — about five minutes, start to finish.

**Tickets and issues.** Engineers assigned Linear and Jira tickets and GitHub
issues to an agent to run end to end. The first team to try it ran out of work
and pulled in more tickets twice.

Three months in, they report AI-authored code up 69%, reported time savings up
37%, and automated PRs up 21 times.

## What parallelism broke

**Review.** PRs tripled and quadrupled, then sat. Earlier AI co-reviewers were
bad enough to annoy people; with AI-ready repos and better tools — they credit
Codex — Block turned it on everywhere, plus an autofix loop where a second agent
commits fixes for flagged issues.

**Hardware.** Agents collided and laptops ran out of memory, so each agent got an
isolated cloud workspace, which also let them run from anywhere.

## Builderbot, then the layoffs

With engineers running four or five agents at once, champions built an
orchestrator, Builderbot, and a machine-readable model of all 25,000 repos and
their dependencies, so agents could explore in parallel while Builderbot planned
across codebases. That was stage five — anyone at the company could @Builderbot
in Slack to fix a bug without touching GitHub.

Then came layoffs. They ask whether it was their fault — whether helping people do
the best work of their careers got them dismissed — and close with three
questions: what are we doing, where are we heading, and are we sure it's where we
want to end up.
