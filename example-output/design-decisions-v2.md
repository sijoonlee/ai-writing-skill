# Design Decisions

*What to slow down on, and how to say why.*

Most of what engineers argue about can be undone in a week. The decisions that outlive everyone
— the data model, the tenancy shape, who owns which fact — get made silently, by whoever writes
the first migration. So this document starts by sorting decisions by what it costs to be wrong,
and says which ones have earned a real conversation. Then it throws out the arguments that were
never trades at all — the ones where a line had already been drawn, by somebody or by arithmetic,
and the only question was whether you cleared it. What survives needs a vocabulary, and today every developer has a
different word for *because* — cleaner, simpler, more maintainable, more pragmatic — so no two
people mean quite the same thing and nobody can read back what anyone actually valued. Fourteen
named values fix that, used in a sentence with two halves: what you bought, and what you paid
with. Naming a trade does not make it a good one, so four tests follow. The rest is reference —
what each kind of decision usually turns on, and what you learn from reading a year of these
sentences back.

---

# 1. Does this decision deserve the effort?

## Cost of being wrong — the deletion test

Sort decisions by what it costs to change your mind later. That reorders the list in a way most
architecture discussion gets backwards.

Asking "is this reversible?" gets you a yes from everyone in the room. Ask the **deletion
test** instead — referred to by that name for the rest of this document:

> **What's the smallest unit here we could delete outright, and what would we have to touch?**

"A module" means you're fine. "A table with two years of rows in it, and the three services that
read from it" means you have work ahead — though notice that phrase bundles two different
difficulties, and you should say which one you have. Rows you can migrate. Readers you own are a
coordinated deploy. Readers you cannot reach are the wall.

The test is an investigation, not a verdict. What it is investigating is **reach**.

## The recovery tiers — how long the old version survives

Every decision here is a version of something, and its **recovery tier** is how long the version
you are replacing sticks around after you ship the replacement.

**The numbers go up as recovery gets harder** — tier 1 is easiest, tier 3 is hardest. That is the
opposite of the Sev-1 and P1 conventions most incident tooling uses, so it is worth saying out
loud once: this is a difficulty scale, not a severity scale. **Tier X is not a number**, and the
letter is doing deliberate work — see below.

| | The old version survives… | Which means |
|---|---|---|
| **Tier 1** | not at all | one unit, one version at a time |
| **Tier 2** | the migration window, and you decide when it closes | two versions coexist, so the change must be sequenced |
| **Tier 3** | past your deploy, and past your reach | the window is not yours to close |
| **Tier X** | irrelevant — there is nothing to recover *from* | never written, or written and no longer usable |

Tiers 1 to 3 sit on one axis: how far you must reach to undo the decision. **Tier X is not
further along that axis — it is off it.** Reach asks how hard it is to get back. Tier X asks
whether there is anything left to get back *from*, which is a different question and is why it
gets its own section below. Note that it does not require the information to have been
destroyed by this decision, or at all: written and no longer usable counts the same as never
written.

Hence the letter. A "tier 4" would claim a position on the scale and invite people to read it as
one-worse-than-3, when tier X does not sit anywhere on that scale at all. Two things make that
concrete:

- **It stacks rather than ranks.** Most tier X decisions are also tier 3, and you need both
  labels: *tier 3* says the rows persist, *tier X* says nothing can rebuild them. A single number
  can only tell you one of those, and a reviewer who hears "tier 3" will reasonably assume a
  migration exists.
- **It attaches to decisions with no reach at all.** "We won't collect visits until next quarter"
  writes no code and touches no storage, so the ladder has no position to give it — and it
  permanently decides that no report reaching back before then can be correct. Not off the top of
  the scale; off the scale.

X keeps the convenience of one label you can say in review while refusing the ordering that would
make it wrong.

Tier 2 includes data migrations. If you can backfill the rows and phase the old shape out on
your own schedule, the window closes when you say so, and that is Tier 2 however many weeks it
takes. Tier 3 is not "data is involved" — it is "the window is not mine."

That is one axis, end to end, and it replaces the question people usually ask. "Is it
reversible?" invites a yes. "How long does the thing I am replacing outlive my deploy?" has an
answer you can check.

It also explains why Tier 2 costs what it does. Not the hours — the fact that you have to keep
both versions working at once and sequence your way out. Expand, migrate, contract. Tier 1 has
no such window; Tier 3's window is not yours to close.

## What counts as a contract

Tier 2 begins at a contract, so the word needs a definition that settles cases rather than
starting arguments:

> **A contract is any interface whose two sides can be at different versions at the same time.**

A function signature inside one module is not one — both sides ship together and the compiler
catches the mismatch. An HTTP API, an event schema, a table two services read, a published
library's public surface, a file format: all contracts, because during any change both versions
exist at once.

**If a change touches a contract, it is at least Tier 2 — whoever owns the other side.** Making
ownership part of the classification means researching the caller before you can place the
decision. Make it a floor instead, then ask the escalating question separately.

And contracts escalate often, by two routes worth naming:

- **Consumers you cannot reach.** Every client version you ever shipped is still out there, and
  your code change cannot reach their code.
- **Data written under the old contract.** The one people miss. Change `amount` from cents to
  dollars and every row stored under the old meaning is now ambiguous. The contract change
  propagated into persisted state without anyone touching the schema.

## Recorded, or computed?

This is what puts a decision above Tier 1, not a separate question. **Data is the version that
does not go away when you finish releasing** — the only question left is whether you can reach
it.

What you **record** accumulates and cannot be un-collected: a field you didn't write in March is
not available in June. What you **compute** is expendable, because it re-runs — change the rule,
recompute, and history moves with it. So computed things sit at Tier 1 or Tier 2 by definition,
since code is enough to regenerate them. Recorded things are the *candidates* for Tier 3 — they
arrive there when the store is one you cannot rewrite, or when the information was never written
at all. A record you can migrate on your own schedule is still Tier 2.

- **For something you record** — if we deleted this, what else would we have to touch?
- **For something you compute** — if we deleted this, could we rebuild it, and from what?

A computed result whose input you kept is cheap to lose. **A computed result whose input you
discarded is now a permanent record, and nobody decided to make one** — which is what happened
the day someone stored a date instead of a timestamp. The diff that does it looks tier 1; the
rows it writes make it tier 3, and the input it dropped makes it tier X. That mismatch between
how small the change looks and where it lands is the single most common way this happens.

## What lands where

**Tier 1 — inside one unit.** One version at a time; a revert puts it back.

There is a condition on that, and it is the one most often missed:

> **A code decision is tier 1 only if reverting it leaves no residue in storage.**

