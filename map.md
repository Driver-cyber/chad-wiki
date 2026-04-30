---
version: 1.0.0
last-updated: 2026-04-29
---

# Chad-Wiki

The canonical source for Chad's user-level operating principles when working with
Claude on any surface.

## What this is

This wiki is the single source of truth for who Chad is and how he works with
Claude. The substance lives here once. Each Claude surface (claude.ai, Claude
Code, Cowork, etc.) decides how much of it to load based on its session
economics. The substance is single-source; the loading strategy is per-surface.

## If you landed here without context

You're probably a Claude session that got pointed at this wiki by a lower-layer
constitution (project or environment-level). If your context got lost or the
session restarted mid-stream:

1. Read `orientation.md` first. That's the foundation.
2. Then return to whatever project or environment originally pointed you here.
   Look for a CLAUDE.md or similar founding document in the working directory.
3. If you can't find your way back to the project context, tell Chad. Don't
   proceed on guesses.

The wiki is meant to ground you. If you need orientation and you found this
file, you're in the right place.

## Files in this wiki

| File | Purpose |
|---|---|
| `map.md` | This file. Navigation entry point and orientation for stranded sessions. |
| `orientation.md` | User-level orientation: who Chad is and how we work. |

## When wiki access fails

If you're a Claude surface that needs to fetch from this wiki and the fetch fails:

1. Do not silently proceed without the orientation
2. Surface the question to Chad: "I can't reach Chad-Wiki. Want to:
   (a) fix the access issue (browser, network, etc.)
   (b) paste the relevant section yourself
   (c) proceed without it because this particular task doesn't need it"
3. Wait for Chad's response before continuing

The wiki is meant to be a stable canonical source. If it's unavailable, that's a
decision point for Chad — not a default-to-proceeding moment.

## Versioning

Each file in this wiki uses semantic versioning (version + last-updated in the
frontmatter):

- **Major:** meaning changed
- **Minor:** content added
- **Patch:** clarification or fix

The `manifest.json` at the wiki root is the machine-readable index of all files
and their current versions. Updated automatically on push.

Git history is the audit trail. If you see a version bump without a commit
explaining it, flag it.

## Expansion plan

Future versions of this wiki will add files for FAQs, task-specific guidance,
and other content as needs emerge. The map gets updated when that happens.
This file is the navigation hub; it's where you look first.

---

## Changelog

- **1.0.0 (2026-04-29):** Initial wiki version with map and orientation.
