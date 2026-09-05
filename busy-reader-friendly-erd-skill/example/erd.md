# Dashboard ERD

## Metadata

|  |  |
| --- | --- |
| **Status** | `Ready for review` |
| **Author** | @sijoon.lee |
| **Reviewers** | @Jacob Foster @Lee Robert @Abul Qasim @Artem Sheika |
| **Last updated** | 2026-09-02 |
| **Related** | [PRD](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4503470081) · [SDD - Users](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4507402261) · [SDD - Lead counting](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4520673281) |

---

## Context

Partners embed our quoters and tracked links; we pay them per lead, at a lead price ($/lead) set in each partner's contract.

The PRD calls for an Overview tab in the partners portal

- What the tab shows
    - Numbers per vertical (P&C and Life for MVP)
    - Four numbers: Users, Unverified Leads, Conversion Rate, and Projected Revenue
    - One chart: Unverified Leads over time
- It states that the lead counts are unverified and the revenue is a projection. (PRD)

**Terminology:**

- the PRD's "Unverified Leads" is the partner-facing name for what engineering calls **conversions** — specifically `SUBMITTED_LEAD` conversion events; UI copy and the PRD keep "Unverified Leads".
- the PRD's "$/lead rate" is called the **lead price** here

**Four-piece work:**

- A way to get unique counts of User into the system \[[SDD](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4507402261)\]
    - Action item:
        - Profile(-bff) sends data to Profile-service on get-session
        - Profile-service stores them in `affiliateVisits` collection
        - Profile-service endpoint provides new endpoints for Profile-bff and Partners-portal
- A way to get contracted lead price($/lead rate) into the system (contracts live outside our partners-portal application)
    - Action item: add new Admin page menu allowing Admin to enter lead price
- A way to count partner-attributed leads continuously \[[SDD](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4520673281)\]
    - Action item: set up new aggregation service `affiliate-conversion-counter`