Once code decides *what gets written down*, its tier is set by the persistence, not by the size
of the code. Reverting the commit does not un-write the rows the commit produced, so a small
change that alters the shape of what you save is not tier 1 — it is tier 3 wearing a tier 1
diff. Everything below passes that condition: none of it changes what ends up in storage.

- **Layering and internal structure** — where business logic lives
- **Error handling and retry patterns**
- **Testing strategy**
- **Module organization, naming conventions**
- **Language and framework**, when the schema is yours and untouched — a port is enormous and
  still tier 1, which is the point

That last entry is what shows the tiers are not sorted by effort. A Python-to-Go port is a
quarter of work and fully recoverable: no data moved, no consumer affected, nothing lost. A day
key that discarded time-of-day took twenty minutes and cannot be recovered at all. Sorting by
labor puts the port first. Sorting by what it costs to be *wrong* — which is what this document
claims to do — puts the day key first, and that inversion is the whole point of §1.

**Tier 2 — crosses a contract.** Still only code, but two versions coexist until you close the
window.

- **Integration style** — sync RPC or async events, orchestration or choreography
- **Deployment topology** — monolith, modular monolith, services. Multi-*region* is tier 3, since
  it moves data
- **Splitting or merging services**, which creates the contracts that then have to be versioned
- **Build vs. buy vs. borrow**, when what you buy is a capability rather than a home for your data
- **Schema changes you can backfill** — a new table, a dual write, a cut-over, dropping the old
  column. Weeks of work and a real plan, but the window is yours

Data migrations are tier 2 **with a tail into tier X**, and that is why they feel heavier than the
tier suggests. A code revert cannot lose anything. A botched backfill can destroy the column you
were reading from, which turns a migration you could run into information nobody can recover. The
tier describes the intended path; the tail is why you soak it.

**Tier 3 — outlives your deploy.** Rows exist, clients are shipped, or somebody else holds the
records. This is where design effort pays.

- **Data model**, in the specific decisions that drop information or denormalize a shape into
  stores you cannot rewrite — identity, history, what granularity gets kept. Ordinary schema
  evolution you can backfill is Tier 2; it is worth knowing which of the two you are doing
- **Records in stores you don't own** — browser cookies and `localStorage`, mobile app local
  state, edge caches, third-party systems, devices in the field
- **Service and domain boundaries** — they become team boundaries, and a team's habits outlive
  any deploy
- **Published contracts** — external consumers you cannot coordinate
- **Multi-tenancy model** — single to multi-tenant is one of the worst migrations there is, and
  it is usually decided by accident on day one
- **Identity and authorization** — the permission *shape* gets recorded, denormalized into
  every query, and is brutal to change
- **Consistency and source-of-truth ownership** — who owns which fact, what is transactional and
  what is eventual. Not "do we use Kafka" — *where does truth live*
- **Datastore choice**, where it decides what gets persisted and in what shape
- **Buying something that holds your records**, because leaving means extracting data in a shape
  the vendor chose

**Tier X — nothing to recover from.** The information was never written, or was written and
discarded. Not a harder tier 3: a different question, which the next-but-one section covers.

- **What granularity gets kept** — a date instead of a timestamp, a count instead of the rows
- **When collection starts** — every day before it is permanently blank
- **What you decline to record at all** — the field nobody asked for until the year it mattered

## Inside tier 3 — why the window isn't yours

Two mechanisms, and they are not the same conversation. Both have escape hatches, which is what
separates them from tier X.

**1. Records in a store you cannot rewrite.** Browser cookies, `localStorage`, app state on
phones that don't update, edge caches, a third-party system you can append to but not correct.
Nobody is withholding cooperation — there is simply no write path. You cannot run a migration,
so you version the record, read both shapes, write the new one, and carry that reader **until
you decide to stop**. You never finish; you choose an expiry.

That makes the compatibility window a decision rather than a consequence, and it deserves a
written trade like anything else:

> **Chose** read-both, write-new, old reader dropped after 90 days
> **For** Economy — the dual-read path stops being maintained and tested forever
> **Accepting** Fidelity — anyone returning after 90 days is treated as new, and whatever the
> record held is gone for them

**The escape hatch is disposability.** If the cookie holds a UI preference, reset everyone and
you have closed the window yourself — that is a demotion to Tier 2, bought with one bad day. If
it holds a consent record, an experiment assignment you need for analytic continuity, or
anything you cannot recreate, there is no reset. Same store, different tier, decided entirely by
whether losing the contents costs you something.

**2. Code you don't own must change.** Consumers you cannot reach, or teams whose tooling and
review habits already assume the current shape. Here somebody *does* have to act, so the escape
hatch is reaching them — versioning the contract, a deprecation window, an actual conversation.
Expensive and slow, but available.

## Tier X — the orthogonal one

The other three tiers measure how hard it is to **undo a decision**. Tier X asks whether there is
anything left to undo it **from**, and those come apart — which is why it needs asking separately
rather than read off the ladder:

> **If this turns out wrong, is there anything still in existence that could reconstruct the
> answer?**

Two examples show why no rung on the reach axis catches this.

**The day key.** A service counts events per day. Each event arrives stamped with the exact
moment it happened — `2026-09-02T23:40:00Z`. The service works out which Toronto calendar day
that moment falls in, adds one to that day's running total, and saves a row that says
`{ day: "2026-09-02", count: 47 }`. The original timestamp is not saved anywhere.

The code doing this lives in one service — one unit, one version — so reverting the commit takes
an afternoon, and from tomorrow every event keeps its full timestamp again. **The diff looks like
tier 1.** It isn't: this is precisely the residue case above, because the commit decided what
gets written down. Every row already saved still says only `day` and `count`, and those rows stay
in storage past the deploy. **Tier 3.**

Then the second, separate fact. When someone asks next year what time of day events arrive, or
whether the day should have been cut at UTC midnight instead, or whether the clock change in
March was handled correctly — every one of those answers was in the timestamp, and the timestamp
was never written down. So this is not a tier 3 you can eventually grind through with a
migration. **Tier 3, and tier X underneath it.**

Two facts, not one: the rows persist *and* nothing can rebuild them. The ladder gives you the
first. Only the destruction question gives you the second.

**Not collecting yet.** "We'll add the visits table when the source ships" is a decision to write
*no code at all*. Its reach is nil, so the ladder has nothing to say — and it permanently
determines that no range reaching back before collection start can ever be correct. Here there
is not even a commit to revert: starting collection tomorrow is free and changes nothing about
yesterday.

Both are tier X, and neither is legible as a reach. So ask the question of every decision
regardless of where it sits on the ladder, and write the answer next to the tier: *tier 3, and
tier X — no replay after 30 days* is a sentence that stops a review, where *tier 3* alone lets a
reviewer assume a migration exists.

