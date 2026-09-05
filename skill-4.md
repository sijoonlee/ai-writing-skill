---
name: concrete-writing
description: Open with the argument's whole line, then write prose the reader can picture.
disable-model-invocation: true
---

# Concrete writing

Sound prose is the floor, not the goal. A model's default output is
already plain, orderly, and grammatical — and recognizable as
nobody's. The work is making it seeable and specific.

## Give the reader the whole line first

Before the first section, walk the argument end to end in a short
paragraph. Not a list of topics — the actual chain, each step
earning the next:

> The outage wasn't the database falling over; the database fell
> over because retries stacked up behind a change nobody flagged as
> risky. The review process couldn't have caught it, because the
> risk lived in the interaction, not the diff. So the fix is a load
> test on the path, not a stricter reviewer.

A reader who stops there still leaves with the argument. A reader
who continues knows where every section sits in it, which is what
lets the sections relax into prose.

Write this paragraph before the draft, the way you'd draft an
outline — if the chain won't connect, the argument has a gap, and
you've found it before spending words on it. Then check it again
against the finished piece.

Two ways it goes wrong. It becomes a table of contents ("first I
cover X, then Y") — no claims, so nothing survives being read
alone. Or it announces itself ("this piece argues that…") instead of
just arguing. Say the thing.

Headers stay useful as landmarks. They just no longer have to carry
the argument alone, so a plain one is fine where a claim would be
clumsy.

## Give every parallel item its own landmark

Headers do two jobs. Carrying the argument is the paragraph's job
now. The other job is retrieval: letting someone who read this last
week find the one part they need without rereading.

So wherever a section holds a run of parallel items — three failure
modes, four options, six checks — each item gets its own visible
handle. A subhead, or a bolded lead-in on its own line. Not six
paragraphs under one header, which is invisible to anyone scanning:
they have to read the whole section to learn it doesn't hold what
they wanted.

The handle names the item in the item's own terms — **The cache
never expires**, not **Failure mode two**. A number tells the reader
where they are in your list, which is a fact about your writing, not
about their problem.

## Assume the reader shares nothing

You are not in a conversation. The reader has no idea what prompted
this, can't ask a question, and won't furrow their brow where you
lost them. Everything they need is on the page or it doesn't exist.

Two habits follow:

- **Name the occasion.** Say what this is about and why it's in
  front of them, in the opening. "This" and "the issue" and "as
  discussed" work in speech because both people were there.
- **Spell out the basics you'd be embarrassed to spell out.** What
  you skip is what you know best. That's the curse of knowledge: the
  more familiar an idea, the less it feels like it needs saying.

When the stakes are high, the real check is a person who doesn't
already know. Nothing you do alone substitutes for that.

## Let the reader form a picture

Prefer the word that projects an image. A rabbit, not a stimulus. A
retry storm that hammers the database, not a degradation event.

Nobody can picture a paradigm, a framework, a level, a perspective,
or a concept. These words are useful *inside* a field — one word
standing in for a body of knowledge everyone present already has.
Outside it, they buy nothing and cost the image.

When you must use an abstraction, land it immediately: the
abstraction, then the thing you can see. "Semantic drift — nobody
breaks a fast at breakfast anymore."

## Pair every generalization with an example

A generalization erases the detail that made it true. An example
without a generalization leaves the reader asking what the point
was. Neither works alone.

So write in pairs. State the claim, then show one case, close enough
together that the reader doesn't have to hold the claim in mind
while waiting. One good example usually beats three; three become a
list the reader skims.

The example has to be specific enough to be checkable. "For
instance, in certain systems, performance may suffer" is a
generalization wearing an example's clothes.

## Cut until it hurts, then check

Impose a real limit — a word count, a screen, a page — and make the
piece fit. Cutting doesn't just shorten; it forces the concrete word
over the woolly phrase, because the woolly phrase costs more
syllables and says less.

Summarizing something? Start at a fifth of the source and adjust
from there. Much longer and the summary competes with the original
instead of replacing it. A fifth leaves room for the qualifications
but not the color, which is the right trade: drop the vivid aside,
keep the caveat that changes how a result should be read.

Cutting also invents numbers. A crisp figure is short and sounds
authoritative, so it's what a tight budget reaches for. Every
number, name, and date has to be traceable to the source you're
writing from — point at the line. Where the source is vague, stay
vague: "about half" is honest, "47 percent" is fabrication.
Compression may drop a fact. It may never sharpen one.

The same pressure invents phrasing. A memorable line is short, so
cutting recruits it — but when you're summarizing a person, a
sentence they never said is theirs now. Freshness applies to your
prose, not to theirs.

Every word is work for the reader. But brevity is not the only
rule: a piece so compressed that one lapse in attention loses the
reader has traded their time for yours.

## Read it aloud

If you stumble saying it, the reader stumbles reading it. Silent
reading still runs the sound.

Listen for the rhythm — prose has beats, and a sentence that fights
them slows the reader down for no reason. Listen for pileups of
hissing syllables and swap a synonym. A little alliteration makes a
line roll; enough to notice makes it look staged.

## Hunt the phrase that isn't already in the language

The failure mode of generated prose is blandness, and it comes from
averaging. Every stock phrase available to you is available because
thousands of people used it first.

Practical version: when a phrase arrives fully formed, suspect it.
"Robust solution," "at the end of the day," "it's not just X, it's
Y," "a delicate balance." Ask what you actually meant and say that
instead.

The child's-explanation test helps. A kid with no jargon to reach
for says smoke is fire vapor — fresh because nothing pre-made was
available. Ask what you'd say if the abstraction didn't exist.

Freshness is not decoration. Humor, persuasion, and memorability all
depend on the reader not having heard it before.

## Before finishing

1. Read the opening paragraph alone. Does it still argue, and does
   it match what the piece became?
2. Can the reader picture the main claim? Point at the words that do
   the picturing.
3. Does every generalization have its example nearby, and every
   example its point?
4. Is every result's qualification in the same breath as the result?
5. Does every run of parallel items have per-item handles a scanner
   can land on?
6. Every number, name, and date — can you point at the line in the
   source it came from?
7. Read it aloud. Fix what you stumble on.
8. Any phrase you've read a hundred times? Replace or cut it.

Fix silently. Don't report the audit.

## Scope

Prose meant to be read by someone who wasn't in the room. Reference
docs, code, and quoted material are exempt.
