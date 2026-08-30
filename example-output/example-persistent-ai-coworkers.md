# AI's Third Era: Persistent AI Coworkers

**Source:** https://www.youtube.com/watch?v=zMvBMfj4cSQ
**Guest:** Tara Seshan, product lead for Codex and ChatGPT work at OpenAI
**Host:** Lenny Rachitsky, Lenny's Podcast
**Length:** ~80 minutes

---

## Three eras: chat, agents, and a coworker that persists

Seshan frames AI products in three eras. The first was chat. The second,
the current one, is working with agents — mostly coding agents so far.
The third, which she thinks may arrive soon, is working with a persistent
coworker that gets things done alongside you, possibly alongside other
people too.

Her job sits on the seam between the second era and the third: bringing
agent capability to the roughly billion people who use ChatGPT but have
never used an agent.

## OpenAI is founders-led, and there is no secret strategy

Seshan joined about a year ago from Stripe, where she was one of the first
five PMs, and expected the familiar things — strong colleagues, high
urgency — which she found. Two things surprised her.

The first is that OpenAI is *founders*-led rather than founder-led. Almost
everyone acts as a founder within their area, top-down direction is
unusually limited, and the distance between an individual and the market
is thin in a way large companies rarely manage.

The second is that she expected to find a treasure trove of internal
strategy — the equivalent of a payments bible — and there wasn't one.
Thinking about how the world should look, how product should be built, or
how the model should behave turns into public product or public messaging
very quickly. The cycle from internal idea to something users can touch is
faster than anywhere she has worked.

## A fast market punishes grand strategy

In a slower or more established market, rigorous first-principles strategy
pays. Payments is dynamic but predictable enough that you can reason
through what a competitor will do, and the market rewards whoever thinks
most rigorously — failing to do so reads as carelessness, because those
decisions were predictable.

This market is not that. It is emergent, fast-changing, and tied closely
to research, so being prolific and empirical beats being academic and
theoretical. Seshan describes the switch as jarring: instead of writing a
long reasoning document resembling a PhD thesis, the question became how
fast she could get to something testable with users. Her early worry was
that she wasn't doing her due diligence. The resolution is that the
thinking still matters, but it concentrates in one place — defining the
core hypothesis as pointedly as possible. Any grand strategy built around
it is not relevant.

## What's left of the PM job is its core

Much of the PM role was trappings: running execution to schedule, writing
particular documents, making presentations. Those have largely fallen
away.

What remains is what the job always was. Identify the question that
determines whether the product works. Test it. Read the results. Feed
them back into a sharper hypothesis and run it again. Doing that well
means understanding users, the market, and the technology, then pulling
all three into the sharpest hypothesis you can and testing it fast.

Seshan's claim is not that this survived the shift but that it became the
most important skill at the company — engineering managers, engineers,
data scientists, and designers have all converged on the same loop.

## Work becomes steering, not rowing

She expects the future of work to look more like steering than rowing,
with agents doing the rowing. The level of steering keeps climbing: from
pressing tab on a line of code, to directing something more comprehensive,
to the goal level, and plausibly higher.

But steering stays with a person. Part of it follows from data, and part
of it is an opinionated call — Seshan thinks intuition is underrated here,
along with a kind of positive determinism: choosing a direction not
because the alternative is unviable but because you want the world to look
that way.

## When everyone has the same tools, taste is the differentiator

If everyone can ask an agent how to win, what separates people is the
person. Seshan reaches for two analogies.

The first is clothing: functional clothes do the job, but what you wear is
a statement about individuality, and it works precisely by contrast with
what everyone else is doing.

The second, which she credits to one of the Collison brothers, is that
software is unlike real estate — you do not put money in and get value
out. It is closer to filmmaking, where a large budget guarantees nothing.
The best films are not the biggest-budget ones. That leaves authorship,
opinionation, and artistry, which requires you or your team to have
something interesting to say.

## The next step is working with agents together