The test is not whether a write happened. It is whether **anything that still exists can
reconstruct the answer**, and that fails in two families.

**Never written.**

- **No records** — nothing was collected. Usually a decision about *when to start*, which arrives
  disguised as a scheduling question.
- **No field** — the rows exist but the answer isn't in them. Usually a decision about
  *granularity*, which arrives disguised as a storage-efficiency question.

**Written, but no longer usable for recovery.**

- **Retention expired** — the raw events held the answer and the window closed. This is the
  common one, and the one people are most surprised by.
- **Overwritten** — a placeholder replaced the unknown. The row is still there and now says `0`,
  which is worse than empty, because nothing distinguishes the guess from a real measurement.
- **Irreversibly transformed** — hashed, truncated, redacted, or aggregated past the point of
  return. The bytes exist and the answer doesn't.

That second family has a consequence the ladder cannot express: **tier X is the only tier that
can arrive later, with nobody doing anything.** A decision sitting safely at tier 3 today —
*we can always replay the stream* — becomes tier X on the day retention expires. Which makes
"we can rebuild it from the events" a claim with an expiry date, and the useful follow-up is
always: *for how long?*

## Why tier X can't be weighed

Tiers 1 to 3 can be priced at the table. A week, a migration and a soak, a deprecation cycle
across three teams — you can put a number on recovery *today*, and argue about whether the number
is worth it.

**Tier X has no such number.** Its cost is whatever question someone asks in two years that you
cannot answer, and nobody in the room knows what that question is. So it does not get weighed
badly — it gets weighed at zero, because there is nothing to put on the scale. That is the whole
reason it needs a flag instead of a place on the ladder: a value you cannot price will lose every
trade it is entered into.

## Latent and active — when you pay

What *is* knowable at the table is whether anything currently consumes the thing you are not
recording. That does not change the permanence; it changes when the bill arrives.

**Latent — nothing reads it.** You stopped collecting, or never started, and no code depends on
it. Going back is free: start collecting tomorrow. The hole stays in history and you may
genuinely never pay. This is the case people point at to argue tier X is overblown, and they are
right about this case and wrong about the category — the cost is not zero, it is unpriced, and it
arrives the day someone asks about last year.

**Active — something expects it.** The record is missing and a consumer needs it now. You pay
immediately, and all three exits are bad:

- **Backfill a placeholder.** ⚠️ The trap, and the one that looks most like a fix. Writing `0`
  where you mean *unknown* destroys the difference between "we measured nothing" and "we didn't
  measure." That is a second tier X created by the repair for the first, and it is worse than the
  original, because the absence is now invisible — nobody can tell afterwards which rows were
  guesses.
- **Make the consumer tolerate absence.** Honest. You carry the null-handling forever, and every
  reader has to understand why it's there.
- **Narrow the product.** Refuse to answer for the affected range, and say so where people read
  the number.

The second and third are the real options. A dashboard that shows an explanatory note instead of
a fabricated `$0`, or that says "since collection began" rather than "since beginning," has taken
the honest exit and paid for it in Ergonomics and scope. That is a trade worth writing down —
where a silent placeholder is not a trade at all, because nobody will ever see what it cost.

There is no escape hatch, no migration and no budget that reaches tier X. The only move is before
the fact: record the input, keep the instant, write the field. Which is why it is the one tier
whose entire treatment is preventive — and why it belongs beside the ladder rather than on it.

## The inversion

**The decisions with the most discussion are mostly Tier 1, and the Tier 3 ones get made
silently.** Nobody holds a design review on the tenancy model or the permission shape. They get
decided by whoever wrote the first migration, on a Tuesday, in twenty minutes. That is where the
2am backfill comes from.

> **Does the stack leak into the schema?**

If your domain model *is* your ORM model, the framework choice has been welded to the schema.
The ORM's conventions — how it names tables, picks primary keys, maps inheritance, expresses
optional — are now shapes in your database with two years of rows in them. So swapping the ORM,
a tier 1 rewrite you should be able to afford, means reshaping tables under live data, which is
a tier 3 cost you can't.

This is the residue principle from §1, arriving as a design rule: the commit that lets the ORM
define your tables decided what gets written down, so its tier came from the rows, not from the
diff. Nobody chose to promote it.

This is the one thing genuinely at stake in the **layered-architecture debate** — clean
architecture, ports and adapters, whether the service layer is anemic, where the repository
interface belongs. Not the principle usually called *purity*, that a domain should never know
the database exists, but the practical business of keeping the framework replaceable. That is
what "hexagonal" and "ports and adapters" are for: the domain declares what it needs from
storage and the framework implements that, rather than the framework defining what the domain is.

So the useful question is narrow: *can the domain types be expressed without the framework?* Yes, pick
a convention and stop arguing. No, and you have promoted a tier 1 decision into tier 3.

Two moves buy most of the protection, and neither requires a full mapping layer. **Hand-write
the migrations**, so the schema is something you designed and the ORM maps to it rather than
defining it. And **keep framework types out of domain signatures** — no session, no query
builder, no lazy proxy in a function the domain owns. An architecture test that fails the build
when the domain package imports the persistence package turns that from a convention people
remember into Enforceability.

## The gate — when a decision deserves deliberation

Three conditions, all required. The rest of this document calls them **the gate**:

1. **Irreversible or expensive to undo** — Tier 3, sometimes Tier 2
2. **More than one genuinely plausible option** — not one option and two strawmen. Doing
   nothing is always one of them, and is usually the one nobody wrote down
3. **Actually contested**, or contested-by-default because nobody has examined it

Everything else gets a convention, a default, or whoever is writing it. Deliberating Tier 1
decisions is how design practice earns its reputation for ceremony.

**The gate governs deliberation, not vocabulary.** Say the sentence anywhere — a review
comment, a commit message, a corridor. *Chose the map here for Legibility, accepting Speed*
costs eight words on a decision nobody will revisit, and saying it on cheap decisions is the
only way the words are available on the Tuesday a Tier 3 decision arrives disguised as a small
one. What the gate withholds from Tier 1 is the meeting, the written trade and the four tests
— not the right to name what you traded.

## Reversibility does two jobs

Most of the time reversibility is **measured, not chosen**. It decides whether a decision earns
deliberation at all — the gate above — and it settles which way a trade should go once you are
inside one (§4: spend the low-tier value to buy the high-tier one). Used that way it is a
property of an option, not something anybody bought, which is why "we can always change it
later" sends a reviewer back here rather than into the vocabulary.

But it can also be **bought**, and then it belongs in the *for* half like any other value. A
shadow table, a dual write, an expand-and-contract migration, a feature flag, a fortnight of
soak before cutover — none of these make the product better. They are built so that being
wrong is cheap, they cost real Economy and Speed, and they are thrown away afterwards. When
that is what you are paying for, say so.

