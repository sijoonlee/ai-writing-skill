# Design Decisions

*What to slow down on, and how to say why.*

Most of what engineers argue about can be undone in a week. The decisions that outlive everyone
— the data model, the tenancy shape, who owns which fact — get made silently, by whoever writes
the first migration. So this document starts by sorting decisions by what it costs to be wrong,
and says which ones have earned a real conversation. Then it throws out the arguments that were
never trades at all — the ones where somebody had already drawn a line and the only question was
whether you cleared it. What survives needs a vocabulary, and today every developer has a
different word for *because* — cleaner, simpler, more maintainable, more pragmatic — so no two
people mean quite the same thing and nobody can read back what anyone actually valued. Twelve
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

"A module" means you're fine. "A table with two years of rows in it, and the three services
that read from it" means you have located the part of the design you only get one attempt at.

## Nouns and verbs answer that question differently

A **noun** is something you record and then accumulate: a customer, a permission, a visit. A
**verb** is something you compute from what you recorded: a daily rollup, an attribution rule, a
report.

Nouns are irreversible because data cannot be un-collected — a field you didn't write in March
is not available in June. Verbs are expendable because they re-run: change the rule, recompute,
and history moves with it.

So the deletion test has two forms:

- **For a noun** — if we deleted this, what else would we have to touch?
- **For a verb** — if we deleted this, could we rebuild it, and from what?

A verb whose input you kept is cheap to lose. **A verb whose input you discarded is a noun you
did not know you were creating** — which is what happened the day someone stored a date instead
of a timestamp.

## The three tiers

**Tier 1 — irreversible.** Data gravity, org gravity, or consumers you don't control. This is
where design effort pays.

- **Data model** — entities, cardinality, identity, history
- **Service and domain boundaries** — because they become team boundaries
- **Published contracts** — external consumers you cannot coordinate
- **Multi-tenancy model** — single to multi-tenant is one of the worst migrations there is, and
  it is usually decided by accident on day one
- **Identity and authorization** — the permission *shape* is a noun, ends up denormalized into
  every query, and is brutal to change
- **Consistency and source-of-truth ownership** — who owns which fact, what is transactional and
  what is eventual. Not "do we use Kafka" — *where does truth live*

**Tier 2 — expensive but bounded.** A rewrite, not a data migration. How far it spreads depends
on how much you let it leak.

- **Tech stack** — languages, frameworks, datastores, infrastructure
- **Integration style** — sync RPC or async events, orchestration or choreography
- **Deployment topology** — monolith, modular monolith, services, regions
- **Build vs. buy vs. borrow** — often the largest lever here, and the least deliberated

**Tier 3 — cheap and local.**

- **Layering and internal structure** — where business logic lives
- **Error handling and retry patterns**
- **Testing strategy**
- **Module organization, naming conventions**

## The inversion

**The decisions with the most discussion are mostly Tier 3, and the Tier 1 ones get made
silently.** Nobody holds a design review on the tenancy model or the permission shape. They get
decided by whoever wrote the first migration, on a Tuesday, in twenty minutes. That is where the
2am backfill comes from.

Layering is the clearest case. It is what engineers argue about hardest — clean architecture,
ports and adapters, the service layer is anemic — and it is among the *most* reversible
decisions available, because it is pure code and code is expendable. You can restructure a
layering scheme in a week with no data touched and no consumer notified.

There is one real irreversibility inside that debate, and it is not the one people argue:

> **Does the stack leak into the nouns?**

If your domain model *is* your ORM model, the framework choice has been welded to the schema.
The ORM's conventions — how it names tables, picks primary keys, maps inheritance, expresses
optional — are now shapes in your database with two years of rows in them. So swapping the ORM,
a Tier 2 rewrite you should be able to afford, means reshaping tables under live data, which is
a Tier 1 cost you can't. The reversible decision has been promoted into the irreversible tier
without anyone deciding to promote it.

That is the actual content of the layering argument. Not the principle usually called
*purity* — that a domain should never know the database exists — but the practical business of
keeping the expendable layer expendable. It is what "hexagonal" and "ports and adapters" are for: the domain declares
what it needs from storage, and the framework implements that, rather than the framework
defining what the domain is.

