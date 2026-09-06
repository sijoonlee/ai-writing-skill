# Design Decisions — Worked Examples

A companion to `design-decisions.md`. One requirement, followed all the way down: which
decisions it hides, which of them are irreversible, and what the vocabulary makes visible that
prose would lose. The second half is the migration plan for when the wrong version is already
in production.

Every decision below is stated in the main document's form — *chose X for A, accepting B* — so
the trades are visible rather than buried in the argument for them.

---

## Worked example — unique visitors per campaign per day

### The situation

The PM's requirement: *know how many unique users visit per day, per campaign.*

The proposed approach:

- Assign a **visitor id** in a browser cookie
- Store a **campaign id** in a cookie when it appears in the URL
- Every visit triggers an API call
- Store the tuple **(visitor_id, campaign_id, date)** — one row per visitor per campaign per day

### Tier, and what it turns on

**Tier X — data model.** Not tier 3: the table is ours and we could migrate it tomorrow. What
puts it off the reach axis entirely is that what you didn't record, you cannot recover — the
visits already happened and nobody will make them again. There is no backfill, so the reach
question has nothing to work with.

Run the deletion test in the form for computed things — *if we deleted this, could we rebuild
it, and from what?* Every number on the dashboard is derived, and the only thing they are derived from is
this table. So the two values that decide it are **Fidelity**, because the model has to match
how visits actually happen, and **Re-derivability**, because nothing here can be rebuilt once
the rule changes.

### What's already right

Storing the **tuple** rather than a count is the key correct decision, and it is worth saying
before the criticism. Counts cannot be re-aggregated; tuples can. Weekly, monthly and
arbitrary-range uniques all stay computable. Had this been `(campaign_id, date, count)`, they
would be permanently impossible.

### The Fidelity pass

*Is this true of the world, or only true of the current requirements?*

| In the model | In the world | Verdict |
|---|---|---|
| A visit happens on a **date** | A visit happens at an **instant** | Requirements-only. "A day" is a reporting policy. |
| A visit has **one campaign** | A visitor can touch several campaigns, even in one day | Requirements-only |
| The cookie's campaign **is** the attribution | Attribution is a *rule* — first-touch? last-touch? 30-day window? | Requirements-only, and currently implicit |
| `visitor_id` = a unique user | `visitor_id` = one browser's cookie instance | Not true, and will be quoted as if it were |

**The date.** `YYYY-MM-DD` in whose timezone? A decision was made, possibly by accident, and
it is not re-derivable. Hour-of-day, business days in another region, "the first 24 hours after
launch" — all unanswerable for any data already collected. *A day is a computation whose output
was stored as the only fact.*

**The attribution.** Overwriting the campaign cookie silently implements **last-touch
attribution with an infinite window**. Nobody chose that; it fell out of "put the campaign id
in a cookie." Two consequences, both unrecoverable: a visitor who clicked in January still
credits that campaign in June, and there is no way to tell whether a row's campaign arrived
fresh from the URL or came off a stale cookie. Those are different facts about the world, and
only one of them means a campaign drove the visit.

### The structural problem

**A rollup is being stored as if it were the source of truth**, with policy baked into it. In
the vocabulary of the main document: the computed result is the only record. The fix is to
separate them.

```
visit_events                          ← append-only, the truth (Tier 3: recorded)
  event_id
  visitor_id
  occurred_at      timestamptz        ← instant, not date
  campaign_id      nullable
  campaign_source  'url' | 'cookie'   ← fresh, or carried?
  campaign_seen_at nullable           ← when the cookie was set
  path, referrer
  ua_hash, bot_suspected

visitor_campaign_day                  ← derived, recomputable
  visitor_id, campaign_id, day, tz    ← the original table, kept

campaign_daily_uniques                ← derived, recomputable
  campaign_id, day, unique_visitors
```

The original table survives as the middle layer, because range uniques genuinely need distinct
tuples rather than counts. It just stops being the thing that can never be rebuilt.

The event log is the **Tier 3 record**, fitted to the domain. The rollups are **Tier 1
computations** — expendable, so the timezone, the attribution rule and the bot filter can all
change and history recomputes behind them. In the original design, none of those are
recoverable.