The test is which direction the sentence runs. *This option happens to be cheap to undo* is
the gate. *We paid to make it cheap to undo* is Reversibility in the for-half.

---

# 2. Which options are even eligible?

## Thresholds are not values

Some things are **pass/fail**, not more-is-better: latency budgets, compliance, data residency,
security requirements, a cost ceiling. Above the bar, more buys you nothing. Below it, you are
disqualified.

Rule them out before anything else. An option that clears 200ms by a mile and violates data
residency is not a strong candidate with one flaw; it is not a candidate. People argue
trade-offs that were actually constraint failures more often than you'd expect.

The tell is the word **enough**. "Fast enough", "cheap enough", "secure enough" all describe a
bar being cleared, and a value has no *enough* — more is better, all else equal, which is what
makes it a value. So when someone says it, ask what the bar is. If they can name one, filter
against it. If they can't, the word was smuggling a preference, and it belongs in the trade
with a proper name on it.

## Who set the bar?

"Enough" tells you what to listen for. This tells you where to look: **ask who drew the line.**

If a regulator, a customer contract, a security policy or a product requirement drew it, it is
a threshold. You are clearing someone else's line, there is nothing to trade, and it gets
filtered here. If nobody drew a line and the argument is simply that more is better, it is a
value and it needs a name from the fourteen in §3 — the verdicts below are drawn from that list.

| Said | Who drew a line? | Verdict |
|---|---|---|
| "it needs to handle 10k rps" | product | threshold |
| "it'll scale better" | nobody | **Headroom** |
| "we encrypt at rest" | security policy | threshold |
| "we could hold less than the policy allows" | nobody | **Data Minimization** |
| "data must stay in-region" | contract or regulator | threshold |
| "fewer copies across fewer vendors" | nobody | **Data Minimization**, or **Isolation** |
| "daily rates don't compose into a range rate" | arithmetic | **found threshold** |
| "we can't backfill it" | nobody — it was never recorded | **found threshold** |
| "this won't scale", no number offered | nobody, and nothing shown | **not a line.** An unfalsifiable veto |

A team can draw its own line — "we keep p99 under 200ms" — and once a line exists it behaves as
a threshold whoever drew it. The question is a way of finding out whether one exists at all,
not a rule that only outside lines count.

## Some lines nobody drew

Sometimes the answer is *nobody, and it is still a line* — the last three rows above. Arithmetic,
physics, an upstream schema and the data you failed to record all disqualify options without
anyone deciding to. So the distinction that matters is not inside or outside the org. It is
**drawn or found**:

- A **drawn line** has a person behind it: a regulator, a contract, a product manager, your own
  team. It could have been different, which means it is negotiable and worth checking.
- A **found line** has no author. Daily unique users do not sum into a range unique count. You
  cannot read a field you never wrote — §1's recorded-versus-computed rule, arriving later as a
  constraint. Nobody chose these and nobody can unchoose them.

Both filter options before anything is compared. They differ in how you check them, and in what
is left when nothing clears the bar.

## A drawn line needs an owner and a date

Drawn lines get invented. "The security policy says so" is often one person's wiki page from
three years ago — and a threshold is the strongest move available in this document, because it
disqualifies options before anyone compares them and leaves nobody to argue with.

So ask two more questions before accepting one. **Who owns it** — who could change it if the
answer mattered enough? And **when was it last true**, because policies calcify around systems
that no longer exist. A line with an owner and a date is a constraint. A line with neither is
somebody's preference wearing a policy's clothes, and it belongs back in the trade with a
value's name on it.

An owner who is not in the room does not make the threshold unchallengeable. It means the
conversation is with them, and the cost of not having it is whichever option you just
discarded.

## A found line needs a demonstration

Found lines get forged too, and it is the more dangerous forgery, because arithmetic is not in
the room to object. The check cannot be who owns it — nobody does. It is whether the line can be
*shown*. Two days and three visitors on a napkin is the whole proof that daily unique counts do
not sum, and the option is gone before anyone weighs it:

| Day | Visitors | Daily uniques |
|---|---|---|
| Day 1 | A, B | 2 |
| Day 2 | A | 1 |
| Range | A, B | **2, not 3** |

A found line nobody can demonstrate — no napkin, no number, no growth story — is a hunch wearing
arithmetic's clothes. That is exactly what "this won't scale" is, and §3 files it as a veto
rather than a reason.

## When nothing clears the bar

Sometimes no option survives the filter. That is not a hard trade-off, it is a different
conversation: the bar moves, the scope shrinks, or the work doesn't happen. Take it to whoever
owns the threshold. Picking the least-disqualified option and hoping is how a compliance
problem becomes a launch problem.

That advice assumes a drawn line. A found line has nobody to take it to, so the bar does not
move and only two options remain: change the scope, or find a shape that does not need the
impossible thing. A rate that cannot be summed does not become summable because the spec asked
for the chart.

---

# 3. Saying why

## One sentence, two halves

> **Chose** daily aggregates **for Re-derivability**, **accepting Legibility**.

The second half is the point. "Because it's cleaner" is unfalsifiable — the only way to disagree
is to disagree with someone's taste. "For Legibility, accepting Optionality" is arguable,
because a reviewer can contest the trade without contesting the person.

The words come from the list below. Use the words, not synonyms — a shared vocabulary only works
if it is shared.

## How many values per half

One or two. A sentence naming five on each side has stopped discriminating: it lists everything
the design touches rather than claiming what was traded. If more than two genuinely apply, the
decision is probably several decisions wearing one name — split it and say why for each.

## The fourteen values

Each one is monotone: more is better, all else equal. The correction always comes from another
value pulling the other way, never from inside one.

