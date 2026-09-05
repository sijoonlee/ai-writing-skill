---
name: busy-reader-friendly-erd
description: Write or restructure an Engineering Requirements Document (ERD) so a busy reader can follow it on one pass and re-enter after looking away, using diagrams, tables, and a first-screen summary instead of dense prose. Use when the user asks to write, draft, rewrite, or "make readable" an ERD, design doc, or SDD, or when a reviewer says a doc is too long, too dense, or has no diagrams.
---

# Busy-reader-friendly ERD

The reader is busy, interrupted, and skims. The document must put everything
on the screen, because anything not on the screen is forgotten. Word count is not the problem.
Load-bearing prose is the problem: sentences the reader has to hold in their
head while reading the next one.

Keep the company template's section list and order. Change what goes inside
each section, not the skeleton. Reviewers know where to look.

## Template

The company ERD template lives next to this file as
[erd-template.md](erd-template.md). Read it before writing or restructuring
any ERD. It defines the section list and order, the Metadata table, the
decision option table, and the per-section guidance that every rule below
assumes. When a rule here says "the template", it means that file.

## Five facts that drive every rule

1. Working memory is small. Never ask the reader to remember something from
   an earlier section. Show it again, or link to it.
2. Pictures survive a lapse; paragraphs do not. A reader who looks away and
   comes back can re-enter at a diagram or table. They restart a paragraph.
3. Starting is the hardest step. The first screen must give the whole plan.
4. Nesting is memory. Every indent level is one more thing held in the head.
5. Rationale is optional reading for most readers. Hide it behind a fold.

## Process: write in this order

1. **Header list first.** Write every header and subheader as a claim, not a
   topic. "Leads are counted once per day by a new consumer", not "Lead
   counting". Read the list straight through. It must read as a summary of the
   whole document. Fix gaps here, before any prose.
2. **Diagram set second.** Pick one or two diagram families and reuse them:
   a system/data-flow diagram, and a simplified UI sketch if there is a UI.
   Draw every diagram before writing prose. Put example data on the arrows.
3. **Tables third.** Anything with repeated attributes is a table: metrics
   (name, formula, source, milestone), data models, decision options,
   milestones, open questions, toy-data examples.
4. **Prose last.** Prose explains the picture or table above it. It never
   carries a fact the picture could carry.
5. **Checklist.** Run the pre-finish checklist at the bottom. Fix silently.

## Layout rules

### First screen: TL;DR

Directly under Metadata, add a **TL;DR** section of at most five bullets,
each starting with a bold label: **What we build**, **Pieces**, **Risky
decision**, **Ships first**, **Still open**. One or two short sentences per
bullet. A reader who stops here knows the plan.

Use a bullet list, not a blockquote and not consecutive lines. Markdown
joins consecutive lines into one paragraph, so the labels run together.

### Every major section opens with a visual

The first element after a section header is a diagram, a table, or a boxed
callout. Prose comes after. If a section has nothing to draw or tabulate,
open with a one-sentence claim in bold.

### Hard limits

| Thing | Limit |
| --- | --- |
| Bullet nesting | 2 levels |
| Items per list | 5 (split into "now / later" or "must / nice" beyond that) |
| Paragraph | 3 lines, one idea |
| Rationale above the fold | 1 sentence: "Chose X because Y. We accept Z." |
| Repeats of a fact | 1; afterwards link to the first occurrence |

### Progressive disclosure

Long rationale, edge cases, timezone and DST notes, failure handling, and
history of how a decision changed go inside a collapsible block:

```html
<details>
<summary>Why daily rates do not add up to the range rate</summary>

...toy-data table and explanation...

</details>
```

The summary line is itself a claim. The reader should be able to skip every
fold and still understand the plan.

### Toy data over argument

When a rule is justified by arithmetic or by an edge case, show three rows of
example data in a table with a one-line caption. Do not argue it in prose.

Bad: "the sum of daily uniques is larger than the range's uniques whenever
anyone returns on a later day."

Good:

| Day | Visitors | Daily uniques |
| --- | --- | --- |
| 1 | A, B | 2 |
| 2 | A | 1 |
| Range | A, B | **2**, not 3 |

### Decisions

Each decision keeps the option table (pros / cons) from
[erd-template.md](erd-template.md). Under it:

- **Chose:** one line.
- **Because:** one line.
- **We accept:** one line.
- `<details>` for everything else.

### Data models

Show a data model as a code block with one field per line and an inline
comment per field. Not as a sentence describing the fields.

### Vocabulary

Define each term once, in a **Terms** table near the top (partner-facing
name, engineering name, one-line meaning). After that, use one name only.
Never switch between the two names in the same document.

## Diagrams

- Use mermaid fenced blocks (```mermaid). Flowchart LR for data flow,
  sequence diagrams for request/response, simple boxes for UI sketches.
- Label arrows with the event or payload name and one example value.
- Keep one diagram under twelve nodes. Split otherwise.
- Reuse the same node names across every diagram in the doc.
- Always leave a blank line before and after a mermaid fence, and put
  nothing else on the fence line. Never place an HTML comment or any raw
  HTML directly above a fence: some markdown previewers hand the mermaid
  renderer empty text and it fails with "No diagram type detected".
- Note for Confluence: mermaid needs the Mermaid macro or an exported image.
  Say so once, in the Appendix, not next to each diagram.

## Prose rules

- Lead every paragraph with its claim.
- **Verbs over abstract nouns.** "Implementing the retry logic slowed
  startup," not "the implementation caused degradation." Verbs force you to
  name the object; nouns let you hide it. Use the noun only when the process
  itself is the topic, or when the previous sentence already pinned it down.
  ERD example: "the consumer stores processed ids", not "idempotency is
  ensured".
- No bare "this", "that", "it" as a subject. Attach a noun.
- Name a number's unit and what it is compared against.
- Matter-of-fact tone. No "note that", no "importantly", no hedging adverbs
  that carry no information.

## Pre-finish checklist

Check every item. Fix silently; do not report the audit.

1. Read only the headers. Do they summarise the document?
2. Read only the TL;DR. Does the reader know the plan?
3. Does every major section open with a diagram, table, or bold claim?
4. Any bullet deeper than two levels? Flatten or tabulate.
5. Any list longer than five? Split.
6. Any paragraph longer than three lines? Split or move to a fold.
7. Any decision with more than one sentence of rationale above the fold?
8. Any fact stated twice? Keep the first, link the second.
9. Any term with two names in use? Pick one.
10. Any argument made in prose that three rows of toy data would make?
11. Every mermaid diagram: under twelve nodes, arrows labelled, node names
    consistent with the other diagrams.

## What not to do

- Do not delete content to hit limits. Move it behind a fold. The document
  must still answer every question the original answered.
- Do not rewrite the section names from [erd-template.md](erd-template.md).
  Add claim subheaders inside.
- Do not replace tables with prose to "make it flow". Flow is the enemy here.
- Do not add a summary at the end. The TL;DR at the top is the summary.
