---
name: busy-reader-friendly-erd
description: Write or restructure an Engineering Requirements Document (ERD) to a line budget, so a busy reader can approve it without scrolling past what they need. Diagrams and tables replace prose rather than accompany it; mechanism detail goes to the SDD. Use when the user asks to write, draft, rewrite, or "make readable" an ERD, design doc, or SDD, or when a reviewer says a doc is too long, too dense, or has no diagrams.
---

# Busy-reader-friendly ERD

A busy reader's cost is scroll length. They decide whether to start by
how long the document looks, and they stop when it outlasts their
attention. Word count is not the measure — lines are.

So this skill is a budget before it is a style. Every rule below either
spends lines or saves them, and the budget is the thing that must hold.

Keep the company template's section list and order. Change what goes
inside each section, not the skeleton.

## Template

The company ERD template lives next to this file as
[erd-template.md](erd-template.md). Read it before writing or
restructuring any ERD. It defines the section list and order, the
Metadata table, the decision option table, and the per-section guidance.

## Set the budget first

Pick the line budget before writing anything, from the size of the work:

| Work | Budget |
| --- | --- |
| Small, well-understood change | 60 lines |
| Normal project, a few milestones | 120 lines |
| Complex or risky, spun-out SDD | 200 lines |

Count every line: headers, table rows, blank lines, diagram source. The
budget is the whole document, counted with `wc -l` on the source, not
estimated. Wrap a paragraph as one source line; the paragraph limit
below is about what the reader sees, the budget is about the file.

If the draft exceeds the budget, cut. Do not fold, and do not shrink
the font of the problem.

A revision that comes out longer than the document it replaced has
failed, whatever else it improved.

## What a line costs

Structure is not free. Under a line budget the usual advice inverts:

| Element | Lines |
| --- | --- |
| Sentence of prose | 1–2 |
| Five-row table | 7 |
| Mermaid diagram | 8–14 |
| `<details>` fold | 4, before any content |
| Bullet | 1 each, plus a blank line around the list |

A table earns its lines when three or more rows share the same
attributes. Two rows is a sentence. A diagram earns its lines when it
replaces a paragraph you would otherwise have to write; a diagram that
illustrates prose already on the page costs double.

## Route detail out of the document

Most length is material that belongs somewhere else. Before cutting for
style, move by destination:

- **The SDD** takes per-field behavior, edge cases, failure handling,
  timezone and DST notes, and anything a reader consults rather than
  reads. Link it from Metadata.
- **Jira** takes task breakdowns and anything with an assignee.
- **An ADR** takes a long-lived decision that outlives this project.
- **Nowhere** takes the history of how the plan changed, restated PRD
  content, and rationale for choices nobody will contest.

Do not use `<details>` to keep material you could not bring yourself to
cut. A fold hides text from the page but not from the document, and it
removes the pressure that makes the rest of the writing good. Reserve
folds for the rare case where a reviewer genuinely needs a lookup table
in place — at most one per document.

## Process: write in this order

1. **Budget.** Pick it from the table above. Write it at the top of your
   draft and delete it before publishing.
2. **Header list.** Every header a claim, not a topic. "Leads are
   counted once per Toronto day by a new consumer", not "Lead counting".
   Read the list straight through: it must summarise the document. Fix
   gaps here.
3. **The one diagram.** Draw the system or data-flow diagram that the
   whole document refers back to. Put example values on the arrows.
   Reuse its node names everywhere else.
4. **Tables.** Only where three or more rows share attributes.
5. **Prose last, in the gaps.** Prose carries what the picture cannot:
   the rule, the constraint, the reason. It never restates the picture.
6. **Count lines. Cut to budget. Run the checklist.**

## Open with the plan, in prose

Under Metadata, before the first section, write one paragraph of four to
six sentences carrying the argument end to end: what the work is, the
piece that is genuinely risky, and what ships first. A reader who stops
there can approve or object.

