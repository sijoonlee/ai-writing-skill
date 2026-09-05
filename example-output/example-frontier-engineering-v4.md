# Frontier engineering at Amazon

Clare Liguori, a senior principal engineer at AWS who works on its Kiro coding assistant, has spent three years on agentic AI, and by her count the first three waves — inline completion, chat, vibe coding — made her maybe 10 to 20 percent faster. Then Amazon teams began reporting a median 4.5x. The cause was not the tool: 90 percent of teams in the largest pilot used the same one, half still under 3x. The fast teams had rebuilt the working day around the agent — writing down what they knew, fixing error messages so a model could read them, handing over tasks complete enough to run for hours unattended — while the slow teams sprinkled it on top of the day they already had. So the gain is not something you install. It costs a couple of slow months up front, moves the bottleneck off code and onto decisions, and burns people out if pushed at everyone at once — her case in a recorded conference talk.

## What a "frontier developer" actually does

**They hand-write almost nothing.** One to two percent of what they ship; agents write the rest.

**They leave the agent alone.** Runs of hours, no intervention.

**They keep several agents busy at once.** Idle time is what they minimize.

## Three results, each with an asterisk

**Bedrock Mantle: 76 days instead of 18 months.** Amazon estimated a new inference data plane for its model-hosting service at 30 people over 18 months; six built it with Kiro in 76 days, up to 20x on commits. The asterisk: two of them were distinguished engineers.

**Prime Video: a 90-week estimate cut to 24.** Six engineers, one 10-day sprint. The asterisk: no on-call, few meetings, and three prior weeks of a senior engineer writing small, tightly scoped tasks for them.

**Amazon Stores: 50 ordinary teams, split down the middle.** Mixed seniority, existing codebases, watched most of a year against deployment velocity — how fast changes reached customers, not how many commits. Half landed under 3x; the rest, a median 4.5x, sometimes over 10x.

## The five habits the fast half built

Habits deliberately, she says: daily practice, not one sprint.

**Write down what's in your head — then prune it.** What normally moves through Slack, standups and code reviews has to land in a file; every agent mistake becomes a question about what the steering files missed. Pruning matters too: the "do nots" written for Sonnet 3.7's quirks were unnecessary by Opus 4.5, and a stale rule only bloats context.

**Slow down to speed up.** Almost every team interviewed saw productivity fall as they adopted the new way, because the engineering comes first: error messages a model can act on, new MCP servers, codebases restructured so an agent can navigate them. Some left untyped Python and JavaScript, where the model guesses, for TypeScript and Rust.

**Feed the agent instead of babysitting it.** Wait 30 seconds to a minute for each batch of code, then review it, and you cannot go do anything else — running agents in parallel gets very hard. Hand over the task and the means to self-validate, so it returns only once the work compiles and passes tests — then put that bar in the steering file.

**Make intent explicit before code exists.** Amazon practices spec-driven development, built into Kiro. Iterating on code is a poor way to fix a misunderstanding: arguing with the model over a document beats arguing over changes strewn across a codebase.

**Shift testing left.** Linters, unit, integration, performance and security tests — hygiene everyone always owed their code, where she thinks the return finally justifies it. Her example: mock services with deterministic responses, so the loop runs on a laptop.

## What it costs, and what leaders have to give up

**Burnout and FOMO.** Engineers staying up late chasing the prompt that runs overnight and leaves a change waiting by morning.

**Cognitive load moves, it doesn't vanish.** Parallel agents mean constant switching between terminal tabs, and reviewing generated code is harder than writing it — especially early in a career, before years of reviewing others' code.

**Leaders who won't fund the slow part.** She includes herself among leaders who ask why a team isn't faster now that models are good.

**Rolling out too broadly, too fast.** Amazon learned what it knows from three experiments run in sequence, and demanding the change from every team at once would have lost the lessons all three produced. She says 2026 is the year Amazon takes frontier engineering from 50 teams to 2,000.

**A new long pole: deciding.** When code took 9 to 12 months, two months to approve a product and two more to approve its launch vanished in the wash. Code now takes one to two months, so approvals are the long pole and cheap reversible decisions are worth more.

Her one takeaway: frontier engineering means intentionally changing how you work, team and organization both, until you are no longer in the loop.
