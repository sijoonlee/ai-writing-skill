# Design Decisions — workbook

One decision per pass. Assumes you've read `design-decisions-v2.md`; this is the short form for
using it, not for learning it.

---

## Step 1 · Is there anything to decide?

**The decision:** ⟨one sentence⟩

Rule out the arguments that were never trades:

- [ ] **Someone drew a line** — regulator, contract, product, security policy, our own team.
      → threshold. Met or not met, no trade. *Who owns it? When was it last true?*
- [ ] **Reality drew a line** — arithmetic, physics, an upstream schema, data never recorded.
      → found threshold. *Can it be demonstrated? If not, it's a hunch wearing arithmetic's clothes.*
- [ ] **Only one option was ever on the table** → not a decision. Name the do-nothing option.

**Options that survive the filter:** ⟨list them, including doing nothing⟩

---

## Step 2 · What tier?

> **How long does the thing I'm replacing outlive my deploy?**

| | The old version survives | Which means |
|---|---|---|
| **1** | not at all | one unit; a revert puts it back |
| **2** | until I close the migration window | a contract — two versions coexist |
| **3** | past my deploy, and past my reach | rows I can't rewrite, code I don't own |
| **X** | — *there's nothing to recover from* | never written, or written and no longer usable |

Check the three that catch people out:

- [ ] **Does this change touch a contract?** — a contract being any interface whose two sides can
      be at different versions at once. → at least tier 2, whoever owns the other side.
- [ ] **Does this change what we save?** — the records, not the code. After this ships, will rows
      land in the database that differ from the rows that would have landed without it? Renaming a
      class, moving a function: no. Choosing a different key, dropping a field, changing what a
      column means: yes.
      → if yes, **not tier 1**, however small the diff. The rows set the tier, not the code.
- [ ] **If this turns out wrong, could we get back what it dropped?** — that is what decides
      whether the decision is permanent, so it is worth two minutes.

      *First, what does it drop?* Deciding what to save also decides what not to, usually without
      anyone saying so: precision (an instant becomes a date), a field nobody has asked for yet,
      rows before a start date, the gap between *zero* and *we didn't measure*. ⟨…⟩ If genuinely
      nothing, you're done with this check.

      *Then, where else does that live?* An event log, an upstream system we could re-read, a
      backup, a vendor's copy. Not "in principle" — a source someone could actually query, and
      how long it keeps it. ⟨…⟩

      → **Nowhere = the decision is permanent. Tier X**, on top of whatever tier you already have.
      X stacks, it doesn't rank.

**Tier:** ⟨n⟩  **Tier X?** ⟨yes / no — and why⟩

If tier X, two dates are worth knowing:

- **Grace period** — nothing reads the records yet and they're cheap to discard. Ends when?
- **Expiry** — retention closes on whatever you'd rebuild from. When?

---

## Step 3 · Say why

> **Chose** ⟨this⟩ over ⟨that⟩
> **For** ⟨value⟩ — ⟨the consequence, not the value name⟩
> **Accepting** ⟨value⟩ — ⟨the consequence, and who pays it⟩
> **If wrong** — tier ⟨n⟩, ⟨what we'd have to touch⟩

One or two values per half. Use the words from the back of this sheet, not synonyms. If more than
two apply per half, it's several decisions wearing one name — split it.

Optional fifth line, when the default version of this trade would have cost something and you
paid not to let it:

> **Not accepting** ⟨value⟩ — ⟨what you did instead, and what that cost⟩

---

## Step 4 · Is it the right trade?

- [ ] **Both halves are consequences.** "Buys Enforceability" justifies nothing. What breaks, how
      often, costing what?
- [ ] **Tiers written beside each half.** Spending low to buy high is fine. The reverse needs a
      much higher bar. *(If both halves are tier 3: can either be made smaller or later? Which
      fails sooner? Which has more consumers you can't reach? All even → not ready, go find
      something out.)*
- [ ] **Bounded or compounding.** Does saying yes once make the next yes easier or harder?
- [ ] **Who pays, and are they in the room?** Convergence, Legibility, Ergonomics and Operability
      are all paid by people who aren't.

---

## Reference · the fourteen values

| | Asks |
|---|---|
| **Fidelity** | Is this how the work actually is? |
| **Re-derivability** | When the rule changes, can we recompute history? |
| **Optionality** | When the requirement changes, what will it cost to say yes? |
| **Headroom** | Does cost grow with the load, or faster than it? |
| **Enforceability** | What stops someone getting this wrong — code, or memory? |
| **Isolation** | When this breaks or changes, how much else moves? |
| **Reversibility** | Did we *pay* to make being wrong cheap? |
| **Legibility** | How fast can someone form a *correct* mental model? |
| **Ergonomics** | How much must they do, and how much know first? |
| **Convergence** | Will the next service reuse this, or add another way? |
| **Operability** | Can I see it, would I find out if it were wrong, can I fix it alone? |
| **Economy** | How much work is this, to build and to keep alive? |
| **Speed** | How soon can we ship it? |
| **Data Minimization** | Why did we keep this, and why for this long? |

## Reference · what gets said, and what it means

| Said | Means |
|---|---|
| "it's cleaner", "simpler" | Legibility or Convergence — ask which |
| "more flexible", "future-proof" | Optionality |
| "that's not how it actually works" | Fidelity |
| "safer" | Enforceability if the compiler catches it, Operability if you'd see it fail |
| "it's standard", "everyone does it this way" | Convergence |
| "nicer API", "less boilerplate" | Ergonomics |
| "it scales" | Headroom — then say what *n* is |
| "fast enough", "under 200ms" | a threshold. Filter it, don't trade it |
| "we can always change it later" | a tier claim, not a reason. Go back to step 2 |
| "we can always replay the events" | tier 3 with an expiry date. Ask what the retention window is |
| "we'll just default them to zero" | a tier X repair that creates a second tier X |
| "it's quicker to build" | Economy — a real reason, so say it plainly |
| "I just don't like X" | unnamed. Convergence, or a value not yet on the list |