Write it as a chain, each step earning the next — not a list of labelled
slots. Slots demand filling, and filling them is what turns a summary
into a duplicate of the document. The paragraph is allowed to leave
things out; it is making a case, not taking inventory.

## Substitution, not accumulation

Every visual replaces the prose it would have needed. It never joins it.

- The diagram replaces the paragraph describing the components.
- The table replaces the list, and the list replaces the paragraph.
- Three rows of toy data replace the argument about arithmetic or an
  edge case. Show the rows, caption them in one line, and stop.
- The option table replaces the prose weighing the options.

After adding any visual, delete the prose it made redundant. If nothing
became redundant, the visual is decoration — cut it.

## Decisions

Most decisions are settled and need one row, not a section. Put them in
a single table: decision, chose, because, accepted.

Expand only the decisions a reviewer might actually fight. Those keep
the option table from [erd-template.md](erd-template.md) and one line
each of **Chose**, **Because**, **We accept**. One or two per document.

## Terms

A Terms table earns its lines only for terms that carry real ambiguity —
a partner-facing name and an engineering name for the same thing, or a
word this document uses more narrowly than the reader expects. Anything
defined by the formula where it is used does not need a row.

Then use one name for the rest of the document. Never alternate.

## Hard limits

| Thing | Limit |
| --- | --- |
| Bullet nesting | 2 levels |
| Items per list | 5 |
| Paragraph | 3 lines, one idea — rendered lines, not source lines |
| Rationale above a link | 1 sentence |
| Repeats of a fact | 1 — but re-show anything short rather than making the reader scroll back; link only what would cost more than a line |
| Folds per document | 1 |

## Diagrams

- Mermaid fenced blocks. Flowchart LR for data flow, sequence diagrams
  for request/response, simple boxes for UI sketches.
- Label arrows with the event or payload name and one example value.
- Make invented example values visibly synthetic: `af_123`, `$PRICE`,
  `2026-01-01`. When the source document gives no real value, a
  plausible-looking one is a fabrication a reviewer may read as fact.
  Never invent a money figure — placeholder it.
- One diagram under twelve nodes. If it needs more, the document needs a
  smaller scope, not a second diagram.
- Reuse node names across the document.
- Leave a blank line before and after a fence, and put nothing else on
  the fence line. Never place an HTML comment or raw HTML directly above
  a fence: some previewers hand the renderer empty text and it fails
  with "No diagram type detected".
- Confluence needs the Mermaid macro or an exported image. Say so once,
  in the Appendix.

## Prose rules

- Lead every paragraph with its claim.
- **Verbs over abstract nouns.** "The consumer stores processed ids",
  not "idempotency is ensured". Verbs force you to name the object.
- No bare "this", "that", "it" as a subject. Attach a noun.
- Name a number's unit and what it is compared against.
- Matter-of-fact tone. No "note that", no "importantly", no hedging
  adverbs that carry no information.

## Pre-finish checklist

Fix silently; do not report the audit.

1. Count the lines. Are you under budget?
2. Is the document shorter than the one it replaced? If not, it failed.
3. Read only the headers. Do they summarise the document?
4. Read only the opening paragraph. Can a reviewer approve or object?
5. Every diagram and table: what prose did it let you delete? If none,
   cut it.
6. Every section: could a reviewer approve the plan without it? If yes,
   route it to the SDD, to Jira, or to nowhere.
7. Any fold beyond the first? Cut or route it.
8. Any paragraph over three lines, list over five, nesting over two?
9. Any term with two names in use? Pick one.
10. Any argument in prose that three rows of toy data would settle?
11. Every mermaid diagram: under twelve nodes, arrows labelled, node
    names consistent.

## What not to do

- Do not keep content by folding it. Route it or cut it.
- Do not rewrite the section names from [erd-template.md](erd-template.md).
  Add claim subheaders inside.
- Do not add a diagram or table to a section that already reads clearly.
  Structure is a substitute for prose, not a supplement to it.
- Do not restate the PRD. Link it.
- Do not add a summary at the end. The opening paragraph is the summary.
