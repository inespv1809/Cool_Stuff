---
name: decision-review
description: >
  The read layer for decision tracking. Analyzes the "Decision Log" to surface how you and Claude work together — who originates which kinds of decisions, and where you lead vs. delegate. Two scopes: an end-of-chat pass over a single project's recent entries to surface forming patterns, and a periodic cross-project pass (grouped by vertical) that produces a working-agreement directive. Trigger on "review the decisions," "what patterns are forming," "let's do the periodic review," "how have we been working together," or at end of a chat that logged several decisions. This is the read half of a pair — the decision-log skill is the record half. Do not use this to add or maintain log entries; that's decision-log.
---

# Decision Review (read layer)

Turns the Decision Log's attribution into insight about how you and Claude divide work. The log answers "why did we choose X"; this skill answers "how do we work, and where should Claude lead vs. defer."

## Source data

The **Decision Log**, related to the **Projects** registry (see `decision-log`'s SKILL.md for the architecture). Vertical comes from the rollup, so grouping by vertical is a property query — no per-page reading needed for the categorical axis. Read page bodies only when you need the reasoning behind a specific decision, not to get attribution or vertical.

## The core analysis (same at both scopes)

Code each decision on two axes:

1. **Who originated it** — from `Decided By` (You / Claude / You + Claude).
2. **What type of decision** — a common split looks like:
   - *You reliably own:* scope (what's in/out), domain facts, values & strategic goals, final content/voice.
   - *You reliably delegate:* framework & structure, analytical synthesis, filtering/exclusion logic, process & methodology.
   (Adjust this split to whatever pattern actually shows up in your log — the categories above are a starting hypothesis, not a rule.)

Surface where the axes line up and, more importantly, where they break the expected pattern — pattern-breakers are where the working relationship is shifting.

**Read `Status` as a second signal.** You-originated decisions trending `Decided` while Claude-originated ones sit `Open` means attribution tracks *who originated the thinking*, not who held authority: Claude proposes, you disposition. Note drift over time.

## Scope 1 — End-of-chat (single project, lightweight)

When a chat logged several decisions, do a short pass over just this session's rows for that project:
- Name any pattern starting to form ("you took every scope call this session; I drove structure").
- A few lines, not the full periodic analysis. Spoken observation only — write nothing back to the log here.

## Scope 2 — Periodic (cross-project, full)

Across the whole Decision Log, grouped by the rolled-up vertical:
1. Query all decisions; group by `Vertical`, then by `Decided By`.
2. Code them on both axes above.
3. Report where you lead vs. delegate, whether the split differs by vertical, and the pattern-breakers.
4. Translate into a concrete operating directive — a short "lead here / defer here / close the loop here" list Claude can act on.
5. Flag sample size honestly; a light period is provisional, not law.

### Writing the directive

If you keep a "how I want Claude to work with me" doc or page, this directive belongs there as its own subsection, appended to (not replacing) anything already there — keep the evolution visible so drift over time stays legible. If you don't have such a doc, just report the directive in chat and ask whether to save it somewhere.

## Write safety — never wipe a page

Same rule as `decision-log`: if your backend has a "replace entire content" operation, don't use it on a page with existing content. Use a targeted insert or find-and-replace instead, and fetch the page first so the edit matches exactly.

## Presenting

Tight. Lead with the finding (where you lead vs. defer), then the one or two pattern-breakers, then the directive. Skip the per-decision recount — the read, not the raw log replayed.

<!--
FILL IN / ADJUST BEFORE USE:
- The "you reliably own / delegate" category split is a starting hypothesis — replace with whatever actually shows up once you have a few dozen logged decisions.
- If you keep a Claude working-agreement doc, name it here so the skill knows where to write the periodic directive.
-->