Most agent work today is one-on-one: you and your agent, perhaps with
sub-agents beneath it, disconnected from what colleagues are doing with
theirs. Internally, people were pasting screenshots of their Codex threads
into Slack to show how they reached a number — which works, but is not a
natural way to collaborate.

The open question she is working on is what interface makes agent work
multiplayer, so a team steers a shared set of agents together rather than
each person steering privately.

## Plumbing matters as much as intelligence

Asked about slow versus fast takeoff, Seshan makes a point that cuts
against the intelligence-centric framing. What has enabled agents to
handle longer, higher-abstraction work is partly intelligence and task
persistence, and partly a set of unglamorous things: cloud infrastructure,
reliability, and above all data access.

Her comparison is a new hire locked in a room with no access to documents,
chat, or the company database. That person would not be useful, and an
isolated cloud agent is no different. These prosaic pieces matter roughly
as much as the broader intelligence questions for whether agents actually
work.

## Ambition is the binding constraint now, not capability

The people who get the most from AI tools don't automate rote tasks — they
expand the set of things they are capable of doing at all.

The old unicorn was the person with product sense who could also engineer
and maybe design, valuable because they collapsed the translation layers
between functions. Seshan's argument is that these tools hand that
superpower to everyone: spin up designs, build a prototype, model the
pricing scenarios. Your ambitions stop being limited by what you can
personally execute or communicate.

Which relocates the bottleneck to your own thinking. She points to
Patrick Collison's list of projects executed unreasonably fast — noting
that every one of them predates these tools, so the number of such
projects should now increase sharply. Citing Tyler Cowen, she argues
people underrate simply asking someone what the more ambitious version of
their plan would be, and that elevating other people's ambitions is now a
large part of the PM job: when someone proposes a timeline or a first
version, ask whether the ceiling isn't meaningfully higher.

Three internal memes capture the operating style: *is this maximally
accelerated?*, *are you mainlining it yet?* — using the product all day,
every day, a sharper version of dogfooding — and *feel the AGI*.

## Build for where models will be in two to three months

Seshan keeps one question in the back of her mind: are we building for
where the models will be in two to three months? Both failure modes are
equally wrong. Build for where models are now and you're overfit to a past
model's limits. Build for where you think they'll be in a year and you're
too early.

Calibrating that horizon is not guesswork. Research effort is directed at
specific capabilities, so staying tightly coupled to the research agenda
tells you roughly where things are heading. The product implication is to
treat model capability as the center of the product and get product
constructs out of the model's way.

## The chat/work toggle is a concession, not the destination

ChatGPT currently presents a choice between ChatGPT and Codex, and within
ChatGPT a toggle between chat and work. The north star is that users
should not make these decisions at all — you describe the task, and the
system picks the right harness and model.

Until then: Codex users should stay in Codex and are missing nothing.
ChatGPT users who want agent capability should use work mode, which is
Codex underneath with the coding-specific UI removed. The difference is
genuinely only at the UI level — how much technical detail and chain of
thought you see — not in capability. Seshan cites OpenAI's corporate
finance team using work mode for things that were previously manual or
required one deep expert.

The framing she gives for the current split: meet people where they are,
because concepts like "harness" are not reasonable things to ask a billion
consumers to understand.

## Conviction beats polish

At previous companies polish was king — shipping late cost little, so an
unpolished corner meant you might as well not ship. In this era, getting
a transformative product into users' hands beats getting it perfect. She
is direct that the ChatGPT app launch drew confusion, and that the answer
was fast iteration on feedback rather than a longer pre-launch polish
cycle.

The Codex story makes the same point from the other side. Asked what
changed internally to produce the visible shift in sentiment toward Codex,
Seshan's answer is that nothing did. The team was always user-focused,
iterating tightly, mainlining the app. What changed was the market
noticing. She gives the team full credit and treats the constancy as the
lesson.

## Roles blur; accountability doesn't

