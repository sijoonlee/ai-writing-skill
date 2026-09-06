# Workbook — worked example

One pass through `design-decisions-workbook.md`, filled in. The decision is the aggregate day key
from the partner dashboard ERD, chosen because it exercises every slot: a threshold that turns out
to be real, a tier that isn't the one the diff suggests, a tier X that hasn't arrived yet but
will, and an option that would have avoided it.

---

## Step 1 · Is there anything to decide?

**The decision:** what key do we store daily lead aggregates under?

- [x] **Someone drew a line** — partner contracts are written in Toronto business days, and the
      lead-price periods admins enter are Toronto-dated. Revenue is `count × price-for-that-day`,
      so counts and prices have to agree on what a day is.
      → **Real threshold.** Owner: the contracts, via product. Last true: current contracts,
      checked with Artem. Any option that buckets on a different calendar is disqualified, not
      merely worse.
- [ ] **Reality drew a line** — none here. Both UTC and Toronto bucketing are arithmetically fine;
      they just disagree about 8pm–midnight.
- [ ] **Only one option was ever on the table** — no, four below.

**Options that survive the filter:**

| | | |
|---|---|---|
| **A** | Toronto `YYYY-MM-DD` label on the aggregate | survives |
| **B** | UTC day bucket | **disqualified** — misfiles 8pm–midnight Toronto traffic into the next day, so counts and price periods disagree |
| **C** | Store the raw instant per conversion, aggregate at read | survives |
| **D** | Store the Toronto day *and* keep the instants underneath | survives |
| **—** | Do nothing — no aggregates, no dashboard | survives, and is the baseline |

---

## Step 2 · What tier?

- [x] **Does this change touch a contract?** The aggregate collection is read only by the portal, which we
      own and deploy together. No external consumer. → not escalated by contract.
- [x] **Does this change what we save?** Yes — this *is* a decision about what lands in the
      database. Without it we'd be saving a row per conversion carrying its instant; with it we
      save one row per partner per day carrying a count. Different records, not just different
      code. → **not tier 1**, whatever the diff looks like. The consumer is one small service and
      the commit is a hundred lines; neither is what sets the tier.
- [x] **If this turns out wrong, could we get back what it dropped?** **Yes — for now.**

      *What does it drop?* The time of day. A conversion arrives stamped `2026-09-02T23:40:00Z`;
      we keep the Toronto date it fell in and drop everything finer. So: hour-of-day, any other
      day boundary, and the ability to tell a 9pm lead from a 9am one.

      *Where else does that live?* The conversions stream. `content.created` travels in the
      payload, and `replay-helper` can reprocess it under any rule — a real source, already
      exercised by the idempotency tests. **But only while the stream is retained, and nobody in
      the review knew that window.** → action: find the number before milestone 2, and put it in
      the SDD.

      → **Recoverable today. Permanent later.** Tier 3 now; tier X the day retention lapses; nobody will know once retention expires

**Tier:** 3 — the rows are ours to rewrite, but only by reprocessing a stream whose retention we
don't control.

**Tier X?** Not yet, and that is the whole problem. This is the case from §1 where tier X arrives
later with nobody doing anything: the decision is recoverable right up until a retention policy
nobody in this review has read closes the window, at which point it silently becomes permanent.

- **Grace period** — the counter ships to dev and writes aggregates nobody reads for two
  milestones. While it runs on test traffic, a wrong day boundary costs `dropCollection()` and a
  redeploy. **Ends when the counter first processes production traffic**, not when the tab ships.
- **Expiry** — the conversions stream's retention window. **Nobody knew the number.** Everything
  in this decision rests on it, and it is the one fact the review did not have.

---

## Step 3 · Say why

> **Chose** the Toronto calendar-day label (A) over keeping the instant timestamps (D)
> **For Fidelity** — partners read these numbers in Toronto business days and are paid against
> Toronto-dated price periods, so the day the count is filed under is the day the money is
> computed for
> **Accepting Economy** — option D costs a second collection holding one row per conversion,
> indefinitely: storage that grows with total lead volume, plus the write path and the backfill
> tooling to go with it
> **If wrong** — tier 3, becoming tier X at an unknown date. Concretely: the day product asks for
> leads per hour — or a different day boundary, or a split around the March clock change — the
> answer is *"yes, if we replay the stream, and only for as far back as it is retained."* Past
> that point the answer becomes *"from today forward, never for history"*, and nothing announces
> the change

**Note for the record:** option D would have removed the dependency on replay entirely. It was not
overlooked — it was declined for Economy, on the argument that the stream already holds the
instants. That argument is sound and has an expiry date, and the expiry date is the number nobody
knew. Whoever revisits this should start by checking whether it has already passed.

---

## Step 4 · Is it the right trade?

- [x] **Both halves are consequences.** Fidelity half: a lead submitted at 9pm Toronto on the 2nd
      is paid at the 2nd's rate, not the 3rd's. Economy half: roughly one row per lead per partner
      held forever, versus one row per partner per day.
- [x] **Tiers beside each half.** Fidelity here is tier 3 (it decides what's recorded). Economy is
      tier 1 — a collection we could add later and backfill by replay. **We spent the low tier to
      buy the high one**, which is the right direction.
      *Caveat, and it is the whole decision:* Economy is only tier 1 **while replay works**. After
      retention closes there is nothing to backfill from, both halves become permanent, and this
      test stops applying to a choice we have already made. The direction is right today and
      silently wrong on a date nobody has looked up.
- [x] **Bounded or compounding?** Bounded. One collection, one key shape. Accepting it does not
      make the next aggregation decision harder.
- [x] **Who pays, and are they in the room?** The Economy is paid by us, now — we're in the room.
      The Fidelity risk is paid by whoever answers the first disputed revenue number, and by the
      analyst who asks an hour-of-day question in 2027. **Neither is in the room.**
      → action: write the retention window and what depends on it into the SDD, so the person who
      arrives later finds out from the document rather than from a replay that returns nothing.

---

## What the pass produced

Three things that would not have come out of writing a paragraph of prose:

1. **Option B was disqualified, not outvoted.** A threshold with a real owner, filtered in step 1
   rather than argued about in step 4.
2. **The tier didn't match the diff.** A hundred-line consumer looked tier 1 and is tier 3,
   because the commit decided what lands in the database.
3. **The load-bearing unknown got named.** Every reassuring line in this decision — recoverable,
   tier 3 not tier X, Economy is the cheap half — rests on the stream's retention window, and
   nobody in the review knew it. Written as prose, "we can always replay it" would have read as
   closing the question. The pass turns it into an action with an owner and a deadline, which is
   the difference between a decision and an assumption nobody noticed making.