| Whose interest | Value | Persona | Asks |
|---|---|---|---|
| The world | **Fidelity** | The practitioner who has done this job for 20 years without your software | Is this how the work actually is? |
| The world | **Re-derivability** | The analyst who has just been told the timezone was wrong | When the rule changes, can we recompute history, or only apply it going forward? |
| The future | **Optionality** | You, a year from now, asked to change it | When the requirement changes, what will it cost us to say yes? |
| The future | **Headroom** | The engineer serving ten times today's traffic | Does the cost of serving more grow with the load, or faster than it? |
| The future | **Enforceability** | The teammate who doesn't know the rule yet | What stops someone getting this wrong — the code, or a convention they must remember? |
| The future | **Isolation** | The tech lead drawing team boundaries | When this breaks or changes, how much else moves? Who owns it? |
| The future | **Reversibility** | The engineer backing it out next week | If this turns out wrong, how cheaply do we find out and undo it? |
| The next developer | **Legibility** | The new hire in week two | How fast can someone form a *correct* mental model? |
| The next developer | **Ergonomics** | The developer on another team wiring this up | How much do they have to do, and how much must they know first? |
| The next developer | **Convergence** | The developer designing the *next* service | Will they reuse what we picked, or add another way of doing this? |
| The operator | **Operability** | The person on call at 3am — and the person who should have been paged and wasn't | Can I see what it's doing, would I find out if it were wrong, and can I fix it without waking anyone else? |
| The payer | **Economy** | The engineer who has to ship it this quarter | How much work is this, to build and to keep alive? |
| The payer | **Speed** | Whoever is waiting on this to be useful | How soon can we ship it? |
| The person in the data | **Data Minimization** | The engineer asked, after a leak, why we still had it | Why did we keep this, and why for this long? |

## Values that get confused

### Optionality, Headroom, reversibility, rollback

Headroom sits beside Optionality on purpose: one is slack for the requirement *changing*, the
other slack for it *growing*. They are independent. A flexible schema that models anything can
still fall over at a thousand requests a second, and a rigid denormalized table can absorb
enormous load while being impossible to reshape.

Optionality is also not reversibility, which is Part 1's gate rather than a value here.
Optionality asks what a *new requirement* costs against this design; reversibility asks what
*undoing this choice* costs. Those come apart too — event sourcing is highly optional and
brutal to back out of, while a lint rule is trivially reversible and grants no optionality at
all. So "we can change it later" belongs to the deletion test, not to the because-clause.

And rollback is neither. Reverting a deploy takes minutes and undoes the code; it does not undo
the rows that code wrote, the column it backfilled, or the events it already consumed. So "can
we roll it back?" earns a confident yes on decisions that are permanent in the only sense the
tier model cares about. Ask it about the deploy, then ask the deletion test about the data.

The complexity class alone does not decide it. The test is **what *n* is, and what makes it
grow**:

- **Bounded by design** — verticals, statuses, a user's own payment methods. O(n) over five
  things is O(1) with extra steps, and optimising it spends Economy for nothing.
- **Grows with one user's activity** — their orders, their documents. Slow, self-limiting,
  usually fine.
- **Grows with total system usage** — every row you have ever written. Here the curve is the
  whole decision, because your own success is what breaks it.

"The number of rows we have ever written" is the answer that should stop the room.

Constant-time reads are nearly always bought at write time — an index, a counter, a cache, a
denormalized column, a pre-computed rollup — so Headroom usually costs Economy and Legibility.
Watch for one specific trap: the cheapest way to make a read O(1) is to store the answer, and a
stored answer is a computed result kept as your only record. Storing `count` instead of the rows
buys Headroom and destroys Re-derivability in the same move, and nobody notices until someone
asks a question the stored number cannot answer.

### Legibility, Ergonomics, Enforceability

Legibility and Ergonomics are the pair people collapse, so keep the short forms handy — together
with Enforceability from the table above, they cover reading, using, and misusing:

- **Legibility** — easy to understand correctly
- **Ergonomics** — easy to use
- **Enforceability** — hard to use wrongly

The word carrying Legibility is *correctly*. A `save()` that also sends an email is instantly
comprehensible and produces a false model, and a reader who leaves confident and wrong is worse
off than one who leaves confused — confusion at least prompts a question.

The two come apart in both directions, which is why they need separate words. Hand-written
boilerplate, every action and every reducer spelled out, is highly legible and tedious to use.
Convention-heavy framework magic is the reverse: one annotation and the endpoint works, and it
is months before anyone can correctly predict what happens at startup. They are also repaired by
opposite moves — you buy Legibility by removing indirection, and Ergonomics by adding a wrapper
that absorbs the boilerplate — so a design cannot be improved on both at once for free.

In review, the two complaints sound like this:

> "I don't understand this" → **Legibility**
> "I understand it fine, it's just annoying" → **Ergonomics**

Where everyone reads everything, the distinction is pedantry: the next developer both reads and
uses. Ergonomics earns its own word where the consumer never opens your source — published
contracts, platform and library work, services other teams call.

### Convergence, and what it isn't

"We already know this" is inertia, not a value — a team that maximises familiarity never adopts
anything. Convergence is about how many different ways of doing one thing the codebase ends up
with, and the pull toward the new thing comes from another value winning on its own terms.

The ramp-up cost people attach to it splits cleanly: mental-model cost is Legibility, staffing
is a hiring question, and "can anyone here run this in production at all" is a threshold.

## What each value is bought at the expense of

The second half of the sentence comes from here.

- **Fidelity ↔ Economy** — modelling the world costs more than modelling the current form
- **Optionality ↔ Enforceability** — every constraint that kills an invalid state also deletes a
  future. `NOT NULL` is correctness today and a migration tomorrow
- **Optionality ↔ Legibility** — the general structure is nearly always the less obvious one. A
  join table permits many, a column permits one, and the column is what a new hire understands
- **Optionality ↔ Ergonomics** — a general interface makes the common case longer
- **Isolation ↔ Legibility** — separate services contain the damage and destroy your ability to
  read a flow end to end
- **Re-derivability ↔ Economy** — keeping the input means an event log *and* rollups where one
  table would have done
- **Headroom ↔ Economy** — capacity you don't need yet is paid for now
- **Headroom ↔ Legibility** — sharding, caching and denormalization all obscure the thing they
  speed up
- **Operability ↔ Economy** — the logs, metrics, traces and runbooks that make a 3am fix
  possible are all built long before anyone needs them, and the reconciliation job that would
  catch a silently wrong number is built for a page that may never come
- **Reversibility ↔ Economy** — a shadow table, a dual write and a soak are built twice and
  thrown away once
- **Reversibility ↔ Speed** — the fortnight that proves the new path is the fortnight you did
  not ship in
- **Speed ↔ Economy** — buying is sooner and dearer, building is cheaper and later. Two
  different currencies, which is why they are two values and not one
- **Speed ↔ Fidelity** — the model that matches the world takes longer than the model that
  matches the current form
- **Data Minimization ↔ Re-derivability** — the raw input you keep in order to recompute is the
  raw input you would have to answer for
- **Data Minimization ↔ Fidelity** — the world's detail is often personal detail
- **Data Minimization ↔ Operability** — you cannot debug what you redacted
- **Convergence ↔ everything** — the conservative pull, and the value most strongly felt and most
  weakly stated

## What people say, and which value they mean

The whole point is to stop these from being interchangeable.