So the useful question is narrow: *can the nouns be expressed without the framework?* Yes, pick
a convention and stop arguing. No, and you have promoted a Tier 2 decision into Tier 1.

Two moves buy most of the protection, and neither requires a full mapping layer. **Hand-write
the migrations**, so the schema is something you designed and the ORM maps to it rather than
defining it. And **keep framework types out of domain signatures** — no session, no query
builder, no lazy proxy in a function the domain owns. An architecture test that fails the build
when the domain package imports the persistence package turns that from a convention people
remember into Enforceability.

## The gate — when a decision deserves deliberation

Three conditions, all required. The rest of this document calls them **the gate**:

1. **Irreversible or expensive to undo** — Tier 1, sometimes Tier 2
2. **More than one genuinely plausible option** — not one option and two strawmen. Doing
   nothing is always one of them, and is usually the one nobody wrote down
3. **Actually contested**, or contested-by-default because nobody has examined it

Everything else gets a convention, a default, or whoever is writing it. Deliberating Tier 3
decisions is how design practice earns its reputation for ceremony.

## What reversibility is not

**Reversibility is not one of the twelve values, and it is never the answer to "I chose A over
B because…".** It does two other jobs instead: it decides whether a decision earns deliberation
at all — the gate above — and it settles which way a trade should go once you are inside one
(§4: spend the reversible value to buy the irreversible one). Nobody has ever chosen an option
*for* reversibility. That is why "we can always change it later" sends a reviewer back here
rather than into the vocabulary.

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
value and it needs a name from the twelve in §3 — the verdicts below are drawn from that list.

| Said | Who drew a line? | Verdict |
|---|---|---|
| "it needs to handle 10k rps" | product | threshold |
| "it'll scale better" | nobody | **Headroom** |
| "we encrypt at rest" | security policy | threshold |
| "we could hold less than the policy allows" | nobody | **Data Minimization** |
| "data must stay in-region" | contract or regulator | threshold |
| "fewer copies across fewer vendors" | nobody | **Data Minimization**, or **Isolation** |

A team can draw its own line — "we keep p99 under 200ms" — and once a line exists it behaves as
a threshold whoever drew it. The question is a way of finding out whether one exists at all,
not a rule that only outside lines count.

## When nothing clears the bar

Sometimes no option survives the filter. That is not a hard trade-off, it is a different
conversation: the bar moves, the scope shrinks, or the work doesn't happen. Take it to whoever
owns the threshold. Picking the least-disqualified option and hoping is how a compliance
problem becomes a launch problem.

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

## The twelve values

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
| The next developer | **Legibility** | The new hire in week two | How fast can someone form a *correct* mental model? |
| The next developer | **Ergonomics** | The developer on another team wiring this up | How much do they have to do, and how much must they know first? |
| The next developer | **Convergence** | The developer designing the *next* service | Will they reuse what we picked, or add another way of doing this? |
| The operator | **Operability** | The person on call at 3am | Can I see what it's doing, and fix it without waking anyone else? |
| The payer | **Economy** | The engineer who has to ship it this quarter | How much work is this, to build and to keep alive? |
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
stored answer is a verb's output kept as your only noun. Storing `count` instead of the rows
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
  possible are all built long before anyone needs them
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
| "do we really need to store that?" | Data Minimization |
| "it's PII" | a threshold if a rule already covers it; Data Minimization if you are choosing to hold less than the rule allows |
| "I just don't like X" | unstated. Convergence, or a value you haven't named yet |

## When the vocabulary is missing a word

If you keep reaching outside the twelve, the set has a hole and should grow. That is a better
outcome than quietly improvising, which is the state this document exists to end.

The bar for adding one: it is monotone, it has a persona who wants it and nothing else, and it
is in tension with something already on the list. And it has to be able to sit in the *for*
half — if nobody would ever choose an option *for* it, it belongs somewhere other than the
vocabulary, the way reversibility does. If it maximises alongside everything else, it is not a
value but a wish.

---

# 4. Is it the right trade?

Naming the trade makes it visible. It does not make it right. Four tests do that.