> **Chose** an append-only event log underneath the existing day table
> **For** Re-derivability — the timezone, the attribution rule and the bot filter become
> parameters that can be re-run, instead of decisions frozen at write time
> **And Fidelity** — a visit happens at an instant, and the model now records one
> **Accepting** Economy — a collection endpoint and an aggregator to own — and Legibility,
> since three tables replace one
> **If wrong** — the event log is additive. Deleting it touches the aggregator and nothing
> downstream, so this half of the decision is cheap; the expensive half was already made the
> day the first row was written.

### Retention: a threshold, or a trade

Ask who drew the line. If a retention policy or a regulator already caps how long raw visit
logs may be kept, this is a **threshold** — unbounded raw events are disqualified and there is
nothing to weigh. If nobody has drawn one, it is a trade, and it is the one place in this
example where two values are in direct opposition:

> **Chose** a 30–90 day window on raw `visit_events`, with rollups kept forever
> **For** Data Minimization — the raw log is PII-adjacent, and almost everything it enables is
> needed within weeks of a mistake, not years
> **Accepting** Re-derivability — past the window, history is frozen at whatever rules were in
> force when it was written

Volume is not what decides this. A campaign-driven website is nowhere near the scale where raw
pageview rows are expensive, so anyone arguing storage cost is arguing the wrong axis.

### Three smaller irreversibilities

- **Name it accurately.** `visitor_id` is a browser cookie, not a person. Safari's ITP caps
  JS-set cookies at seven days, so "unique users" inflates steadily, and it is per-device
  regardless. Don't let the column be called `user_id`, and say "unique browsers" to the PM
  once — otherwise the number becomes a KPI and its drift becomes your problem.
- **Record consent state.** If a banner can block the cookie, some visits go uncounted, and
  "traffic fell" becomes indistinguishable from "consent rate fell."
- **Keep enough signal to reclassify bots.** Unique-visitor numbers are always eventually
  challenged as inflated. A UA hash on the event allows retroactive filtering; a bare distinct
  tuple does not, and the bots stay in history permanently.

### The questions to ask, in world-form

Not "will this change?" — they will say no, and be wrong. Ask about the world:

1. "If someone clicks campaign A on Monday and campaign B on Tuesday, whose visitor are they on
   Wednesday?" — forces the attribution rule into the open
2. "When we say a day, do we mean UTC, our office timezone, or the visitor's?" — they will ask
   whether it matters; it matters once, forever
3. "How long after a click does a campaign still get credit?" — the window nobody has stated

You don't need the answers to start building. You need them to stop being **implicit**,
because right now all three are being decided by cookie mechanics rather than by a person.

---

## Migration plan — when the old model is already in production

**The migration is easy. The backfill is impossible.** That split is the whole argument in
miniature, and saying it first sets expectations correctly before any work starts.

Easy, because nothing is reshaped: the event log is added *underneath* an unchanged table.
Impossible, because the existing rows already discarded time-of-day, campaign source, user
agent and consent state. The new fidelity begins at the cutover date, and history stays as it
is.

### The day table is recomputed, not converted

Today the collapse from visits to one-row-per-day happens **at write time**, inside the API
handler, and the timestamp is gone a millisecond later. Afterwards the same collapse happens in
a job, over data you kept. Same collapse, later, from something you still have.

```sql
-- produces exactly what the upsert produces today
SELECT DISTINCT visitor_id, campaign_id, (occurred_at AT TIME ZONE 'UTC')::date AS day
FROM visit_events;
```

```
visit_events            ← facts, kept
      │  DISTINCT + date truncation
      ▼
visitor_campaign_day    ← unchanged in schema, name and consumers
      │  COUNT
      ▼
campaign_daily_uniques
```

The middle box already exists. What changes is only what sits above it: today that arrow
starts from the HTTP requests themselves, which nobody keeps.

The payoff is that the truncation becomes a **parameter**. Change `'UTC'` to `'Asia/Seoul'`,
re-run, and a 23:58Z visit correctly moves to the next day. Hour-of-day, launch-window
analysis, retroactive bot exclusion and attribution windows are all the same query over the
same events — and all permanently unanswerable against a table of dates.

### Shape of the change

- **A new collection endpoint** takes the richer payload and appends to `visit_events`. The
  existing endpoint is left completely untouched, so the data downstream depends on keeps being
  produced exactly as it is today, and the kill switch is "stop calling the new endpoint."
- **An aggregator** re-derives `visitor_campaign_day` from the events, so downstream consumes
  what it always did.