| What gets said | What it means |
|---|---|
| "it's cleaner" / "simpler" | Legibility or Convergence — say which |
| "more flexible", "future-proof" | Optionality |
| "that's not how it actually works" | Fidelity |
| "more maintainable" | usually Legibility plus Convergence; sometimes Isolation |
| "safer" | Enforceability if the compiler catches it, Operability if you'd see it fail |
| "it's standard", "everyone does it this way" | Convergence |
| "nicer API", "less boilerplate" | Ergonomics |
| "we could re-run it" | Re-derivability |
| "we'd own less of it" | Isolation |
| "it's quicker to build" | Economy — a real reason, so say it plainly |
| "it scales", "it'll scale" | Headroom — then say what *n* is |
| "it's faster" | Headroom if the curve is better, Economy if it's cheaper to run, a threshold if there is a latency budget |
| "fast enough", "handles our peak", "under 200ms" | a threshold. Met or not met — filter it, don't trade it |
| "this won't scale", with no number and no growth story | not a reason. An unfalsifiable veto |
| "it's testable" | derived: Isolation plus Enforceability. Score the parts |
| "can we roll it back?" | The deploy, not the design — code reverts, the rows it wrote don't. Operability at best; never the gate |
| "we can always change it later" | a reversibility claim, so it's the gate, not a value. Run the deletion test |
| "this won't box us in", "we're not locked in" | Optionality |
| "let's put it behind a flag", "we can back it out" | Reversibility — and say what building the escape hatch costs |
| "we need it by the demo" | a threshold if someone drew that date; **Speed** if the date is ambition |
| "we'd never finish in time building it" | Speed, bought with Economy |
| "do we really need to store that?" | Data Minimization |
| "it's PII" | a threshold if a rule already covers it; Data Minimization if you are choosing to hold less than the rule allows |
| "I just don't like X" | unstated. Convergence, or a value you haven't named yet |

## When the vocabulary is missing a word

If you keep reaching outside the fourteen, the set has a hole. That is worth knowing either way,
because the alternative is quietly improvising, which is the state this document exists to end.

But growing the set is the second thing to try, not the first. More often the word is already
here and its row is drawn too tight — the persona was written for one situation and the question
followed the persona. Operability was originally scoped to the person on call at 3am, which
quietly excluded the case where nothing pages anyone and a wrong number accumulates for a year;
that reads as a missing value right up until you notice it is the same value with a narrow
definition. The tell is the tension line. **If the word you want would be bought with the same
artifacts, at the expense of the same value, as one already on the list, it is that value and the
row needs widening.** Two values that cost the same thing are one value.

The bar for adding one: it is monotone, it has a persona who wants it and nothing else, and it
is in tension with something already on the list. And it has to be able to sit in the *for*
half — if nobody would ever choose an option *for* it, it belongs somewhere other than the
vocabulary, the way reversibility does. If it maximises alongside everything else, it is not a
value but a wish.

---

# 4. Is it the right trade?

Naming the trade makes it visible. It does not make it right. Four tests do that, and each one is
shown below on a sentence that passes the vocabulary and still fails the test.

## Test 1 — Restate both halves as consequences

A value names the axis, not the amount. *Buys Enforceability* justifies nothing, because nobody
can tell whether it bought a lot of it or a little.

> ✗ **Chose** an append-only event log **for** Re-derivability, **accepting** Economy.

Both halves are correctly named and neither is checkable. A reviewer who thinks this is a bad
idea has nothing to point at.

> ✓ **Chose** an append-only event log
> **For Re-derivability** — the timezone, the attribution rule and the bot filter stop being
> frozen at write time and become parameters we can re-run over history
> **Accepting Economy** — a collection endpoint and an aggregator to own: about three weeks, and
> a second service on the on-call rotation

Now the argument is *three weeks and an extra pager against being able to fix the attribution
rule*, which two people can actually disagree about. **Once both halves are consequences, people
argue about size instead of taste.**

## Test 2 — Check the recovery asymmetry

**Spend the low-tier value to buy the high-tier one.** The reverse needs a much higher bar,
because you are paying with the thing you cannot get back.

The quickest way to run this is to write the tier beside each half:

> ✓ **Chose** `zod` over hand-written validators
> **For Fidelity (tier 3)** — the schema stops accepting states the domain says are impossible
> **Accepting Convergence (tier 1)** — a second validation library; deleting it later is an
> afternoon

Arrow points the right way: a package you can delete bought a shape that goes into every row.

> ✗ **Chose** the ORM's generated schema
> **For Ergonomics (tier 1)** — no migrations to hand-write
> **Accepting Fidelity (tier 3)** — table shapes are whatever the ORM's inheritance mapping
> produces

Same sentence structure, arrow reversed, and the annotation is what makes it obvious. You bought
a convenience you could have had for a week's work with the shape of two years of rows.

**When both halves are tier 3**, this test has nothing to say — it assumes one side is cheap.
A schema against a published contract is the usual case: you need a new field, and either the
schema bends to match the API's JSON or the API bends to match the schema. Nothing on the board
is expendable, so three questions replace the one.

- **Can either half be made smaller, or later?** Version the contract so only one shape is
  published at a time. Buy information with a spike before committing the schema. A tier 3
  decision you can defer is not yet tier 3, and shrinking one side is usually cheaper than
  choosing between two permanent things.
- **Which failure surfaces sooner?** Constrain the side that fails loudly. A broken client tells
  you within minutes; a wrongly shaped write produces a row that looks fine for a year. Between
  two permanent mistakes, take the one you will hear about.
- **Which has more consumers you cannot reach?** Your schema has one consumer — your code. A
  published contract has every version you ever shipped. Keep flexible whichever one would need
  more people coordinated to change it.

If all three come out even, that is not a tie to be broken. It means the decision is not ready,
and the next move is to go and find something out rather than to pick.

## Test 3 — Bounded, or compounding?

> **Does accepting this once make the next one easier or harder?**

Bounded losses you can pay repeatedly. Compounding ones re-price every similar decision after
them, which is why they deserve a different bar even when the immediate cost looks identical.

> **Bounded** — "one date library, in one module." If it goes unmaintained, one module changes.
> Accept it, and the next such decision is exactly as free as this one was.

> **Compounding** — "just this one service in Kotlin." The immediate cost reads the same: one
> unit, one team, revert-able. But it arrives with a build toolchain, a CI lane, a second set of
> libraries to patch, an on-call rotation that now needs two languages, and a hiring
> conversation. And having said yes once, the argument against the third language is weaker than
> the argument against the second was.

Both are tier 1 by reach. The tier is not the whole answer, which is exactly why this is a
separate test.

## Test 4 — Name who pays

Convergence, Legibility, Ergonomics and Operability are all paid by people who are not in the
room. Those are literally their personas: the next service's author, the new hire in week two,
the developer on another team, whoever is on call.