**Restate both halves as consequences.** The value names the axis, not the amount. "Buys
Enforceability" justifies nothing. "The compiler catches the null-handling class that caused
three incidents last quarter" is checkable, and so is "one more runtime in CI, one more thing in
onboarding." Once both halves are consequences, people argue about size instead of taste.

**Check the reversibility asymmetry.** Spend the reversible value to buy the irreversible one,
and treat the reverse as needing a much higher bar. Convergence is usually the cheapest thing you
own — you can delete the odd package and standardise later. Fidelity in a schema is not. That is
why adopting one new library for a real correctness gain reads as obviously fine, and why letting
the ORM shape the schema reads as obviously not.

**Ask whether the loss is bounded or compounding.** One package inside one module is bounded, and
the exit is cheap. A new *language* compounds — it pulls in tooling, CI, hiring, and on-call
knowledge, and it re-prices every similar decision after it. Test: does accepting this once make
the next one easier or harder?

**Name who pays.** Convergence, Legibility and Ergonomics are all paid by people who are not in
the room — the next service's author, whoever is on call, the new hire in week two. Those are
literally the personas. When the chooser also pays, the bar is low. When someone absent pays, ask
them. This is also what makes "I just don't like Kafka" suspicious: the person deciding bears
none of the cost.

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

For a Tier 1 decision, add the deletion test's answer in concrete form, because that is what
tells a reviewer how hard to push:

> **If wrong** — a table with two years of rows and three readers. No backfill; the instants were
> never recorded.

## A weak because-clause, repaired

> "Chose Postgres because it's more robust."

Nothing here can be argued with. *Robust* is not one of the twelve, there is no second half, and
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
| Tier 3 — layering, patterns, naming | Legibility, Convergence | — |

Two of these are worth stating explicitly, because they are where teams most often optimise the
wrong half:

**Tenancy, authorization and consistency are decisions about nouns.** They describe who exists, who may see what,
and who owns which fact, so the domain has an answer and the spec probably has not asked for it.
*Does an account really have one organization? Can a person really hold only one role?* The
answer is almost always "no, we just don't support it yet" — the same mistake as a single
payment-method column on a customer, at a scale you cannot migrate out of. Convergence is explicitly not the value here: "it's
how the framework does auth" is how a permission shape gets chosen by a library that never met
your customers.

**Tier 3 decisions do not get this treatment.** Ask which one this team will read correctly,
default to what you already do, and move on.

---

# 6. Reading it back

The vocabulary pays off twice. Once per decision, when a reviewer can see what was traded. And
again in aggregate — wherever these sentences end up living, whether that is a design doc, a
pull request, or a decision log, they can be read back together.

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

| Flag | What it means |
|---|---|
| A because-clause with only one half | Nothing was traded, so nothing was decided |
| "It's cleaner" | Legibility or Convergence, unnamed. Ask which |
| "We can always change it later" | The gate, not a reason. Run the deletion test |
| Latency or compliance argued as a trade-off | A threshold treated as a value |
| Only one approach was ever considered | There was no decision to explain |
| A trade written out for a layering or naming choice | Ceremony on a Tier 3 decision |
| Nobody can say why the tenancy or permission shape is what it is | A Tier 1 decision made silently by the first migration |
| Convergence bought a Tier 1 asset | You optimised the reversible layer and guessed at the irreversible one |
| The accepting half is always the same value | Either a real org priority, or a blind spot. Worth knowing which |

---

# Compressed

- Sort by cost of being wrong. Score it with the deletion test, in noun form or verb form.
- Nouns accumulate and cannot be un-collected. Verbs recompute. Never store a verb's output as
  your only noun.
- Tier 1 is irreversible — data model, boundaries, contracts, tenancy, authorization,
  consistency. That is where the effort belongs.
- The loudest debates are Tier 3 and reversible. Convention, not deliberation.
- Say it in two halves: **chose X for A, accepting B.** Use the twelve words, not synonyms.
- Thresholds are not values. Filter them first.
- Spend the reversible value to buy the irreversible one. The reverse needs a much higher bar.
- Name who pays. If they are not in the room, ask them.
- Read the records back once a year. That is what tells you what this organization values.