Seshan has always preferred environments where everything and nothing is
your responsibility — Stripe worked this way, with no boundaries between
what an engineer, PM, or designer could do. What she cares about is that
someone holds accountability for whether the product is used, wanted, and
good. Who does the work can follow from affinity and capability.

She is candid that this leaves a real question unresolved. Craft is part
of why people love their disciplines, and some of it is being abstracted
away by models. Engineers mourn the flow state of writing code by hand.
Her honest position is that she does not have an answer, and that everyone
is working it out at once.

## Writing to think stays human; writing to report gets automated

The distinction she draws is between writing as thinking — a brief arguing
for a product, a strategy, a spicy take — and writing as reporting, such
as a weekly status summary or a launch plan.

Reporting she automates happily. Thinking she will not automate at all.
Outlining, turning it into prose, cutting, editing, iterating: that is how
her ideas get straight. She thinks the common broad-brush positions — never
use models for writing, always use them — are both wrong, and the line runs
between these two kinds of writing.

What has changed is the artifact. A long document is no longer evidence of
thought, since anyone can produce one that proves the opposite. So she has
moved from docs to mocks, prototypes, and results from an actual A/B test,
which communicate far better. She still writes hundreds of documents, but
now for herself rather than for other people — the single biggest change
in her day-to-day.

Two practices sit alongside this. A manager once told her to write a
document to 70% and take it to the people whose buy-in she needs for the
last 30%, because polished ideas repel new ones while rough edges invite
collaboration. And on the risk of atrophy from over-relying on AI, she
holds herself to investing at least the collective time she expects
readers to spend — the same rule she applies to calling a meeting.

## Knowledge work can't be verified the way code can

The most product-relevant insight in the conversation is that coding and
knowledge work differ in how you check the result. Coding is
output-oriented: run the tests, try it, see if it works. Knowledge work
isn't. You cannot read a number off a finished deck and believe it — you
need the process, the inputs, and the reasoning that produced it.

That drives concrete product work: making ChatGPT a collaborator you can
watch, exposing in-progress work, citations, and inputs, so you arrive at
the output having followed the journey. It also raises open questions,
such as whether the thread — a surface well suited to coding — is the right
container for knowledge work at all.

## What stays human: accountability, expression, and each other

Asked where human brains stay valuable, Seshan names three things.

**Accountability.** Someone owns the outcome. You can think of your agent
as a report; the question of who is answerable for whether the result was
good stays with a person, especially in regulated industries or where a
human interface is required.

**Expression.** The filmmaking analogy again — what you choose to build and
how it feels is a human question, and authorship is visible in software.

**Each other.** The part of her work that involves talking with her team,
building shared enthusiasm, learning together, and elevating each other's
ambitions has become more important, not less.

## Notes from the rest of the conversation

Two tools she recommends: **sites**, which she now builds constantly in
place of docs and decks — including one tracking elevation and food for a
backpacking trip — and which she describes as finally delivering Alan Kay's
idea of malleable personal software, since a site is just a prompt; and
**/visualize** in Codex, for turning data into a presentable visualization.

On **Sutter Hill**, where she spent time as an EIR: the lesson was that
product-market fit is not a dark art. There is a repeatable playbook. The
specific surprise was how badly she had underrated *product marketing*
fit — the narrative and positioning can and should be tested before the
product is built, by pitching it a hundred times and refining, and doing
that excellently can be what makes a company succeed.

Her recommendations, briefly: *Barbarian Days* by William Finnegan, for the
idea that you can be deeply devoted to something without being good at it;
*Anna Karenina*, as a book that reads differently at 13, 17, and 30, which
she connects to needing to transform yourself repeatedly in this era; and
Kurosawa's *Rashomon*, which prompts her ambition point — he made something
perfect with a fraction of the tools now in her phone.

Her motto, from Toni Morrison's essay on work, comes down to four ideas:
do the work well for yourself rather than the boss; you make the job, it
doesn't make you; your real life is with your family; and you are not the
work you do, you are the person you are.