- The dashboard views that join the three above.
    - Action item:
        - set up 'read' of external data
            - \[1\] the number of Conversions
            - \[2\] the number of Users
        - set up view with calculated values
            - the conversion rate = \[1\]/\[2\] \* 100 (%)
            - the projected revenue = sum over each day in the range of (that day's \[1\] × the lead price in effect that day)

## Goals & non-goals

**Goals**

- Count partner-attributed leads per affiliate id, per vertical, per day, from the platform's existing event stream — no new tracking added to the quoter funnel for lead counting.
- Dashboard date-range queries (30 days / 90 days / YTD / since beginning) answer from pre-aggregated daily records; no scan over raw conversions at request time.
- Admins can enter and maintain each partner's lead price per vertical, as effective-dated periods, in the existing Admin page.
- Projected Revenue is derived, never stored: daily lead counts × the lead price in effect that day. Correcting a lead price retroactively corrects the displayed revenue.
- Timezone rule, applied consistently across the consumer, the lead-price records, and the display: **instants are stored as UTC timestamps** (the conversion's `created`, processing times, "data last refreshed"); **the aggregate's day key is a plain** `YYYY-MM-DD` label meaning the Toronto calendar day (`America/Toronto`) of the event's `content.created` — the conversion's creation instant, which travels in the event payload and is stable under event replay. The label is not a timezone-bearing value — the Toronto boundary is baked in at write time, because aggregation discards time-of-day and the choice cannot be revisited at read time.
- Per-partner visibility control for the Mortgage vertical, default hidden (separate from the Milestone-1 access gates).

**Non-goals**

- **No lead verification pipeline.** Manual review by BUD/PM/stakeholders happens outside this system; the dashboard knowingly shows unverified counts and "Projected" revenue that will not update after payout cleanup (PRD's pipeline constraint).
- **No payout or invoicing.** The lead-price records feed a projection on a dashboard; actual payment stays wherever it is today.
- **No sub-vertical breakdown.** Auto/home/condo are one P&C bucket for MVP; sub-tabs can come later because the aggregates keep `productType` granularity (see Design decisions).

## Technical vision

This is Four-piece work

1. **Users**
    - The number of _unique_ people who arrived via the partner's tracked link (`aff_id`) or landed in our quoter through the Quoter Launcher, within the selected date range.
    - The same person landing 7 days in a row is 1 user (and 7 landings — landings are not the metric).
    - Users cannot be a plain daily counter — unique counts don't sum across days (7 daily counts of the same person add to 7, but the range's true count is 1), so the source must deduplicate across the selected date range
    - Uniqueness is assumed from their visitor id
        - _if they use different visitor id, they are different users_
    - See the [System Design doc](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4507402261) for the data source and the range-dedup mechanism
2. **Lead counting**
    - The number of partner-attributed conversions (the PRD's "Unverified Leads") per `aff_id`, per `productType`, per Toronto day.
    - A new service, `affiliate-conversion-counter`, consumes the platform's existing conversions event stream, keeps the events that pass a per-BU filter (ex: Insurance keeps assigned, monetized `SUBMITTED_LEAD` events), and upserts a daily aggregate document per `(aff_id, productType, torontoDay)`.
    - Consumption is idempotent, backed by stored state: each daily document holds the set of processed conversion ids, and the day's count is _derived from that set's size_ — a bare counter cannot be idempotent under Kafka's at-least-once delivery or event replay.
    - Aggregates are keyed by `aff_id` (what the event carries), not partner org; the portal resolves org → aff_id through the existing affiliates service at read time.
    - See the [SDD - Lead counting](https://ratehub.atlassian.net/wiki/spaces/RET/pages/4520673281) for which event type to consume, the filter mechanism, the data model, and failure handling.
3. **Lead Price**
    - A lead price comes with a period: `(partner, vertical, $/lead, start date, end date?)`, dates inclusive, Toronto time; an open end date means "until further notice"
    - Lead Prices are persisted in a standalone collection, one document per lead-price period (see Design decisions):
        - `{ partnerId, vertical, leadPrice: number, startDate: string, endDate?: string, createdBy, createdAt }`
        - `createdBy`/`createdAt` make every lead price attributable — this is money, and "who set this and when" will be asked the first time a projected-revenue number is disputed
    - Overlapping periods within a vertical must be forbidden
    - Admins manage them in the Partners tab
    - The admin UI supports multiple lead-price rows per vertical, each row a window with a start date and an end date
        - No overlapping time windows are allowed within a vertical
        - The end date can be empty only on the latest window — an open end means "until further notice"
    - When a vertical has no lead-price row at all, the dashboard still shows Users and lead counts, but no Projected Revenue until an Admin provides a lead price (see Dashboard below)
4. **Dashboard.**
    - The Overview shows four numbers for the selected range
        - Users: the range's unique visitor count (deduplicated across the whole range, not a sum of daily counts)
        - Unverified Leads: the sum of the daily aggregates in the range
        - Conversion Rate: Unverified Leads ÷ Users, as a %, both taken over the same range. A range with zero Users shows no rate rather than a division by zero.
        - Projected Revenue: each day's count multiplied by the lead price in effect that day, summed. A partner with no lead price for a vertical sees leads but no revenue figure (with an explanatory note) rather than a fabricated $0.
    - The chart shows Unverified Leads over time, one point per day, for the same range as the numbers
        - Daily lead counts sum to the range total, so the chart always reconciles with the Unverified Leads number above it. A conversion-rate chart cannot offer that (see Design decisions: chart scope).

## Design decisions

**Decision: source of lead counts**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — consume** `conversions:status-changed` _(chosen)_ | Carries `aff_id` and `productType` in-payload; standard `@ratehub/sdk` events consumer; no upstream changes | `affiliate` field is free-form — needs defensive parsing |
| B — consume `profiles` document events | Fires at document creation | No `aff_id` in payload; requires a profile fetch per event |
| C — query existing conversion stores at request time | No new consumer | No store is queryable by `aff_id`; couples the portal to membership-team internals; slow, unbounded queries |

- **Decision:** consume the conversions stream.
- **Rationale & tradeoff accepted:** it is the one place attribution and vertical already meet. We accept parsing a loosely-typed field and depending on `track-conversion-from-inquiry` staying the attribution point.

**Decision: pre-aggregated daily records vs. aggregate-on-read**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — consumer maintains daily aggregates _(chosen)_** | Range queries are trivial sums; dashboard latency independent of traffic; YTD/monthly free (PRD note) | A new long-running consumer to own; needs idempotency |
| B — store raw events, aggregate per request | No aggregation logic; maximal flexibility | Every dashboard view scans a growing event set; retention question from day one |
| C / do nothing — no counting | — | The dashboard has no data |

- **Decision:** daily aggregates, keyed `(aff_id, productType, torontoDay)`, upserted idempotently. `torontoDay` is a `YYYY-MM-DD` label — the Toronto calendar date of the event's `content.created` — while every stored instant remains a UTC timestamp.
- **Rationale & tradeoff accepted:** the dashboard's read patterns are all period sums, so paying the aggregation cost once at write is strictly better. Keeping `productType` (not the collapsed P&C bucket) in the key costs a few rows per day and buys the future sub-vertical tabs without reprocessing. We accept owning idempotency as the price of at-least-once delivery and replayability — implemented as the processed conversion ids stored _in_ the daily document (`$addToSet`, single-document atomicity, count derived from the set's size), chosen over a separate dedup collection because two documents can't be updated atomically without transactions and this key's cardinality (one partner × one product type × one day) keeps the id set tiny. Revisit if a vertical ever produces thousands of leads per partner per day. The Toronto day boundary is deliberate, not an oversight of "store UTC": a UTC-day bucket would misfile roughly 8 pm–midnight Toronto traffic into the next day, misalign counts against the Toronto-dated lead-price periods that revenue is computed from, and — since aggregation discards time-of-day — the boundary could never be corrected at read time.

**Decision: lead-price storage shape**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — effective-dated periods per (partner, vertical) _(chosen)_** | History preserved; supports campaigns/promotions; retroactive corrections possible | More rows and UI than a single number |
| B — one editable price field per partner-vertical | Simplest | Price changes rewrite history; past revenue silently recomputed at today's lead price |

- **Decision:** dated periods — already settled during PRD review (PRD Personal notes) and reflected in the built admin UX.
- **Rationale & tradeoff accepted:** revenue over a range is only computable if we know the lead price on each day of the range. We accept the heavier admin UI, which is already built.

**Decision: where lead prices are stored**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — standalone collection, one document per lead-price period _(chosen)_** | Clean write-path ownership (admin-only, separate from the onboarding-owned `organization` doc); per-period attribution (`createdBy`/`createdAt`) is just fields on the row, like the blocklist's `blockedBy`; matches the portal's precedent for admin-managed per-partner config (the feature-store's partner rows); history accumulates in its own collection | Overlap refusal loses single-document atomicity — needs a careful upsert or transaction; one extra (indexed, tiny) query on dashboard render |
| B — embedded in the `organization` record (`LeadPrices` map) | Prices arrive free with the org fetch; overlap check is atomic within one document; no new collection | Two writers with different auth paths (onboarding flow and admin actions) share one document; per-entry attribution is awkward in nested arrays; as promotional periods accumulate over years, the org document slowly becomes a changelog that every org fetch (auth checks, admin lists) drags along |

- **Decision:** standalone collection.
- **Rationale & tradeoff accepted:** the deciding concern is that financial history doesn't belong inside a shared identity document — embedded price periods would turn `organization` into a slowly growing changelog with mixed ownership, and money data needs the per-change attribution a row-per-period shape gives naturally. We accept implementing the overlap check without single-document atomicity and one extra query per dashboard view.

**Decision: where Projected Revenue is computed**

| Option | Pros | Cons |
| --- | --- | --- |
| **A — at read time: counts × lead-price-per-day _(chosen)_** | Self-correcting — fixing a mistyped lead price heals history; aggregates stay money-free and vertical-agnostic | Portal does a small join (days × price periods) per view |
| B — at write time: consumer stamps each day's revenue | Reads are pure sums | Freezes typos into history; consumer must know lead prices, coupling it to the portal's admin data |

- **Decision:** read time.
- **Rationale & tradeoff accepted:** the join is tiny (≤ 365 daily rows × a handful of price periods) and correctness under price corrections matters more than saving it. We accept that the portal owns the revenue math.

**Decision: chart scope**

| Option | Pros | Cons |
| --- | --- | --- |
| A — both, sequenced: leads-over-time first, conversion-rate after Users | Mock's chart ships early with no new data source | Two chart deliveries; the conversion-rate chart has the reconciliation problem described under C |
| **B — leads-over-time chart, with Conversion Rate as a number _(chosen)_** | Daily counts sum to the range total, so the chart reconciles with the Unverified Leads number; needs only the counter's daily aggregates, so it ships with the numbers and does not wait for the Users source; the PRD's intent (conversion effectiveness) is kept as a range-level number, which is the only level at which the rate is correct | Matches the original mock rather than the 2026-08-25 PRD text; the Conversion Rate number still waits for the Users source |
| C — conversion-rate over time | Matches the 2026-08-25 PRD text | **Daily rates do not compose into the range rate.** Each point's denominator is that day's unique users, but the range's Users number is deduplicated across the whole range, and the sum of daily uniques is larger than the range's uniques whenever anyone returns on a later day. A partner who reads the chart against the numbers above it sees figures that cannot be reconciled. Also blocked on the Users source, so the tab ships chartless until then |
| D — no chart in MVP | PRD allows it | Tab is bare tiles; both docs expected a chart |

- **Decision:** leads-over-time chart plus a Conversion Rate number — agreed with PM, 2026-09-02 (supersedes the 2026-08-25 decision for a conversion-rate-only chart).
- **Rationale & tradeoff accepted:** the deciding fact is arithmetic. Example: A and B visit on day 1, A visits again on day 2. Daily uniques are 2 and 1; the range's uniques are 2, not 3. A conversion-rate chart is correct per point but wrong the moment a reader totals it, and a dashboard that invites that comparison will confuse partners. Leads over time has no such trap because daily lead counts do sum to the range total. Conversion Rate stays on the dashboard as a number computed over the selected range, where both numerator and denominator are range-level and the rate is exact. We accept that the chart no longer shows conversion effectiveness directly, and that the Conversion Rate number is the one thing on the tab that waits for the Users source.

## Milestone list

1. **Lead prices are real** — Build the lead-price admin UI, allows the lead-price to be persisted; entered prices survive reload and are visible to both admins, demoable on dev.
2. **Leads are counted** — the consumer service runs on dev, processes `conversions:status-changed`, and daily aggregate records can be shown for a test affiliate id after submitting a quote on dev.
3. **Overview tab, leads and revenue** — P&C and Life tabs show Unverified Leads and Projected Revenue from real aggregates and lead prices, plus the Unverified-Leads-over-time chart from the same aggregates, with the disclaimer copy and the no-price fallback; Mortgage hidden by default.
4. **Range filter + refreshed-at** — 30/90/YTD/since-beginning filters and the "data last refreshed" timestamp.
5. **Users + Conversion Rate** — once the Users source has collected and been sanity-checked: the Users number and the Conversion Rate number (Unverified Leads ÷ Users over the selected range). Until this milestone the Overview tab shows two numbers and the chart.

## Test plan

_To be co-owned with QA; summary of intent:_

- **Unit:** lead-price-period logic (overlap refusal, open ends, Toronto day boundaries — including the DST transitions), revenue math (counts × dated prices, price changes mid-range), consumer event parsing (missing/malformed `aff_id`, unknown `productType`).
- **Integration:** consumer idempotency — the same conversion delivered twice yields one count; replay over processed events changes nothing.
- **E2E/manual:** submit a quote on deployed dev with a test partner's `aff_id` and watch it land on that partner's dashboard (local onboarding can't complete by design, so this is dev-environment work).

## Deployment plan

- The consumer service ships first and runs silently — it writes aggregates nobody reads yet, so it can bake with zero user impact and its numbers can be sanity-checked against known dev traffic before any UI exists.
- The Overview tab ships behind the portal's existing per-partner feature gating if a gradual rollout is wanted; the tab is read-only, so rollback is un-deploying the UI with no data implications.

## Open questions

| Question | Assignee | Answer |
| --- | --- | --- |
| Should we backfill numbers for existing partners? | Sijoon/Artem | **No backfill.** Users cannot be backfilled (nothing recorded visits before collection starts). Leads could be replayed from the Kafka topic, but backfilling only Leads would divide a full lead count by a partial Users count and inflate Conversion Rate for every range reaching before collection start. So neither is backfilled: all numbers begin at collection start. Consequence: "since beginning" reads as "since collection began" for partners who were already with us before this shipped, and the dashboard copy must say so. |
| Mortgage per-partner visibility — reuse the existing feature-gate machinery or a dedicated flag? | Sijoon | re-use the flag system |
| Final disclaimer copy per tab (We only have it for P&C) | Artem |  |

## Appendix

- Conversion event schema: `services/membership/track-conversion-from-inquiry/src/schema/zod_conversion_schema.ts`
- Attribution (`aff_id` attachment): `services/plat/profile/src/common/enrichRequestMetadata.js`
- Affiliates service: `services/membership/affiliates` (org ↔ affiliate id)
- Event replay tooling: `services/plat/replay-helper`