> ✗ **Chose** the generated API client **for** Ergonomics, **accepting** Legibility.

Everyone in the review gets the Ergonomics. Nobody in the review pays the Legibility — it lands
on someone who joins in March and cannot work out what the generated code does at startup.

**When the chooser also pays, the bar is low. When someone absent pays, go and ask them**, or at
minimum write down who they are:

> ✓ **Accepting Legibility** — paid by the next person onboarding onto this service, who will
> read generated code instead of a call. Checked with two people who joined this quarter: both
> said the generated client was the confusing part of week one.

This is also what makes *"I just don't like Kafka"* suspicious. It is not that the objection is
wrong — it is that the person raising it bears none of the cost of the alternative, so nothing
in the sentence can be weighed.

## The full form

For anything that clears the gate, the sentence is worth expanding until every line can be
argued with:

> **Chose** `new-pkg` over `old-pkg`
> **For** Enforceability — invalid states fail at compile time, and this class caused three
> incidents last quarter
> **Accepting** Convergence — a second library doing a job we already have one for; paid by the
> next service's author and by on-call
> **Bounded?** Yes — one module, no tooling or CI changes
> **Exit** — if it goes unmaintained, swapping costs about a day

Every line is contestable, which is what a review needs. Compare it with "chose `new-pkg`
because it's cleaner," which cannot be argued with at all.

The form is also what makes a *restructured* trade visible — the most valuable move it can
surface, and the one that gets lost when rationale is a sentence of prose:

> **Chose** pre-aggregated daily records over aggregating on read
> **For** Headroom — every range query is a period sum, so read cost stops tracking total
> conversion volume
> **Accepting** Economy — a new long-running consumer to own — and Legibility, since the count
> now lives in a maintained document rather than in a query anyone can read
> **Not accepting** Re-derivability — the usual price of this optimisation is storing the answer
> and losing the ability to recompute it, so the raw events are kept underneath and the rollup
> stays derived

The last line is the interesting one. The default version of this trade destroys
Re-derivability; this team paid extra not to. Without the vocabulary that move is invisible,
and someone repeats the cheap version next quarter.

Close any decision above tier 1 with an **If wrong** line: the recovery tier, then the deletion
test's answer in concrete form. That is what tells a reviewer how hard to push, and it is worth
writing on the cheap decisions too — a bare *tier 1* waves things through, and tier X lives
mostly among the cheap ones.

> **If wrong** — tier 3, and tier X underneath it: a table with two years of rows and three
> readers we don't own. No backfill either; the instants were never recorded.

Naming both is the point. The tier says how far you would have to reach; the tier X clause says
that reaching won't help.

## A weak because-clause, repaired

> "Chose Postgres because it's more robust."

Nothing here can be argued with. *Robust* is not one of the fourteen, there is no second half, and
no line states a consequence — so a reviewer who disagrees is left disagreeing with the author's
judgement in general, which is not a conversation anyone wins.

> **Chose** Postgres over the document store
> **For** Enforceability — the constraints we need are declarable once in the schema instead of
> maintained in every writer
> **Accepting** Ergonomics — a migration for every shape change, where the document store needed
> none
> **Bounded?** Yes — one service, and the migration tooling already exists

Now disagreement has somewhere to land. A reviewer can say those constraints don't matter for
this data, or that shape changes are frequent enough that the migration cost dominates. Both are
specific, both are answerable, and neither is about the author.

---

# 5. What each decision usually turns on

Defaults, not rules. They tell you which two values to reach for first.

| Decision | Usually bought | Usually paid with |
|---|---|---|
| Data model | Fidelity | Economy, Legibility |
| Derived data — rollups, reports | Headroom, Re-derivability | Economy, Legibility |
| Integration style, deployment topology | Headroom, Isolation | Economy, Legibility |
| Service and domain boundaries | Isolation | Legibility |
| Tenancy, authorization, consistency | Fidelity | Economy |
| Published contracts | Ergonomics, Enforceability | Optionality |
| Tech stack | Convergence, Operability | Optionality |
| Build vs. buy vs. borrow | Economy, Isolation | Optionality, Operability |
| Retention and logging | Data Minimization | Re-derivability, Operability |
| Tier 1 — layering, patterns, naming | Legibility, Convergence | — |

Two of these are worth stating explicitly, because they are where teams most often optimise the
wrong half:

**Tenancy, authorization and consistency decide what gets recorded.** They describe who exists, who may see what,
and who owns which fact, so the domain has an answer and the spec probably has not asked for it.
*Does an account really have one organization? Can a person really hold only one role?* The
answer is almost always "no, we just don't support it yet" — the same mistake as a single
payment-method column on a customer, at a scale you cannot migrate out of. Convergence is explicitly not the value here: "it's
how the framework does auth" is how a permission shape gets chosen by a library that never met
your customers.

**Tier 1 decisions do not get this treatment.** Ask which one this team will read correctly,
default to what you already do, and move on.

---

# 6. Reading it back

The vocabulary pays off once for certain: per decision, when a reviewer can see what was
traded. That payoff needs nothing but the sentence.

The second payoff is conditional, and the condition is worth stating plainly rather than
implying it. Reading a year of decisions back only works if the sentences are somewhere you
can find them, in a shape you can search. Scattered through the prose of two hundred pull
request descriptions they are not a record, and this section is a wish. One decision fixes it
— a fixed heading in whatever your teams already write, or a line format grep can find — and
without that decision, skip to the three sources below, which need no infrastructure at
all.

*"We chose Convergence over Fidelity fourteen times last year"* is a finding, and an actionable
one. So is discovering that Economy never appears in the accepting half — which would mean
nobody has ever knowingly spent money to get something better, which is unlikely to be true and
therefore means people are hiding it.

Reading your values back does not require waiting for a year of sentences to pile up. Three
places the answer already exists:

- **Rewrites.** Something got built, then replaced. The axis it was replaced *along* is a value
  this team holds and has never written down.
- **Rejections.** A proposal that was cheap, wanted, obviously useful — and still got killed.
  There was an unstated value in the room.
- **Recurring review comments.** A thing three reviewers independently flag is a value.

Two cautions when mining that record. **Constraint or value?** A team that only shipped small
things might value Economy, or might have had two engineers — same trace, opposite conclusion.
You can separate them only where a decision was made *against* the constraint, or where the
constraint lifted and behaviour didn't change. **Missing denominator.** The options that lost are
rarely written down, so you recover the values you used, bounded by your past imagination.

---

# Red flags

Things you can catch in a review without re-reading this document. The left column is what
actually reaches you — a sentence someone says, or a shape a document has. The right column is
the move, because a flag that stops at a diagnosis just leaves you knowing something is wrong.

