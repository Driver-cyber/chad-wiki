---
version: 1.0.0
last-updated: 2026-05-02
---

[← map](./map.md)

# Collaboration patterns

> [`orientation.md`](./orientation.md) covers who Chad is and how he wants
> to be worked with at the disposition level. This file covers process —
> the patterns we use whenever a piece of work has more than one move. New
> patterns get added here as they emerge. If anything below is unclear,
> return to `map.md`.

## Detect → propose → confirm

The default for any new multi-item process — sweep, scaffold, batch edit,
multi-file refactor, retrofit, cleanup pass, anything where multiple
discrete items could be acted on.

**The pattern:** describe what's there, lay out what could happen (with
options when reasonable), wait for Chad's per-item OK before applying.

**Why it's the default:** Chad is the editorial hand; Claude is the typing
layer. He wants to remain involved by default and hand over autonomy
intentionally, not have it taken. Silent bulk action — even when the right
move seems obvious — short-circuits the editorial role.

**Bulk opt-in is per-occurrence.** Recognized opt-in phrases include:
*"do them all"*, *"process them yourself"*, *"sweep all"*,
*"session-end automated"*, *"run the whole thing"*,
*"go ahead and just do it"*, or equivalent handoff language. The opt-in
applies only to that sweep; the default reverts to interactive on the next
run.

**Failure mode:** noticing that all items obviously want the same treatment
and proceeding without asking. Even when the answer seems clear, ask the
first time — the cost of one extra confirmation is small; the cost of an
unwanted batch action can be much larger.

## Measure twice, cut once — extended to multi-file work

[`orientation.md`](./orientation.md) names the principle. For multi-file or
multi-artifact changes specifically:

- Propose the file list and the outline of each before writing any of them.
- Wait for explicit sign-off on the list and outlines.
- Only then start writing.
- Within the writing, prefer one focused commit per coherent change over
  many small commits that fragment the diff.

The friction of one extra confirmation is much smaller than the cost of
throwing away three files because the second one revealed a structural
problem with the first.

## Single source, many surfaces

This wiki is the example. Substance lives once — orientation, protocols,
checklists, patterns. Each surface (project `CLAUDE.md`, Claude Code skill,
chat conversation) decides how much of that substance to load based on its
session economics.

**Implications:**

- **Link, don't copy.** When a project's `CLAUDE.md` needs to invoke the
  session-end protocol, it links to
  [`session-end-protocol.md`](./session-end-protocol.md) instead of
  duplicating the body. Updates to the canonical doc propagate everywhere.
- **Inline only with a reason.** A skill may inline a snapshot of a wiki
  doc for offline use or fetch latency, but should also expose a `--resync`
  affordance and stamp the inline copy with a "last-synced" date.
- **One change, one place.** If you find yourself editing the same paragraph
  in two files, that's a signal the paragraph belongs in the wiki and the
  files should reference it.

## Adding new patterns

When a new working pattern emerges — anything Chad would want future
sessions to default to — append it here as a new section. Bump this file's
`version` according to the wiki's semver rules (Minor for content added,
Major for meaning changed) and note the addition in the Changelog.

The bar for inclusion: the pattern is something Chad would want active by
default across projects, and would otherwise have to re-explain in every
new session.

---

## Changelog

- **1.0.0 (2026-05-02):** Initial version. Three patterns:
  detect → propose → confirm; measure twice, cut once (extended to
  multi-file work); single source, many surfaces.
