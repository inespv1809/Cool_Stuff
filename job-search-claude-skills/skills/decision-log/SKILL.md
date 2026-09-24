---
name: decision-log
description: >
  The record layer for decision tracking. Logs decisions to a "Decision Log" database, one row per decision, with reasoning in each page's body and a relation to a "Projects" registry. Use whenever the user is working on a multi-step project and has opted into decision tracking. Trigger on "log this decision," "add that to the log," "track this," or an accepted choice on a project being tracked. This is the record half of a pair — the decision-review skill is the read half that analyzes the log later. Do not use this to analyze or summarize patterns; that's decision-review.
---

# Decision Log (record layer)

Captures decisions so future-you can reconstruct *why* a choice was made, and so decision-review can later mine *how* you and Claude divide the work. One decision per row; the table stays categorical (short chips only); all prose lives in the page body.

## Architecture (pick a backend, then don't reinvent it)

Two linked structures under `<YOUR_WORKSPACE>` (a Notion workspace, a database, or a spreadsheet — whatever you use to track projects):

- **Projects** — registry of every project and sub-project you want decisions tracked against. Suggested properties: `Name`, `Type` (Project / Sub-project), `Vertical` (a category — e.g. Personal / Professional / Academic / Clubs / Creative, or your own set), `Parent` (relation to a parent project, if sub-projects apply).
- **Decision Log** — one row per decision. Suggested properties: `Decision` (title), `Project` (relation to the registry), `Decided By` (You / Claude / You + Claude), `Status` (Decided / Open / Superseded), `Date`, and `Vertical` (rolled up from the related project — never set by hand).

If using Notion, locate both by name via search rather than hardcoding IDs (they can move/rename). Confirm you've got the right ones by checking the schema matches the above before writing.

## The one rule that isn't obvious: the vertical rule

Every project row carries a `Vertical` **except** a parent that has sub-projects, which may leave it blank (its children carry the real signal). Enforcement when touching the registry:

- Creating a **leaf** project (no children): always set a `Vertical`.
- Creating a **sub-project**: set `Type` = Sub-project, set its `Parent` relation, set its `Vertical`.
- A parent that has sub-projects may be blank. **Never strip an existing vertical** from a project just because it gained a child — leave it as-is. Blank is allowed for parents, not forced.

## Relationship to a standing preference

If the user has a standing preference that asks, at the start of a multi-step project, whether to track decisions: **this skill executes once they say yes — it doesn't re-ask.** If they have a `decision_log_methodology.md` uploaded, follow it instead of this default.

## Workflow

1. **Identify the project** the decision belongs to — the specific one, including sub-projects. A decision about a sub-project relates to that sub-project, not its parent; granular linking still rolls up to the parent's vertical for review.

2. **Ensure the project exists in the registry.** Search the Projects registry. If the project (or sub-project) isn't there, create it first, applying the vertical rule above. If it's a new sub-project, set its `Parent`. This keeps the registry populated as a byproduct of logging — never a separate chore.

3. **Create one Decision Log row per decision.** Set `Decision` (a short label — one line, never reasoning), `Project` (relation), `Decided By`, `Status`, `Date`. Put the **reasoning, options considered, and any narrative in the page body**, not in properties. `Vertical` fills itself via rollup — don't set it.

4. **Attribution is the signal, so be honest.** `Decided By` records who *originated the thinking*: You (you directed/originated it), Claude (Claude proposed it), or You + Claude (genuinely joint). When Claude proposes and you accept, that's `Claude` originated — and it often sits `Open` until you disposition it. Don't mark something `Decided` just because it was proposed.

## One decision per row — don't bundle

The most important integrity rule: **when several distinct decisions happen in one exchange, give each its own row — never collapse them into one.** Bundling corrupts attribution in two directions at once:

- It **over-credits joint work**: fold several you-originated calls plus one genuinely-joint call into a single row, and the row inherits a "You + Claude" tag that misattributes the content.
- It **buries genuinely-joint decisions**: the one call that was truly joint vanishes as its own entry, smeared into a neighbor.

So: separate the intent from the method, the hook from the tone from the framing — if they're distinct choices, they're distinct rows, even when they occurred in the same message. A row should carry exactly one decision and therefore exactly one clean attribution. If you can't give a row a single unambiguous `Decided By`, it's probably two decisions wearing one title — split it.

And per the attribution definition: **accepting a proposal is not originating it.** "Yes, let's" in response to a Claude proposal is `Claude` originated, not `You + Claude`. Reserve `You + Claude` for genuine co-origination, never as a hedge for "we talked about it."

## Maintaining across the session

- Log decisions as they're made, not one batch dump at the end (though do a catch-all sweep at end of chat).
- Supersede, don't delete: when a decision is replaced, set the old row's `Status` to `Superseded` and add the new one. The trail stays intact.
- If the user has a "show plan before batch operations" preference, show the rows you intend to create before writing more than one or two.

## Body template (keep it skimmable)

```
## Reasoning
<why this was chosen — the asset; capture even when terse>

## Options considered
<alternatives and why they lost, when relevant>
```

## Write safety — never wipe a page

If your backend supports a "replace entire content" operation, treat it as prohibited on any page that already has content — it destroys anything not in the new string.

- Append an entry with an insert/append operation.
- Edit one section with a targeted find-and-replace.
- Fetch the page first so a targeted edit matches exactly.
- If a full-page rewrite ever genuinely seems necessary, stop and confirm with the user first, and carry every piece of existing content forward verbatim in the new string.

Logging new rows and superseding (a status change) are both non-destructive by nature — this rule mainly matters when editing a page body (a reasoning note, a legacy prose log).

## End-of-chat sweep

When the chat is wrapping up, scan for decisions made but not yet logged and offer to add them as a short confirm-list — don't write silently. This is a record action only; surfacing *patterns* is decision-review's job.

<!--
FILL IN BEFORE USE:
- <YOUR_WORKSPACE> — where your Projects registry and Decision Log live (a Notion workspace, an Airtable base, a spreadsheet, etc.)
- Adjust the `Vertical` category list to whatever groupings make sense for you, or drop the field entirely if you don't need it.
-->