| What you hear or see | What it means | What to do |
|---|---|---|
| *"Chose Postgres because it's more robust."* | A because-clause with one half. Nothing was traded, so nothing was decided | Ask what it cost. If the answer is "nothing", there was no decision |
| *"The new structure is cleaner."* | Legibility or Convergence, unnamed — and they pull in different directions | Ask which. "Easier to understand" and "same as everywhere else" are not the same claim |
| *"We can always change it later."* | A claim about recovery, so it belongs in the tier, not the because-clause | Run the deletion test. What survives your deploy — rows, clients, nothing? |
| *"We need to weigh p99 against developer experience."* | A threshold argued as a value. Above the bar, more buys nothing | Filter it: met or not met. Then trade what's left |
| *"The security policy says so."* / *"That won't scale."* | A line with no owner, no date, and no demonstration | For a drawn line, ask who owns it and when it was last true. For a found one, ask for the napkin |
| A design doc with one option and no alternatives section | There was no decision to explain, only a plan to ratify | Name the do-nothing option out loud. If nobody can, say so in the doc |
| A full *for / accepting* block on where a helper function lives | Ceremony on a tier 1 decision — the deliberation costs more than being wrong would | Pick what this team reads correctly and move on |
| *"Why is `org_id` on every table?"* → nobody knows | A tier 3 decision made silently by whoever wrote the first migration | Reconstruct it now, while the row count is still small enough to change |
| *"If wrong — tier 1."* and nothing else | The reach was answered; the destruction question wasn't asked | Also ask: does this discard anything? And check the residue — if the commit changed what gets written, it was never tier 1 |
| *"That's low risk, we've done this before."* | Confidence is a probability claim. The tier already assumes you were wrong | Re-ask it as: *if* this is wrong, what does recovery cost? |
| *"We'll just use the framework's user model."* | Convergence chose a tier 3 asset. A library picked the shape that goes into every row | Split the question: is the library good, and separately, does its shape fit our domain? |
| *"We'll just default the missing ones to zero."* | A tier X repair that creates a second tier X — `0` and *unknown* stop being distinguishable | Carry the null, or narrow what you promise. Never write a guess into a permanent record |
| *"We can always replay the events."* | A tier 3 claim with an expiry date attached to it | Ask what the retention window is. Past it this is tier X, and it arrives with nobody doing anything |

---

# Compressed

- Sort by **recovery tier**, not by effort. Numbers go up as recovery gets harder — the opposite
  of Sev-1. One axis for 1–3: **how long does the thing I am replacing outlive my deploy?**
- **Tier 1**, not at all — one unit, a revert puts it back. **Tier 2**, until you close the
  migration window — a contract, so two versions coexist. **Tier 3**, past your deploy and past
  your reach — rows you can't rewrite, clients you can't call.
- A contract is any interface whose two sides can be at different versions at once. Touching one
  is at least tier 2, whoever owns the other side.
- A migration you can backfill and phase out is tier 2, however long it takes — with a tail into
  tier X, because a botched backfill destroys what you were reading from.
- Tier 3 is where the effort belongs — identity, history, boundaries, contracts, tenancy,
  authorization, consistency, and anything held in a store you can't rewrite. Two mechanisms:
  records you can't reach (expire them, if they're disposable) and code you don't own (reach them).
- **Tier X is off the axis**: not "hard to undo" but "nothing left to recover from." Ask it
  separately of every decision — *could anything that still exists reconstruct this?* Never
  written, or written and no longer usable. It stacks onto a tier rather than ranking above one.
- Tier X can arrive **later, with nobody doing anything** — retention expiry turns tier 3 into
  tier X on a schedule. "We can replay the stream" has an expiry date; ask what it is.
- A code change is tier 1 only if reverting it leaves **no residue in storage**. Once the commit
  decides what gets written down, the rows set the tier, not the diff.
- Tier X can't be priced at the table, so it gets weighed at zero. What you *can* check is what
  the fix costs whoever reads the records:
  - **Nothing** — the consumer already copes with records that lack the field. Start recording
    and move on.
  - **Real work** — the fix leaves two populations, before and after, and every consumer has to
    handle both from then on.
- The placeholder backfill is not a repair. `0` where you mean *unknown* is a second tier X, and
  it hides the first. Carry the null, or narrow what you promise.
- What you record accumulates and cannot be un-collected. What you compute re-runs. Never store a
  computed result as your only copy of its input.
- The loudest debates are tier 1. Convention, not deliberation. A huge port is still tier 1.
- Say it in two halves: **chose X for A, accepting B.** Use the fourteen words, not synonyms.
- Thresholds are not values. Filter them first — some are drawn and negotiable, some are found
  and not.
- Spend the low-tier value to buy the high-tier one. The reverse needs a much higher bar — write
  the tier beside each half and look at which way the arrow points.
- Name who pays. If they are not in the room, ask them.
- Read the records back once a year. That is what tells you what this organization values.

---

# Relevant approaches

Where these ideas come from, and where this document parts company with them.

**One-way and two-way doors** (Jeff Bezos, Amazon shareholder letters). Decisions split into
irreversible and reversible, with deliberation reserved for the first kind. The tiers here are
that split with the middle case named — most software decisions are neither a one-way door nor
free, and Tier 2 is where they live. The doors metaphor also asks about the decision; this
document asks about the thing being replaced and how long it survives, which is a property you
can check rather than a judgement you have to make.

**Architecture as the decisions that are hard to change** (Martin Fowler, building on Ralph
Johnson). The same axis, used to define what counts as architecture. This document borrows the
definition and asks what follows from it on a Tuesday afternoon.

**Architecture Decision Records** (Michael Nygard). Write down the decision, its context and its
consequences. ADRs assume you already know which decisions are architecturally significant;
§1 is an attempt to answer that, and the *for / accepting* sentence is meant to sit inside an
ADR rather than replace it.

**Magic Lenses, from the Foundation Sprint** (Jake Knapp and John Zeratsky). The origin of the
values idea: you pick an approach by naming what you value, not by scoring a rubric. Diverged
from deliberately — the 2x2 grids are gone, because a position on a plane records a verdict and
throws away the reasoning.

**Real options, and the last responsible moment** (Chris Matts and Olav Maassen; Mary and Tom
Poppendieck). Deferring a commitment has value, and that value can be reasoned about rather
than felt. It is the argument behind *if all three come out even, the decision is not ready* —
not a tie to break, but a signal to go and buy information.

**Event sourcing and CQRS.** The recorded-versus-computed distinction in §1 is the same shape as
events against projections: facts accumulate, views rebuild. If you already think that way about
storage, §1 is that idea applied to decisions.