Name the new route as a different **resource** (`/events`), not a version (`/v2/collect`). It
genuinely is a different thing, and resource names age better once the old one is gone.

### Sequence

1. **Launch the aggregator, writing to a shadow table.** Nothing downstream can be affected by
   a bug in the new path.
2. **Diff shadow against live** until they match. Two weeks is a reasonable soak. This is the
   step that earns trust for everything after it.
3. **Deploy the client** to call the new endpoint. Consider sampling — 10% for a few days —
   before going to 100%, and `navigator.sendBeacon` so the call doesn't compete with page load.
4. **Watch the old endpoint's traffic decay.** Retire it on a **threshold, not a date**: cached
   JS keeps stale browsers calling it for a cache TTL or two, and those visits are lost
   outright once it's gone.
5. **Stop the old endpoint upserting.** Either off entirely, or — to lose nothing — keep the
   route and have it append a `visit_events` row with the new fields null. Prefer "keep the
   route, stop writing" over deleting it; a dead path generates console errors and retry noise.
6. **The aggregator is now the only writer** to the day table. Set retention on `visit_events`.

Steps 1–4 are all reversible. The first irreversible moment is 5, and by then the new path has
already been proven to reproduce the old numbers.

### Aggregator properties that matter more than where it runs

Make it a scheduled **job** in the service you already run, not a new deployable — chosen
**for** Convergence and Operability, **accepting** Isolation, since the rollup now shares a fate
with whatever hosts it. The rollup has no independent scaling story, so Isolation is the cheap
half to spend here. Use a separate service only if you already have a job platform where that
*is* the standard thing.

- **Batch recompute, not incremental append.** Recompute a trailing window — say three days —
  on a schedule, rather than processing since a watermark. Late events are guaranteed: a visit
  at 23:59:58 whose call lands at 00:00:02. Incremental logic gets those permanently wrong,
  while a recompute window heals itself on the next run, and a bug fix becomes "re-run the
  range."
- **Idempotent upsert.** Same input range, same output rows, every time. That property is what
  makes recompute safe and re-derivation possible.

### Two writers, and why it mostly doesn't matter

While both the old endpoint and the aggregator write the day table, they produce the
*identical* tuple. It is a set union, idempotent by construction, with nothing to reconcile.
The overlap only matters in two narrow places:

- **Extra columns.** With `tz`, `source` or `first_seen_at`, the two writers agree the row
  exists but disagree about its attributes, because the old endpoint has no timestamp to offer.
- **Recompute.** Re-deriving a range means deleting it and regenerating from events. Rows
  written by the old endpoint have no backing event, so they are deleted and never return —
  silent, small and permanent. Scope the delete to `source = 'derived'`, or finish step 5
  first.

Both dissolve once the aggregator is the sole writer.

### Don't synthesize history

Never backfill `visit_events` from the old rows. Writing `occurred_at = day + 00:00 UTC`
produces facts that look real and aren't, and someone will plot hour-of-day a year later and
find a midnight spike spanning the entire pre-cutover period. Leave the old rows untouched,
record the cutover date where a human will find it, and accept a visible seam.

Two optional additive columns make that seam legible — both nullable, no rewrite:

- **`tz`** — which timezone this row's `day` was computed in. This is what marks a row as
  *recomputable*; nulls identify the rows that can never move.
- **`source`** — `'legacy'` vs `'derived'`, if marking the seam explicitly beats remembering a
  date.

### Expect the numbers to move, and say so first

Once the day boundary comes from a real timestamp in a declared timezone, some visits land on a
different date than they used to, and the midnight dedupe boundary shifts with them. The delta
is small but nonzero, and someone downstream will notice a KPI change on cutover day.

The soak gives you the exact number in advance. *"Uniques will move by about X% on this date,
here's why"* is a trivial conversation beforehand and a very unpleasant one afterwards.

### If the migration slips — the one-hour version

Two additive columns on the existing table, `ALTER TABLE ADD COLUMN` with a null default, no
rewrite, no downtime:

- **`first_seen_at timestamptz`** — set on insert, untouched on conflict. Recovers time-of-day
  and makes the timezone question re-derivable from here on.
- **`campaign_source`** — `'url'` if the campaign arrived fresh in the address bar, `'cookie'`
  otherwise. This is the fact that cannot be reconstructed later, and it separates "a campaign
  drove this visit" from "a stale cookie was present."

Neither requires the full migration, and they stop the two most expensive losses today.
