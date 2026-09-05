# Design Decisions — Worked Example 2

Client-computed JSON diffs on save. A companion to `design-decisions.md`, in the same form as
`design-decisions-worked-example1.md`: one ordinary-looking choice, followed down to the part
that cannot be undone.

---

## The situation

A web application and a back-end server share a data model. When the user saves, the browser
computes a JSON diff between the record it loaded and the record on screen, and sends the diff.
The server applies it to the row.

It reads as a payload optimisation — a Tier 3 detail about how one endpoint is shaped.

## Tier, and what it turns on

Run the deletion test. *What's the smallest unit we could delete outright, and what would we
have to touch?*

Deleting the diff mechanism touches every client save path, the server's apply path, and
anything that has stored a diff. The first two are code. The third decides everything, and it
is the question nobody asks — see below.

But the tier is not set by the mechanism. It is set by who the consumers are.

### The wire format was always Tier 1

Tier 1 includes published contracts — external consumers you cannot coordinate. Teams exclude
the browser from that list because both ends are in-house, which is a fact about org chart and
deploy pipeline, not about coupling.

Deployed JavaScript is cached. Tabs stay open for days. A laptop wakes on Tuesday holding
Friday's bundle. You cannot upgrade every client at once, so **every version of the client you
have ever shipped is a consumer you do not control**. The browser is not internal; it is the
least coordinated consumer you have.

Nothing was promoted here. The contract was Tier 1 from the first deploy, and was being managed
as if it were Tier 3.

## What diffs weld together

A diff is expressed as paths into the document: `/customer/addresses/2/postcode`. So the wire
protocol now contains the schema's shape.

Rename a field, change the nesting, reorder an array, and every in-flight diff from a stale
client addresses a location that no longer exists. The data model and the wire contract can no
longer move independently.

This is the same shape as an ORM welded to a schema, with one difference that makes it worse.
There, a Tier 2 decision was bound to a Tier 1 asset, and you could always sacrifice the
framework. Here **both sides are Tier 1**, so there is no expendable half to give up.

## Three failures that damage data instead of breaking loudly

**Diffs are not idempotent.** A full-record write applied twice is the same write. "Remove the
array element at index 2" applied twice removes two elements. Retries, double submits and
at-least-once delivery all corrupt quietly.

**Conflicts become invisible.** A diff computed against version N and applied to version N+2
produces a record that is structurally valid and semantically wrong. A full-record write with
an etag fails loudly instead. Diffs detect this only if the client also sends the base version,
which is the part implementations leave out.

**The server cannot check invariants.** It sees `set /status to approved` without the record
around it, so validation and authorization become path-based — a permission model expressed
over JSON paths, which is the noun-shaped permission problem in its least tractable form.

Each of these is a Tier 1 failure in disguise: the symptom is bad rows, and you cannot un-write
a row.

## The fork: are the diffs stored?

This decides how much of the damage is recoverable, and it is worth answering before anything
else.

**Transient.** If diffs exist only in flight, the format is a wire contract. Painful to change,
bounded by cache TTL, effectively Tier 2 — you can support both shapes for a release window and
retire the old one on a threshold.

**Persisted.** If a diff is written anywhere — an audit trail, an event stream, an undo history
— then the diffs are **nouns**. Path semantics are frozen permanently, every future schema
change has to keep old paths interpretable, and no migration recovers it, because the records
those paths described are gone. This is the version you cannot get out of.

Same mechanism, two tiers, decided entirely by whether someone once wrote a diff to disk to
make debugging easier.

## Commands instead of diffs

Send operations named by intent rather than by location: `approve`, `add-address`,
`change-postcode`.

Intent survives refactoring; paths do not. A command carries enough context for the server to
validate the whole record against its invariants, it is idempotent if you give it an id, and it
mentions nothing about the document's shape — so the schema can be reshaped freely without
invalidating anything a client already holds.

It also stores well. A persisted command is a fact about what someone did, which stays true
however the model changes. A persisted diff is a fact about where bytes used to live.

## The record

As shipped:

> **Chose** client-computed structural diffs on save
> **For** Economy — payload proportional to what changed — and Isolation of concurrent edits,
> since two users touching different fields do not clobber each other
> **Accepting** Enforceability, because the server can no longer validate a whole record, and
> Optionality on the data model, because the wire format now encodes the document's shape
> **If wrong** — every client save path, the server apply path, and any stored diffs. Bounded
> if diffs are transient; unbounded if they are persisted

Written out, the asymmetry is the finding. What was bought is bandwidth and merge behaviour,
both recoverable. What was paid is the ability to change the schema, which is not. That is
spending the irreversible to buy the reversible, and it needs a far higher bar than it got.

The alternative:

> **Chose** intent-named commands over structural diffs
> **For** Optionality — the wire format stops encoding the schema, so the model can be reshaped
> without breaking clients that are already deployed — and Enforceability, since the server
> validates whole records again
> **Accepting** Economy — more endpoints or a larger command surface to define and maintain —
> and some Ergonomics, since the client must name what it did rather than diffing two objects
> **If wrong** — the command handlers. A module.

## If it is already in production

You cannot recall the clients. Everything below assumes old versions keep arriving for at least
one cache lifetime.

### The one-hour version

**Require a base version on every diff, and reject mismatches.** One field on the request, one
comparison on the server, a 409 when it fails.

This changes nothing about the coupling and does not need a migration. What it does is convert
silent corruption into a loud conflict — and silent corruption is the part you cannot detect
afterwards, because a wrongly-applied diff leaves a record that looks fine.

Do this first, whatever else you decide.

### The sequence

1. **Add the version check.** Loud conflicts from here on.
2. **Add a command endpoint alongside the diff endpoint.** The old one keeps working, untouched,
   so nothing deployed breaks.
3. **Have the server validate the whole record after applying a diff**, not just the paths the
   diff touched. Diffs stop being able to produce states the model forbids.
4. **Move the client to commands**, one operation at a time, starting with the ones whose paths
   you most want to reshape.
5. **Watch the diff endpoint's traffic decay.** Retire it on a threshold, not a date — cached
   bundles keep calling it well past any deploy.
6. **Freeze the schema shape until then.** While stale clients can still send paths, a rename is
   a breaking change to a consumer you cannot reach.

Steps 1–3 are additive and reversible. Step 4 is the real work, and it is the first one that
touches anything you cannot roll back cheaply.

### If diffs were persisted

Stop writing new ones. Then record a schema version alongside every stored diff from here on,
so future readers know which shape the paths referred to — that does not fix the existing rows,
but it stops the pile growing.

The old diffs stay uninterpretable past the next reshape. Do not synthesize replacements for
them: a reconstructed diff looks like a fact and is not one, and someone will build an audit
report on it. Record the cutover date, accept the seam, and treat what came before as opaque.
